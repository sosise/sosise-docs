# Docker
Sosise supports docker out of the box. You can build your docker image with `build-docker-image.sh` script which can be found in the root directory

```sh
./build-docker-image.sh
```

> Just run it and follow the instructions

## The docker Directory
The `docker` directory contains all necessary files which are needed for building a docker image. Feel free to change anything you need for your project.

## Docker-compose
There is also a `docker-compose.yml` file.

```sh
docker-compose up
```

> Please make sure that you have `.env`, `cron` files, which are necessary to run the container.

## How to use cron inside your docker container
In the project root is an example cron file, which contains information on how to run jobs in your container. To make it work just run the project by using `docker-compose` or mount the `cron` file to `/etc/crontabs/root` inside the container.

> When you edit the cron file, you have to restart the container, or just restart cron inside your container with supervisord.