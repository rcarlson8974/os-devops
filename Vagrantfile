# -*- mode: ruby -*-
# vi: set ft=ruby :

unless Vagrant.has_plugin?("vagrant-hostsupdater")
  system("vagrant plugin install vagrant-hostsupdater")
  puts "Dependencies installed, please try the command again."
  exit
end

# Specify Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = "2"

# Require 'yaml' module
require 'yaml'

# Read YAML file with box details
servers = YAML.load_file('servers_vagrant.yml')

# Create boxes
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Iterate through entries in YAML file
  servers.each do |servers|
    config.vm.define servers["name"] do |srv|
      srv.vm.box = servers["box"]
      srv.vm.box_url = servers["box_url"]
      srv.vm.hostname = servers["name"]
      srv.vm.network :forwarded_port, guest: 80, host: servers["forwarded_port"]
      # srv.vm.network :forwarded_port, guest: servers["guest_port"], host: servers["forwarded_port"]

      # Copy SSH key
      id_rsa_ssh_key_pub = File.read(File.expand_path('~') + '/.ssh/id_rsa.pub');
      config.vm.provision :shell, :inline => "echo 'Copying local id_rsa SSH Key to VM auth_keys for auth purposes (login into VM included)...' && echo '#{id_rsa_ssh_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"

      srv.vm.network "private_network", ip: servers["ip"]

      srv.vm.provider :virtualbox do |vb|
        vb.name = servers["name"]
        vb.memory = servers["ram"]
      end

      # Experiencing some issues running the playbooks from here.  Works outside of Vagrant.....
      config.vm.provision "ansible" do |ansible|
         ansible.verbose = "v"
         ansible.inventory_path = "inventory_local"
         ansible.playbook = "playbooks/" + servers["playbook"]
      end

    end
  end

  config.ssh.insert_key = false

end