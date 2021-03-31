# LXD Lightning Talk

>>>
:warning: Before you can run the config examples, you must run this setup:

```sh
export _GID=$(id -u)
export _UID=$(id -g)
export SSH_KEY=$(cat ~/.ssh/id_ed25519.pub)

envsubst < configs/cloud-init.tpl.yml > configs/cloud-init.yml
envsubst < configs/gui.tpl.yml > configs/gui.yml
```
>>>

Some nice resources:

- https://ubuntu.com/blog/feeling-at-home-in-a-lxd-container
- https://lxd.readthedocs.io/en/latest/

### Basic Image and Container handling

One can use `docker` images for application deployment (mostly one process).
And `lxd` for opersting system deployment.

```sh
podman search debian

podman run -dt debian:buster /bin/sh

podman ps

podman container top
```

```sh
lxc image ls images:debian

lxc lauch images:debian/buster c1

lxc info c1

lxc config show c1

lxc exec c1 bash

pstree 0
```

### Configuring via Cloud Init

To configure containers one can use [cloud-init](https://cloudinit.readthedocs.io/en/latest/)

```sh
lxc launch ubuntu:20.04 c1 < configs/cloud-init.yml

lxc exec c2 -- su --login $USER

# get container IP
lxc exec c2 -- ip -f inet addr show eth0 | awk '/inet / {print $2}'

ssh CONTAINER_IP
```

### General Container Configuration

- https://lxd.readthedocs.io/en/latest/instances/

One can do a lot of things inside an lxd container, here are a few examples:

```sh
lxc config device add c2 share disk path=/home/$USER/demo source=$PWD/demo

lxc config device add c2 awesome-page proxy listen=tcp:0.0.0.0:8000 connect=tcp:0.0.0.0:8000

lxc exec c2 -- su --login $USER

cd demo && python3 -m http.server
```

Have you ever wondered wether a `t2.micro` instance is enough for your application?
A full list of supported types can be found here: https://github.com/dustinkirkland/instance-type

```sh
lxc launch --type t2.micro ubuntu:20.04 micro
lxc config show micro
```

This can even be live updated!

```sh
lxc exec micro -- watch -n 0.5 nproc

# in another terminal
lxc config edit micro
```

This resource limitation enables to simulate the whole internet:

 - https://github.com/nsec/the-internet
 - https://stgraber.org/2015/12/

### Run Graphical Applications Inside an LXD Container

One can run graphical applications inside a container by mounting `/tmp.X11-unix/X0`
with the correct id mapping.

```sh
lxc launch ubuntu:20.04 < configs/gui.yml c3

lxc exec c3 -- bash

sudo apt update && sudo apt install --yes x11-apps && xeyes
```

### Run nested Containers

With lxd it is very simple to run an lxd container inside another lxd container.

```sh
lxc launch ubuntu:20.04 nested -c security.nesting=true

lxc exec nested -- bash

# inside the container
lxd init --auto
lxc launch images:debian/buster c1

lxc exec c1 -- bash
```

There is also a guide on how to run docker containers inside an lxd container:
https://lxd.readthedocs.io/en/latest/#how-can-i-run-docker-inside-a-lxd-container

### Creating Snapshots

```sh
lxc snapshot c1 snap0

# view snapshots
lxc info c1

lxc exec c1 -- touch after-snap0
lxc exec c1 -- ls

lxc restore c1 snap0
lxc exec c1 -- ls
```

### Run virtual Machines

One can very easily run VM's with the `--vm` flag

```sh
lxc launch ubuntu:20.04 v1 --vm
lxc console v1
lxc exec v1 -- bash
```

### Use Containers in the GitLab Pipeline

It is very simple to use lxd containers in a gitlab pipeline,
you need to setup a custom runner for this: https://docs.gitlab.com/runner/executors/custom_examples/lxd.html


```yaml
run-lxd-container:
  tags: [lxd]
  image: ubuntu:20.04
  script:
    - echo "Hello lxd container!"
```
