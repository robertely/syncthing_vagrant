# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "Syncthing" do |ubuntu|
    ubuntu.vm.hostname = "Syncthing"
    ubuntu.vm.box = "ubuntu/xenial64"

    config.vm.network "public_network",  use_dhcp_assigned_default_route: true, bridge: "enp5s0"
    config.vm.network "private_network", ip: "192.168.5.100"
 
    #For if you don't want to use a public network. 
    # config.vm.network "forwarded_port",  guest: 8443,  host:  8443, protocol: "tcp" # Web
    # config.vm.network "forwarded_port",  guest: 22000, host: 22000, protocol: "tcp" # Syncthing
    # config.vm.network "forwarded_port",  guest: 21027, host: 21027, protocol: "udp" # Syncthing
    # config.vm.network "forwarded_port",  guest: 1900,  host:  1900, protocol: "udp" # upnp, You won't want this. It can only work for one VM
 
    config.vm.synced_folder "syncthing.config/", "/home/ubuntu/.config/syncthing/", create: true
    config.vm.synced_folder "/apollo-prime/",    "/apollo-prime", type: "nfs", create: true

    ubuntu.vm.provider "virtualbox" do |vb|
      vb.memory = "256"
      vb.cpus = "2"
      vb.customize ["modifyvm", :id, "--memory", "2048"] 
      vb.customize ["modifyvm", :id, "--cpus", "2"] 
      #upnp magic
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"] 
      vb.customize ["modifyvm", :id, "--nictype2", "Am79C973"] 
    end

    ubuntu.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
    config.vm.provision "file", source: "syncthing.service", destination: "/tmp/syncthing.service"
    ubuntu.vm.provision "shell", inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      echo ================ Loading Syncthing Repo ================
      curl -s https://syncthing.net/release-key.txt | sudo apt-key add -
      echo "deb http://apt.syncthing.net/ syncthing release" | sudo tee /etc/apt/sources.list.d/syncthing.list
      echo ================= Installing packages ==================
      sudo apt-get -qq update
      sudo apt-get install -y nfs-common vim-tiny syncthing
      sudo apt-get clean
      echo =================== Starting Service ====================
      sudo mv /tmp/syncthing.service /etc/systemd/system/syncthing.service
      sudo service syncthing start
      echo WEB: https://$(/sbin/ifconfig enp0s8 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}'):8443
    SHELL
  end
end
