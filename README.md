# Docker host
A Vagrant-based virtual machine to run a Docker CE host.

## Installation

### Prerequisites
To install this virtual machine you require the following tooling on your host machine:
* Git
* [Vagrant](https://www.vagrantup.com)
* [Virtualbox](https://www.virtualbox.org)
* [Docker CE](https://docs.docker.com/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* On Windows: [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) with [Vagrant installed within WSL](https://www.vagrantup.com/docs/other/wsl.html).

### 1. Get the source
```bash
git clone git@github.com:nijens/docker-host.git
```

### 2. Configure the virtual machine as Docker host
```bash
export DOCKER_HOST=tcp://192.168.33.100:2375
```

### 3. Build and start the virtual machine
```bash
vagrant up
```

## Configuration
`config.yaml`

### Virtual machine configuration

| Configuration key   | Default value  | Description                                                                                                                                                         |
|---------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cpus                | 2              | The number of CPUs attached to the virtual machine.                                                                                                                 |
| memory              | 3072           | The amount of RAM (in Megabytes) attached to the virtual machine.                                                                                                   |
| ip                  | 192.168.33.100 | The IP address of the virtual machine. Please note that this is a private network not reachable from the network.                                                   |
| docker_storage_size | 50             | The size (in Gigabytes) of the additional virtual disk added to the virtual machine. This virtual disk is used to store all the container images. (/var/lib/docker) |

```yaml
# ./config.yaml
cpus: 2
memory: 3072
ip: 192.168.33.100
docker_storage_size: 50
```

```bash
vagrant reload
```

### Mounting volumes

```yaml
# ./config.yaml
volumes:
    /mnt/c/projects/my-dockerized-application:
```

```yaml
# ./config.yaml
volumes:
    /mnt/c/projects/my-dockerized-application:
        type: rsync
```
