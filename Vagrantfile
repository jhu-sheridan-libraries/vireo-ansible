# -*- mode: ruby -*-
# vi: set ft=ruby :

# require_relative './script/authorize_key'

domain          = "test"
setup_complete  = false

# NOTE: currently using the same OS for all boxen
OS="centos" # "debian" || "centos"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  package=""
  if OS=="debian"
    config.vm.box = "debian/jessie64"
    package="_apt"
  elsif OS=="centos"
    config.vm.box = "centos/7"
    package="_yum"
  else
    puts "you must set the OS variable to a valid value before continuing"
    exit
  end

  {
    # 'vireodb'    => '10.51.30.102',
    'vireo'   => '10.51.30.101'
  }.each do |short_name, ip|
    config.vm.define short_name do |host|
      host.vm.network 'private_network', ip: ip
      host.vm.hostname = "#{short_name}.#{domain}"
      # presumes installation of https://github.com/cogitatio/vagrant-hostsupdater on host
      host.hostsupdater.aliases = ["#{short_name}"]
      # avoinding "Authentication failure" issue
      host.ssh.insert_key = false
      host.vm.synced_folder "mount",
        "/vagrant",
        type: "nfs",
        mount_options: ['rw', 'vers=3', 'tcp', 'fsc' ,'actimeo=1'],
        create: true

      host.vm.provider "virtualbox" do |vb|
        vb.name = "#{short_name}.#{domain}"
        vb.memory = 1024
        vb.linked_clone = true
      end

      # # do minimal provisioning (in order to do further work with Ansible)
      # host.vm.provision "prerequisites", type: "shell", path: "script/prereqs#{package}.sh"
      #
      # # add authorized key to user created by the prereqs script
      # authorize_key host, auto_user, auto_key

      if short_name == "vireo" # last in the list
        setup_complete = true
      end

      if setup_complete
        # workaround for https://github.com/mitchellh/vagrant/issues/8142
        host.vm.provision "shell",
          inline: "sudo service network restart"

        host.vm.provision "ansible" do |ansible|
          ansible.galaxy_role_file = "requirements.yml"
          ansible.inventory_path = "inventory/vagrant"
          ansible.playbook = "setup.yml"
          ansible.limit = "all"
          ansible.verbose = "v"
        end
      end
    end
  end
end
