# -*- mode: ruby -*-
# vi: set ft=ruby :

san_ip = "192.168.33.100"
san_iqn = "iqn.2019-04.com.accelstor:test"

cluster = {
  "node1" => { :ip => "192.168.33.11", :cpus => 1, :mem => 1536 },
  "node2" => { :ip => "192.168.33.12", :cpus => 1, :mem => 1536 }
}

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.vm.define :san do |san|
    san.vm.hostname = "san"
    san.vm.network "private_network", ip: san_ip
    san.vm.provider "virtualbox" do |vb|
      vb.name = "san"
      vb.memory = "1536"
    end
    san.vm.provision "shell", inline: <<-SHELL
      yum install -y epel-release
      yum install -y targetcli
      targetcli /backstores/ramdisk create volume 512m
      targetcli /iscsi create #{san_iqn}
      targetcli /iscsi/#{san_iqn}/tpg1 set attribute authentication=0
      targetcli /iscsi/#{san_iqn}/tpg1 set attribute generate_node_acls=1
      targetcli /iscsi/#{san_iqn}/tpg1 set attribute demo_mode_write_protect=0
      targetcli /iscsi/#{san_iqn}/tpg1/luns create /backstores/ramdisk/volume
    SHELL
  end

  cluster.each_with_index do |(hostname, info), index|

    config.vm.define hostname do |cfg|      
      cfg.vm.provider "virtualbox" do |vb, override|
        vb.name = hostname
        vb.memory = "#{info[:mem]}"
        vb.cpus = "#{info[:cpus]}"
        override.vm.hostname = hostname
        override.vm.network "private_network", ip: "#{info[:ip]}"
      end
      cfg.vm.provision "shell", inline: <<-SHELL
        yum install -y epel-release
        yum install -y iscsi-initiator-utils
        yum install -y rgmanager lvm2-cluster gfs2-utils
        yum groupinstall -y 'High Availability' 'Resilient Storage'
        iscsiadm -m node -o new -p #{san_ip} -T #{san_iqn}
        iscsiadm -m node -l
        systemctl enable pcsd
        systemctl start pcsd
        cp /etc/corosync/corosync.conf.example /etc/corosync/corosync.conf
        echo -e "myclusterpassword\nmyclusterpassword" | passwd hacluster
        pcs cluster auth #{info[:ip]} -u hacluster -p myclusterpassword
      SHELL
    end

  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   yum install -y epel-release
  # SHELL
end
