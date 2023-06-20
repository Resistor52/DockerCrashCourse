## Task 12 - Enable Vulnerability Scanning

Another amazing feature of Docker Hub is image vulnerability scanning. Let's enable that next. Back in the Docker Hub Web UI, click on the "settings" tab under our "my_tools" repository.

Then select the "Advanced image analysis provided by Docker Scout" radio button as shown in the image below:

![Image insight settings with Advanced image analysis selected](/images/dockerhub_scanning.png)

Not sure if you noticed, but the shell for our ubuntu user is `sh` and perhaps we want it to be `bash`

Lets change lines 1-7 to read:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -s /bin/bash -m hitc && apt update && apt install -y sudo git && echo "hitc ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER hitc
WORKDIR /home/hitc
SHELL ["/bin/bash", "-c"]
```

Now when we build the image and then push it to Docker Hub, it should kick off an automated vulnerability scan. First to build and tag the image:

```
docker build -t my_tools .
docker tag my_tools $DHUSER/my_tools
docker images
```

Don't forget that we will need to login again:

```
docker login
```

And then push the image:

```
docker push $DHUSER/my_tools:latest
```

When you look in the Docker Hub Web Page under the "General" tab, it should show that you just pushed your image. Click on the image tagged "latest" to see the details. It may take several minutes for the scan results to show up, but when they do, you will see that even the latest versions of our CLI's have vulnerabilities.

![Scan results from Docker Scout](/images/scan_results.png)

