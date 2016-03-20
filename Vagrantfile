# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT
GALAXY=/usr/local/bin/ansible-galaxy
echo '#!/usr/bin/env python2
import sys
import os

args = sys.argv
if args[1:] == ["--help"]:
  args.insert(1, "info")

os.execv("/usr/bin/ansible-galaxy", args)
' | sudo tee $GALAXY
sudo chmod 0755 $GALAXY
SCRIPT

unless Vagrant.has_plugin?("vagrant-cachier")
  raise 'Following vagrant plugin is missing vagrant-cachier'
end

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"
  #config.vm.box = "centos/7"
  config.vm.box_check_update = true

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.provider :virtualbox do |vb, override|
    vb.gui = true
    vb.customize [
      "modifyvm", :id,
      "--memory", "2048",
      "--cpus", "4",
      "--natdnspassdomain1", "off",
      ]
  end

  config.vm.provider :vmware_fusion do |v, override|
      v.vmx["memsize"] = 1024
      v.vmx["numvcpus"] = 4
  end

## NOTE: "_" in box names are converted into "-" so that DNS works.
boxes = [
  { :name => :privacyidea, :ip => '10.0.0.20', :memory => 2048, :ansibleplaybook => 'ansible/tasks.yml' }
]

boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:memory] ]
      end
      config.vm.provider :vmware_fusion do |v|
        v.vmx["memsize"] =  opts[:memory]
      end
      config.vm.network   :private_network, ip: opts[:ip]
      config.vm.hostname  = "%s.sandbox.internal" % opts[:name].to_s.gsub('_', '-')
      config.vm.provision :shell, :inline => $script
      config.vm.provision "ansible_local" do |ansible|
        ansible.playbook = opts[:ansibleplaybook]
        ansible.install  = true
      end
    end
  end
end
