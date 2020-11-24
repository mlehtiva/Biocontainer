---
title: Docker Volumes
---

# Learning Objectives
In scientific applications, the usage of docker volumes is essential to persist data when working with containers. In this session, you will be able to learn:
- How docker enables `data persistence` 
- Different storage types and how they are used

## Description

Docker images are stored as read-only layers. When we start a container from an image, Docker takes the read-only image and adds a read-write layer on top (Union File System). If a file in the running container is modified, the file is copied out of the underlying read-only layer and put it into the top-most read-write layer where the changes are applied. As containers are ephemeral, any changes made inside the container are lost permanently once the container is removed.

In order to save (persist) data and also to share data between containers, Docker came up with the concept of volumes. Quite simply, volumes are directories (or files) that are outside of the default Union File System and exist as normal directories and files on the host filesystem.

we have mainly two different ways of mounting persistent data with docker containers:

- `Bind mounts` (=Mounting a Host Directory inside a container)
- `Docker volumes`

> **Note**: Another type of mount called, **tmpfs mount** is stored in the host system’s memory only and is never written to the host file system. When the container stops, the tmpfs mount is removed, and files written there won’t be persisted. This may be your option If you don't want the data to persist either on the host machine or within the container.  This may be for security reasons or to protect the performance of the container when your application needs to write a large volume of non-persistent state data.

## Bind mounts ##
When you use a bind mount option, a file or directory on the host directory is mounted into a container.  One needs to specify this option during runtime. While it is easy to use, non-docker processes on the Docker host can modify them at any time.

#### Bind mounts example

Let's use *fastqc* container as an example here. For now, don't worry about the context of this container if you are not familiar with *fastqc* software.

Pull the latest Docker image of *fastqc* if it is not already available on host machine as below:

