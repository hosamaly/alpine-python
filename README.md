# alpine-python

[![Docker Stars](https://img.shields.io/docker/stars/jfloff/alpine-python.svg)][hub]
[![Docker Pulls](https://img.shields.io/docker/pulls/jfloff/alpine-python.svg)][hub]
[![Build Status](https://travis-ci.org/jfloff/alpine-python.svg?branch=master)][travis]

[hub]: https://hub.docker.com/r/jfloff/alpine-python/
[travis]: https://travis-ci.org/jfloff/alpine-python

A small Python Docker image based on [Alpine Linux](http://alpinelinux.org/).

<!-- MDTOC maxdepth:6 firsth1:0 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [alpine-python](#alpine-python)
  - [Supported tags](#supported-tags)
  - [Why?](#why)
  - [Details](#details)
  - [Usage](#usage)
  - [Usage of `onbuild` images](#usage-of-onbuild-images)
  - [Usage of `slim` images](#usage-of-slim-images)
      - [Via `docker run`](#via-docker-run)
      - [Pip Dependencies](#pip-dependencies)
      - [Run-Time Dependencies](#run-time-dependencies)
      - [Build-Time Dependencies](#build-time-dependencies)
      - [Creating Images](#creating-images)
      - [Debugging](#debugging)
      - [Additional Arguments](#additional-arguments)
  - [Ecosystem](#ecosystem)
  - [License](#license)

<!-- /MDTOC -->

## Supported tags
* **`2.7` ([2.7/Dockerfile](https://github.com/jfloff/alpine-python/blob/master/2.7/Dockerfile))**
* **`2.7-onbuild` ([2.7-onbuild/Dockerfile](https://github.com/jfloff/alpine-python/blob/master/2.7-onbuild/Dockerfile))**
* **`2.7-slim` ([2.7-slim/Dockerfile](https://github.com/jfloff/alpine-python/blob/master/2.7-slim/Dockerfile))**
* **`3.6` ([Dockerfile](https://github.com/jfloff/alpine-python/blob/master/3.6/Dockerfile))**
* **`3.6-onbuild` ([Dockerfile](https://github.com/jfloff/alpine-python/blob/master/3.6-onbuild/Dockerfile))**
* **`3.6-slim` ([Dockerfile](https://github.com/jfloff/alpine-python/blob/master/3.6-slim/Dockerfile))**
* **`3.7` `latest` ([Dockerfile](https://github.com/jfloff/alpine-python/blob/master/3.7/Dockerfile))**
* **`3.7-onbuild` `latest-onbuild` ([Dockerfile](https://github.com/jfloff/alpine-python/blob/master/3.7-onbuild/Dockerfile))**
* **`3.7-slim` `latest-slim` ([Dockerfile](https://github.com/jfloff/alpine-python/blob/master/3.7-slim/Dockerfile))**

**NOTES:**
- `onbuild` images install the `requirements.txt` of your project from the get go. This allows you to cache your requirements right in the build. _Make sure you are in the same directory of your `requirements.txt` file_.

## Why?
The default docker python images are too [big](https://github.com/docker-library/python/issues/45), much larger than they need to be. Hence I built this simple image based on [docker-alpine](https://github.com/gliderlabs/docker-alpine), that has everything needed for the most common python projects - including `python-dev` (which is not common in most minimal alpine python packages).

|REPOSITORY|TAG|SIZE|
|-|-|-|
|jfloff/alpine-python|2.7-slim|60MB|
|python|2.7-slim|120MB|
|python|2.7-alpine|61.2MB|
||||
|jfloff/alpine-python|2.7|235MB|
|python|2.7|912MB|
||||
|jfloff/alpine-python|3.6-slim|76.3MB|
|python|3.6-slim|138MB|
|python|3.6-alpine|79MB|
||||
|jfloff/alpine-python|3.6|252MB|
|python|3.6|922MB|
||||
|jfloff/alpine-python|3.7-slim|80.4MB|
|python|3.7-slim|86.7MB|
|python|3.7-alpine|143MB|
||||
|jfloff/alpine-python|3.7|256MB|
|python|3.7|927MB|

Perhaps this could be even smaller, but I'm not an Alpine guru. **Feel free to post a PR.**

## Details
* Installs `build-base` and `python-dev`, allowing the use of more advanced packages such as `gevent`
* Installs `bash` allowing interaction with the container
* Just like the main `python` docker image, it creates useful symlinks that are expected to exist, e.g. `python3` > `python`, `pip2.7` > `pip`, etc.)
* Added `testing` and `community` repositories to Alpine's `/etc/apk/repositories` file

## Usage

This image runs `python` command on `docker run`. You can either specify your own command, e.g:
```shell
docker run --rm -ti jfloff/alpine-python python hello.py
```

You can also access `bash` inside the container:
```shell
docker run --rm -ti jfloff/alpine-python bash
```

## Usage of `onbuild` images

These images can be used to bake your dependencies into an image by extending the plain python images. To do so, create a custom `Dockerfile` like this:
```dockerfile
FROM jfloff/alpine-python:3.6-onbuild

# for a flask server
EXPOSE 5000
CMD python manage.py runserver
```

Don't forget to build that `Dockerfile`:
```shell
docker build --rm=true -t jfloff/app .

docker run --rm -t jfloff/app
```

Personally, I build an extended `Dockerfile` version (like shown above), and mount my specific application inside the container:
```shell
docker run --rm -v "$(pwd)":/home/app -w /home/app -p 5000:5000 -ti jfloff/app
```

## Usage of `slim` images

These images are very small to download, and can install requirements at run-time via flags. The install only happens the first time the container is run, and dependencies can be baked in (see Creating Images).

#### Via `docker run`
These images can be run in multiple ways. With no arguments, it will run `python` interactively:
```shell
docker run --rm -ti jfloff/alpine-python:2.7-slim
```

If you specify a command, they will run that:
```shell
docker run --rm -ti jfloff/alpine-python:2.7-slim python hello.py
```

#### Pip Dependencies
Pip dependencies can be installed by the `-p` switch, or a `requirements.txt` file.

If the file is at `/requirements.txt` it will be automatically read for dependencies. If not, use the `-P` or `-r` switch to specify a file.
```shell
# This runs interactive Python with 'simplejson' and 'requests' installed
docker run --rm -ti jfloff/alpine-python:2.7-slim -p simplejson -p requests

# Don't forget to add '--' after your dependencies to run a custom command:
docker run --rm -ti jfloff/alpine-python:2.7-slim -p simplejson -p requests -- python hello.py

# This accomplishes the same thing by mounting a requirements.txt in:
echo 'simplejson' > requirements.txt
echo 'requests' > requirements.txt
docker run --rm -ti \
  -v requirements.txt:/requirements.txt \
  jfloff/alpine-python:2.7-slim python hello.py

# This does too, but with the file somewhere else:
echo 'simplejson requests' > myapp/requirements.txt
docker run --rm -ti \
  -v myapp:/usr/src/app \
  jfloff/alpine-python:2.7-slim \
    -r /usr/src/app/requirements.txt \
    -- python /usr/src/app/hello.py
```

#### Run-Time Dependencies
Alpine package dependencies can be installed by the `-a` switch, or an `apk-requirements.txt` file.

If the file is at `/apk-requirements.txt` it will be automatically read for dependencies. If not, use the `-A` switch to specify a file.

You can also try installing some Python modules via this method, but it is possible for Pip to interfere if it detects a version problem.
```shell
# Unknown why you'd need to do this, but you can!
docker run --rm -ti jfloff/alpine-python:2.7-slim -a openssl -- python hello.py

# This installs libxml2 module faster than via Pip, but then Pip reinstalls it because Ajenti's dependencies make it think it's the wrong version.
docker run --rm -ti jfloff/alpine-python:2.7-slim -a py-libxml2 -p ajenti
```

#### Build-Time Dependencies
Build-time Alpine package dependencies (such as compile headers) can be installed by the `-b` switch, or a `build-requirements.txt` file. They will be removed after the dependencies are installed to save space.

If the file is at `/build-requirements.txt` it will be automatically read for dependencies. If not, use the `-B` switch to specify a file.

`build-base`, `linux-headers` and `python-dev` are always build dependencies, you don't need to include them.
```shell
docker run --rm -ti jfloff/alpine-python:2.7-slim \
  -p gevent \
  -p libxml2 \
  -b libxslt-dev \
  -b libxml-dev \
  -- python hello.py
```

#### Creating Images
Similar to the onbuild images, dependencies can be baked into a new image by using a custom `Dockerfile`, e.g:
```dockerfile
FROM jfloff/alpine-python:2.7-slim
RUN /entrypoint.sh \
  -p ajenti-panel \
  -p ajenti.plugin.dashboard \
  -p ajenti.plugin.settings \
  -p ajenti.plugin.plugins \
  -b libxml2-dev \
  -b libxslt-dev \
  -b libffi-dev \
  -b openssl-dev \
&& echo
CMD ["ajenti-panel"]
# you won't be able to add more dependencies later though-- see 'Debugging'
```

#### Debugging
The `/entrypoint.sh` script that manages dependencies in the slim images creates an empty file, `/requirements.installed`, telling the script not to install any dependencies after the container's first run. Removing this file will allow the script to work again if it is needed.

You can use the `-x` flag to see everything the `/entrypoint.sh` script is doing.

You can also access `bash` inside the container:
```shell
docker run --rm -ti jfloff/alpine-python:2.7-slim bash
```

#### Additional Arguments

`-q`: silences output from `/entrypoint.sh`
`-x`: turns on Bash debugging, making the output very verbose.

## Ecosystem

These are some of the images that use `jfloff/alpine-python` as base image. *If you have another image that uses this as base image, please submit an issue or PR for it to be added. Image has to be published on Docker Hub.*

- **[jfloff/alscipy](https://github.com/jfloff/docker-alscipy)** [![Docker Stars](https://img.shields.io/docker/stars/jfloff/alscipy.svg)][alscipy-hub] [![Docker Pulls](https://img.shields.io/docker/pulls/jfloff/alscipy.svg)][alscipy-hub] : image with common packages for Science in Alpine Python.
- **[jfloff/pywfm](https://github.com/jfloff/pywfm)** [![Docker Stars](https://img.shields.io/docker/stars/jfloff/pywfm.svg)][pywfm-hub] [![Docker Pulls](https://img.shields.io/docker/pulls/jfloff/pywfm.svg)][pywfm-hub] : image from the python wrapper for Steffen Rendle's factorization machines library libFM.

[alscipy-hub]: https://hub.docker.com/r/jfloff/alscipy/
[pywfm-hub]: https://hub.docker.com/r/jfloff/alscipy/




## License
The code in this repository, unless otherwise noted, is MIT licensed. See the `LICENSE` file in this repository.
