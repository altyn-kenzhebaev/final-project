# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME'] 

MACHINES = {
    :nginx => {
        :box_name => "almalinux/9",
        :ip_addr => '192.168.50.100',
        :net_adp => 3,
        :cpus => 2,
        :memory => 1024,
        :nets => {
            :net2 => {
              :ip => '10.10.0.10',
              :adapter => 2,
              :netmask => "255.255.255.0",
              :virtualbox__intnet => "localnet"
            },
          }
    },
    :mysql => {
        :box_name => "almalinux/9",
        :cpus => 2,
        :memory => 1024,
        :disks => {
          :sata1 => {
              :dfile => home + '/VirtualBox VMs/mysql/sata1.vdi',
              :size => 768,
              :port => 1
          },
      },
        :nets => {
            :net2 => {
              :ip => '10.10.0.110',
              :adapter => 2,
              :netmask => "255.255.255.0",
              :virtualbox__intnet => "localnet"
            },
          }
    },
    :pcmk1 => {
        :box_name => "almalinux/9",
        :cpus => 2,
        :memory => 1536,
        :disks => {
            :sata1 => {
                :dfile => home + '/VirtualBox VMs/pcmk1/sata1.vdi',
                :size => 1024,
                :port => 1
            },
        },
        :nets => {
            :net2 => {
              :ip => '10.10.0.101',
              :adapter => 2,
              :netmask => "255.255.255.0",
              :virtualbox__intnet => "localnet"
            },
          }
    },
    :pcmk2 => {
        :box_name => "almalinux/9",
        :cpus => 2,
        :memory => 1536,
        :disks => {
            :sata1 => {
                :dfile => home + '/VirtualBox VMs/pcmk2/sata1.vdi',
                :size => 1024,
                :port => 1
            },
        },
        :nets => {
            :net2 => {
              :ip => '10.10.0.102',
              :adapter => 2,
              :netmask => "255.255.255.0",
              :virtualbox__intnet => "localnet"
            },
          }
    },
    :syslog => {
        :box_name => "almalinux/9",
        :cpus => 2,
        :memory => 4096,
        :nets => {
            :net2 => {
              :ip => '10.10.0.200',
              :adapter => 2,
              :netmask => "255.255.255.0",
              :virtualbox__intnet => "localnet"
            },
        }
    },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
      boxconfig[:nets].each do |netname, netconf|
        box.vm.network "private_network", 
        ip: netconf[:ip], 
        adapter: netconf[:adapter], 
        netmask: netconf[:netmask], 
        virtualbox__intnet: netconf[:virtualbox__intnet]
      end
      
      if boxname.to_s == 'nginx'
        box.vm.network "private_network", adapter: boxconfig[:net_adp], ip: boxconfig[:ip_addr]
      end
 
      box.vm.provider :virtualbox do |vb|
        vb.name = boxname.to_s
        vb.memory = boxconfig[:memory]
        vb.cpus = boxconfig[:cpus]
        #if (vb.name != "nginx") && (vb.name != "syslog")
        #if ["pcmk1", "pcmk2", "mysql"].include?(vb.name)
        if !["nginx", "syslog"].include?(vb.name)
            boxconfig[:disks].each do |dname, dconf|
              unless File.exist?(dconf[:dfile])
                vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
              end
              vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
            end
        end
      end
    end
  end
end