```
docker pull biocontainers/fastqc:v0.11.9_cv7

````
copy few toy examples of fastq files from Puhti supercomputer as below (use your own CSC username):

```
scp your-csc-username@puhti.csc.fi:/scratch/project_2003682/Trinity/*.gz .

```
> **Note**: Check that you have approved Allas and Puhti as services for this course project in [my.csc.fi](https://docs.csc.fi/accounts/how-to-add-service-access-for-project/).

Run *fastqc* analaysis on some of example files as shown below:

```
docker run biocontainers/fastqc:v0.11.9_cv7 \
      fastqc reads.left.fq.gz
```
> issue:
> Skipping '/data/reads.left.fq.gz' which didn't exist, or couldn't be read

The above issue has arised as container file system is isolated from that of host system. In order to persist data, one has to mount host's directory inside the docker container with the `-v` flag in `docker run command` as shown below:

``` bash
docker run --rm -v /home/biouser/Downloads:/data biocontainers/fastqc:v0.11.9_cv7  fastqc /data/reads.left.fq.gz

```

It should work fine now and all results will be written to the "host directory" that was mounted inside the container. However, pay attention to the file permissions of newly created files by docker containers. 
```
> ls -l /home/biouser/Downloads

-rw------- 1 biouser biouser 14905493 Nov 19 16:08 lung3e_1_subset.fq.gz
-rw------- 1 biouser biouser 14955814 Nov 19 16:08 lung3e_2_subset.fq.gz
-rw-rw---- 1 biouser biouser  1252784 Nov 19 14:22 reads2.left.fq.gz
-rw-rw---- 1 biouser biouser  1274471 Nov 19 14:22 reads2.right.fq.gz
-rw-r--r-- 1 root    root      263555 Nov 20 02:07 reads.left_fastqc.html
-rw-r--r-- 1 root    root      345718 Nov 20 02:07 reads.left_fastqc.zip
-rw-rw---- 1 biouser biouser  1251148 Nov 19 14:22 reads.left.fq.gz
-rw-rw---- 1 biouser biouser  1272939 Nov 19 14:22 reads.right.fq.gz
```
You can easily realise that files written by container are owned by root. What if you are not a root user? It will become difficult to view those files that are owned by a root user. One tedious way is to go inside the container and change the permissions of files written by the container. 

Alernative option though is to set Docker user when starting your container:

```bash     
docker run  --user "$(id -u):$(id -g)" --rm -v /home/biouser/Downloads:/data biocontainers/fastqc:v0.11.9_cv7  fastqc /data/reads.right.fq.gz

```
Above flagt with *–user “$(id -u):$(id -g)“* in `docker run command` informs the container to run with current user id and group id which are obtained dynamically through bash command substitution.

With new command, you can see the changes in the permissions of files written this time by fastqc as below:

```bash
-rw------- 1 biouser biouser 14905493 Nov 19 16:08  lung3e_1_subset.fq.gz
-rw------- 1 biouser biouser 14955814 Nov 19 16:08  lung3e_2_subset.fq.gz
-rw-rw---- 1 biouser biouser  1252784 Nov 19 14:22  reads2.left.fq.gz
-rw-rw---- 1 biouser biouser  1274471 Nov 19 14:22  reads2.right.fq.gz
-rw-r--r-- 1 root    root      263555 Nov 20 02:07  reads.left_fastqc.html
-rw-r--r-- 1 root    root      345718 Nov 20 02:07  reads.left_fastqc.zip
-rw-rw---- 1 biouser biouser  1251148 Nov 19 14:22  reads.left.fq.gz
-rw-r--r-- 1 biouser biouser   259532 Nov 20 02:17  reads.right_fastqc.html
-rw-r--r-- 1 biouser biouser   340578 Nov 20 02:17  reads.right_fastqc.zip
-rw-rw---- 1 biouser biouser  1272939 Nov 19 14:22  reads.right.fq.gz

````
## Docker Volumes ##
You can use a named or anonymous volume to store external data. When you choose to use this type of volume, a new directory is created within Docker’s storage area on the host machine and Docker manages that directory’s contents.

Docker volumes can be created in three different ways:

* By explicitly creating it with `docker volume create <volume_name>` command
* By creating a named volume at container creation time with `docker container run -d -v DataVolume:/opt/app/data ...`
* By creating an anonymous volume at container creation time with `docker container run -d -v /opt/app/data ...`

Let's try to make a data volume called `mydata`:

```bash
docker volume create mydata
```

Above docker command creates a volume named `mydata` and can be viewd using `docker volume ls`  command as below:

```
> docker volume ls
DRIVER              VOLUME NAME
local               mydata
```

Unlike `bind mount`, you don't need to specify where `mydata` directory is stored on host path. As mentioned earlier, it is actually stored in host's file system managed by docker.

if you are curious, you can use  `inspect` command to view `mdata` volume created by docker. 

```bash
#command
> docker volume inspect mydata
# output
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": {},
        "Scope": "local"
    }
]
```

You can see that `mydata` volume is mounted at `/var/lib/docker/volumes/mydata/` on the host's file system. Let's mount `mydata` volume inside `fastqc` container and write some file into it as below:

```
docker run --rm -v mydata:/data_volume biocontainers/fastqc:v0.11.9_cv7  touch /data_volume/text.txt
```

You can actually view `text.txt` file in the Docker volume now on host machine as shown below:

```
sudo ls -l /var/lib/docker/volumes/mydata/_data

-rw-r--r-- 1 root root 0 Nov 20 02:32 text.txt

```

## Tips and tricks

As you have seen, the `-v` flag can create a bind mount or named volume depending on the syntax. If the first argument begins with a / or ~/ you're creating a *bind mount*. Remove that, and you're naming the volume. For example:

* `-v /path:/path/in/container` mounts the host directory, `/path` at the `/path/in/container`
* `-v path:/path/in/container` creates a volume named `path` to the host machine.

# Conclusion
Bind mounts and docker volumes are the Docker way of persisting data beyond the lifetime of containers. In this exercise, we saw how to use these two different types of persisting data. 
