# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Path where the extra disks should be stored when using VirtualBox
  vbox_vm_path = "d:\\VMs\\" # VirtualBox ONLY! Please use "\\" as directory name delimiter on WIndows as example "C:\\Virtualbox\\"
  vbox_dvd_path = vbox_vm_path + 'CentOS-7-x86_64-DVD-1908.iso'  # iso name of Centos 7 Binary DVD. Please download this dvd in advance.
  # Storage pool where the extra disks should be stored when using libVirt
  libvirt_storage_pool = "/data/storage/" # libVirt ONLY!
  # Storage pool where the extra disks should be stored when using libVirt

  # Memory configuration
  c7_master_memory = 15000 # Recommended: 2048 MiB / Minimum: 1512 MiB Achtung! memory must be not less Minimum. or you will got error in ipa install https://www.reddit.com/r/linuxadmin/comments/9jrbsm/freeipa_server_install_error/ 
  c7_node1_memory = 4000 # Recommended: 1024 MiB / Minimum: 512 MiB
  c7_node2_memory = 4000 # Recommended: 1024 MiB / Minimum: 512 MiB

  c7_workstation_memory = 1500 # Recommended: 2048 MiB / Minimum: 1024 MiB

  extra_disk_size = 10 # Recommended: 10 GiB / Minimum: 2 GiB

  # VM vagrant box name
  config.vm.box = "dmi3mis/centos7"

  config.vagrant.plugins = ["vagrant-vbguest", "vagrant-timezone"]
  config.timezone.value = :host

  
  # Don't modify beyond here
  conname = "Wired connection 1"
  devname = "eth1"

  config.vm.provider "virtualbox" do |vbox, override|
    # vbox.gui = true
    if ENV['VBOX_VM_PATH']
      vbox_vm_path = ENV['VBOX_VM_PATH']
    end
  end

  config.vm.provider "libvirt" do |libvirt, override|
    if ENV['LIBVIRT_STORAGE_POOL']
      libvirt_storage_pool = ENV['LIBVIRT_STORAGE_POOL']
    end
    libvirt.driver = "kvm"
    libvirt.storage_pool_name = libvirt_storage_pool
  end
  config.vm.provision :shell, path: "scripts/add-users"
  
  config.vm.define :"c7-master" do |c7_master_config|
    c7_master_config.vm.hostname = "c7-master.lab.dmi3mis.ml"
    c7_master_config.vm.box = "dmi3mis/centos7"
    c7_master_config.vm.network "private_network", ip: "192.168.2.254", auto_config: false
    c7_master_config.vm.network :forwarded_port, guest: 22, host: 6001, id: "ssh"
    c7_master_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.254/24 ipv4.dns 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_master_config.vm.provision :shell, path: "scripts/c7-nat"
    c7_master_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus   = 4
      vbox.memory = c7_master_memory
      vbox.name   = "c7-master.lab.dmi3mis.ml"
      #vbox.linked_clone = true
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      vbox.customize ["modifyvm", :id, "--vram", "32"]
      vbox.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vbox.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
      vbox.customize ["modifyvm", :id, "--usb", "on"]
      
      vbox.customize ["modifyvm", :id, "--usbehci", "on"]
      vbox.auto_nat_dns_proxy = false
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
	    vbox.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', "2", '--device', "0", '--type', 'dvddrive', '--medium', vbox_dvd_path ]
	    vbox.customize ["modifyvm", :id, "--boot1", "disk", "--boot2", "dvd"]
    end
    c7_master_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_master_memory
    end
	end

  config.vm.define :"c7-node1" do |c7_node1_config|
    c7_node1_config.vm.box = "dmi3mis/centos7"
    c7_node1_config.vm.hostname = "c7-node1.lab.dmi3mis.ml"
    c7_node1_config.vm.network "private_network", ip: "192.168.2.10", auto_config: false
    c7_node1_config.vm.network :forwarded_port, guest: 22, host: 6010, id: "ssh"
    c7_node1_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.10/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_node1_config.vm.network "private_network", ip: "192.168.2.11", auto_config: false
    c7_node1_config.vm.provision :shell, path: "scripts/localrepo"
    c7_node1_config.vm.provision :shell, path: "scripts/ipa-attach"
    c7_node1_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_node1_memory
      vbox.name = "c7-node1.lab.dmi3mis.ml"
      #vbox.linked_clone = true      
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      vbox.customize ["modifyvm", :id, "--vram", "32"]
      vbox.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vbox.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
      vbox.customize ["modifyvm", :id, "--usb", "on"]
      vbox.customize ["modifyvm", :id, "--usbehci", "on"]
      vbox.auto_nat_dns_proxy = false
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      if !File.exist?(vbox_vm_path + 'c7-node1_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-node1_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-node1_disk2.vdi']
    end

    c7_node1_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_node2_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  
  config.vm.define :"c7-node2" do |c7_node2_config|
    c7_node2_config.vm.box = "dmi3mis/centos7"
    c7_node2_config.vm.hostname = "c7-node2.lab.dmi3mis.ml"
    c7_node2_config.vm.network "private_network", ip: "192.168.2.20", auto_config: false
    c7_node2_config.vm.network :forwarded_port, guest: 22, host: 6030, id: "ssh"
    c7_node2_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.20/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10  ipv4.method manual && nmcli con up '#{conname}'"
    c7_node2_config.vm.provision :shell, path: "scripts/localrepo"
    c7_node2_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_node2_memory
      vbox.name = "c7-node2.lab.dmi3mis.ml"
      #vbox.linked_clone = true
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      vbox.customize ["modifyvm", :id, "--vram", "32"]
      vbox.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vbox.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
      vbox.customize ["modifyvm", :id, "--usb", "on"]
      vbox.customize ["modifyvm", :id, "--usbehci", "on"]
      vbox.auto_nat_dns_proxy = false
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      if !File.exist?(vbox_vm_path + 'c7-node2_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-node2_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-node2_disk2.vdi']
    end
    c7_node2_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_node2_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  config.vm.define :"c7-workstation" do |c7_workstation_config|
    c7_workstation_config.vm.box = "dmi3mis/centos7_desktop"
    c7_workstation_config.vm.hostname = "c7-workstation.lab.dmi3mis.ml"
    c7_workstation_config.vm.network "private_network", ip: "192.168.2.40", auto_config: false
    c7_workstation_config.vm.network :forwarded_port, guest: 22, host: 6050, id: "ssh"
    c7_workstation_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.40/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_workstation_config.vm.provision :shell, path: "scripts/localrepo"
    c7_workstation_config.vm.provision :shell, path: "scripts/c7-client"
    c7_workstation_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_workstation_memory
      vbox.name = "c7-workstation.lab.dmi3mis.ml"
      #vbox.linked_clone = true
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      vbox.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
      vbox.customize ["modifyvm", :id, "--vram", "32"]
      vbox.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vbox.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
      vbox.customize ["modifyvm", :id, "--usb", "on"]
      vbox.customize ["modifyvm", :id, "--usbehci", "on"]
      vbox.auto_nat_dns_proxy = false
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      if !File.exist?(vbox_vm_path + 'c7-workstation_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-workstation_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-workstation_disk2.vdi']
    end
    c7_workstation_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_workstation_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end
    

end
