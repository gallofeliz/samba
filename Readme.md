# Samba

A [Docker](http://docker.com) file to build images for many platforms (linux/amd64, linux/arm64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6) with a installation of [Samba](https://www.samba.org/) that is the standard Windows interoperability suite of programs for Linux and Unix. This is my own Multi-architecture docker recipe.

> Be aware! You should read carefully the usage documentation of every tool!

## Thanks to

- [Samba](https://www.samba.org/)
- [dastrasmue rpi-samba](https://github.com/dastrasmue/rpi-samba)

## Details

- [GitHub](https://github.com/DeftWork/samba)
- [Deft.Work my personal blog](http://deft.work/Samba)

| Docker Hub | Docker Pulls | Docker Stars | Size/Layers |
| --- | --- | --- | --- |
| [samba](https://hub.docker.com/r/elswork/samba "elswork/samba on Docker Hub") | [![Docker Pulls](https://img.shields.io/docker/pulls/elswork/samba.svg)](https://hub.docker.com/r/elswork/samba "elswork/samba on Docker Hub") | [![Docker Stars](https://img.shields.io/docker/stars/elswork/samba.svg)](https://hub.docker.com/r/elswork/samba "elswork/samba on Docker Hub") | [![Size/Layers](https://images.microbadger.com/badges/image/elswork/samba.svg)](https://microbadger.com/images/elswork/samba "elswork/samba on microbadger.com") |

## Build Instructions

Build for amd64, armv7l, aarch64 architecture (thanks to its [Multi-Arch](https://blog.docker.com/2017/11/multi-arch-all-the-things/) base image)

``` sh
docker build -t elswork/samba .
```

## Usage

I use it to share files between Linux and Windows, but Samba has many other capabilities.

ATTENTION: This is a recipe highly adapted to my needs, it might not fit yours.
Deal with local filesystem permissions, container permissions and Samba permissions is a Hell, so I've made a workarround to keep things as simple as possible.
I want avoid that the usage of this conainer would affect current file permisions of my local system, so, I've "synchronized" the owner of the path to be shared with Samba user. This mean that some commitments and limitations must be assumed.

Container will be configured as samba sharing server and it just needs:

- host directories to be mounted,
- users (one or more uid:gid:username:usergroup:password tuples) provided,
- shares defined (name, path, users).

-u uid:gid:username:usergroup:password

- uid from user p.e. 1000
- gid from group that user belong p.e. 1000
- username p.e. alice
- usergroup (the one to whom user belongs) p.e. alice
- password (The password may be different from the user's actual password from your host filesystem)

-s name:path:rw:user1[,user2[,userN]][:recycle]

- add share, that is visible as 'name', exposing contents of 'path' directory for read+write (rw) or read-only (ro) access for specified logins user1, user2, .., userN ; Add optional recycle arg to activate recycle with value as dirname

### Serve

Start a samba fileshare.

``` sh
docker run -d -p 139:139 -p 445:445 \
  -- hostname any-host-name \ # Optional
  -e TZ=Europe/Madrid \ # Optional
  -v /any/path:/share/data \ # Replace /any/path with some path in your system owned by a real user from your host filesystem
  elswork/samba \
  -u "1000:1000:alice:alice:put-any-password-here" \ # At least the first user must match (password can be different) with a real user from your host filesystem
  -u "1001:1001:bob:bob:secret" \
  -u "1002:1002:guest:guest:guest" \
  -s "Backup directory:/share/backups:rw:alice,bob" \
  -s "Alice (private):/share/data/alice:rw:alice:.recycle-bin" \
  -s "Bob (private):/share/data/bob:rw:bob" \
  -s "Documents (readonly):/share/data/documents:ro:guest,alice,bob"
```

This is my real usage command:

``` sh
docker run -d -p 139:139 -p 445:445 -e TZ=Europe/Madrid \
    -v /home/pirate/docker/makefile:/share/folder elswork/samba \
    -u "1000:1000:pirate:pirate:put-any-password-here" \
    -s "SmbShare:/share/folder:rw:pirate"
```

or this if the user that owns the path to be shared match with the user that raise up the container:

``` sh
docker run -d -p 139:139 -p 445:445 --hostname $HOSTNAME -e TZ=Europe/Madrid \
    -v /home/pirate/docker/makefile:/share/folder elswork/samba \
    -u "$(id -u):$(id -g):$(id -un):$(id -gn):put-any-password-here" \
    -s "SmbShare:/share/folder:rw:$(id -un)"
```

On Windows point your filebrowser to `\\host-ip\` to preview site.

---
**[Sponsor me!](https://github.com/sponsors/elswork) Together we will be unstoppable.**
