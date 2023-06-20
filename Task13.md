## Task 13 - OPTIONAL - Github Integration

**NOTE:: This Task requires a paid Docker Hub Account. At a minumum, you can sign up for 1 month at $5/month**

Wouldn't it be cool if we could connect a Github code repo with our Docker Hub image repo such that every time we committed a code change to our main branch, Docker Hub would automatically build the image for us?

Well, we will be doing that next. Note that this capability assumes that you have a paid Docker Hub subscription.

Log into your Github account and create a new **public** Git Repo called "my_tools." Use the following blurb for the description:

 ```
A Docker container of cloud tools. Includes the AWS, Azure, and GCloud command line interfaces.
 ```

Click the "Add a README file" and chose the "MIT License." Click the green "Create Repository" button.

Next, let's add our Dockerfile to the Git Repo. In the Web UI, click the "Add file" button and select "Create new file." Call the new file Dockerfile and paste in the following contents:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -s /bin/bash -m sans && apt update && apt install -y sudo git && echo "sans ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER sans
WORKDIR /home/sans
SHELL ["/bin/bash", "-c"]

# AWS CLI
RUN sudo apt update && sudo apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && sudo apt update -y && sudo apt install google-cloud-sdk -y
```

Click the "Commit new file" button at the bottom of the page.

Now that we have a Git Repo with our Dockerfile, lets connect that to Docker Hub.

Back in the Docker Hub web page, looking at the "my_tools" repository, click on the "Builds" tab. Click the "Link to Github" button. This will open up a new page. On the line that lists Github, click the "Connect" hyperlink and link your Github account to Docker Hub.

Click the button again to inform Docker Hub of which Git Repo to use.

* Set the Source Repository. (In my case it is "Resistor52 / my_tools")
* Change the Autotest setting to "Internal Pull Requests"
* Change the Build Rules so that the "Source" field reads "main" instead of "master" **NOTE:** The default branch in github for new repos in now "main" so as to be more inclusive.
* Click "save"

Great. Now let's make some changes to our Dockerfile in Github and verify that Docker Hub gets triggered to build a new image when we commit the change to Github.

Using the Web UI for Github, edit the Docker file. We want to make two changes. First, we want to install `git`, so add that to line 4 after the word "sudo." While we are at it, let's use the line continuation character "`\`" to make our code more readable.  

Once your code looks like the following, go ahead and commit the change:

```
FROM ubuntu

# Add the Non-privileged user
RUN useradd -s /bin/bash -m sans && apt update && apt install -y sudo git && \
echo "sans ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
USER sans
WORKDIR /home/sans
SHELL ["/bin/bash", "-c"]

# AWS CLI
RUN sudo apt update && sudo apt install -y curl unzip sudo \
&& curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
&& unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg \
&& curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor \
| sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) \
&& echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" \
| sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" \
| sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list \
&& curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - \
&& sudo apt update -y && sudo apt install google-cloud-sdk -y
```

Back in the Docker Hub Web UI, looking at the Builds tab. Click refresh. After a few minutes, you should now see "IN PROGRESS" under the "Latest Build Status" column. It will take a while, but eventually Docker Hub will finish building the new image.

# Test the New Image

Once the latest Build status reads "SUCCESS" we can test our new image. Back on the VM, let's run the new image right from the repo.

First, to demonstrate this, lets zap everything local using `docker system prune --all`

Now run a container based on our Docker Hub image:

```
docker run --rm -it $DHUSER/my_tools
```  

Note that the output reads "unable to find image locally" so docker has to pull it down before docker can run it.:

```
ubuntu@ubuntu:~$ docker run --rm -it resistor52/my_tools
Unable to find image 'resistor52/my_tools:latest' locally
latest: Pulling from resistor52/my_tools
7b1a6ab2e44d: Pull complete
1b60658ffa6f: Pull complete
4f4fb700ef54: Pull complete
aa816e75ae95: Pull complete
1aded45ec442: Pull complete
907e0e173b79: Pull complete
Digest: sha256:d0039763ccc5c07fd905fe5ef5aa053fe624b63a2a43225c69efdd5498a4b429
Status: Downloaded newer image for resistor52/my_tools:latest
sans@0a84b95a1da9:~$
```

Since we have the container's prompt, let's see if the container has git installed. Run `git --version`:

```
sans@0a84b95a1da9:~$ git --version
git version 2.25.1
```

Awesome.