# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Path where the extra disks should be stored when using VirtualBox
  vbox_vm_path = "C:\\VMs\\" # VirtualBox ONLY! Please use "\\" as directory name delimiter on WIndows as example "C:\\Virtualbox\\"
  vbox_dvd_path = vbox_vm_path + 'CentOS-7-x86_64-DVD-1810.iso'  # iso name of Centos 7 Binary DVD. Please download this dvd in advance.
  # Storage pool where the extra disks should be stored when using libVirt
  libvirt_storage_pool = "/data/storage/" # libVirt ONLY!
  # Storage pool where the extra disks should be stored when using libVirt

  # Memory configuration
  c7_nat01_memory =    1512 # Recommended: 2048 MiB / Minimum: 1512 MiB Achtung! memory must be not less Minimum. or you will got error in ipa install https://www.reddit.com/r/linuxadmin/comments/9jrbsm/freeipa_server_install_error/ 
  c7_server01_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  c7_server02_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  c6_server01_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  c7_client01_memory = 2048 # Recommended: 2048 MiB / Minimum: 1024 MiB
  c7_client02_memory = 2048 # Recommended: 2048 MiB / Minimum: 1024 MiB
  extra_disk_size = 2 # Recommended: 10 GiB / Minimum: 2 GiB

  # VM vagrant box name
  config.vm.box = "dmi3mis/centos7"

  config.vagrant.plugins = ["vagrant-vbguest", "vagrant-timezone", "vagrant-hosts"]
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
  
  config.vm.define :"c7-nat01" do |c7_nat01_config|
    c7_nat01_config.vm.hostname = "c7-nat01.ll-100.local"
    c7_nat01_config.vm.box = "dmi3mis/centos7"
    c7_nat01_config.vm.network "private_network", ip: "192.168.2.254", auto_config: false
    c7_nat01_config.vm.network :forwarded_port, guest: 22, host: 6001, id: "ssh"
    c7_nat01_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.254/24 ipv4.dns 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_nat01_config.vm.provision :shell, path: "scripts/c7-nat"
    c7_nat01_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus   = 1
      vbox.memory = c7_nat01_memory
      vbox.name   = "c7-nat01.LL-100.local"
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
    c7_nat01_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_nat01_memory
    end
	end

  config.vm.define :"c7-server01" do |c7_server01_config|
    c7_server01_config.vm.box = "dmi3mis/centos7"
    c7_server01_config.vm.hostname = "c7-server01.ll-100.local"
    c7_server01_config.vm.network "private_network", ip: "192.168.2.10", auto_config: false
    c7_server01_config.vm.network :forwarded_port, guest: 22, host: 6010, id: "ssh"
    c7_server01_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.10/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_server01_config.vm.network "private_network", ip: "192.168.2.11", auto_config: false
    c7_server01_config.vm.provision :shell, path: "scripts/localrepo"
    c7_server01_config.vm.provision :shell, path: "scripts/ipa-attach"
    c7_server01_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_server01_memory
      vbox.name = "c7-server01.LL-100.local"
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
      if !File.exist?(vbox_vm_path + 'c7-server01_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-server01_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-server01_disk2.vdi']
    end

    c7_server01_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_server02_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  
  config.vm.define :"c7-server02" do |c7_server02_config|
    c7_server02_config.vm.box = "dmi3mis/centos7"
    c7_server02_config.vm.hostname = "c7-server02.ll-100.local"
    c7_server02_config.vm.network "private_network", ip: "192.168.2.20", auto_config: false
    c7_server02_config.vm.network :forwarded_port, guest: 22, host: 6030, id: "ssh"
    c7_server02_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.20/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10  ipv4.method manual && nmcli con up '#{conname}'"
    c7_server02_config.vm.provision :shell, path: "scripts/localrepo"
    c7_server02_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_server02_memory
      vbox.name = "c7-server02.LL-100.local"
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
      if !File.exist?(vbox_vm_path + 'c7-server02_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-server02_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-server02_disk2.vdi']
    end
    c7_server02_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_server02_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  
  
  config.vm.define :"c6-server01" do |c6_server01_config|
    c6_server01_config.vm.box = "dmi3mis/centos6"
    c6_server01_config.vm.hostname = "c6-server01.ll-100.local"
    c6_server01_config.vm.network "private_network",type: "dhcp"
    c6_server01_config.vm.network :forwarded_port, guest: 22, host: 6040, id: "ssh"
    c6_server01_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c6_server01_memory
      vbox.name = "c6-server01.LL-100.local"
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
      if !File.exist?(vbox_vm_path + 'c6-server01_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c6-server01_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c6-server01_disk2.vdi']
    end
    c6_server01_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c6_server01_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end


  config.vm.define :"c7-client01" do |c7_client01_config|
    c7_client01_config.vm.box = "dmi3mis/centos7_desktop"
    c7_client01_config.vm.hostname = "c7-client01.ll-100.local"
    c7_client01_config.vm.network "private_network", ip: "192.168.2.40", auto_config: false
    c7_client01_config.vm.network :forwarded_port, guest: 22, host: 6050, id: "ssh"
    c7_client01_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.40/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_client01_config.vm.provision :shell, path: "scripts/localrepo"
    c7_client01_config.vm.provision :shell, path: "scripts/c7-client"
    c7_client01_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_client01_memory
      vbox.name = "c7-client01.LL-100.local"
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
      if !File.exist?(vbox_vm_path + 'c7-client01_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-client01_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-client01_disk2.vdi']
    end
    c7_client01_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_client01_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end
    
  config.vm.define :"c7-client02" do |c7_client02_config|
    c7_client02_config.vm.box = "dmi3mis/centos7_desktop"
    c7_client02_config.vm.hostname = "c7-client02.ll-100.local"
    c7_client02_config.vm.network "private_network", ip: "192.168.2.50", auto_config: false
    c7_client02_config.vm.network :forwarded_port, guest: 22, host: 6060, id: "ssh"
    c7_client02_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 192.168.2.50/24 ipv4.dns 192.168.2.254 ipv4.gateway 192.168.2.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    c7_client02_config.vm.provision :shell, path: "scripts/localrepo"
    c7_client02_config.vm.provision :shell, path: "scripts/c7-client"
    c7_client02_config.vm.provider "virtualbox" do |vbox, override|
      vbox.cpus = 1
      vbox.memory = c7_client02_memory
      vbox.name = "c7-client02.LL-100.local"
      #vbox.linked_clone = true
      vbox.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
      vbox.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
      vbox.customize ["modifyvm", :id, "--vram", "32"]
      vbox.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      vbox.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
      vbox.customize ["modifyvm", :id, "--usb", "on"]
      vbox.customize ["modifyvm", :id, "--usbehci", "on"]
      vbox.auto_nat_dns_proxy = false
      vbox.customize ["modifyvm", :id, "--usb", "on"]
      vbox.customize ["modifyvm", :id, "--usbehci", "on"]
      if !File.exist?(vbox_vm_path + 'c7-client02_disk2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'c7-client02_disk2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'c7-client02_disk2.vdi']
    end
    c7_client02_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = c7_client01_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  
  end

end
