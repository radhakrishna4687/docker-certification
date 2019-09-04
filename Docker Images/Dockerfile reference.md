# Dockerfile reference
## Usage
- The docker build command builds an image from a Dockerfile and a context. 
- The build’s context is the set of files at a specified location PATH or URL.
- The PATH is a directory on your local filesystem.
- The URL is a Git repository location.
- A context is processed recursively. 
- So, a PATH includes any subdirectories and the URL includes the repository and its submodules.
- This example shows a build command that uses the current directory as context:
```
docker build .
Sending build context to Docker daemon  6.51 MB
```
- The build is run by the Docker daemon, not by the CLI
- The first thing a build process does is send the entire context (recursively) to the daemon
- In most cases, it’s best to start with an empty directory as context and keep your Dockerfile in that directory.
-  Add only the files needed for building the Dockerfile.
- To increase the build’s performance, exclude files and directories by adding a .dockerignore file to the context directory. 
- Traditionally, the Dockerfile is called Dockerfile and located in the root of the context. 
- You use the `-f` flag with docker build to point to a Dockerfile anywhere in your file system.
```
docker build -f /path/to/a/Dockerfile .
```
- You can specify a repository and tag at which to save the new image if the build succeeds:
```
docker build -t shykes/myapp .
```
- To tag the image into multiple repositories after the build, add multiple -t parameters when you run the build command:
```
docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```
- Before the Docker daemon runs the instructions in the Dockerfile, it performs a preliminary validation of the Dockerfile and returns an error if the syntax is incorrect:
```
docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```
- The Docker daemon runs the instructions in the Dockerfile one-by-one, committing the result of each instruction to a new image if necessary, before finally outputting the ID of your new image. 
- Note that each instruction is run independently, and causes a new image to be created - so `RUN cd /tmp` will not have any effect on the next instructions.
- Whenever possible, Docker will re-use the intermediate images (cache), to accelerate the docker build process significantly. 
- If you wish to use build cache of a specific image you can specify it with `--cache-from` option. 
# BuildKit
Starting with version `18.09`, Docker supports a new backend for executing your builds that is provided by the `moby/buildkit` project

The BuildKit backend provides many benefits compared to the old implementation.
- Detect and skip executing unused build stages
- Parallelize building independent build stages
- Incrementally transfer only the changed files in your build context between builds
- Detect and skip transferring unused files in your build context
- Use external Dockerfile implementations with many new features
- Avoid side-effects with rest of the API (intermediate images and containers)
- Prioritize your build cache for automatic pruning

To use the BuildKit backend, you need to set an environment variable `DOCKER_BUILDKIT=1` on the CLI before invoking `docker build`.

# Format
Here is the format of the Dockerfile:
```
# Comment
INSTRUCTION arguments
```
- The instruction is not case-sensitive. 
- However, convention is for them to be UPPERCASE to distinguish them from arguments more easily.
- Docker runs instructions in a Dockerfile in order.
- A Dockerfile must start with a `FROM` instruction. 
- The FROM instruction specifies the Base Image from which you are building. 
- FROM may only be preceded by one or more ARG instructions, which declare arguments that are used in FROM lines in the Dockerfile.
- Docker treats lines that begin with # as a comment, unless the line is a valid parser directive. 
- A # marker anywhere else in a line is treated as an argument. This allows statements like:
```
# Comment
RUN echo 'we are running some # of cool things'
```
- Line continuation characters are not supported in comments.
# Parser directives
- Parser directives are optional, and affect the way in which subsequent lines in a Dockerfile are handled.
- Parser directives do not add layers to the build, and will not be shown as a build step.
- Parser directives are written as a special type of comment in the form # directive=value. 
- A single directive may only be used once.
- Once a comment, empty line or builder instruction has been processed, Docker no longer looks for parser directives.
- Instead it treats anything formatted as a parser directive as a comment and does not attempt to validate if it might be a parser directive. 
- Therefore, all parser directives must be at the very top of a Dockerfile.
- Parser directives are not case-sensitive.
- However, convention is for them to be lowercase. 
- Convention is also to include a blank line following any parser directives.
- Line continuation characters are not supported in parser directives.

