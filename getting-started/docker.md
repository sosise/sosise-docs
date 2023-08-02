# Docker Integration
Sosise seamlessly supports Docker. To build your Docker image, use the `build-docker-image.sh` script located in the root directory. 

```sh
./build-docker-image.sh
```

> Note: Simply execute the script and follow the prompted instructions.

## Docker Directory
The `docker` directory encompasses all the requisite files for building a Docker image. Feel free to modify these to suit the unique requirements of your project.

## Docker-compose
A `docker-compose.yml` file is also included for your convenience.

```sh
docker-compose up
```

> Important: Ensure the presence of the `.env` and `cron` files as they are critical for running the container.

## Utilizing Cron Inside Docker Container
An example cron file, outlining how to execute jobs within your container, can be found in the project's root. To activate it, either run the project using `docker-compose` or mount the `cron` file to `/etc/crontabs/root` within the container.

> Caution: Any modifications to the cron file necessitate a container restart, or you can opt to simply restart cron within your container using supervisord.