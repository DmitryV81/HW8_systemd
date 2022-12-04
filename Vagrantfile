# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  #config.vm.box = "generic/centos8s"
  #config.vm.box_version = "v4.2.4"
  config.vm.provider "virtualbox" do |v|
    v.memory = 512
    v.cpus = 1
  end
  config.vm.define "systemd" do |nfss|
    nfss.vm.network "private_network", ip: "192.168.50.10",
    virtualbox__intnet: "net1"
    nfss.vm.hostname = "systemd"
    #nfss.vm.provision "shell", path: "rpm.sh"
  end
end
