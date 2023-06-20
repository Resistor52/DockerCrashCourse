## Task 11 - Pull the Image from Docker Hub

Now that we have stored the image in Docker Hub, we can pull it down whenever we need it, to any system that has Docker installed. And since it is a public repository, we don't even need to be logged in.

Run `docker logout` to prove this.

Next, let's blow away all images and containers on our VM. Type `docker system prune --all` and then hit "y" when prompted.

This will output something similar to the following:

```
ubuntu@ubuntu:~$ docker system prune  --all
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
Deleted Containers:
9e0fe56a4cc1cd4c3a4538bc6bc1ef641a4c06053bab7d8a466e965f04ce8901
f89cceb2743d0770acae4b1f3139890064ce37e1b1d546814f165c69ce1f3868
e40e715751ca037508188767294a6f609a8792e239068a49f4691fc3eaa04a0b

Deleted Images:
deleted: sha256:2b96b00ce668bf8c55bd8ff40a3224d7ab1645fe73272f374783b13f5f83c6d5
deleted: sha256:e07df04bf2cb41754b70158d34afff88b0ae675bcbde1417096e3fb4144f7eac
untagged: khman52/my_tools:latest
untagged: khman52/my_tools@sha256:d6ac6ed27434fe37822665381fe80705cd44ed198db2ab74179c60641ac6d068
untagged: my_tools:latest
deleted: sha256:506981686a337743bf0fba6a1a30700ee7f25de08132078755718d53eb6ea542
untagged: ubuntu:latest
untagged: ubuntu@sha256:ac58ff7fe25edc58bdf0067ca99df00014dbd032e2246d30a722fa348fd799a5
deleted: sha256:1f6ddc1b2547b2e38dc25b265ac585238a3c23da63976722864dab2a069c74f4
untagged: sans:latest
deleted: sha256:903e689852f4f3eb6a6b095a74fac3abb805013875c1ed32ba196b38a533ae5e
deleted: sha256:fec2910474cee5450b514d515a663a6b0662f63bbbefc3168a360881adbd63fa

Deleted build cache objects:
dg0yuw48s5iqiwgzoeqz1zto3
l0n5ui07wuxt61a3hjrf6d02s
yaj7otsyxmmatmont7cbjuami
mtaktu2ta7b2y8vfqddye20vr
7peuk2ej6p7upbzx68r9h5aat
ygfkfmx613xzqvf5pzgga3inc
fbtzfypxptmercfoy8hyapffa
ltleifghm9jcj6et5j1c6a7wd
zpt1wcpjbcnjkrmkb9blod3dh
mkjfpd4bz2hs924pfjvrwlsw8
34fxmav9dq80z2n1gzhs11fiv
3tgbcp2ej6dqpscl97qgj57ex
lmvreyv51za6bizxkp2w0d1f4
mmlyoxwailn56b9lmauqyhpmo
x8s8c8acpo4l8y8y91ifrm1ty
pdcfkimifgc33kihykb8i1jhd
nwz50xalezfb2h1t556jqfykp
injy0vtu7ubp03jrojlxq9iv3
9xnsr4br4sm82r58q086hkweg
v318wmm8cutx3iwslxr7riic9

Total reclaimed space: 4.29GB

```

We can verify that we do not have any images on the local system:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

Ok, lets pull down our image:

```
docker pull $DHUSER/my_tools:latest
```

Now when we run `docker images` we can verify that we have a local copy:

```
ubuntu@ubuntu:~$ docker images
REPOSITORY              TAG       IMAGE ID       CREATED             SIZE
resistor52/my_tools   latest    b677d5555ad4   About an hour ago   2.04GB
```

And we can run it:

```
docker run --rm -it $DHUSER/my_tools
```

And then confirm our tools are on the container:

```
which aws az gcloud
```

And you should see:

```
sans@5aae26738dd4:~$ which aws az gcloud
/usr/local/bin/aws
/usr/bin/az
/usr/bin/gcloud
```

Exit the container.