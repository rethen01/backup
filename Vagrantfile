# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :bserver => {
        :box_name => "centos/7",
        :box_version => "2004.01",
    :disks => {
        :sata01 => {
            :dfile => '/home/rethen/Desktop/backup/sata01.vdi',
            :size => 2048,
            :port => 1
        },
    }
        
  },
}

Vagrant.configure("2") do |config|
 
  MACHINES.each do |boxname, boxconfig|
      config.vm.define "bclient" do |bclient|
       bclient.vm.box = "centos/7" 
       bclient.vm.network "private_network", ip: "192.168.56.150"
       bclient.vm.host_name = "client"
      
      config.vm.provision "shell", path: "script.sh"
      end

      config.vm.define boxname do |box|
 
        box.vm.box = boxconfig[:box_name]
        box.vm.box_version = boxconfig[:box_version]
        box.vm.network "private_network", ip: "192.168.56.160"     
        box.vm.host_name = "server"
 
        box.vm.provider :virtualbox do |vb|
              vb.customize ["modifyvm", :id, "--memory", "1024"]
              needsController = false
        boxconfig[:disks].each do |dname, dconf|
              unless File.exist?(dconf[:dfile])
              vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
         needsController =  true
         end
        end
            if needsController == true
                vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                boxconfig[:disks].each do |dname, dconf|
                vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                end
             end
          end
      config.vm.provision "shell", path: "script.sh"
  
    end
  end
end