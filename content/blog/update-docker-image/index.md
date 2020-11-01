---
title: Update Your Docker Image
date: 2020-11-01T22:15
---

If you have your own docker image/s set up, here is how you can push new builds.

Make the necessary changes in your Dockerfile then change to the command line and navigate into the folder where our Dockerfile resides. First we build our image.

```shell
docker build -t imagename .
```

Next we tag it, so it is easy to reference at a later point.

```shell
docker tag imagename rafhun/php:tag
```

Then push the image to the Docker repository. If you do not yet have a Docker ID go ahead and create one [here](https://hub.docker.com/signup). Then make sure you are logged into Docker Hub.

```shell
docker login
```

Now we are ready to push the image to Docker Hub.

```shell
docker push rafhun/php:tag
```

**Hint:** Make sure to also add the `latest` tag to the version you are pushing and push it to Docker Hub as well.

After pushing a new tag to the repository make sure to build images when you are restarting containers by running the following command to start them up.

```shell
docker-composer up -d --build
```

**Hint:** If you are using a specific tag (other than `latest`) adjust it before you rebuild the image.
