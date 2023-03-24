# -*- mode: ruby -*-
# vi: set ft=ruby :

# vagrant plugin install vagrant-vbguest --plugin-version 0.21
# Providers: VirtualBox, VMware, Hyper-V
# vagrant up --provider=PROVIDER`

required_plugins = %w(vagrant-vbguest vagrant-hostsupdater hashie)
plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

default_ip="10.8.21.140"
default_cpus=1
default_memory=1024
# centos
# default_vm_box="centos/6"
# default_vm_box_version="1902.01" # CentOS 6.10
default_vm_box="bento/centos-7"
default_vm_box_version="202303.13.0"
# debian
#default_vm_box="debian/stretch64"
#default_vm_box_version="9.2.0"

# NOTE" makes use of the Hashie gem to parse Ansible's YAML inventory files
# install before using like:
# vagrant plugin install hashie

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # this same file is used for ansible inventory
  inventory_path = 'inventory/vagrant'
  inventory = YAML.load_file(inventory_path)
  inventory.extend Hashie::Extensions::DeepFind
  ansible_hosts = inventory.deep_find_all('hosts').inject(:merge)

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  ansible_hosts.each_with_index do |(host_name, host_vars), index|
    # set empty host_vars, if none provided
    if not host_vars
      host_vars = Hash.new
    end

    # increment the default ip in case we need it
    default_ip = default_ip.succ

    config.vm.define host_name do |host|
      # set up box image for each host
      host.vm.box = host_vars['vagrant_box_image'] || default_vm_box
      host.vm.box_version = host_vars['vagrant_box_version'] || default_vm_box_version

      # configure network
      host.vm.network 'private_network', ip: host_vars['ansible_ip'] || default_ip
      host.vm.hostname = host_name

      # presumes installation of https://github.com/cogitatio/vagrant-hostsupdater on host
      if host_vars['aliases']
        host.hostsupdater.aliases = host_vars['aliases']
      end

      # avoiding "Authentication failure" issue
      host.ssh.insert_key = false
      # host.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
      # host.ssh.extra_args = ["-t", "mkdir -p /vagrant ; cd /vagrant; bash --login"]
      host.vm.synced_folder ".", "/vagrant", disabled: true
      # host.vm.synced_folder ".", "/home/vagrant/provision", type: "rsync"
      # host.vm.synced_folder "mount",
      #   "/vagrant",
      #   type: "nfs",
      #   mount_options: ['rw', 'vers=3', 'tcp', 'fsc' ,'actimeo=1'],
      #   create: true

      host.vm.provider "virtualbox" do |vb|
        vb.name = host_name
        vb.memory = host_vars['memory'] || default_memory
        vb.cpus = host_vars['cpus'] || default_cpus
        vb.linked_clone = true
      end

      if index == (ansible_hosts.size - 1)
        host.vm.provision "ansible" do |ansible|
          ansible.galaxy_role_file = "requirements.yml"
          ansible.inventory_path = inventory_path
          ansible.playbook = "setup.yml"
          ansible.limit = "all,localhost"
          ansible.verbose = "v"
        end
      end
    end
  end
end
