# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# MongoDB cluster nodes configuration
NODES = [
  { name: "mongo1", ip: "192.168.8.21", primary: true },
  { name: "mongo2", ip: "192.168.8.22", primary: false },
  { name: "mongo3", ip: "192.168.8.23", primary: false }
]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Use Ubuntu 22.04 LTS
  config.vm.box = "ubuntu/jammy64"
  
  # Disable automatic box update checking
  config.vm.box_check_update = false

  NODES.each_with_index do |node, index|
    config.vm.define node[:name] do |node_config|
      node_config.vm.hostname = node[:name]
      node_config.vm.network "private_network", ip: node[:ip]

      # VirtualBox specific configuration
      node_config.vm.provider "virtualbox" do |vb|
        vb.name = node[:name]
        vb.memory = 4096
        vb.cpus = 2
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      end

      # Add hosts entries for all nodes
      NODES.each do |n|
        node_config.vm.provision "shell", inline: <<-SHELL
          grep -q "#{n[:ip]} #{n[:name]}" /etc/hosts || echo "#{n[:ip]} #{n[:name]}" >> /etc/hosts
        SHELL
      end

      # Run Ansible provisioner only on the last node
      if index == NODES.length - 1
        node_config.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbooks/site.yml"
          ansible.limit = "all"
          ansible.become = true
          
          # Auto-generate inventory groups
          ansible.groups = {
            "mongodb_nodes" => NODES.map { |n| n[:name] },
            "mongodb_primary" => NODES.select { |n| n[:primary] }.map { |n| n[:name] },
            "mongodb_secondary" => NODES.reject { |n| n[:primary] }.map { |n| n[:name] }
          }
          
          ansible.extra_vars = {
            mongodb_replica_set_name: "rs0",
            mongodb_nodes: NODES.map { |n| { name: n[:name], ip: n[:ip], primary: n[:primary] } }
          }
        end
      end
    end
  end
end
