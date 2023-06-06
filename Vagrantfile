# -*- mode: ruby -*-
$script = <<-SCRIPT
cat >> /etc/hosts <<EOF
192.168.56.101 master
192.168.56.102 node01
192.168.56.103 node02
EOF

## stop swap
sudo swapoff -a
sudo sed -i '/swap.img/ s/^\(.*\)$/#\1/g' /etc/fstab

## enable netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

## enable ip bridge
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

# remove old
sudo apt-get remove docker docker-engine docker.io containerd runc

# upgrade
sudo apt-get update
sudo apt-get -y upgrade

# install basic
sudo apt-get -y install ca-certificates curl gnupg lsb-release

# add GPG key
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# install docker
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable docker.service
sudo systemctl restart docker.service
  
# add k8s repository
sudo apt install curl apt-transport-https -y
curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# install k8s
sudo apt-get update
sudo apt install -y kubelet=1.26.5-00 kubeadm=1.26.5-00 kubectl=1.26.5-00
# disable hold k8s versionva
# sudo apt-mark hold kubelet kubeadm kubectl

sudo mv /etc/containerd/config.toml /etc/containerd/config.toml.bak
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl enable containerd
sudo systemctl daemon-reload && sudo systemctl restart containerd

SCRIPT


Vagrant.configure("2") do |config|


   config.vm.define "master" do |master|
	master.vm.box = "ubuntu-20.04LTS"
	master.vm.box_check_update = false
	master.vbguest.auto_update = false
	master.vm.provider "virtualbox" do |vb|
		vb.gui = false
		vb.name = "master"
		vb.memory = 2048
		vb.cpus = 2
	end

	master.vm.network "private_network", ip: "192.168.56.101"
	master.vm.provision "shell", inline: $script
  end
  
  config.vm.define "node01" do |node01|
	node01.vm.box = "ubuntu-20.04LTS"
	node01.vm.box_check_update = false
	node01.vbguest.auto_update = false
	node01.vm.provider "virtualbox" do |vb|
		vb.gui = false
		vb.name = "node01"
		vb.memory = "2048"
		vb.cpus = 2
	end
	
	node01.vm.network "private_network", ip: "192.168.56.102"
	node01.vm.provision "shell", inline: $script
  end
  
  config.vm.define "node02" do |node02|
	node02.vm.box = "ubuntu-20.04LTS"
	node02.vm.box_check_update = false
	node02.vbguest.auto_update = false
	node02.vm.provider "virtualbox" do |vb|
		vb.gui = false
		vb.name = "node02"
		vb.memory = "2048"
		vb.cpus = 2
	end
	
	node02.vm.network "private_network", ip: "192.168.56.103"
	node02.vm.provision "shell", inline: $script
  end

end
