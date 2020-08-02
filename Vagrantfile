# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# author : Pramode P
#
# Pospose : Vagrant script provisions a kubernetes cluster with 1 master node and 2 worker nodes
#
# Using yaml to load external configuration files
require 'yaml'
require 'fileutils'

Vagrant.configure("2") do |config|
  # Using the hostmanager vagrant plugin to update the host files
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  
  # Create a data dir to mount with in the VM information
  dirname = File.dirname("./../data/")
  unless File.directory?(dirname)
  FileUtils.mkdir_p(dirname)
  end
  
  # Loading in the VM configuration information
  servers = YAML.load_file('servers.yaml')
  
  servers.each do |servers| 
    config.vm.define servers["name"] do |srv|
      srv.vm.box = servers["box"] # Speciy the name of the Vagrant box file to use
      srv.vm.hostname = servers["name"] # Set the hostname of the VM
#     Add a second adapater with a specified IP
      srv.vm.network "private_network", ip: servers["ip"], :adapater=>2 
#     srv.vm.network :forwarded_port, guest: 22, host: servers["port"] # Add a port forwarding rule
      srv.vm.synced_folder ".", "/vagrant", type: "virtualbox"
      srv.vm.synced_folder "./../data/" , "/data", type: "virtualbox", owner: "root", group: "root"
	  srv.ssh.insert_key = false
	  
      srv.vm.provider "virtualbox" do |vb|
        vb.name = servers["name"] # Name of the VM in VirtualBox
        vb.cpus = servers["cpus"] # How many CPUs to allocate to the VM
        vb.memory = servers["memory"] # How much memory to allocate to the VM
#       vb.customize ["modifyvm", :id, "--cpuexecutioncap", "10"] # Limit to VM to 10% of available 
      end
	  
	  if servers["name"].include? "master" then
		srv.vm.provision "shell", inline: <<-SHELL
		       
			sudo mkdir -p /data/$cluster	
			sudo cp /etc/kubernetes/admin.conf /data/$cluster/config
					
		SHELL
	  end
	  
	  if servers["name"].include? "worker" then
	    srv.vm.provision "shell", inline: <<-SHELL
		        
            echo "--- Join as worker node "
			sudo sed -i  "s~192.168.100.3~$IPADDR_ENP0S8~g" /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
            sudo chmod +x /data/$cluster/kubeadm_join_cmd.sh
            sudo sh /data/$cluster/kubeadm_join_cmd.sh
            
            echo "--- create dummy bootstart if not exist"
            [ -f /etc/kubernetes/bootstrap-kubelet.conf ] || sudo touch /etc/kubernetes/bootstrap-kubelet.conf
		   
		SHELL
      end
	  	 
   end
 end
end