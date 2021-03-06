# 172.28.128
# -*- mode: ruby -*-
# vi: set ft=ruby :
#

agents = ENV['PUPPET_AGENTS'] || 2


installers = Dir.entries(File.dirname(__FILE__)).select { |f| f =~ /puppet-enterprise.*.tar.gz/ }
if installers.empty?
  fail("No installer found, please place a Puppet Enterprise tgz file in the same directory as Vagrantfile")
end

# If there is more than one installer in the directory we
# use the latest one
installer = installers.last


# Set ports for exposing Puppet services to the host
network = ENV['PE_VAGRANT_NETWORK'] || '172.28.128'
console_port = ENV['PE_VAGRANT_CONSOLE_PORT'] || '8443'
orchestrator_port = ENV['PE_VAGRANT_ORCHESTRATOR_PORT'] || '8143'
rbac_port = ENV['PE_VAGRANT_RBAC_PORT'] || '4433'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos/7"
  config.vm.define 'puppet' do |master|
    #master.vm.network :private_network, ip: "#{network}.99"
    master.vm.network :private_network, type: "dhcp"
    master.vm.network "forwarded_port", guest: 443, host: console_port
    master.vm.network "forwarded_port", guest: 4433, host: rbac_port
    master.vm.network "forwarded_port", guest: 8143, host: orchestrator_port
    master.vm.hostname = 'puppet.tickbox'
    master.vm.provider :virtualbox do |v|
      v.memory = 4096
    end

    master.vm.provision :hosts, :sync_hosts => true
    master.vm.provision "shell", inline: %Q{
      mkdir /root/pe
      tar -xv -C /root/pe --strip 1 -f /vagrant/#{installer}
      cd /root/pe
      ./puppet-enterprise-installer -y -c /vagrant/pe.conf
      echo '*' > /etc/puppetlabs/puppet/autosign.conf
      (/opt/puppetlabs/bin/puppet agent -t ; true)
    }
      
  end
  (1..agents.to_i).each do |sec|
    config.vm.define "agent#{sec}" do |agent|
      agent.vm.hostname = "agent#{sec}.tickbox"
      #agent.vm.network :private_network, ip: "#{network}.1#{sec}"
      agent.vm.network :private_network, type: "dhcp"
      agent.vm.provision :hosts, :sync_hosts => true
      agent.vm.provision "shell", inline: %Q{
        mkdir /root/pe
        tar -xv -C /root/pe --strip 1 -f /vagrant/#{installer}
        yum install -y /root/pe/packages/el-7-x86_64/puppet-agent*
        (/opt/puppetlabs/bin/puppet agent -t ; true)
      }
    end
  end
  
end


