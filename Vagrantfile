DOMAIN="k8s.sr"

servers=[
  {
    :hostname => "master." + DOMAIN,
    :ip => "192.168.56.200",
	:mac => "5CA1AB1E0001",
    :type => "master",
    :ram => 3072,
    :port => 8080,
	 :cpu => 2
  },
  {
    :hostname => "node1." + DOMAIN,
    :ip => "192.168.56.211",
	:mac => "5CA1AB1E0002",
    :type => "node",
    :ram => 3072,
	  :port => 8080,
	  :cpu => 2
  },
  {
    :hostname => "node2." + DOMAIN,
    :ip => "192.168.56.212",
	:mac => "5CA1AB1E0003",
    :type => "node",
    :ram => 3072,
	  :port => 8080,
	  :cpu => 2
  }
]


Vagrant.configure("2") do |config|

servers.each do |machine|
	ip=machine[:ip]
	port=machine[:port]
	config.vm.define machine[:hostname] do |node|
		node.vm.box = "sbeliakou/centos"
		config.vm.box_check_update = true

		node.vm.hostname = machine[:hostname]
		 node.vm.network "private_network", ip: machine[:ip],  :mac =>machine[:mac]
		#node.vm.network "private_network", type: "dhcp"	
		node.vm.provider "virtualbox" do |vb|
			vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
			vb.customize ["modifyvm", :id, "--cpus", machine[:cpu]]
			vb.name = machine[:hostname]
			#vb.customize ["modifyvm", :id, "--nic1", "myNatNetwork"]

		end
		node.vm.provision "shell", inline: <<-SHELL
				systemctl stop firewalld
				systemctl disable firewalld
				sudo setenforce 0 #disable selinux 
				sed -i "7c\SELINUX=disabled" /etc/selinux/config #disable selinux
				
				swapoff -a #disable swap file
				sed -i "11s@/dev/@#dev/@" /etc/fstab  #disablew swap file
# create kube repo file
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
	
			 	yum update -y
				systemctl enable sshd
				systemctl start sshd
				yum install docker -y
				systemctl enable docker
				systemctl start docker
#				yum clean all
				
				yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes #install kube tools
				systemctl enable kubelet
				systemctl start kubelet
				
				# disable autoupdate packages
				echo "exclude=kubeadm, kubelet, kubectl" >> /etc/yum.conf
				
				
				
		SHELL
		
		
		if (machine[:type] == "master") # init kube on master
			node.vm.provision "shell", inline: <<-SHELL1
				kubeadm init --apiserver-advertise-address 192.168.56.200 --pod-network-cidr 172.17.0.0/16 --ignore-preflight-errors All > kube_init_result
				echo '#!/bin/bash' > /vagrant/node_join.sh #create script to join workers
				echo  "$(tail -2 kube_init_result)" >> /vagrant/node_join.sh
				
				
				# create kube config for root user
				mkdir -p /root/.kube  #create kube config for root
				sudo cp -i /etc/kubernetes/admin.conf /root/.kube/config
				sudo chown 0:0 /root/.kube/config
				
				#install calico for networking
				curl https://docs.projectcalico.org/manifests/calico.yaml -O
				kubectl apply -f calico.yaml		

				echo "Hello world" > /vagrant/for-text.txt
				SHELL1
		else
			node.vm.provision "shell", inline: <<-SHELL2
				# join workers to cluster
				cp /vagrant/node_join.sh ~
				chmod +x node_join.sh
				~/node_join.sh
				SHELL2
		


	end
end
					
					
end
end
