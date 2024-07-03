# -*- mode: ruby -*-
# vi: set ft=ruby :
NUM_WORKER_NODES    = 3
IP_NW               = "192.168.56."
IP_START            = 10
CPUS_MASTER_NODE    = 4
MEMORY_MASTER_NODE  = 2048
CPUS_WORKER_NODE    = 2
MEMORY_WORKER_NODE  = 1024
VAGRANT_BOX_IMAGE   = "ubuntu/focal64"  # Ubuntu 20.04 LTS

Vagrant.configure("2") do |config|
  config.vm.provision "shell", env: {"IP_NW" => IP_NW, "IP_START" => IP_START}, inline: <<-SHELL
      apt-get update -y
      echo "$IP_NW$((IP_START))  master-node  master-node.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+1))  worker-node01  worker-node01.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+2))  worker-node02  worker-node02.hellocloud.io" >> /etc/hosts
      echo "$IP_NW$((IP_START+3))  worker-node03  worker-node03.hellocloud.io" >> /etc/hosts
  SHELL

  config.ssh.insert_key = false
  config.ssh.private_key_path = ["~/.ssh/id_rsa", "~/.vagrant.d/insecure_private_key"]
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"

  config.vm.define "master" do |master|
    master.vm.box = VAGRANT_BOX_IMAGE
    master.vm.hostname = "master-node.hellocloud.io"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "master-node"
      vb.cpus = CPUS_MASTER_NODE
      vb.memory = MEMORY_MASTER_NODE
      vb.gui = false
    end
    master.vm.provision "shell", run: "always", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y net-tools zip curl jq tree unzip wget siege apt-transport-https ca-certificates software-properties-common gnupg lsb-release bash-completion
      netstat -tunlp
      echo "Hello from Master-node"
    SHELL
  end

  (1..NUM_WORKER_NODES).each do |i|
    config.vm.define "node0#{i}" do |node|
      node.vm.hostname = "worker-node0#{i}.hellocloud.io"
      node.vm.box = VAGRANT_BOX_IMAGE
      node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "worker-node0#{i}"
        vb.cpus = CPUS_WORKER_NODE
        vb.memory = MEMORY_WORKER_NODE
        vb.gui = false
      end
      node.vm.provision "shell", run: "always", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y net-tools zip curl jq tree unzip wget siege apt-transport-https ca-certificates software-properties-common gnupg lsb-release bash-completion
        netstat -tunlp
        echo "Hello from Worker-node#{i}"
      SHELL
    end
  end
end