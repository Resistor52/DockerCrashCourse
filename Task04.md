## Task 4 - Create a Custom Image

The most common use case for docker is to run web services, so let's create a simple web server to illustrate some important concepts. First, lets create a basic web page called "index.html" in a directory called "html"

```
mkdir html
echo "SANS Docker Crash Course" > html/index.html

```

With that done, we need to create a `Dockerfile`

Images are typically built from a base image adding and removing stuff as needed. Our base image will be 'nginx' since it already has the web server installed. Then we will copy our "html" folder onto the image stomping on the existing  /usr/share/nginx/html

```
echo 'FROM nginx' > Dockerfile
echo 'COPY html /usr/share/nginx/html' >> Dockerfile

```

That creates our Dockerfile, thanks to the magic of file redirection. Let's double-check our Dockerfile. Run `cat Dockerfile` and you should see:

```
ubuntu@ubuntu:~$ cat Dockerfile
FROM nginx
COPY html /usr/share/nginx/html

```

Now let's build a custom image based on our Dockerfile:

```
docker build -t sans .
```

Note that the "." indicates "the current directory"

```
ubuntu@ubuntu:~$ docker build -t sans .
Sending build context to Docker daemon  14.85kB
Step 1/2 : FROM nginx
latest: Pulling from library/nginx
07aded7c29c6: Pull complete
bbe0b7acc89c: Pull complete
44ac32b0bba8: Pull complete
91d6e3e593db: Pull complete
8700267f2376: Pull complete
4ce73aa6e9b0: Pull complete
Digest: sha256:765e51caa9e739220d59c7f7a75508e77361b441dccf128483b7f5cce8306652
Status: Downloaded newer image for nginx:latest
 ---> f8f4ffc8092c
Step 2/2 : COPY html /usr/share/nginx/html
 ---> 995b0de5245b
Successfully built 995b0de5245b
Successfully tagged sans:latest
ubuntu@ubuntu:~$

```

Now when we run `docker images` we see two new images along with the 'ubuntu' image from before:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
sans         latest    995b0de5245b   44 seconds ago   133MB
ubuntu       latest    597ce1600cf4   47 hours ago     72.8MB
```

The "sans" is the image that we just created.

Ok, lets spin up a container from this image.

```
docker run --name crashcourse -d -p 8080:80 sans
```

And we get:

```
ubuntu@ubuntu:~$ docker run --name crashcourse -d -p 8080:80 sans
61747901bb594c88ec03736a50886985b404aca4a4a3d85b2b9608d190a8c626
```

Now, lets hit it with curl:

```
curl localhost:8080
```

You should see something like:

```
ubuntu@ubuntu:~$ curl localhost:8080
SANS Docker Crash Course
ubuntu@ubuntu:~$

```

Note that the `-d` option ran the container in the background and the `-p` option maps port 8080 on the host to port 80 in the container.

We can run `docker ps` to see that it is still running in the background:

```
ubuntu@ubuntu:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
61747901bb59   sans      "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   crashcourse
```

If we want to see what processes are running inside the container, we can use `docker top [CONTAINER]` like so:

```
ubuntu@ubuntu:~$ docker top crashcourse
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                10642               10616               0                   01:47               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            10693               10642               0                   01:47               ?                   00:00:00            nginx: worker process

```

If we stop the container, we will no longer be able to connect to port 8080:

```
ubuntu@ubuntu:~$ docker stop crashcourse
crashcourse
ubuntu@ubuntu:~$ curl localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

Well, there you go. We installed Docker Engine Community Edition, learned some basic Docker commands, and launched a very basic website. 

Before proceeding to Task 5, clean up the ubuntu user's home directory:

```
rm -r html Dockerfile
```