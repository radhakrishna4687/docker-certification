# docker build
Build an image from a Dockerfile
```
docker build [OPTIONS] PATH | URL | -
```
- The docker build command builds Docker images from a Dockerfile and a “context”. 
- A build’s context is the set of files located in the specified PATH or URL. 
- The build process can refer to any of the files in the context. 
- For example, your build can use a COPY instruction to reference a file in the context.

The URL parameter can refer to three kinds of resources: 
1. Git repositories, 
2. pre-packaged tarball contexts and 
3. plain text files.
# Git repositories
- When the URL parameter points to the location of a Git repository, the repository acts as the build context. 
- The system recursively fetches the repository and its submodules. 
- The commit history is not preserved. 
- A repository is first pulled into a temporary directory on your local host. 
- After that succeeds, the directory is sent to the Docker daemon as the context. 
- Local copy gives you the ability to access private repositories using local user credentials, VPN’s, and so forth.
- Git URLs accept context configuration in their fragment section, separated by a colon :. 
- The first part represents the reference that Git will check out, and can be either a branch, a tag, or a remote reference. 
- The second part represents a subdirectory inside the repository that will be used as a build context.

For example, run this command to use a `directory` called `docker` in the branch `container`
```
$ docker build https://github.com/docker/rootfs.git#container:docker
```
# Tarball contexts
If you pass an URL to a remote tarball, the URL itself is sent to the daemon:
```
$ docker build http://server/context.tar.gz
```
# Text files
Instead of specifying a context, you can pass a single Dockerfile in the URL or pipe the file in via STDIN. To pipe a Dockerfile from STDIN:
```
$ docker build - < Dockerfile
```
- If you use `STDIN` or specify a `URL` pointing to a plain text file, the system places the contents into a file called Dockerfile, and any -f, --file option is ignored. 
- In this scenario, there is no context.
- By default the `docker build` command will look for a Dockerfile at the root of the build context. 
- The `-f`, `--file`, option lets you specify the path to an alternative file to use instead. 
- This is useful in cases where the same set of files are used for multiple builds. 
- The path must be to a file within the build context. 
- If a relative path is specified then it is interpreted as relative to the root of the context.
- In most cases, it’s best to put each Dockerfile in an empty directory. 
- Then, add to that directory only the files needed for building the Dockerfile. 
- To increase the build’s performance, you can exclude files and directories by adding a .dockerignore file to that directory as well. 
- For information on creating one, see the .dockerignore file.
- If the Docker client loses connection to the daemon, the build is canceled. 
- This happens if you interrupt the Docker client with CTRL-c or if the Docker client is killed for any reason. 
- If the build initiated a pull which is still running at the time the build is cancelled, the pull is cancelled as well.
# Build with PATH
```
docker build .
```
- This example specifies that the PATH is ., and so all the files in the local directory get tard and sent to the Docker daemon. 
- The PATH specifies where to find the files for the “context” of the build on the Docker daemon. 
- Remember that the daemon could be running on a remote machine and that no parsing of the Dockerfile happens at the client side (where you’re running docker build). 
- That means that all the files at PATH get sent, not just the ones listed to ADD in the Dockerfile.
- The transfer of context from the local machine to the Docker daemon is what the docker client means when you see the “Sending build context” message.
- If you wish to keep the intermediate containers after the build is complete, you must use --rm=false. This does not affect the build cache.

# Build with URL
```
$ docker build github.com/creack/docker-firefox
```
- This will clone the GitHub repository and use the cloned repository as context. 
- The Dockerfile at the root of the repository is used as Dockerfile. 
- You can specify an arbitrary Git repository by using the git:// or git@ scheme.
```
docker build -f ctx/Dockerfile http://server/ctx.tar.gz
```
# Build with -
```
docker build - < Dockerfile
```
- This will read a Dockerfile from STDIN without context. 
- Due to the lack of a context, no contents of any local directory will be sent to the Docker daemon. 
- Since there is no context, a Dockerfile ADD only works if it refers to a remote URL.
```
$ docker build - < context.tar.gz
```
This will build an image for a compressed context read from STDIN. Supported formats are: bzip2, gzip and xz.

# Use a .dockerignore file
# Tag an image (-t)
```
$ docker build -t vieux/apache:2.0 .
```
- This will build like the previous example, but it will then tag the resulting image. 
- The repository name will be vieux/apache and the tag will be 2.0. Read more about valid tags.
- You can apply multiple tags to an image. 
- For example, you can apply the latest tag to a newly built image and add another tag that references a specific version. 
- For example, to tag an image both as `whenry/fedora-jboss:latest` and `whenry/fedora-jboss:v2.1`, use the following:
```
$ docker build -t whenry/fedora-jboss:latest -t whenry/fedora-jboss:v2.1 .
```
# Specify a Dockerfile (-f)
## Specify docker file 
```
$ docker build -f Dockerfile.debug .
```
## STDIN dockerfile
```
$ curl example.com/remote/Dockerfile | docker build -f - .
```
- The above command will use the current directory as the build context and read a Dockerfile from stdin.
## Build twice using two different docker file
```
$ docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .
$ docker build -f dockerfiles/Dockerfile.prod  -t myapp_prod .
```
- The above commands will build the current build context (as specified by the .) twice, once using a debug version of a Dockerfile and once using a production version.

## Custom location for Dockerfile and context
```
$ cd /home/me/myapp/some/dir/really/deep
$ docker build -f /home/me/myapp/dockerfiles/debug /home/me/myapp
$ docker build -f ../../../../dockerfiles/debug /home/me/myapp
```
- These two docker build commands do the exact same thing. 
- They both use the contents of the debug file instead of looking for a Dockerfile and will use `/home/me/myapp` as the root of the build context. 
- Note that debug is in the directory structure of the build context, regardless of how you refer to it on the command line.
# Set build-time variables (--build-arg)
- You can use ENV instructions in a Dockerfile to define variable values. 
- These values persist in the built image. 
- However, often persistence is not what you want. 
- Users want to specify variables differently depending on which host they build an image on.
- A good example is http_proxy or source versions for pulling intermediate files. 
- The ARG instruction lets Dockerfile authors define values that users can set at build-time using the --build-arg flag:
```
$ docker build --build-arg HTTP_PROXY=http://10.20.30.2:1234 --build-arg FTP_PROXY=http://40.50.60.5:4567 .
```
You may also use the --build-arg flag without a value, in which case the value from the local environment will be propagated into the Docker container being built:

```
$ export HTTP_PROXY=http://10.20.30.2:1234
$ docker build --build-arg HTTP_PROXY .
```
# Specifying target build stage (--target)
- When building a Dockerfile with multiple build stages, `--target` can be used to specify an intermediate build stage by name as a final stage for the resulting image. 
- Commands after the target stage will be skipped.
```
$ docker build -t mybuildimage --target build-env .
```




























