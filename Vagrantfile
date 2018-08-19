# coding: utf-8
# -*- mode: ruby -*-
#

Vagrant.configure(2) do |config|

  # ノード１ 仮想マシンの起動
  #
  config.vm.define 'node1' do |machine|
    machine.vm.box = "ubuntu/xenial64"
    machine.vm.hostname = 'node1'
    machine.vm.network :private_network,ip: "172.16.20.12"
    #machine.vm.network :public_network, ip: "192.168.1.92", bridge: "en0: Ethernet"
    machine.vm.provider "virtualbox" do |vbox|
      vbox.gui = false        
      vbox.cpus = 1
      vbox.memory = 1024
    end

    # ノード１ docker & k8sのインストール 
    #
    machine.vm.provision "ansible_local" do |ansible|
      ansible.playbook       = "ansible-playbook/kubernetes.yml"
      ansible.version        = "2.6.3"
      ansible.verbose        = false
      ansible.install        = true
      ansible.limit          = "node1"      
      ansible.inventory_path = "ansible-playbook/hosts"
    end
    #machine.vm.provision "shell", inline: <<-EOF
    #  apt-get update && apt-get install -y python-minimal
    #EOF
  end


  # ノード２ 仮想マシンの起動
  #
  config.vm.define 'node2' do |machine|
    machine.vm.box = "ubuntu/xenial64"
    machine.vm.hostname = 'node2'
    machine.vm.network :private_network,ip: "172.16.20.13"
    #machine.vm.network :public_network, ip: "192.168.1.93", bridge: "en0: Ethernet"
    machine.vm.provider "virtualbox" do |vbox|
      vbox.gui = false        
      vbox.cpus = 1
      vbox.memory = 1024
    end

    # ノード２ docker & k8sのインストール 
    #
    machine.vm.provision "ansible_local" do |ansible|
      ansible.playbook       = "ansible-playbook/kubernetes.yml"
      ansible.version        = "2.6.3"
      ansible.verbose        = false
      ansible.install        = true
      ansible.limit          = "node2"
      ansible.inventory_path = "ansible-playbook/hosts"
    end
    #machine.vm.provision "shell", inline: <<-EOF
    #  apt-get update && apt-get install -y python-minimal
    #EOF
  end


  #
  # マスタ 仮想マシンの起動
  #
  config.vm.define 'master' do |machine|
    machine.vm.box = "ubuntu/xenial64"
    machine.vm.hostname = 'master'
    machine.vm.network :private_network,ip: "172.16.20.11"
    #machine.vm.network :public_network, ip: "192.168.1.91", bridge: "en0: Ethernet"
    machine.vm.provider "virtualbox" do |vbox|
      vbox.gui = false        
      vbox.cpus = 1
      vbox.memory = 1024
    end

    #
    # マスタ docker & k8sのインストール 
    #
    machine.vm.provision "ansible_local" do |ansible|
      ansible.playbook       = "ansible-playbook/kubernetes.yml"
      ansible.version        = "2.6.3"
      ansible.verbose        = false
      ansible.install        = true
      ansible.limit          = "master" # or only "nodes" group, etc.
      ansible.inventory_path = "ansible-playbook/hosts"
    end

    # マスタのセットアップ
    #
    machine.vm.provision "ansible_local" do |ansible|
      ansible.playbook      = "ansible-playbook/k8s_master.yml"
      ansible.version        = "2.6.3"
      ansible.verbose        = false
      ansible.install        = true
      ansible.limit          = "master"
      ansible.inventory_path = "ansible-playbook/hosts"
    end

    # ノードのセットアップ
    #
    machine.vm.provision "ansible_local" do |ansible|
      ansible.playbook      = "ansible-playbook/k8s_nodes.yml"
      ansible.version        = "2.6.3"
      ansible.verbose        = false
      ansible.install        = true
      ansible.limit          = "nodes"
      ansible.inventory_path = "ansible-playbook/hosts"
    end

  end
  
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

end


