#  How to configure Docker to use external DNS server
Either via 
```
$ docker run --dns 10.0.0.2 busybox nslookup google.com
```
or edit your `/etc/docker/daemon.json` to have something like:
```
{
    "dns": ["10.0.0.2", "8.8.8.8"]
}
```
then restart docker service  
```
$ sudo systemctl docker restart
```
