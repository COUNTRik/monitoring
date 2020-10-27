# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :monitor => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.150'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "200"]
          end
          
          # box.vm.provision "shell", path: "scripts/monitoring.sh"

          box.vm.provision "ansible" do |ansible|
            ansible.playbook = "playbooks/monitor.yml"
          end

      end
  end
end