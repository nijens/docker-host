# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

# Default configuration.
configuration = {
    "cpus" => 2,
    "memory" => 3072,
    "ip" => "192.168.33.100",
    "docker_storage_size" => 20, # Size in GB
    "volumes" => {},
	"ports" => []
}

# Merge default configuration with configuration from config.yaml
configuration_file = "./config.yaml"
if File.exists?(configuration_file)
    configuration = configuration.merge(YAML.load_file configuration_file)
end

# Location of the Docker storage virtual disk.
docker_storage_virtual_disk_file = File.realpath(".").to_s+"/docker_storage.vmdk"
virtualbox_docker_storage_virtual_disk_file = docker_storage_virtual_disk_file
if virtualbox_docker_storage_virtual_disk_file[0..6] == "/mnt/c/" # TODO: Should only be used when within WSL. Also shouldn't be limited to C drive.
    virtualbox_docker_storage_virtual_disk_file = docker_storage_virtual_disk_file[5..5]
    virtualbox_docker_storage_virtual_disk_file += ":"
    virtualbox_docker_storage_virtual_disk_file += docker_storage_virtual_disk_file[
        6,
        docker_storage_virtual_disk_file.length
    ]
end

Vagrant.configure("2") do |config|
    # Load Debian 9 Vagrant box.
    config.vm.box = "debian/stretch64"

    # Require Vagrant plugins.
    config.vagrant.plugins = ["vagrant-vbguest"]

    # Configure VM inside a private network.
    config.vm.network "private_network", ip: configuration["ip"]

    # Disable default Vagrant share as it's not being used.
    config.vm.synced_folder ".", "/vagrant", disabled: true

    # Configure mounts from config.yaml.
    configuration["volumes"].each do |volume, volumeOptions|
        volumeOptions ||= {}
        if volumeOptions
            volumeOptions = volumeOptions.inject({}){|memo,(k,v)| memo[k.to_sym] = v; memo}
        end

        host_path = volume
        guest_path = volume
        if volumeOptions.has_key?(:mount_as)
            guest_path = volumeOptions.delete(:mount_as)
        end

        config.vm.synced_folder(host_path, guest_path, volumeOptions)
    end
	
    # Configure ports from config.yaml
    configuration["ports"].each do |port|
        if port.is_a? Integer
            config.vm.network "forwarded_port", guest: port, host: port
        else
            config.vm.network "forwarded_port", guest: port["guest"], host: port["host"]
        end
    end

    # Configure VM hardware.
    config.vm.provider "virtualbox" do |vb|
        vb.cpus = configuration["cpus"]
        vb.memory = configuration["memory"]

        if !File.exist?(docker_storage_virtual_disk_file)
            vb.customize [
                'createhd',
                '--filename', virtualbox_docker_storage_virtual_disk_file,
                '--format', 'VMDK',
                '--size', configuration["docker_storage_size"] * 1024
            ]

            vb.customize [
                'storageattach', :id,
                '--storagectl', 'SATA Controller',
                '--port', 1,
                '--device', 0,
                '--type', 'hdd',
                '--medium', virtualbox_docker_storage_virtual_disk_file
            ]
        end
    end

    # Create filesystem partition on Docker storage virtual disk.
    config.vm.provision "shell", inline: <<-SHELL
        echo 'type=83' | sfdisk /dev/sdb
        mkfs.ext4 /dev/sdb1
        eval `blkid /dev/sdb1 -o export`
        echo "" >> /etc/fstab
        echo "UUID=$(blkid -o value -s UUID /dev/sdb1) /var/lib/docker ext4    defaults          0       0" >> /etc/fstab
        mkdir -p /var/lib/docker
        mount /var/lib/docker
    SHELL

    # Install and configure the Docker Container Engine.
    config.vm.provision "shell", inline: <<-SHELL
        # Install utilities to ease the installation process.
        apt-get update
        apt-get install -y \
             apt-transport-https \
             ca-certificates \
             curl \
             gnupg2 \
             software-properties-common

        # Install Docker Container Engine.
        curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

        add-apt-repository \
            "deb [arch=amd64] https://download.docker.com/linux/debian \
            $(lsb_release -cs) \
            stable"

        apt-get update
        apt-get install -y docker-ce

        # Add Docker daemon configuration in JSON format.
        echo '{' > /etc/docker/daemon.json
        echo '    "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"]' >> /etc/docker/daemon.json
        echo '}' >> /etc/docker/daemon.json

        # Remove options from Systemd service configuration.
        mkdir -p /etc/systemd/system/docker.service.d
        echo '[Service]' > /etc/systemd/system/docker.service.d/override.conf
        echo 'ExecStart=' >> /etc/systemd/system/docker.service.d/override.conf
        echo 'ExecStart=/usr/bin/dockerd' >> /etc/systemd/system/docker.service.d/override.conf

        systemctl daemon-reload
        systemctl restart docker.service
    SHELL
end
