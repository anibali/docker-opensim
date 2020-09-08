## OpenSim Docker image

Ubuntu + [OpenSim](https://simtk.org/projects/opensim)


### Requirements

In order to use this image you must have Docker Engine installed. Instructions
for setting up Docker Engine are
[available on the Docker website](https://docs.docker.com/engine/installation/).


### Usage

Firstly you will need to give the appropriate display server access permissions
using `sudo xhost +local:root` (these permissions can be revoked later using
`sudo xhost -local:root`). Then run the OpenSim GUI using the following command:

```bash
$ mkdir -p opensim_home_volume
$ docker run --rm \
  --user="$(id -u):$(id -g)" \
  --volume="$PWD/opensim_home_volume:/home/user:rw" \
  -e "DISPLAY" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
  docker.pkg.github.com/anibali/docker-opensim/opensim
```
