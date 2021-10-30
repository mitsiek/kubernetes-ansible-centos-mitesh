require 'getoptlong'

Vagrant.require_version ">= 1.8.1"

tags=Array.new
numberOfVms=4
vmMemory=2048

sshKey="#{Dir.home}/.ssh/id_rsa.pub"

Vagrant.configure("2") do |config|
  config.vm.box_check_update = "false"

  # Provisioning kubernetes nodes
  (1..numberOfVms).each do |i|
    config.vm.define "node#{i}" do |kubernetes|
      kubernetes.vm.box = "centos/7"
      kubernetes.vm.hostname = "node#{i}"
      kubernetes.vm.network "private_network", ip: "192.168.120.1#{i}"

      kubernetes.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{sshKey}").first.strip
        s.inline = <<-SHELL
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          mkdir /root/.ssh && chmod 700 /root/.ssh
          touch /root/.ssh/authorized_keys && chmod 600 /root/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      end
      
      kubernetes.vm.provision "shell" do |s|
        s.inline = "ifup eth1"
      end

      kubernetes.vm.provider "virtualbox" do |v|
        v.name = "node#{i}"
        v.cpus = 2
        v.memory = vmMemory
      end
    end
  end

  # Provisioning ansible controller machine
  config.vm.define "ansible" do |ansible|
    ansible.vm.box = "centos/7"
    ansible.vm.hostname = "ansible"
    ansible.vm.network "private_network", ip: "192.168.120.10"

    ansible.vm.provision "shell" do |s|
      s.inline = <<-SHELL
        yum update -y
        yum install epel-release -y
        yum install ansible -y
      SHELL
    end

    ansible.vm.provision "file", source: "ansible", destination: "$HOME/ansible"

    ansible.vm.provider "virtualbox" do |v|
        v.name = "ansible"
        v.memory = "1024"
        v.cpus = "2"
    end
  end
end
