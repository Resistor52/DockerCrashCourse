## Task 2 - Explore Docker

Verify that there are no docker images on the local system:

```
sudo docker images
```

Note that if we try to run the `docker images` command with out sudo we get a permissions error:

```
ubuntu@ubuntu:~$ docker images
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/json": dial unix /var/run/docker.sock: connect: permission denied
```

We can fix that by adding the "ubuntu" user to the "docker" group:

```
sudo usermod -aG docker ubuntu
newgrp docker       # Trick to load new group permissions
```

Now we can run `docker images` without being root 

Next, run the Docker version of the classic 'hello world' test:

```
docker run hello-world
```

You should see output that looks like the following:

```
ubuntu@ubuntu:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:9ade9cc2e26189a19c2e8854b9c8f1e14829b51c55a630ee675a5a9540ef6ccf
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

Let's rerun the `docker images` and see what changed:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    feb5d9fea6a5   8 days ago   13.3kB

```

Ok, great. We don't need that image around anymore. So what command do we need to delete the image? Run `docker help` to see a list of all the possible docker commands.

It looks like it would be `docker rmi`, so let's try it:

```
ubuntu@ubuntu:~$ docker rmi hello-world
Error response from daemon: conflict: unable to remove repository reference "hello-world" (must force) - container 39c0bbdce667 is using its referenced image feb5d9fea6a5
```

Well, that didn't work. We can't remove the image while there is a container referencing it. Let's see what containers are running, using `docker ps`

```
ubuntu@ubuntu:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Well, that didn't show us what we wanted to see, let's rerun it with the `--all` option:

```
ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS     NAMES
39c0bbdce667   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago             nifty_turing
```

The `--all` option shows all containers, not just the running containers. (For more info see [https://docs.docker.com/engine/reference/commandline/ps/](https://docs.docker.com/engine/reference/commandline/ps/))

Now that we know the container id that is referencing our `hello-world` image, we can delete it using the `docker rm` command, as follows:

```
docker rm 39c0bbdce667
```

Now if we rerun the `docker ps --all` command, we see that the container is gone:

```
ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Once the container is zapped, we can delete the image:

```
ubuntu@ubuntu:~$ docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:9ade9cc2e26189a19c2e8854b9c8f1e14829b51c55a630ee675a5a9540ef6ccf
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
```

And just to confirm that it is actually deleted:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
