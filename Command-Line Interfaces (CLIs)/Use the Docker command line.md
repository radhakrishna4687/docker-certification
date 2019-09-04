# Use the Docker command line
To list available commands, either run docker with no parameters or execute docker help:
# Configuration files
- By default, the Docker command line stores its configuration files in a directory called .docker within your $HOME directory. 
- However, you can specify a different location via the DOCKER_CONFIG environment variable or the --config command line option. 
- If both are specified, then the --config option overrides the DOCKER_CONFIG environment variable. 
For example:
```
docker --config ~/testconfigs/ ps
```

- Docker manages most of the files in the configuration directory and you should not modify them. 
- However, you can modify the config.json file to control certain aspects of how the docker command behaves.
- Currently, you can modify the docker command behavior using environment variables or command-line options. 
- You can also use options within `config.json` to modify some of the same behavior. 
- When using these mechanisms, you must keep in mind the order of precedence among them.
- Command line options override environment variables and environment variables override properties you specify in a config.json file.
- The config.json file stores a JSON encoding of several properties:
```
{
  "auths" : {
    "https://665312134720.dkr.ecr.us-east-1.amazonaws.com" : {

    }
  },
  "HttpHeaders" : {
    "User-Agent" : "Docker-Client/17.09.1-ce (darwin)"
  }
}
```
## HttpHeaders
- The property `HttpHeaders` specifies a set of headers to include in all messages sent from the Docker client to the daemon. 
- Docker does not try to interpret or understand these header; it simply puts them into the messages. 
- Docker does not allow these headers to change any headers it sets for itself.
## psFormat
- The property `psFormat` specifies the default format for docker ps output. 
- When the `--format` flag is not provided with the docker ps command, Docker’s client uses this property. 
- If this property is not set, the client falls back to the default table format. For a list of supported formatting directives, see the Formatting section in the docker ps documentation
## imagesFormat
- The property `imagesFormat` specifies the default format for docker images output. 
- When the `--format` flag is not provided with the docker images command, Docker’s client uses this property. 
- If this property is not set, the client falls back to the default table format. 
- For a list of supported formatting directives, see the Formatting section in the docker images documentation
## pluginsFormat
- The property `pluginsFormat` specifies the default format for docker plugin ls output. 
- When the `--format` flag is not provided with the docker plugin ls command, Docker’s client uses this property. 
- If this property is not set, the client falls back to the default table format. 
- For a list of supported formatting directives, see the Formatting section in the docker plugin ls documentation
## servicesFormat
- The property `servicesFormat` specifies the default format for docker service ls output. 
- When the --format flag is not provided with the docker service ls command, 
- Docker’s client uses this property. 
- If this property is not set, the client falls back to the default json format. 
- For a list of supported formatting directives, see the Formatting section in the docker service ls documentation
## serviceInspectFormat
## statsFormat
## secretFormat
## nodesFormat
## configFormat
## credsStore
## stackOrchestrator











