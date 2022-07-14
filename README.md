# mrefd_autoconf
This scrpt is intended to use with https://github.com/n7tae/mrefd and Docker allowing the configuration files to be written using passed in environment variables. It then automates the build process and starts the service.

## Background
`mrefd` requires some configuration prior to building. The concept in this Docker deployment is that the configuration is supplied via environemnt variables and it is built every time the container is started for a completely self contained container.

The script is mostly code from the mredf `rconfig` script that is usually used to walk the user through the config process prior to building. 

This script just automates all that..but also in a way that does not require a fork of the mrefd code.

Though it shouldn't matter, I use Portainer for managing my containers so YMMV

## How it works
The script takes the following actions:
* Deletes any previous git clone and built configuration of mrefd
* Clones mrefd project
* Uses the environment variables from `docker-compose` to create the three config files `rconfig` usually makes
* Copies over the whitelist, blacklist, and interlink default files (as per the mrefd instructions)
* Uses the respective comma separated values in the Docker environment variables to append lines to each of the files as needed
* Creates some folders that are not present in the Alpine image due to missing services, so that the installer doesn't error out
* Make and Install (see note)
* Starts the mrefd service

Note: The base Alpine linux image does not have systemctl so technically the install fails because it cannot start the service without systemctl. We simply ignore the error and run mrefd ourselves.


## Image Setup
The base image is the super lightweight Alpine.  
* `bash` and `git` are added for obvious reasons
* clones this project and chmod the script so it can be our entry point for the container

Dockerfile:
```
FROM alpine:latest
RUN apk add bash git alpine-sdk  
RUN cd ~ &&\
  git clone https://github.com/turbo2ltr/mrefd_autoconf.git &&\
  chmod 777 ~/mrefd_autoconf/autoconf
```

## Container setup
* The container entry point is the `autoconf` script in this package which is already in the image.
* Forward UDP port 17000 
* No volume is needed

### Environment Variables
All config variables are set up as docker environment variables
* MREFD_CALLSIGN : Reflector callsign (M17-xxx)
* MREFD_NB_OF_MODULES : Number of modules
* MREFD_LISTEN_IPV4 : IPv4 IP address to listen on
* MREFD_LISTEN_IPV6 : IPv6 IP address to listen on
* MREFD_NB_OF_CLIENTS : Allow multiple clients on the same IP
* MREFD_DEBUG : Debugging Support
* MREFD_WHITELIST : Comma separated string of whitelist entries (e.g. A1AA*,B2BB*,C3CC*)
* MREFD_BLACKLIST : Comma separated string of whitelist entries (e.g. A1AA*,B2BB*,C3CC*)
* MREFD_INTERLINK : Comma separated string of interlink entried (e.g. M17-M17 xx.xx.xx.xx ACD)

### docker-compose example:
Update the image name to match the name of the image you created

```
version: "3.5"
services:
  mrefd:
    image: mrefd-auto
    network_mode: "bridge"
    container_name: mrefd 
    ports:
      - "17000:17000/udp"
    restart: always
    entrypoint: /bin/bash /root/mrefd_autoconf/autoconf
    stdin_open: true 
    tty: true      
    environment:
      - MREFD_CALLSIGN=M17-MBB
      - MREFD_NB_OF_MODULES=26
      - MREFD_LISTEN_IPV4=0.0.0.0
#      - MREFD_NB_OF_CLIENTS=false
#      - MREFD_LISTEN_IPV6=
#      - MREFD_DEBUG=false
#      - MREFD_WHITELIST=
#      - MREFD_BLACKLIST=
#      - MREFD_INTERLINK=
```



  
