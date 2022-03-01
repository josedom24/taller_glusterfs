# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  config.vm.define :nodo1 do |nodo1|
      nodo1.vm.box = "debian/bullseye64"
      nodo1.vm.hostname = "nodo1"
      nodo1.vm.synced_folder ".", "/vagrant", disabled: true
      nodo1.vm.network :private_network, ip: "10.1.1.101"
      nodo1.vm.provider :libvirt do |libvirt|
        libvirt.storage :file, :size => '2048M'
        libvirt.memory = 1024
      end
    end
    config.vm.define :nodo2 do |nodo2|
      nodo2.vm.box = "debian/bullseye64"
      nodo2.vm.hostname = "nodo2"
      nodo2.vm.synced_folder ".", "/vagrant", disabled: true
      nodo2.vm.network :private_network, ip: "10.1.1.102"
      nodo2.vm.provider :libvirt do |libvirt|
        libvirt.storage :file, :size => '2048M'
        libvirt.memory = 1024
      end
    end
    config.vm.define :nodo3 do |nodo3|
      nodo3.vm.box = "debian/bullseye64"
      nodo3.vm.hostname = "nodo3"
      nodo3.vm.synced_folder ".", "/vagrant", disabled: true
      nodo3.vm.network :private_network, ip: "10.1.1.103"
      nodo3.vm.provider :libvirt do |libvirt|
        libvirt.storage :file, :size => '2048M'
        libvirt.memory = 1024
      end
    end
    config.vm.define :cliente do |cliente|
      cliente.vm.box = "debian/bullseye64"
      cliente.vm.hostname = "cliente"
      cliente.vm.synced_folder ".", "/vagrant", disabled: true
      cliente.vm.network :private_network, ip: "10.1.1.104"
    end
  end
  
