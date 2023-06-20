## Task 6 - Add the Azure CLI

Now to repeat the process for the Azure CLI:

From the [Azure documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt), we can see that the commands that we need to use are:

```
sudo apt-get update
sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt-get update
sudo apt-get install azure-cli
```

For the sake of space and consistency, let's use the short form of `apt` versus `apt-get` and add the `-y` switch to all of the install commands to make them non-interactive:

```
sudo apt update
sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list
sudo apt update
sudo apt install -y azure-cli
```

Let's append that to our Dockerfile as a RUN command, separating each of the above commands using `&&`:

```
echo "# Azure CLI" >> Dockerfile

echo 'RUN sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null && AZ_REPO=$(lsb_release -cs) && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list && sudo apt update && sudo apt install -y azure-cli' >> Dockerfile

echo " " >> Dockerfile

```

Now, test our latest Dockerfile:

```
docker build -t my_tools .
docker run --rm  -it my_tools bash
```

Once we get the prompt from the container, we can run `az --version` and the output will look something like:

```
ubuntu@ubuntu:~$ docker run --rm -it my_tools bash
root@963a51baed25:/# az --version
azure-cli                         2.49.0

core                              2.49.0
telemetry                          1.0.8

Dependencies:
msal                              1.20.0
azure-mgmt-resource               22.0.0

Python location '/opt/az/bin/python3'
Extensions directory '/root/.azure/cliextensions'

Python (Linux) 3.10.10 (main, May 19 2023, 08:20:31) [GCC 11.3.0]

Legal docs and information: aka.ms/AzureCliLegal


Your CLI is up-to-date.
```

Very good. Exit from the container.
