# -*- mode: ruby -*-
# vi: set ft=ruby :
module OS
  def OS.windows?
    (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
  end

  def OS.mac?
    (/darwin/ =~ RUBY_PLATFORM) != nil
  end

  def OS.unix?
    !OS.windows?
  end

  def OS.linux?
    OS.unix? and not OS.mac?
  end
end

# Ensure that the environment file exists
if !File.exists?('.env')
    abort ".env in application root not found. Please run `cp .env.dist .env`"
end

require 'json'
require 'dotenv'
Dotenv.load

# Check required plugins
required_plugins = %w(vagrant-hostsupdater vagrant-env)
need_restart = false
required_plugins.each do |plugin|
    unless Vagrant.has_plugin? plugin
        unless system "vagrant plugin install #{plugin}"
            abort "Installation of #{plugin} has failed. Aborting."
        end
        need_restart = true
    end
end
exec "vagrant #{ARGV.join(' ')}" if need_restart

HOSTUPDATER_HOSTS = Array.new
array = JSON.parse(ENV['VAGRANT.HOSTS'])
array.each { | item |
  HOSTUPDATER_HOSTS.push(item['host'])
}

Vagrant.configure('2') do |config|
    config.ssh.forward_agent = true
    config.ssh.insert_key = false
    config.ssh.keep_alive = true

    config.env.enable
    config.vm.box_check_update = true
    config.vm.box = "ubuntu/bionic64"
    config.vm.network :private_network, ip: ENV['VAGRANT.SERVER.IP']
    config.vm.hostname = "docker.local"
    config.hostsupdater.aliases = HOSTUPDATER_HOSTS
    config.vm.synced_folder ".", "/vagrant"

    config.vm.provider "virtualbox" do |v|
        v.memory = ENV['VAGRANT.SERVER.MEM']
        v.cpus = ENV['VAGRANT.SERVER.CPUS']
    end

    config.vm.provision "shell", inline: "
        sudo apt-add-repository ppa:ansible/ansible -y &&
        sudo apt-get update &&
        sudo apt install ansible -y
    "

    config.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
      s.inline = <<-SHELL
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
      SHELL
    end
    config.vm.provision "file", source: "~/.gitconfig", destination: "/home/vagrant/.gitconfig"

    config.vm.provision "ansible", type: "ansible_local" do |ansible|
      ansible.playbook = "/vagrant/ansible/site.yaml"
      ansible.inventory_path  = "/vagrant/ansible/inventories/development/hosts"
      ansible.limit = "all"
    end
end
