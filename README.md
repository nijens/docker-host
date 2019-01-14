# Docker host
A Vagrant-based virtual machine to run a Docker CE host.

## Installation

### Prerequisites
To install this virtual machine you require the following tooling on your host machine:
* Git
* [Vagrant](https://www.vagrantup.com/intro/getting-started/install.html)
* [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
* [Docker CE](https://docs.docker.com/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* On Windows: [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) with [Vagrant installed within WSL](https://www.vagrantup.com/docs/other/wsl.html).

### 1. Get the source
Clone this repository to get the configuration of the virtual machine:
```bash
git clone git@github.com:nijens/docker-host.git
```

### 2. Reference the virtual machine as Docker host
To configure the virtual machine as the default Docker host export the following environment variable:
```bash
export DOCKER_HOST=tcp://192.168.33.100:2375
```

Both the Docker CLI and Docker Compose binaries will listen look at this environment variable. It's recommended to add
the above line to the `.bashrc` file of your user.

### 3. Build and start the virtual machine
The following command will initialize and start the virtual machine:
```bash
vagrant up
```

## Configuration
To customize the virtual machine you're able to create `config.yaml` file next to the `Vagrantfile`.

### Virtual machine configuration
The following configuration options are available to customize the virtual machine:
```yaml
# ./config.yaml
cpus:                2              # The number of CPUs attached to the virtual machine.
memory:              3072           # The amount of RAM (in Megabytes) attached to the virtual machine.
ip:                  192.168.33.100 # The IP address of the virtual machine. Please note that this is a private network
                                    # not reachable from the network.
docker_storage_size: 50             # The size (in Gigabytes) of the additional virtual disk added to the virtual machine.
                                    # This virtual disk is used to store all the container images. (/var/lib/docker)
```

To apply the configuration changes, except for the `docker_storage_size`, run the following command:
```bash
vagrant reload
```

Updating the `docker_storage_size` requires you to rebuild the virtual machine. This will destroy all the built
images and persistent volumes you have created. Although this shouldn't be an issue on a development environment,
it's something you should be aware of when applying changes to this configuration option.

To rebuild the virtual machine after a change to `docker_storage_size`, execute the following commands:
```bash
vagrant destroy
vagrant up
```

### Mounting application/project volumes
When developing applications with Docker you need to have your application source accessible for the Docker host.
To allow access you're required to configure the path of your application source for the virtual machine.

The following example shows how to configure a directory as volume for the virtual machine and Docker CE without
any additional options.
```yaml
# ./config.yaml
volumes:
    /mnt/c/projects/my-dockerized-application:
```

To apply the configuration changes, run the following command:
```bash
vagrant reload
```

The configuration of volumes also allows you to change the mount types and mount specific configuration options Vagrant
already offers:
```yaml
# ./config.yaml
volumes:
    /mnt/c/projects/my-dockerized-application:
        type: rsync
```

For more information about the different mount configuration options, see the
[Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) chapter within
the Vagrant documentation.
