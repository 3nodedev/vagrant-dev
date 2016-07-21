# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = 'boxcutter/ubuntu1404'
  config.ssh.forward_agent = true

  config.ssh.shell = %{bash -c 'BASH_ENV=/etc/profile exec bash'}

  # Required for NFS to work, pick any local IP
  config.vm.network :private_network, type: "dhcp"

  # config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 4200, host: 4200
  config.vm.network "forwarded_port", guest: 443, host: 44300
  config.vm.network "forwarded_port", guest: 5432, host: 5433
  config.vm.network "forwarded_port", guest: 3001, host: 3001
  config.vm.network "forwarded_port", guest: 5857, host: 5858
  config.vm.network "forwarded_port", guest: 49152, host: 49152
  config.vm.network "forwarded_port", guest: 35729, host: 35729

  config.vm.hostname = "vagrant-dev"
  config.vm.synced_folder "data", "/data", create: true, nfs: true,
    mount_options: %w{nolock,vers=3,udp,noatime,actimeo=1}

  host = RbConfig::CONFIG['host_os']
  if host =~ /darwin/
    cpus = `sysctl -n hw.ncpu`.to_i / 4
    memory = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
  elsif host =~ /linux/
    cpus = `nproc`.to_i / 4
    memory = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
  else
    cpus = 2
    memory = 4096
  end

  config.vm.provider 'virtualbox' do |vb|
    vb.customize ['modifyvm', :id, '--memory', memory]
    vb.customize ['modifyvm', :id, '--cpus', cpus]

    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']

    vb.name = config.vm.hostname
  end

  # VMware Workstation/Fusion settings
  ['vmware_fusion', 'vmware_workstation'].each do |provider|
    config.vm.provider provider do |vmw, override|
      # override.vm.box = 'puppetlabs/ubuntu-16.04-64-puppet'

      vmw.vmx['memsize'] = memory
      vmw.vmx['numvcpus'] = cpus

      vmw.name = config.vm.hostname
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
    ansible.verbose = "true"
  end

  # Configure vm to use host ssh keys

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y git
    mkdir -p ~/.ssh
    chmod 700 ~/.ssh
    ssh-keyscan -H github.com >> ~/.ssh/known_hosts
  SHELL
end