Invalid due to line continuation:
```
# direc \
tive=value
```
Invalid due to appearing twice:
```
# directive=value1
# directive=value2

FROM ImageName
```
Treated as a comment due to appearing after a builder instruction:
```
FROM ImageName
# directive=value
```
Treated as a comment due to appearing after a comment which is not a parser directive:
```
# About my dockerfile
# directive=value
FROM ImageName
```
The unknown directive is treated as a comment due to not being recognized. In addition, the known directive is treated as a comment due to appearing after a comment which is not a parser directive.
```
# unknowndirective=value
# knowndirective=value
```
Non line-breaking whitespace is permitted in a parser directive. Hence, the following lines are all treated identically:
```
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```
The following parser directives are supported:
- syntax
- escape
## syntax parser directive
```
# syntax=[remote image reference]
```
Examples
```
# syntax=docker/dockerfile
# syntax=docker/dockerfile:1.0
# syntax=docker.io/docker/dockerfile:1
# syntax=docker/dockerfile:1.0.0-experimental
# syntax=example.com/user/repo:tag@sha256:abcdef...
```
- This feature is only enabled if the BuildKit backend is used.

Custom Dockerfile implementation allows you to:
- Automatically get bugfixes without updating the daemon
- Make sure all users are using the same implementation to build your Dockerfile
- Use the latest features without updating the daemon
- Try out new experimental or third-party features
### Official releases
Stable channel follows semantic versioning. For example:
```
docker/dockerfile:1.0.0 - only allow immutable version 1.0.0
docker/dockerfile:1.0 - allow versions 1.0.*
docker/dockerfile:1 - allow versions 1..
docker/dockerfile:latest - latest release on stable channel
```
## escape directive parser
```
# escape=\ (backslash)
```
Or 
```
# escape=` (backtick)
```
- The escape directive sets the character used to escape characters in a Dockerfile. If not specified, the default escape character is `\`.
- The escape character is used both to escape characters in a `line`, and to escape a `newline`.
- This allows a Dockerfile instruction to span multiple lines.
- Note that regardless of whether the escape parser directive is included in a Dockerfile, escaping is not performed in a RUN command, except at the end of a line.
- Setting the escape character to \` is especially useful on Windows, where `\` is the directory path separator. \` is consistent with Windows PowerShell.

Consider the following example which would fail in a non-obvious way on Windows. 
```
FROM microsoft/nanoserver
COPY testfile.txt c:\\
RUN dir c:\
```
-  The second `\` at the end of the second line would be interpreted as an escape for the newline, instead of a target of the escape from the first `\`
- Similarly, the `\` at the end of the third line would, assuming it was actually handled as an instruction, cause it be treated as a line continuation. 
- The result of this dockerfile is that second and third lines are considered a single instruction

Results in:
```
PS C:\John> docker build -t cmd .
Sending build context to Docker daemon 3.072 kB
Step 1/2 : FROM microsoft/nanoserver
 ---> 22738ff49c6d
Step 2/2 : COPY testfile.txt c:\RUN dir c:
GetFileAttributesEx c:RUN: The system cannot find the file specified.
PS C:\John>
```
By adding the escape parser directive, the following Dockerfile succeeds as expected with the use of natural platform semantics for file paths on Windows:
```
# escape=`

