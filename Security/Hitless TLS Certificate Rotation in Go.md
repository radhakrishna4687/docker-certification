# Hitless TLS Certificate Rotation in Go
- One of the core security goals of Docker's Swarm mode is to be secure by default. 
- To achieve that, when a new Swarm gets created it generates a self-signed Certificate Authority (CA) and issues short-lived certificates to every node, allowing the use of Mutually Authenticated TLS (MTLS) for node-to-node communications.
![](https://diogomonica.com/content/images/2017/01/swarm-with-mutual-tls-simple.png)
- Unfortunately, and much to the annoyance of every infrastructure engineer, there is an old TLS maxim that states that:
> If a certificate got issued, it will have to be rotated.
- Rotating TLS certificates manually may quickly get out of hand—particularly when we have to manage hundreds of certificates—and becomes completely unmanageable if we issue certificates that expire within hours, instead of months.
- In this post I'll go over two different ways of doing hitless certificate rotation in Go, so that we can follow the only logical path out of this certificate management nightmare: automate the hell out of TLS certificate rotation.

# Why do I need hitless rotation?
- There are some use-cases where replacing the TLS certificate on disk and restarting the application is a completely valid way of doing certificate rotation.
- In fact, if we do a rolling deploy of our application—by adding new application instances with the new certificate before shutting down the instances with the old certificate—we can achieve hitless rotation[2] and none of the incoming requests to the application will be dropped.
![](https://diogomonica.com/content/images/2017/01/load-balancer-hitless-rotation-1.png)

Unfortunately, there are two main issues with this approach:
1. There might be side-effects to shutting down the old instances (e.g., applications losing their caches).
2. We either have to wait for all the currently active TCP connections of our old instances to finish (i.e., we might have to wait a long time) or forcefully terminate all the current open connections.

As a concrete example, consider the Docker Swarm architecture, depicted in the following Figure. To create highly-available clusters, Swarm uses special manager nodes participating in a consensus protocol called Raft. Manager nodes keep long-lived connections between each other, and all the other nodes in the system (workers) maintain a long-lived connection to one of the managers.
![](https://diogomonica.com/content/images/2017/01/swarm-raft-cluster-1.png)

If we were to use the previously described rolling–deploy method to rotate the certificates on the manager nodes, we would:
- Cause a thundering herd of workers attempting to reconnect their terminated connections with the managers.
- Potentially cause a leader election between the managers, bringing unnecessary disruption to our cluster while Raft converges to a new leader.

To make matters worse, if our certificates have short expiration times, these two issues would occur several times a day.

Fortunately, there is a better way.

# Certificate selection during the TLS handshake
- In a TLS handshake, the certificate presented by a remote server is sent alongside the ServerHello message. 
- At this point in the connection, the remote server has received the ClientHello message, and that is all the information it needs to decide which certificate to present to the connecting client.
![](https://diogomonica.com/content/images/2017/01/begining-tls-handshake-1.png)

- It turns out that Go supports passing a callback in a TLS Config that will get executed every time a TLS ClientHello is sent by a remote peer. 
- This method is conveniently called GetCertificate, and it returns the certificate we wish to use for that particular TLS handshake.

- The idea of GetCertificate is to allow the dynamic selection of which certificate to provide to a particular remote peer. 
- This method can be used to support virtual hosts, where one web server is responsible for multiple domains, and therefore has to choose the appropriate certificate to return to each remote peer.
![](https://diogomonica.com/content/images/2017/01/certificate-selection-during-handshake-1.png)

Using GetCertificate is easy. The first thing we need to do is to create a struct that implements the GetCertificate(clientHello *tls.ClientHelloInfo) method.
```
type wrappedCertificate struct {
	sync.Mutex
	certificate *tls.Certificate
}

func (c *wrappedCertificate) getCertificate(clientHello *tls.ClientHelloInfo) (*tls.Certificate, error) {
	c.Lock()
	defer c.Unlock()

	return c.certificate, nil
}
```
After this, we can create a TLS Config that makes use of this method, and a TLS listener that makes use of this config:
```
wrappedCert := &wrappedCertificate{}
config := &tls.Config{
	GetCertificate: wrappedCert.getCertificate,
	PreferServerCipherSuites: true,
	MinVersion:               tls.VersionTLS12,
}
network := "0.0.0.0:8080"
listener, _ := tls.Listen("tcp", network, config)
```
Every time a TLS handshake is about to occur, our getCertificate method is going to get called, and the current certificate stored inside wrappedCertificate will be returned.

However, we are missing a way of replacing the internal certificate that is returned by getCertificate. Let's fix that:

```
func (c *wrappedCertificate) loadCertificate(cert, key []byte) error {
	c.Lock()
	defer c.Unlock()

	certAndKey, err := tls.X509KeyPair(cert, key)
	if err != nil {
		return err
	}

	c.certificate = &certAndKey

	return nil
}
```
This loadCertificate() method allows updating the certificate stored inside wrappedCertificate, successfully achieving our goal of doing certificate rotation without killing the currently active connections.

Here's a diagram of what is happening:
![](https://diogomonica.com/content/images/2017/01/golang-new-certificate-being-served.png)
Old established connections using the previous certificate will remain active, but new connections coming in to our TLS server will use the most recent certificate.

Here is a simple example of a TLS server that rotates its certificates every second. Every new certificate gets generated with random Organization (O=); running this example and doing a few handshakes shows us that the server is indeed rotating certificates at every second:
```
go build certificate_rotation.go; ./certificate_rotation
Generating new certificates.
Generating new certificates.
```
```
 ~ openssl s_client -connect localhost:8080 -no_ssl3 -no_ssl2 | openssl x509 -text | grep "O="
depth=0 /O=YSvkxjrK1UGexUg1KubNtrfXRhyRF-AxPPtXZxXkiKk=
...
➜  ~ openssl s_client -connect localhost:8080 -no_ssl3 -no_ssl2 | openssl x509 -text | grep "O="
depth=0 /O=aQIkDOpBwUDLdCLAGvnY8C5vRlmV0eDn2hRf_zTgpxk=
...
```
# Choosing the TLS config before the TLS handshake
- There is another way of achieving the same goal of certificate rotation in Go. 
- Instead of relying on GetCertificateto be called on every TLS handshake and choosing which certificate gets used, we can create a new TLS server for every accepted TCP connection, and provide whatever TLS config is active at the time.
- The major advantage of this particular route is the fact that we are no longer stuck with the same TLS config parameters for every connection, and we can change any TLS Config parameters on the fly.
# Certificate rotation in Docker Swarm
- One of our objectives with Docker Swarm is to support transparent root rotation. 
- The idea is to allow an administrator to force the whole cluster to migrate away from an old root CA transparently, removing its existence from the trust stores of all the nodes participating in the Swarm. 
- This means that we need control over the whole TLS config, instead of controlling only which certificate is currently being served.
- To slightly complicate matters, we have to rotate not only the server certificates, but also the client certificates being actively used for Mutually Authenticated TLS by every node.
- Finally, Swarm also makes heavy use of gRPC, which is a a high-performance, open-source, universal RPC framework. Therefore, we will have to integrate our wrappedConfig style config rotation with the gRPC's authentication model.
- It turns out that gRPC provides a simple authentication API that allows us to define custom methods for performing the client and server handshakes by simply implementing the TransportCredentials interface:
# Changes coming in Golang 1.8
If the previous method seems a bit clunky to you, it's because it is. Thankfully, Golang 1.8 is bringing some changes that will help simplify this use-case. In particular, the addition of GetConfigForClient will allow us to dynamically change the server's TLS Config behavior based on the ClientHello message.

Additionally, Golang 1.8 is also adding support for ChaCha20-Poly1305 based cipher suites and the VerifyPeerCertificate method, which enables custom certificate checking logic.

# Conclusion
The ability of doing hitless TLS certificate rotation is critical to continue our quest of reducing certificate expiration times, while keeping our sanity intact.












