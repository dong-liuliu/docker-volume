[![Build Status](https://travis-ci.org/quobyte/docker-volume.svg?branch=master)](https://travis-ci.org/quobyte/docker-volume)

# Quobyte volume plug-in for Docker

This plugin allows you to use [Quobyte](https://www.quobyte.com) with Docker without installing the Quobyte client on the host system (e.q. Rancher/CoreOS) for more information look at [docs/coreos.md](docs/coreos.md).

## Tested

OS              | Docker Version
--------------- | :------------:
CentOS 7.2      |     1.10.3
Ubuntu 16.04    |     1.11.2
Ubuntu 16.04    |     1.12.0
CoreOS 1097.0.0 |     1.11.2

## Setup

### Binary

Download the binary and the systemd files from the [releases](https://github.com/quobyte/docker-volume/releases) page. To download version `1.1` of this plugin run the following step:

```bash
curl -LO https://github.com/quobyte/docker-volume/releases/download/v1.1/docker-quobyte-plugin.tar.gz
tar xfz docker-quobyte-plugin.tar.gz
```

### Create a user in Quobyte for the plug-in:

This step is optional and should only be used if Quobytes build-in user database is used.

```
$ qmgmt -u <api-url> user config add docker <email>
```

### Systemd Files

Install systemd files and set your variables in systemd/docker-quobyte.env.sample

```
$ cp systemd/docker-quobyte.env.sample /etc/quobyte/docker-quobyte.env
$ cp bin/docker-quobyte-plugin /usr/local/bin/
$ cp systemd/docker-quobyte-plugin.service /lib/systemd/system

$ systemctl daemon-reload
$ systemctl start docker-quobyte-plugin
$ systemctl enable docker-quobyte-plugin
$ systemctl status docker-quobyte-plugin
```

### Configuration

Configuration is done mainly through the systemd environment file:

```
# Maximum number of filesystem checks when a Volume is created before returning an error
MAX_FS_CHECKS=5
# Maximum wait time for filesystem checks to complete when a Volume is created before returning an error
MAX_WAIT_TIME=30
# Group to create the unix socket
SOCKET_GROUP=root
QUOBYTE_API_URL=http://localhost:7860
QUOBYTE_API_PASSWORD=quobyte
QUOBYTE_API_USER=admin
QUOBYTE_MOUNT_PATH=/run/docker/quobyte/mnt
QUOBYTE_MOUNT_OPTIONS=-o user_xattr
QUOBYTE_REGISTRY=localhost:7861
# ID of the Quobyte tenant in whose domain volumes are managed by this plugin
QUOBYTE_TENANT_ID=replace_me
# Default volume config for new volumes, can be overridden via --opt flag 'configuration_name'
QUOBYTE_VOLUME_CONFIG_NAME=BASE
```

### Usage

The cli is faily simple:

```
$ bin/docker-quobyte-plugin -h
Usage of bin/docker-quobyte-plugin:
  -version
      Shows version string
```

## Examples

### Create a volume

```
$ docker volume create --driver quobyte --name <volumename> 
# Set user, group and specific volume configuration for the new volume
$ docker volume create --driver quobyte --name <volumename> --opt user=docker --opt group=docker --opt configuration_name=SSD_ONLY
```

### Delete a volume

__Important__: Be careful when using this. The volume removal allows removing any volume accessible in the configured tenant!

```
$ docker volume rm <volumename>
```

### List all volumes

```
$ docker volume ls
```

### Attach volume to container

```
$ docker run --volume-driver=quobyte -v <volumename>:/vol busybox sh -c 'echo "Hello World" > /vol/hello.txt'
```

## Development

### Build

Get the code

```
$ go get -u github.com/quobyte/docker-volume
```

#### Dependency Management

For the dependency management we use [golang dep](https://github.com/golang/dep)

#### Linux

```
$ go build -ldflags "-s -w" -o bin/docker-quobyte-plugin .
```

#### OSX/MacOS

```
$ GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" -o bin/docker-quobyte-plugin .
```

#### Docker

```
$ docker run --rm -v "$GOPATH":/work -e "GOPATH=/work" -w /work/src/github.com/quobyte/docker-volume golang:1.8 go build -v -ldflags "-s -w" -o bin/quobyte-docker-plugin
```
