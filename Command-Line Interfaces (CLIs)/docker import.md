# docker import
Import the contents from a tarball to create a filesystem image
```
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```
- You can specify a URL or - (dash) to take data directly from STDIN. 
- The URL can point to an archive (.tar, .tar.gz, .tgz, .bzip, .tar.xz, or .txz) containing a filesystem or to an individual file on the Docker host. 
- If you specify an archive, Docker untars it in the container relative to the / (root). 
- If you specify an individual file, you must specify the full path within the host. 
- To import from a remote location, specify a URI that begins with the http:// or https:// protocol.


## --change , -c
Apply Dockerfile instruction to the created image
## --message , -m
Set commit message for imported image
## --platform	
Set platform if server is multi-platform capable
# Examples
## Import from a remote location
```
$ docker import http://example.com/exampleimage.tgz
```
## Import from a local file
### Import to docker via pipe and STDIN.
```
$ cat exampleimage.tgz | docker import - exampleimagelocal:new
```
### Import with a commit message.
```
$ cat exampleimage.tgz | docker import --message "New image imported from tarball" - exampleimagelocal:new
```
### Import to docker from a local archive.
```
  $ docker import /path/to/exampleimage.tgz
```
## Import from a local directory
```
$ sudo tar -c . | docker import - exampleimagedir
```
