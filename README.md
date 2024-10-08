# k8s-vm-calico-localreg-ecr-setup
Install k8s on VM (Calico CNI) with Local Registry run on different VM and Pull images from AWS ECR

##  Need to Ready 3 VM running ( one Master node, one Worker node and one for running Registry), Will need AWS Account for testing connect with ECR

- Vagrantfile sample need to create ssh-key first before “vagrant up”
```bash
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
```

## Command need to run on master node

```
$ sh command.sh
$ sh master.sh

###Install CNI Calico###
$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml

###Modify the pod CIDR firt###
$ kubectl apply -f custom-resources.yaml
```

## 
Command need to run on client nodes

```
$ sh command.sh
$ sh node.sh (No Need)

###Join the K8s cluster###(Generated from master node)
$ kubeadm join 192.168.56.10:6443 --token z05q57.1pnh06cqvyp0ndd9 \
        --discovery-token-ca-cert-hash sha256:fd2480726d04b19aaaf72eac24c5c4ced08909943aa2cc49e727ac153c1fcc3f 
```

## After Successfully Joining Worker node run on master node

```
$ kubectl get nodes
$ kubectl label node worker-node01 node-role.kubernetes.io/worker=worker-new
```

## All k8s running resource output after calico installed

