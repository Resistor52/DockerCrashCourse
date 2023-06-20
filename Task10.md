## Task 10 - Push the Image to Docker Hub

Before we can push the image we need to tag it with our repository name and log into Docker Hub from the VM. Let's start by looking at the images that we have on the system:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
my_tools     latest    747e211c3dbc   9 minutes ago    2GB
<none>       <none>    fb0bdaa3f3df   21 minutes ago   2GB
<none>       <none>    94294c2a8081   27 minutes ago   1.14GB
<none>       <none>    8f79b2b824f5   30 minutes ago   522MB
sans         latest    8b3f354b7385   37 minutes ago   187MB
ubuntu       latest    99284ca6cea0   11 days ago      77.8MB
```

Notice the untagged images. These are the images that we created in Tasks 5, 6, and 7. 

Our image is "my_tools" but it needs to have our Docker Hub username as part of the tag. my Docker Hub user name is "$DHUSER" so I will use the following command to tag my image:

```
docker tag my_tools $DHUSER/my_tools
```

Now when we run `docker images` we see two lines with the same Image ID and we see that an entry with our user name has been added.

Now, to login to docker hub. Type `docker login` and provide your Docker Hub credentials:

```
ubuntu@ubuntu:~$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: resistor52
Password:
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

Cool, now we can push our image up to the Docker Hub repo:

```
docker push $DHUSER/my_tools:latest
```

It will take a little bit while each layer is pushed up, but eventually, you will get something that looks like:

```
ubuntu@ubuntu:~$ docker push $DHUSER/my_tools:latest
The push refers to repository [docker.io/resistor52/my_tools]
246e1b6ea448: Pushed
5779442fb74b: Pushed
f89896336d37: Pushed
5f70bf18a086: Pushed
91b8fb9a0655: Pushed
966e94ab6e16: Mounted from library/ubuntu
latest: digest: sha256:d6ac6ed27434fe37822665381fe80705cd44ed198db2ab74179c60641ac6d068 size: 1586
```

Now, refresh the Docker Hub webpage that is showing your repository and you will see the image with the 'latest' tag is showing.

Click on the "latest" tag as it is a hyperlink. This new page shows the layers. if we select a layer, we can see the commands that are part of the layer.