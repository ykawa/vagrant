# -*- mode: ruby -*-
# vi: set ft=ruby :

CPU     = 2
MEM     = 2048
BOX     = "bento/ubuntu-16.04"
NAME    = "docker"
INFLATE = 10000


Vagrant.configure("2") do |config|

  config.vm.box = BOX
  config.vm.box_check_update = false
  config.vm.hostname = NAME
  config.vm.provider :virtualbox do |vb|
    vb.name = NAME
    vb.cpus = CPU
    vb.memory = MEM
    vb.customize ["modifyvm",  :id,
      "--natdnsproxy1",        "on",
      "--natdnshostresolver1", "on"
    ]
  end

  #config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network :public_network

  # Port forwarding for basics.
  [
    { :id => :ssh,   :port =>  22 },
    { :id => :http,  :port =>  80 },
    { :id => :https, :port => 443 },

  ].each do |pmap|
    config.vm.network "forwarded_port",
        guest:   pmap[:port],
        host:    pmap[:port] + INFLATE,
        id:      pmap[:id],
        auto_correct: true
  end

  # Port forwarding for Applications.
  config.vm.usable_port_range = ( INFLATE+1000..INFLATE+2000 )
  [ *3000..3008, *5000..5008, *8000..8008 ].each do |port|
    config.vm.network "forwarded_port",
        guest:   port,
        host:    port,
        auto_correct: true
  end

  config.vm.synced_folder ".",       "/vagrant", mount_options: ['dmode=777', 'fmode=777']
  config.vm.synced_folder "../data", "/data",    mount_options: ['dmode=777', 'fmode=777'], create: true

  vbox_install_path = ENV['VBOX_MSI_INSTALL_PATH']
  vbox_manage = File.join(vbox_install_path, "VBoxManage")
  interfaces  = %x("#{vbox_manage}" list bridgedifs)
  #puts interfaces

  config.vm.provision :ansible_local do |ansible|
    ansible.playbook = "provision/playbook.yml"
    ansible.install  = true
    ansible.limit    = "all"
    ansible.verbose  = "vvv"
  end

  config.ssh.insert_key = false
  config.ssh.forward_agent = true
end

