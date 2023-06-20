## Task 9 - Introducing Docker Hub

Navigate over to [hub.docker.com](https://hub.docker.com/) and sign up, assuming that you do not have an account yet. Note that the second half of what we will be covering during today's episode assumes that you have a paid subscription  (Pro, Team or Business). The Pro subscription is $5 per user per month and that is what I have, but I paid the annual fee which is $60/year.

Now that you have a Docker Hub account, create a repository by clicking the blue button at the top right side of the web page. Name your new repository `my_tools` and set the visibility to "public." You can leave the description blank for now and leave the Build Settings alone for now. We will be setting that up in a little bit.

Once the repository has been created in Docker Hub, we can see a black box titled "Docker commands." This box contains the docker command that we will need to use to push our image up to Docker Hub.

Next, set your Docker Hub Username to an environment variable named `DHUSER`. For example my Docker Hub User Name is `resistor52` so I would use the following command:

```
export DHUSER=resistor52
```
