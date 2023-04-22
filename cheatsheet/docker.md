# Docker

## Running a Toolbox Image

* `docker run --rm -ti --name toolbox ubuntu:latest bash` (single run)

## Image Management

List locally present images
* `docker image ls`

Remove images not tagged and not in use by a container
* `docker image prune`

Downloads the latest image, possibly overwriting an outdated local image when necessary.
* `docker pull hashicorp/terraform:light`

## Container Management

Remove all stopped containers
* `docker container prune`

List existing containers (running and paused)
* `docker ps -a`

Print logs that the container produced. Works also for a stopped container.
* `docker logs name`

Like tail -f for a container
* `docker logs -f name`

Shows process list inside the container
* `docker top name`

Show containers as process list with CPU,Mem,Net usage.
* `docker stats [container]`

## Running, Interacting

Run a “tool container” (executing ENTRYPOINT or CMD), attaches to the running command and, after stopping, the container deleted right away.
* `docker run --rm -t -i tastybug/someclient`

In an already running container, execute an interactive task (/bin/bash)
* `docker exec -ti ubuntu /bin/bash`

In an already running container, execute some_command without attaching to it
* `docker exec -d ubuntu some_command`

## docker-compose

Runs, stops docker-compose.yml
* `docker-compose up|stop|kill`

Removes containers previously created with UP, like “docker rm NAME”
* `docker-compose rm`

Show ongoing sessions
* `docker-compose ps`

Dump or tail logs of an ongoing session
* `docker-compose logs [-f]`
