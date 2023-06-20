## Task 8 - Non-privilaged User


The cloud command line interfaces are designed to be run as a non-privileged user, so let's make that change to our
Dockerfile. Using a text editor, add the following lines right below the `FROM ubuntu` line at the top of the file:

```
# Add the Non-privileged user
RUN useradd -m sans && apt update && apt install -y sudo && echo "sans ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER sans
WORKDIR /home/sans
```

What do these commands do? First, we are adding a new user called **sans** complete with a home directory. Next we install `sudo` so that the **sudoers** file is created as part of the install process. lastly, we need to add a line to the **sudoers** file to prevent the prompting for a password when `sudo` is run from the **sans** user context.  

The USER instruction tells docker to switch to the "sans" user for the following steps while the WORKDIR instruction changes the working directory. To learn more about these instructions, see the [official documentation](https://docs.docker.com/engine/reference/builder/#user)

Make sure that your Docker file looks like:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -m sans && apt update && apt install -y sudo && echo "sans ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER sans
WORKDIR /home/sans

# AWS CLI
RUN sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y
```

Great, let's test the build. Note that it will take longer because Docker will use cached layers if it can, but since we are now installing the three CLI's under the `sans` user context, all of the intermediate containers will need to be rebuilt.:

```
docker build -t my_tools .

```

Bonk! We have an error:

```
---> Running in 76d84d16d559

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Reading package lists...
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
The command '/bin/sh -c apt update && apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip' returned a non-zero code: 100
```

Upon inspection of the error we can see that it is in the RUN command that installs the AWS CLI and it is a "Permission denied" error. Since it worked before we changed the execution context to the "sans" user, it is clear that we are missing some `sudo` commands.

Change the AWS RUN instruction as follows:

```
RUN sudo apt update && sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip
```

And, let's test it again:

```
docker build -t my_tools .

```

Another error:

```
---> Running in eac768a8ebaf
deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main
tee: /etc/apt/sources.list.d/google-cloud-sdk.list: Permission denied
The command '/bin/sh -c echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -yI' returned a non-zero code: 1
```

It's the same type of error as before, but this time in the GCloud RUN instruction. Change the last two lines of the Dockerfile to read as follows:

```
# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && sudo apt update -y && sudo apt install google-cloud-sdk -y
```

Now it should build without an error:

```
docker build -t my_tools .
```

Awesome! Now we can run it and put it thru it's paces:

```
ubuntu@ubuntu:~$ docker run --rm -it my_tools bash
sans@9e703ed5f1de:~$ aws --version
aws-cli/2.12.0 Python/3.11.3 Linux/5.19.0-43-generic exe/x86_64.ubuntu.22 prompt/off
sans@9e703ed5f1de:~$ az --version
azure-cli                         2.49.0

core                              2.49.0
telemetry                          1.0.8

Dependencies:
msal                              1.20.0
azure-mgmt-resource               22.0.0

Python location '/opt/az/bin/python3'
Extensions directory '/home/sans/.azure/cliextensions'

Python (Linux) 3.10.10 (main, May 19 2023, 08:20:31) [GCC 11.3.0]

Legal docs and information: aka.ms/AzureCliLegal


Your CLI is up-to-date.
sans@9e703ed5f1de:~$ gcloud --version
Google Cloud SDK 435.0.1
alpha 2023.06.14
beta 2023.06.14
bq 2.0.93
bundled-python3-unix 3.9.16
core 2023.06.14
gcloud-crc32c 1.0.0
gsutil 5.24
sans@9e703ed5f1de:~$
```

Everything looks good!

Well, there you go. We created a Docker image that has each of the command line interfaces for AWS, Azure, and GCP. Along the way, we performed some troubleshooting and configured the image to execute as a non-privileged user. At this point, you should have confidence creating your own custom docker images.