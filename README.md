<!--
# Copyright © (C) 2017 Emory Merryman <emory.merryman@gmail.com>
#   This file is part of run-injector.
#
#   Run-injector is free software: you can redistribute it and/or modify
#   it under the terms of the GNU General Public License as published by
#   the Free Software Foundation, either version 3 of the License, or
#   (at your option) any later version.
#
#   Run-injector is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#   GNU General Public License for more details.
#
#   You should have received a copy of the GNU General Public License
#   along with run-injector.  If not, see <http://www.gnu.org/licenses/>.
-->
# Run Injector

## Synopsis
Injects a dependency into a container via its volumes.

## Getting Started
You need [docker](https://www.docker.com/).
You shoud not need anything else.

## Example

Create some volumes
```
SUDO=$(docker volume create) &&
BIN=$(docker volume create) &&
SBIN=$(docker volume create)
```

Inject into the volumes.
```
docker \
       run \
       --volume /var/run/docker.sock:/var/run/docker.sock:ro \
       emorymerryman/run-injector:0.0.0 \
       --sudo ${SUDO} \
       --bin ${BIN} \
       --sbin ${SBIN} \
       --program-name hello \
       --image emorymerryman/base:0.1.1 \
       --user user \
       --command echo hello world
```

Containers with those volumes can use the injected dependencies.

```
docker \
       run \
       --interactive \
       --tty \
       --rm \
       --volume ${SUDO}:/etc/sudoers.d:ro \
       --volume ${BIN}:/usr/local/bin:ro \
       --volume ${SBIN}:/usr/local/sbin:ro \
       --volume /var/run/docker.sock:/var/run/docker.sock:ro \
       --user user \
       emorymerryman/base:0.1.1 \
       hello
```

## Options

Most of the options are from [docker run](https://docs.docker.com/engine/reference/run/).

The special options are:
1. sudo - This identifies the volume where a sudo file will be written.
2. bin - This indicates the volume where the user script will be written.
3. sbin - This indicates the volume where the root script will be written.
4. image - This indicates the docker image to be run
5. command - This indicates the command (overriding the command from the Dockerfile).


## Motivation
We should keep docker images small.
We prefer minimal bases.
We do not want to install software.
We prefer to use docker to run things.

This is not an image that invokes 'docker run'.
Instead it creates a script that invokes 'docker run'.
It puts this script in the 'sbin' volume.
It is anticipated that the target container will have docker installed.

It is anticipated that the target container will not be run as root and that the regular user will not be part of the docker group.
For this reason, a user script will be put in the bin volume and a sudo file will be put in the sudo volume.
The user script invokes the root script and the sudo file permits this.