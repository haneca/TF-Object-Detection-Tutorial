# -*- coding: utf-8 -*-
# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
Vagrant.require_version '>= 2.1'

Vagrant.configure('2') do |config|
  tf_version = '1.0.0'

  # Load Configuration File
  _conf = YAML.load(
    File.open(
      File.join(File.dirname(__FILE__), 'provision/default.yml'),
      File::RDONLY
    ).read
  )

  # check existance of ansible configuration files
  if File.exists?(_conf['ansible_playbook_path'])
    ansible_playbook_path = _conf['ansible_playbook_path']
  # convert relative path to absolute path
  elsif File.exists?(File.join(File.dirname(__FILE__), _conf['ansible_playbook_path']))
    ansible_playbook_path = File.join(File.dirname(__FILE__), _conf['ansible_playbook_path'])
  # if cannot find ansible file
  else
    puts 'Can\'t find ' + _conf['ansible_playbook_path'] + '. Please check ansible_playbook_path in the config.'
    exit 1
  end

  config.vm.define _conf['hostname'] do |v|
  end

  config.vm.box = 'bento/ubuntu-18.04'
  config.ssh.forward_agent = true
  config.vm.box_check_update = true

  # OS settings
  config.vm.hostname = _conf['hostname']
  config.vm.network :private_network, ip: _conf['ip']

  # change /etc/hosts on host machine
  if Vagrant.has_plugin?('vagrant-hostsupdater')
    config.hostsupdater.remove_on_suspend = true
  end

  # guest tools installation
  config.vbguest.auto_update = true
  config.vbguest.no_remote = true

  config.vm.provider :virtualbox do |vb|
    vb.name = _conf['hostname']
    vb.memory = _conf['memory'].to_i
    vb.cpus = _conf['cpus'].to_i
    if 1 < _conf['cpus'].to_i
      vb.customize ['modifyvm', :id, '--ioapic', 'on']
    end
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
  end

  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.synced_folder _conf['sync_folder'], _conf['document_root'], create: true, mount_options: ['dmode=755', 'fmode=644']

  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'playbook.yml'
  end
end
