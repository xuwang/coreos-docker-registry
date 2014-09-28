# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'

Vagrant.require_version ">= 1.6.0"

CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), "user-data")
CONFIG = File.join(File.dirname(__FILE__), "config.rb")
TEST_ROOT_CA_PATH = File.join(File.dirname(__FILE__), "apps/nginx/certs/generate/rootCA.pem")

# Defaults for config options defined in CONFIG
$num_instances = 1
$update_channel = "alpha"
$enable_serial_logging = false
$vb_gui = false
$vb_memory = 1024
$vb_cpus = 1

# Customized configuration
expose_etcd_tcp = true

# Attempt to apply the deprecated environment variable NUM_INSTANCES to
# $num_instances while allowing config.rb to override it
if ENV["NUM_INSTANCES"].to_i > 0 && ENV["NUM_INSTANCES"]
  $num_instances = ENV["NUM_INSTANCES"].to_i
end

if File.exist?(CONFIG)
  require CONFIG
end

Vagrant.configure("2") do |config|
  config.vm.box = "coreos-%s" % $update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % $update_channel
  
  # To enable ssh agent fowarding and
  # add vagrant insecure_private_key to ssh-agent for fleet ssh
  # if you are using a different private_keys vs the default vagrant's insecure_private_key,
  # add them to ssh-agent: 
  #      ssh-add <your_cluster_private_key>
  config.ssh.forward_agent = true
  %x( if [[ `ssh-add -l` != *insecure_private_key* ]];then ssh-add ~/.vagrant.d/insecure_private_key; fi )

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant_vmware_fusion.json" % $update_channel
  end

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "core-%02d" % i do |config|
      config.vm.hostname = vm_name

      if $enable_serial_logging
        logdir = File.join(File.dirname(__FILE__), "log")
        FileUtils.mkdir_p(logdir)

        serialFile = File.join(logdir, "%s-serial.txt" % vm_name)
        FileUtils.touch(serialFile)

        config.vm.provider :vmware_fusion do |v, override|
          v.vmx["serial0.present"] = "TRUE"
          v.vmx["serial0.fileType"] = "file"
          v.vmx["serial0.fileName"] = serialFile
          v.vmx["serial0.tryNoRxLoss"] = "FALSE"
        end

        config.vm.provider :virtualbox do |vb, override|
          vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
        end
      end

      if $expose_docker_tcp
        config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
      end

      if $expose_etcd_tcp
        config.vm.network "forwarded_port", guest: 4001, host: ($expose_etcd_tcp + i - 1), auto_correct: true
      end

      config.vm.provider :vmware_fusion do |vb|
        vb.gui = $vb_gui
      end

      config.vm.provider :virtualbox do |vb|
        vb.gui = $vb_gui
        vb.memory = $vb_memory
        vb.cpus = $vb_cpus
      end

      ip_base = "172.17.8"
      ip = "#{ip_base}.#{i+100}"
      config.vm.network :private_network, ip: ip

      # enable NFS for sharing the host machine into the coreos-vagrant VM.
      config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']
      
      # The demo docker registry is using a self-signed certs for https://registery.docker.local.
      # To make the self-signed certs truesed for clients in vbox, update the system bundled certs with the rootCA.
      # This should be done before docker.service starts so it can pick up the testing rootCA.
      config.vm.provision :file, :source => "#{TEST_ROOT_CA_PATH}", :destination => "/tmp/XXX-Dockerage.pem"
      config.vm.provision :shell, :inline => "cd /etc/ssl/certs && mv /tmp/XXX-Dockerage.pem . && update-ca-certificates", :privileged => true

      # enable ssh-agent forwarding in coreos-vagrant VM.
      config.vm.provision \
        :shell, \
        :inline => "echo -e 'Host #{ip_base}.* registry*\n  StrictHostKeyChecking no\n  ForwardAgent yes' > .ssh/config; chmod 600 .ssh/config", \
        :privileged => false

      if File.exist?(CLOUD_CONFIG_PATH)
        config.vm.provision :file, :source => "#{CLOUD_CONFIG_PATH}", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end 


      config.vm.provision :shell, :inline => "cd /var/lib && ln -sf /home/core/share/apps apps && ln -sf /home/core/share/data apps-data", :privileged => true
      if i == $num_instances
        # start all initial feet units, do it only on last node
        config.vm.provision :shell, :inline => "cd /var/lib/apps/units && ./all start", :privileged => true
      end  
    end
  end
end
