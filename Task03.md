## Task 3 - The Ubuntu Image

Great, now let's pull down a different image. Let's pull down the official Ubuntu container image from Docker hub:

```
docker pull ubuntu
```

You should see:

```
ubuntu@ubuntu:~$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
f3ef4ff62e0d: Pull complete
Digest: sha256:44ab2c3b26363823dcb965498ab06abf74a1e6af20a732902250743df0d4172d
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

We can see that it was downloaded:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    597ce1600cf4   46 hours ago   72.8MB
```

Next, let's run the container in an interactive mode so that we execute various commands from the bash shell within the container. To do that, we need two options:

* -i --> Interactive, Keep STDIN open even if not attached
* -t --> Allocate a pseudo-TTY

While we are at it, we will assign a name to the container with the `--name` option.

So our command becomes:

```
docker run --name test -it ubuntu bash
```

And then we see our prompt change. Now we can run some commands that are executed inside the container:

```
ubuntu@ubuntu:~$ docker run --name test -it ubuntu bash
root@a843c11e5313:/# whoami
root
root@a843c11e5313:/# hostname
a843c11e5313
root@a843c11e5313:/# pwd
/
root@a843c11e5313:/# ls /
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@a843c11e5313:/# ls /home
root@a843c11e5313:/# exit
exit
ubuntu@ubuntu:~$
```

The `exit` command kills the container executing in the foreground and returns us to our original prompt.

List the container:

```
ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS                     PORTS     NAMES
a843c11e5313   ubuntu    "bash"    4 minutes ago   Exited (0) 3 minutes ago             test
```

We can run the container again by using the `docker start` command, remembering to pass in the interactive option:

```
ubuntu@ubuntu:~$ docker start -i test
root@a843c11e5313:/#
root@a843c11e5313:/# exit
exit
ubuntu@ubuntu:~$
```  

If we use the `--rm` option with the run command, it cleans up the container when the command passed into the container exits:

```
docker run --name test2 -it --rm ubuntu ls
```

In this case, the ls command runs in the container, and then the container exits:

```
ubuntu@ubuntu:~$ docker run --name test2 -it --rm ubuntu ls
bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  etc  lib   lib64  media   opt  root  sbin  sys  usr
ubuntu@ubuntu:~$
```

Note that we named the container "test2" but when we run `docker ps --all` we do not see it listed, because of the `--rm` switch.

```

ubuntu@ubuntu:~$ docker ps --all
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                         PORTS     NAMES
a843c11e5313   ubuntu    "bash"                   2 hours ago      Exited (0) About an hour ago             test
```
