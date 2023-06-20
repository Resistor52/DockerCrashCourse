## Task 7 - Add the  GCloud CLI to the Image

One more cloud service provider to go--Google Cloud Platform.

The nice thing about the [Google documentation](https://cloud.google.com/sdk/docs/install) is it provides you with the Docker run command!

```
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y

```

Let's use a text editor this time to modify the Docker file. It should look like this when done:

```
FROM ubuntu

# AWS CLI
RUN apt update && apt install -y curl unzip sudo && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip && sudo ./aws/install && rm awscliv2.zip

# Azure CLI
RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli

# GCloud CLI
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y
```

Time to test it again:

```
docker build -t my_tools .
docker run --rm -it my_tools bash
```

When we get the container's prompt, run `gcloud --version` and the results should look like:

```
root@93d257522d99:/# gcloud --version
Google Cloud SDK 435.0.1
alpha 2023.06.14
beta 2023.06.14
bq 2.0.93
bundled-python3-unix 3.9.16
core 2023.06.14
gcloud-crc32c 1.0.0
gsutil 5.24
```

It works, so exit from the container.