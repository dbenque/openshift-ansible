# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"

unless Vagrant.has_plugin?("vagrant-hostmanager")
  raise 'vagrant-hostmanager plugin is required'
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  deployment_type = ENV['OPENSHIFT_DEPLOYMENT_TYPE'] || 'origin'
  num_nodes = (ENV['OPENSHIFT_NUM_NODES'] || 2).to_i

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |vbox, override|
    override.vm.box = "chef/centos-7.1"
    vbox.memory = 1024
    vbox.cpus = 2

    # Enable multiple guest CPUs if available
    vbox.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.provider "libvirt" do |libvirt, override|
    libvirt.cpus = 2
    libvirt.memory = 1024
    libvirt.driver = 'kvm'
    override.vm.box = "centos-7.1"
    override.vm.box_url = "https://download.gluster.org/pub/gluster/purpleidea/vagrant/centos-7.1/centos-7.1.box"
    override.vm.box_download_checksum = "b2a9f7421e04e73a5acad6fbaf4e9aba78b5aeabf4230eebacc9942e577c1e05"
    override.vm.box_download_checksum_type = "sha256"
  end

  num_nodes.times do |n|
    node_index = n+1
    config.vm.define "node#{node_index}" do |node|
      node.vm.hostname = "ose3-node#{node_index}.example.com"
      node.vm.network :private_network, ip: "192.168.100.#{200 + n}"
    end
  end

  config.vm.define "master" do |master|
    master.vm.hostname = "ose3-master.example.com"
    master.vm.network :private_network, ip: "192.168.100.100"
    master.vm.network :forwarded_port, guest: 8443, host: 8443
    master.vm.provision "ansible" do |ansible|
      ansible.limit = 'all'
      ansible.sudo = true
      ansible.groups = {
        "masters" => ["master"],
        "nodes"   => ["node1", "node2"],
      }
      ansible.extra_vars = {
        openshift_deployment_type: "origin",
      }
      ansible.playbook = "playbooks/byo/config.yml"
    end
  end
end
