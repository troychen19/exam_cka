# -*- mode: ruby -*-
$script = <<-SCRIPT

# remove old
sudo apt-get remove docker docker-engine docker.io containerd runc

# upgrade
sudo apt-get update
sudo apt-get -y upgrade

# set host
cat >> /etc/hosts <<EOF
192.168.153.101 master
192.168.153.102 node01
192.168.153.103 node02
EOF

# disable 127.0.2.1 host
sudo sed -i '/127.0.2.1/d' /etc/hosts

# stop swap
sed -i '/swap/d' /etc/fstab
sudo swapoff -a

# disabel ufw
systemctl disable --now ufw >/dev/null 2>&1

# enable netfilter
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


# install basic
sudo apt-get -y install ca-certificates curl gnupg lsb-release

# add GPG key
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# install containerd
sudo apt-get update
sudo apt-get -y install  containerd.io 
#docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin
#sudo systemctl enable docker.service
#sudo systemctl restart docker.service

# config crictl
cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
EOF

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
	master.vm.box = "bento/ubuntu-20.04"
	master.vm.host_name="master"
	master.vm.box_check_update = false
	master.vbguest.auto_update = false
	master.vm.provider "vmware_desktop" do |vb|
		vb.gui = false
		vb.vmx["displayName"]="master"
		vb.vmx["numvcpus"]="2"
		vb.vmx["memsize"]="2048"
	end

	master.vm.network :private_network, ip: "192.168.153.101"
	master.vm.provision "shell", inline: $script
  end
  
  config.vm.define "node01" do |node01|
	node01.vm.box = "bento/ubuntu-20.04"
	node01.vm.host_name="node01"
	node01.vm.box_check_update = false
	node01.vbguest.auto_update = false
	node01.vm.provider "vmware_desktop" do |vb|
		vb.gui = false
		vb.vmx["displayName"]="node01"
		vb.vmx["numvcpus"]="2"
		vb.vmx["memsize"]="2048"
	end
	
	node01.vm.network :private_network, ip: "192.168.153.102"
	node01.vm.provision "shell", inline: $script
  end
  
  config.vm.define "node02" do |node02|
	node02.vm.box = "bento/ubuntu-20.04"
	node02.vm.host_name="node02"
	node02.vm.box_check_update = false
	node02.vbguest.auto_update = false
	node02.vm.provider "vmware_desktop" do |vb|
		vb.gui = false
		vb.vmx["displayName"]="node02"
		vb.vmx["numvcpus"]="2"
		vb.vmx["memsize"]="2048"
	end
	
	node02.vm.network :private_network, ip: "192.168.153.103"
	node02.vm.provision "shell", inline: $script
  end

end