FROM microsoft/nanoserver
COPY testfile.txt c:\
RUN dir c:\
```
# Environment replacement
- Environment variables (declared with the ENV statement) can also be used in certain instructions as variables to be interpreted by the Dockerfile. 
- Escapes are also handled for including variable-like syntax into a statement literally.
- Environment variables are notated in the Dockerfile either with `$variable_name` or `${variable_name}`. 
- They are treated equivalently and the brace syntax is typically used to address issues with variable names with no whitespace, like `${foo}_bar`.
- The `${variable_name}` syntax also supports a few of the standard bash modifiers as specified below:
1. `${variable:-word}` indicates that if variable is set then the result will be that value. If variable is not set then word will be the result.
2. `${variable:+word}` indicates that if variable is set then word will be the result, otherwise the result is the empty string.

Example
```
FROM busybox
ENV foo /bar
WORKDIR ${foo}   # WORKDIR /bar
ADD . $foo       # ADD . /bar
COPY \$foo /quux # COPY $foo /quux
```
Environment variables are supported by the following list of instructions in the Dockerfile:
- ADD
- COPY
- ENV
- EXPOSE
- FROM
- LABEL
- STOPSIGNAL
- USER
- VOLUME
- WORKDIR

Environment variable substitution will use the same value for each variable throughout the entire instruction. In other words, in this example:
```
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```
- will result in def having a value of hello, not bye
- However, ghi will have a value of bye because it is not part of the same instruction that set abc to bye

# .dockerignore file
- Before the docker CLI sends the context to the docker daemon, it looks for a file named .dockerignore in the root directory of the context. 
- If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it.
# FROM
```
FROM <image> [AS <name>]
```
Or
```
FROM <image>[:<tag>] [AS <name>]
```
Or
```
FROM <image>[@<digest>] [AS <name>]
```
- The `FROM` instruction initializes a new build stage and sets the Base Image for subsequent instructions. 
- As such, a valid `Dockerfile` must start with a `FROM` instruction. 
- The image can be any valid image 

Following are few more points.
- `ARG` is the only instruction that may precede `FROM` in the Dockerfile
- `FROM` can appear multiple times within a single `Dockerfile` to create multiple images or use one build stage as a dependency for another.
- Each `FROM` instruction clears any state created by previous instructions.
- Optionally a name can be given to a new build stage by adding `AS name` to the `FROM` instruction.
- The name can be used in subsequent `FROM` and `COPY --from=<name|index>` instructions to refer to the image built in this stage.
- The tag or digest values are optional
- If you omit either of them, the builder assumes a latest tag by default
## Understand how ARG and FROM interact
- `FROM` instructions support variables that are declared by any `ARG` instructions that occur before the first `FROM`.
```
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```
- An `ARG` declared before a `FROM` is outside of a build stage, so it can’t be used in any instruction after a `FROM`
- To use the default value of an `ARG` declared before the first `FROM` use an `ARG` instruction without a value inside of a build stage
```
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```
# RUN
RUN has 2 forms
1. `RUN <command>` - shell form, the command is run in a shell, which by default is `/bin/sh -c` on Linux or `cmd /S /C` on Windows
2. `RUN ["executable", "param1", "param2"]` - exec form

- The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.
- Layering `RUN` instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image’s history, much like source control.
- The `exec form` makes it possible to avoid shell string `munging`, and to `RUN` commands using a base image that does not contain the specified shell executable.
- The default shell for the shell form can be changed using the SHELL command.
- In the shell form you can use a \ (backslash) to continue a single RUN instruction onto the next line. 
```
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
```
Together they are equivalent to this single line:
```
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```
- To use a different shell, other than ‘/bin/sh’, use the exec form passing in the desired shell. 
```
RUN ["/bin/bash", "-c", "echo hello"]
```
- The exec form is parsed as a JSON array, which means that you must use double-quotes (“) around words not single-quotes (‘).
- Unlike the `shell` form, the `exec` form does not invoke a command shell. This means that normal shell processing does not happen.

- The cache for RUN instructions isn’t invalidated automatically during the next build
-  The cache for an instruction like `RUN apt-get dist-upgrade -y` will be reused during the next build.
- The cache for RUN instructions can be invalidated by using the `--no-cache `flag, for example `docker build --no-cache`
- The cache for RUN instructions can be invalidated by ADD instructions
 
 # CMD
The `CMD` instruction has three forms
 1. `CMD ["executable","param1","param2"]`  - exec form, this is the preferred form
 2. `CMD ["param1","param2"]` - as default parameters to ENTRYPOINT 
 3. `CMD command param1 param2` -  shell form
- There can only be one `CMD` instruction in a Dockerfile
- If you list more than one `CMD` then only the last `CMD` will take effect.
- The main purpose of a `CMD` is to provide defaults for an executing container
- These defaults can include an executable, or they can omit the executable, in which case you must specify an `ENTRYPOINT` instruction as well.
-  If `CMD` is used to provide default arguments for the `ENTRYPOINT` instruction, both the `CMD` and `ENTRYPOINT` instructions should be specified with the JSON array format
- When used in the shell or exec formats, the `CMD` instruction sets the command to be executed when running the image.
- If you use the shell form of the `CMD`, then the `<command>` will execute in `/bin/sh -c`
```
FROM ubuntu
CMD echo "This is a test." | wc -
```
- If you want to run your `<command>` without a shell then you must express the command as a JSON array and give the full path to the executable. This array form is the preferred format of `CMD`
```
FROM ubuntu
CMD ["/usr/bin/wc","--help"]
```
- If you would like your container to run the same executable every time, then you should consider using `ENTRYPOINT` in combination with `CMD`
- If the user specifies arguments to docker run then they will override the default specified in `CMD`.
- Don’t confuse RUN with CMD. RUN actually runs a command and commits the result; CMD does not execute anything at build time, but specifies the intended command for the image.

# LABEL
```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```
- The `LABEL` instruction adds metadata to an image
- A `LABEL` is a key-value pair
- To include spaces within a `LABEL` value, use quotes and backslashes as you would in command-line parsing.
A few usage examples:
```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```
- An image can have more than one label. 
- You can specify multiple labels on a single line. 
- Labels included in base or parent images (images in the FROM line) are inherited by your image. 
- If a label already exists but with a different value, the most-recently-applied value overrides any previously-set value.
- To view an image’s labels, use the `docker inspect` command.
# MAINTAINER (deprecated)
```
MAINTAINER <name>
```
- The `MAINTAINER` instruction sets the Author field of the generated images. 
- The `LABEL` instruction is a much more flexible version of this and you should use it instead, as it enables setting any metadata you require, and can be viewed easily, for example with `docker inspect`.
```
LABEL maintainer="SvenDowideit@home.org.au"
```
# EXPOSE
```
EXPOSE <port> [<port>/<protocol>...]
```
- The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime.
- You can specify whether the port listens on `TCP` or `UDP`, and the default is `TCP` if the protocol is not specified.
- The `EXPOSE` instruction does not actually publish the port.
- It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published.
- To actually publish the port when running the container, use the `-p` flag on `docker run` to publish and map one or more ports, or the `-P` flag to publish all exposed ports and map them to high-order ports.
- By default, EXPOSE assumes TCP. You can also specify UDP:
```
EXPOSE 80/udp
```
- To expose on both TCP and UDP, include two lines:
```
EXPOSE 80/tcp
EXPOSE 80/udp
```
In this case, if you use `-P` with docker run, the port will be exposed once for `TCP` and once for `UDP`. Remember that `-P` uses an ephemeral high-ordered host port on the host, so the port will not be the same for TCP and UDP.
- Regardless of the EXPOSE settings, you can override them at runtime by using the -p flag. For example
```
docker run -p 80:80/tcp -p 80:80/udp ...
```
# ENV
```
ENV <key> <value>
ENV <key>=<value> ...
```
- The `ENV` instruction sets the environment variable `<key>` to the value `<value>`
- This value will be in the environment for all subsequent instructions in the build stage and can be replaced inline in many as well.
- The ENV instruction has two forms. The first form, `ENV <key> <value>`, will set a single variable to a value. The entire string after the first space will be treated as the `<value>` - including whitespace characters. The value will be interpreted for other environment variables, so quote characters will be removed if they are not escaped.
- The second form, `ENV <key>=<value> ...`, allows for multiple variables to be set at one time. Like command line parsing, quotes and backslashes can be used to include spaces within values.
For example:
```
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```
- The environment variables set using ENV will persist when a container is run from the resulting image.
- You can view the values using `docker inspect`, and change them using `docker run --env <key>=<value>`.

# ADD
ADD has two forms
1. `ADD [--chown=<user>:<group>] <src>... <dest>`
2. `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]`
Note: The `--chown` feature is only supported on `Dockerfiles` used to build Linux containers, and will not work on Windows containers.
- The `ADD` instruction copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`
- Multiple `<src>` resources may be specified but if they are files or directories, their paths are interpreted as relative to the source of the context of the build.
- Each `<src>` may contain wildcards and matching will be done using Go’s filepath.Match rules.
```
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```
- The `<dest>` is an absolute path, or a path relative to `WORKDIR`, into which the source will be copied inside the destination container.
```
ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```
- When adding files or directories that contain special characters (such as [ and ]), you need to escape those paths following the Golang rules to prevent them from being treated as a matching pattern.
- All new files and directories are created with a `UID` and `GID` of `0`, unless the optional `--chown` flag specifies a given `username`, `groupname`, or `UID`/`GID` combination to request specific ownership of the content added.
- The format of the `--chown `flag allows for either `username` and `groupname` strings or direct integer `UID` and `GID` in any combination.
- Providing a username without groupname or a UID without GID will use the same numeric UID as the GID.
- If a `username` or `groupname` is provided, the container’s root filesystem `/etc/passwd` and `/etc/group` files will be used to perform the translation from name to integer `UID` or `GID` respectively.
- The following examples show valid definitions for the `--chown` flag:
```
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```
- If the container root filesystem does not contain either `/etc/passwd` or `/etc/group` files and either `user` or `group` names are used in the `--chown` flag, the build will fail on the `ADD` operation. 
- Using numeric IDs requires no lookup and will not depend on container root filesystem content.
- In the case where `<src>` is a remote file URL, the destination will have permissions of `600`.
- If the remote file being retrieved has an HTTP Last-Modified header, the timestamp from that header will be used to set the mtime on the destination file. 
- However, like any other file processed during an ADD, mtime will not be included in the determination of whether or not the file has changed and the cache should be updated.
Note: If you build by passing a Dockerfile through STDIN (`docker build - < somefile`), there is no build context, so the Dockerfile can only contain a URL based ADD instruction. 
Note: You can also pass a compressed archive through STDIN: (`docker build - < archive.tar.gz`), the Dockerfile at the root of the archive and the rest of the archive will be used as the context of the build.
Note: If your URL files are protected using authentication, you will need to use RUN wget, RUN curl or use another tool from within the container as the ADD instruction does not support authentication.
Note: The first encountered `ADD` instruction will invalidate the cache for all following instructions from the Dockerfile if the contents of `<src>` have changed. This includes invalidating the cache for `RUN` instructions. See the Dockerfile Best Practices guide for more information.

