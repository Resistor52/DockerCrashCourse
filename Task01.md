# Task1 - Setup

For this workshop, we will assume that you have a fresh Ubuntu 20.01 (or greater) virtual machine running and have a SSH connection to it. We also assume that you have a user named "ubuntu." If you do not, run the following commands:

```
sudo mkdir /home/ubuntu
sudo useradd ubuntu -d /home/ubuntu -s /bin/bash -G sudo
sudo chown ubuntu:ubuntu /home/ubuntu
echo "Set the password for the new user: ubuntu"
sudo passwd ubuntu
sudo su - ubuntu
```

For your reference, this section follows the steps outlined in the [Docker installation documentation](https://docs.docker.com/engine/install/ubuntu/#installation-methods).

First, set up the repo by running the following commands:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

Then add Docker’s official GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

Next, set up Docker Engine (Community Edition):

```
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

```
