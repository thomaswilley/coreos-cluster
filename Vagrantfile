# -*- mode: ruby -*-
# vi: set ft=ruby :

# note: this is the Vagrantfile that came with the coreos download, credit all goes to coreos for this one!
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  (1..3).each do |i|
    config.vm.define vm_name = "core-%02d" % i do |config|
      config.vm.box = "coreos"
      config.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant.box"

      config.vm.hostname = vm_name

      config.vm.network :private_network, ip: "192.168.65.#{i+1}"
      config.vm.provision "ansible" do |ansible|
        ansible.playbook = "provision/playbook.yml"
        ansible.inventory_path = "provision/hosts"
      end
    end
  end
end
