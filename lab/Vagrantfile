# -*- mode: ruby -*-
# vi: set ft=ruby  :

machines = {
  "master"   => {"memory" => "2048", "cpu" => "2", "ip" => "100", "image" => "ubuntu/bionic64"},
  "node01"   => {"memory" => "1024", "cpu" => "2", "ip" => "110", "image" => "ubuntu/bionic64"},
  "node02"   => {"memory" => "1024", "cpu" => "2", "ip" => "120", "image" => "centos/7"},
  "registry" => {"memory" => "2048", "cpu" => "2", "ip" => "200", "image" => "ubuntu/bionic64"}
}

Vagrant.configure("2") do |config|

  machines.each do |name, conf|
    config.vm.define "#{name}" do |machine|
      machine.vm.box = "#{conf["image"]}"
      machine.vm.hostname = "#{name}.docker-dca.example"
      machine.vm.network "private_network", ip: "10.20.20.#{conf["ip"]}"
      machine.vm.provider "virtualbox" do |vb|
        vb.name = "#{name}"
        vb.memory = conf["memory"]
        vb.cpus = conf["cpu"]
        vb.customize ["modifyvm", :id, "--groups", "/Docker-DCA"]
      end
      machine.vm.provision "shell", path: "provision.sh"
      machine.vm.provision "shell", inline: "hostnamectl set-hostname #{name}.docker-dca.example"
    end
  end
end
