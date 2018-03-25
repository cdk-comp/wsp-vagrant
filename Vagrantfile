# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

# Load the settings file
settings = YAML.load_file(File.join(File.dirname(__FILE__), "wsp-conf.yaml"))

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Detect host OS for different folder share configuration
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

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
config.vm.box = "bento/ubuntu-16.04"

  config.vm.provider "parallels"
  config.vm.provider "virtualbox"
  
  if !Vagrant.has_plugin?('vagrant-cachier')
    puts "The vagrant-cachier plugin is required. Please install it with \"vagrant plugin install vagrant-cachier\""
    exit
  end

  if !Vagrant.has_plugin?('vagrant-hostmanager')
    puts "The vagrant-hostmanager plugin is required. Please install it with \"vagrant plugin install vagrant-hostmanager\""
    exit
  end
  
  if OS.windows?      
    if !Vagrant.has_plugin?('vagrant-winnfsd')         
      puts "The vagrant-winnfsd plugin is required. Please install it with \"vagrant plugin install vagrant-winnfsd\""   
      exit
    end
  end
    
  if Vagrant.has_plugin?('vagrant-vbguest')
      config.vbguest.auto_update = true
  end
  
  config.cache.scope = :box

  # Vagrant hardware settings
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", settings["memory"] ||= "2048"]
    vb.customize ["modifyvm", :id, "--cpus", settings["cpus"] ||= "2"]
  end

  config.vm.provider "parallels" do |v|
    v.memory = settings["memory"] ||= "2048"
    v.cpus = settings["cpus"] ||= "2"
  end
  
  # vagrant-hostmanager config (https://github.com/smdahlen/vagrant-hostmanager)
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  config.vm.define "project" do |node|
    node.vm.hostname = settings['hostname'] ||= 'sp-vagrant'
    node.vm.network :private_network, ip: settings['ip'] ||= '192.168.100.100'

    node.hostmanager.aliases = [settings['aliases']]

    config.vm.network :forwarded_port, guest: 80, host: 8080
  end

  # Configure The Public Key For SSH Access
  if settings.include? 'authorize'
    if File.exists? File.expand_path(settings["authorize"])
      config.vm.provision "shell" do |s|
        s.inline = "echo $1 | grep -xq \"$1\" /home/vagrant/.ssh/authorized_keys || echo \"\n$1\" | tee -a /home/vagrant/.ssh/authorized_keys"
        s.args = [File.read(File.expand_path(settings["authorize"]))]
      end
    end
  end

  # Copy The SSH Private Keys To The Box
  if settings.include? 'keys'
    if settings["keys"].to_s.length == 0
      puts "Check your wsp-conf.yaml file, you have no private key(s) specified."
      exit
    end
    settings["keys"].each do |key|
      if File.exists? File.expand_path(key)
        config.vm.provision "shell" do |s|
          s.privileged = false
          s.inline = "echo \"$1\" > /home/vagrant/.ssh/$2 && chmod 600 /home/vagrant/.ssh/$2"
          s.args = [File.read(File.expand_path(key)), key.split('/').last]
        end
      else
        puts "Check your wsp-conf.yaml file, the path to your private key does not exist."
        exit
      end
    end
  end

  # Disabling the default /vagrant share
  config.vm.synced_folder '.', '/vagrant', disabled: true

  # Register All Of The Configured Shared Folders
  if settings.include? 'folders'
    settings["folders"].each do |folder|
      # Check the host filepath and not the guest (waiting serverpilot to create this folder before) Need to learn ruby lol!
      if File.exists? File.expand_path(folder["map"])
        if OS.windows?
          config.vm.synced_folder folder["map"], folder["to"], type: "nfs", owner: "serverpilot", group: "serverpilot", disabled: true
        else
          config.vm.synced_folder folder["map"], folder["to"], owner: "serverpilot", group: "serverpilot", disabled: true
        end
      end
    end
  end

  config.ssh.forward_agent = true

  # curl packages install (run only once)
  config.vm.provision "shell", inline: "apt-get update && apt-get install -y curl"

  # Configure The Serverpilot server name
  if settings["hostname"].to_s.length == 0
    puts "Check your wsp-conf.yaml file, you have no Serverpilot server name specified."
    exit
  else
    config.vm.provision "shell" do |s|
      s.inline = "echo $1$2 | grep -xq \"$1$2\" /home/vagrant/.bash_profile || echo \"\n$1$2\" | tee -a /home/vagrant/.bash_profile"
      s.args   = ['export server_name=', settings["hostname"]]
    end
  end

  # Configure The Serverpilot Client ID
  if settings["serverpilot_cliend_id"].to_s.length == 0
    puts "Check your wsp-conf.yaml file, you have no Serverpilot Client ID specified."
    exit
  else
    config.vm.provision "shell" do |s|
      s.inline = "echo $1$2 | grep -xq \"$1$2\" /home/vagrant/.bash_profile || echo \"\n$1$2\" | tee -a /home/vagrant/.bash_profile"
      s.args   = ['export serverpilot_client_id=', settings["serverpilot_cliend_id"]]
    end
  end

  # Configure The Serverpilot API key
  if settings["serverpilot_api_key"].to_s.length == 0
    puts "Check your wsp-conf.yaml file, you have no Serverpilot API key specified."
    exit
  else
    config.vm.provision "shell" do |s|
      s.inline = "echo $1$2 | grep -xq \"$1$2\" /home/vagrant/.bash_profile || echo \"\n$1$2\" | tee -a /home/vagrant/.bash_profile"
      s.args   = ['export serverpilot_api_key=', settings["serverpilot_api_key"]]
    end
  end

  # Serverpilot Install
  config.vm.provision "shell", inline: "curl -sSL cdk.mk/pss > /home/vagrant/app-create.sh"
  config.vm.provision "shell", inline: "curl -sSL cdk.mk/nss > /home/vagrant/wsp-setup.sh"
  config.vm.provision "shell", inline: "curl -sSL cdk.mk/adm > /home/vagrant/admin-create.sh"
  config.vm.provision "shell", inline: "source /home/vagrant/.bash_profile && bash wsp-setup.sh"
  config.vm.provision "shell", inline: "source /home/vagrant/.bash_profile && bash admin-create.sh"
end