ADD obeys the following rules:
- The `<src>` path must be inside the context of the build; you cannot `ADD ../something /something`, because the first step of a docker build is to send the context directory (and subdirectories) to the docker daemon.
- If `<src>` is a URL and `<dest>` does not end with a trailing slash, then a file is downloaded from the URL and copied to `<dest>`.
- If `<src>` is a URL and `<dest>` does end with a trailing slash, then the filename is inferred from the URL and the file is downloaded to `<dest>/<filename>`. For instance, `ADD http://example.com/foobar /` would create the file `/foobar`. The URL must have a nontrivial path so that an appropriate filename can be discovered in this case (http://example.com will not work).
- If `<src>` is a directory, the entire contents of the directory are copied, including filesystem metadata.
Note: The directory itself is not copied, just its contents.
- If `<src>` is a local tar archive in a recognized compression format (identity, gzip, bzip2 or xz) then it is unpacked as a directory. Resources from remote URLs are not decompressed. When a directory is copied or unpacked, it has the same behavior as tar -x, the result is the union of:
1. Whatever existed at the destination path and
2. The contents of the source tree, with conflicts resolved in favor of “2.” on a file-by-file basis.
- If `<src>` is any other kind of file, it is copied individually along with its metadata. In this case, if `<dest>` ends with a trailing slash /, it will be considered a directory and the contents of `<src>` will be written at <dest>/base(<src>).
- If multiple `<src>` resources are specified, either directly or due to the use of a wildcard, then `<dest>` must be a directory, and it must end with a slash `/`.
- If `<dest>` does not end with a trailing slash, it will be considered a regular file and the contents of `<src>` will be written at `<dest>`.
- If `<dest>` doesn’t exist, it is created along with all missing directories in its path.

# COPY
COPY has two forms:
1. `COPY [--chown=<user>:<group>] <src>... <dest>`
2. `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]`
- The COPY instruction copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.
- Multiple `<src>` resources may be specified but the paths of files and directories will be interpreted as relative to the source of the context of the build.

Most of the rules followed as `ADD`. Following are few examples
```
COPY hom* /mydir/        # adds all files starting with "hom"
COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
COPY test /absoluteDir/  # adds "test" to /absoluteDir/
COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
```
Note: If you build using STDIN (docker build - < somefile), there is no build context, so COPY can’t be used.

- Optionally `COPY` accepts a flag `--from=<name|index>` that can be used to set the source location to a previous build stage (created with `FROM .. AS <name>`) that will be used instead of a build context sent by the user. 
- The flag also accepts a numeric index assigned for all previous build stages started with FROM instruction.

COPY obeys the following rules:
- The `<src>` path must be inside the context of the build; you cannot COPY ../something /something, because the first step of a docker build is to send the context directory (and subdirectories) to the docker daemon.
- If `<src>` is a directory, the entire contents of the directory are copied, including filesystem metadata.

# ENTRYPOINT
ENTRYPOINT has two forms:

1. `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred)
2. `ENTRYPOINT command param1 para`m2 (shell form)

An `ENTRYPOINT` allows you to configure a container that will run as an executable. For example, the following will start nginx with its default content, listening on port 80
```
docker run -i -t --rm -p 80:80 nginx
```
- Command line arguments to `docker run <image>` will be appended after all elements in an exec form ENTRYPOINT, and will override all elements specified using `CMD`
- This allows arguments to be passed to the entry point, i.e., `docker run <image> -d` will pass the `-d` argument to the entry point. 
- You can override the `ENTRYPOINT` instruction using the `docker run --entrypoint` flag.
- The shell form prevents any `CMD` or `run` command line arguments from being used, but has the disadvantage that your `ENTRYPOINT` will be started as a subcommand of `/bin/sh -c`, which does not pass signals.
- This means that the executable will not be the container’s PID 1 - and will not receive Unix signals - so your executable will not receive a `SIGTERM` from `docker stop <container>`
- Only the last `ENTRYPOINT` instruction in the Dockerfile will have an effect.

## Exec form ENTRYPOINT example
- You can use the exec form of `ENTRYPOINT` to set fairly stable default commands and arguments and then use either form of `CMD` to set additional defaults that are more likely to be changed.
```
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```
- When you run the container, you can see that top is the only process:
```
docker run -it --rm --name test  top -H
```
- The following Dockerfile shows using the ENTRYPOINT to run Apache in the foreground (i.e., as PID 1):
```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
## Shell form ENTRYPOINT example
- You can specify a plain string for the ENTRYPOINT and it will execute in /bin/sh -c. 
- This form will use shell processing to substitute shell environment variables, and will ignore any CMD or docker run command line arguments.
- To ensure that `docker stop` will signal any long running ENTRYPOINT executable correctly, you need to remember to start it with exec:
```
FROM ubuntu
ENTRYPOINT exec top -b
```
# Understand how CMD and ENTRYPOINT interact
Both `CMD` and `ENTRYPOINT` instructions define what command gets executed when running a container. There are few rules that describe their co-operation.
- Dockerfile should specify at least one of `CMD` or `ENTRYPOINT` commands.
- `ENTRYPOINT` should be defined when using the container as an executable.
- `CMD `should be used as a way of defining default arguments for an `ENTRYPOINT` command or for executing an ad-hoc command in a container.
- `CMD` will be overridden when running the container with alternative arguments.
# VOLUME
```
VOLUME ["/data"]
VOLUME ["/data","/var/log"]
VOLUME /var/log
VOLUME /var/log /var/db
```
- The `VOLUME` instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers. 
- The `docker run` command initializes the newly created volume with any data that exists at the specified location within the base image. 
```
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```
This Dockerfile results in an image that causes `docker run` to create a new mount point at `/myvol` and copy the greeting file into the newly created volume.
## Notes about specifying volumes
Keep the following things in mind about volumes in the `Dockerfile`
### Volumes on Windows-based containers
When using Windows-based containers, the destination of a volume inside the container must be one of:
1. a non-existing or empty directory
2. a drive other than C:
### Changing the volume from within the Dockerfile
If any build steps change the data within the volume after it has been declared, those changes will be discarded.
### The host directory is declared at container run-time
- The host directory (the mountpoint) is, by its nature, host-dependent.
- This is to preserve image portability, since a given host directory can’t be guaranteed to be available on all hosts.
- For this reason, you can’t mount a host directory from within the Dockerfile. 
- The VOLUME instruction does not support specifying a host-dir parameter. You must specify the mountpoint when you create or run the container.

# USER
```
USER <user>[:<group>] or
USER <UID>[:<GID>]
```
- The `USER` instruction sets the `user name` (or `UID`) and optionally the `user group` (or `GID`) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile
- When the user doesn’t have a primary group then the image (or the next instructions) will be run with the `root` group.
- On `Windows`, the user must be created first if it’s not a built-in account. This can be done with the net user command called as part of a Dockerfile.
```
FROM microsoft/windowsservercore
# Create Windows user in the container
RUN net user /add patrick
# Set it for subsequent commands
USER patrick
```
# WORKDIR
```
WORKDIR /path/to/workdir
```
- The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile
- If the `WORKDIR` doesn’t exist, it will be created even if it’s not used in any subsequent Dockerfile instruction
- The `WORKDIR` instruction can be used multiple times in a Dockerfile
- If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction. For example:
```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
The output of the final `pwd` command in this Dockerfile would be `/a/b/c`.
- The `WORKDIR` instruction can resolve environment variables previously set using `ENV`. You can only use environment variables explicitly set in the Dockerfile
```
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```
The output of the final pwd command in this Dockerfile would be /path/$DIRNAME
# ARG
```
ARG <name>[=<default value>]
```
- The `ARG` instruction defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag
- If a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning.
- A Dockerfile may include one or more ARG instructions. For example, the following is a valid Dockerfile:
```
FROM busybox
ARG user1
ARG buildno
```
## Default values
An ARG instruction can optionally include a default value:
```
FROM busybox
ARG user1=someuser
ARG buildno=1
```
## Scope
An ARG variable definition comes into effect from the line on which it is defined in the Dockerfile not from the argument’s use on the command-line or elsewhere. For example, consider this Dockerfile:
```
1 FROM busybox
2 USER ${user:-some_user}
3 ARG user
4 USER $user
```
A user builds this file by calling:
```
$ docker build --build-arg user=what_user .
```
The USER at line 2 evaluates to some_user as the user variable is defined on the subsequent line 3. The USER at line 4 evaluates to what_user as user is defined and the what_user value was passed on the command line. Prior to its definition by an ARG instruction, any use of a variable results in an empty string.

An ARG instruction goes out of scope at the end of the build stage where it was defined. To use an arg in multiple stages, each stage must include the ARG instruction.
```
FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS

FROM busybox
ARG SETTINGS
RUN ./run/other $SETTINGS
```
## Using ARG variables
You can use an ARG or an ENV instruction to specify variables that are available to the RUN instruction. Environment variables defined using the ENV instruction always override an ARG instruction of the same name. Consider this Dockerfile with an ENV and ARG instruction.
```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER
```
## Predefined ARGs
- HTTP_PROXY
- http_proxy
- HTTPS_PROXY
- https_proxy
- FTP_PROXY
- ftp_proxy
- NO_PROXY
- no_proxy
To use these, simply pass them on the command line using the flag:
```
--build-arg <varname>=<value>
```
By default, these pre-defined variables are excluded from the output of docker history. Excluding them reduces the risk of accidentally leaking sensitive authentication information in an HTTP_PROXY variable.
For example, consider building the following Dockerfile using --build-arg HTTP_PROXY=http://user:pass@proxy.lon.example.com
```
FROM ubuntu
RUN echo "Hello World"
```
In this case, the value of the HTTP_PROXY variable is not available in the docker history and is not cached. If you were to change location, and your proxy server changed to http://user:pass@proxy.sfo.example.com, a subsequent build does not result in a cache miss.

If you need to override this behaviour then you may do so by adding an ARG statement in the Dockerfile as follows:
```
FROM ubuntu
ARG HTTP_PROXY
RUN echo "Hello World"
```
When building this Dockerfile, the HTTP_PROXY is preserved in the docker history, and changing its value invalidates the build cache.
## Automatic platform ARGs in the global scope
This feature is only available when using the BuildKit backend. 

Docker predefines a set of ARG variables with information on the platform of the node performing the build (build platform) and on the platform of the resulting image (target platform). The target platform can be specified with the --platform flag on docker build.

The following ARG variables are set automatically:

- TARGETPLATFORM - platform of the build result. Eg linux/amd64, linux/arm/v7, windows/amd64.
- TARGETOS - OS component of TARGETPLATFORM
- TARGETARCH - architecture component of TARGETPLATFORM
- TARGETVARIANT - variant component of TARGETPLATFORM
- BUILDPLATFORM - platform of the node performing the build.
- BUILDOS - OS component of BUILDPLATFORM
- BUILDARCH - OS component of BUILDPLATFORM
- BUILDVARIANT - OS component of BUILDPLATFORM

These arguments are defined in the global scope so are not automatically available inside build stages or for your RUN commands. To expose one of these arguments inside the build stage redefine it without value.
```
FROM alpine
ARG TARGETPLATFORM
RUN echo "I'm building for $TARGETPLATFORM"
```
## Impact on build caching
- ARG variables are not persisted into the built image as ENV variables are. 
- However, ARG variables do impact the build cache in similar ways. 
- If a Dockerfile defines an ARG variable whose value is different from a previous build, then a “cache miss” occurs upon its first usage, not its definition. 
- In particular, all RUN instructions following an ARG instruction use the ARG variable implicitly (as an environment variable), thus can cause a cache miss. 
- All predefined ARG variables are exempt from caching unless there is a matching ARG statement in the Dockerfile.

For example, consider these two Dockerfile:
```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 RUN echo $CONT_IMG_VER
```
```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 RUN echo hello
```

# ONBUILD
```
ONBUILD [INSTRUCTION]

```
- The ONBUILD instruction adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.
- The trigger will be executed in the context of the downstream build, as if it had been inserted immediately after the FROM instruction in the downstream Dockerfile.
- Any build instruction can be registered as a trigger.
- This is useful if you are building an image which will be used as a base to build other images, for example an application build environment or a daemon which may be customized with user-specific configuration.
- For example, if your image is a reusable Python application builder, it will require application source code to be added in a particular directory, and it might require a build script to be called after that. 
- You can’t just call ADD and RUN now, because you don’t yet have access to the application source code, and it will be different for each application build. You could simply provide application developers with a boilerplate Dockerfile to copy-paste into their application, but that is inefficient, error-prone and difficult to update because it mixes with application-specific code.
- The solution is to use ONBUILD to register advance instructions to run later, during the next build stage.

Here’s how it works:
1. When it encounters an ONBUILD instruction, the builder adds a trigger to the metadata of the image being built. The instruction does not otherwise affect the current build.
2. At the end of the build, a list of all triggers is stored in the image manifest, under the key OnBuild. They can be inspected with the docker inspect command.
3. Later the image may be used as a base for a new build, using the FROM instruction. As part of processing the FROM instruction, the downstream builder looks for ONBUILD triggers, and executes them in the same order they were registered. If any of the triggers fail, the FROM instruction is aborted which in turn causes the build to fail. If all triggers succeed, the FROM instruction completes and the build continues as usual.
4. Triggers are cleared from the final image after being executed. In other words they are not inherited by “grand-children” builds.

For example you might add something like this:
```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```
- Warning: Chaining ONBUILD instructions using ONBUILD ONBUILD isn’t allowed.
- Warning: The ONBUILD instruction may not trigger FROM or MAINTAINER instructions.

# STOPSIGNAL
```
STOPSIGNAL signal
```
The STOPSIGNAL instruction sets the system call signal that will be sent to the container to exit. This signal can be a valid unsigned number that matches a position in the kernel’s syscall table, for instance 9, or a signal name in the format SIGNAME, for instance SIGKILL.
# HEALTHCHECK
The HEALTHCHECK instruction has two forms:
1. HEALTHCHECK [OPTIONS] CMD command (check container health by running a command inside the container)
2. HEALTHCHECK NONE (disable any healthcheck inherited from the base image)

The HEALTHCHECK instruction tells Docker how to test a container to check that it is still working. This can detect cases such as a web server that is stuck in an infinite loop and unable to handle new connections, even though the server process is still running.

When a container has a healthcheck specified, it has a health status in addition to its normal status. This status is initially starting. Whenever a health check passes, it becomes healthy (whatever state it was previously in). After a certain number of consecutive failures, it becomes unhealthy.

The options that can appear before CMD are:
```--interval=DURATION (default: 30s)
--timeout=DURATION (default: 30s)
--start-period=DURATION (default: 0s)
--retries=N (default: 3)```

The health check will first run interval seconds after the container is started, and then again interval seconds after each previous check completes.


If a single run of the check takes longer than timeout seconds then the check is considered to have failed.

It takes retries consecutive failures of the health check for the container to be considered unhealthy.

start period provides initialization time for containers that need time to bootstrap. Probe failure during that period will not be counted towards the maximum number of retries. However, if a health check succeeds during the start period, the container is considered started and all consecutive failures will be counted towards the maximum number of retries.

There can only be one HEALTHCHECK instruction in a Dockerfile. If you list more than one then only the last HEALTHCHECK will take effect.

The command after the CMD keyword can be either a shell command (e.g. HEALTHCHECK CMD /bin/check-running) or an exec array (as with other Dockerfile commands; see e.g. ENTRYPOINT for details).

For example, to check every five minutes or so that a web-server is able to serve the site’s main page within three seconds:
```
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
# SHELL
```
SHELL ["executable", "parameters"]
```
The SHELL instruction allows the default shell used for the shell form of commands to be overridden. The default shell on Linux is ["/bin/sh", "-c"], and on Windows is ["cmd", "/S", "/C"]. The SHELL instruction must be written in JSON form in a Dockerfile.

The SHELL instruction is particularly useful on Windows where there are two commonly used and quite different native shells: cmd and powershell, as well as alternate shells available including sh.

The SHELL instruction can appear multiple times. Each SHELL instruction overrides all previous SHELL instructions, and affects all subsequent instructions. For example:

```
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S", "/C"]
RUN echo hello
```

