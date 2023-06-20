## Task 5 - AWS CLI

Now that we understand the basics of creating a custom container, we will expand on that knowledge by installing some additional packages and addressing typical situations that may come up.

Recall that a Docker image is created using a Dockerfile, so to start, add a line to a new Dockerfile to specify that our base image will be `ubuntu`

```
echo 'FROM ubuntu' > Dockerfile
echo " " >> Dockerfile
```

Next, we need to look up the commands to install the AWS CLI onto Ubuntu.

According to [https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html) The commands are:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

But let's test them in a container based on our "ubuntu" image to see if we are missing any dependencies. Spin up the container in interactive mode:

```
docker run --name dev -it ubuntu
```

and then paste in the first command:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

We see that we get an error:

```
root@b0dabf129639:/# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
bash: curl: command not found

```

As suspected, curl is not installed yet. Let's install it:

```
apt update
apt install -y curl

```

Ok, try the curl command again:

```
root@b0dabf129639:/# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 42.4M  100 42.4M    0     0  50.3M      0 --:--:-- --:--:-- --:--:-- 50.3M
```

Good, try the next command:

```
unzip awscliv2.zip
```

It is not installed either, so install it:

```
apt install -y unzip

```

Now we can unzip the file that we downloaded to our container:

```
unzip awscliv2.zip

```

At this point, we can try to install the AWS CLI:

```
sudo ./aws/install
```
We get:

```
root@2883310a829c:/# sudo ./aws/install
bash: sudo: command not found
```

Oops, `sudo` is not installed either. Let's add it as well:

```
apt install -y sudo

```

And now, finally, we can run the AWS CLI installer:

```
sudo ./aws/install

```

And we get:

```
root@2883310a829c:/# sudo ./aws/install
You can now run: /usr/local/bin/aws --version
```

To test that the CLI was properly installed, just run:

```
aws --version
```

And we should get something like:

```
root@2883310a829c:/# aws --version
aws-cli/2.2.45 Python/3.8.8 Linux/5.4.0-1045-aws exe/x86_64.ubuntu.20 prompt/off
```

All right, hence we conclude that the full set of commands to install the AWS CLI are:

```
apt update
apt install -y curl unzip sudo
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
rm awscliv2.zip
```

For good housekeeping, we removed the zip file as it is no longer necessary. Also, since these commands were executed as root, we technically did not need to install the CLI using `sudo` however, toward the end of the episode we will be modifying our container so that we can use the CLI tools from a non-privileged user and that will require that we create our image with `sudo` installed.

Type `exit` to exit the interactive container.

In a Dockerfile, it is considered a best practice to separate related commands with `&&` which means _execute the following command only if the previous command ran successfully_ (returned an exit code of zero).

Let's add this to our Dockerfile as a RUN command:

```
echo "# AWS CLI" >> Dockerfile
echo 'RUN apt update && apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip' >> Dockerfile
echo " " >> Dockerfile

```

Every RUN command creates a new layer (or intermediate image) and that's why related commands are grouped into a single RUN command. See [https://docs.docker.com/engine/reference/builder/#run](https://docs.docker.com/engine/reference/builder/#run) to learn more.

Now let's test our Dockerfile. First by building our image:

```
docker build -t my_tools .
```

and then by running it:

```
docker run --rm  -it my_tools bash
```

Now we should be able to run `aws --version` and see a result similar to:

```
root@5fe7341eff0d:/# aws --version
aws-cli/2.2.43 Python/3.8.8 Linux/5.4.0-1045-aws exe/x86_64.debian.10 prompt/off
```

As before, type `exit` to terminate the container.

