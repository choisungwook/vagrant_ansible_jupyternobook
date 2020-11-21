INTNET = "ansible-cluster"
IMAGE_NAME = "centos/7"
IP_MASTER = "192.168.50.200"
IP_CLIENT = "192.168.50."
N = 1

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  # ansible-cleint001
  (1..N).each do |i|
    config.vm.define "ansible-client#{i}" do |cfg|
      cfg.vm.box = IMAGE_NAME
      cfg.vm.network "private_network", ip: IP_CLIENT + "#{i+10}" , virtualbox__intnet: INTNET
      cfg.vm.hostname = "ansible-client#{i}"
      
      cfg.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
        v.name = "ansible-client#{i}"
      end
      cfg.vm.provision "shell", inline: <<-SCRIPT
        sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SCRIPT
    end
  end

  # ansible-server
  config.vm.define "ansible-server" do |cfg|
    cfg.vm.box = IMAGE_NAME
    cfg.vm.hostname = "ansible-server"
    cfg.vm.network "private_network", ip: IP_MASTER, virtualbox__intnet: INTNET

    cfg.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.name =  "ansible-server"
    end
    cfg.vm.provision  "shell", inline: <<-SCRIPT
      yum install epel-release -y
      yum install ansible -y
    SCRIPT

    # copy ansible files
    cfg.vm.provision "file", source: "./install_ansible", destination: "install_ansible"
    cfg.vm.provision "shell", inline: "ansible-playbook ./install_ansible/add_hosts.yaml"
    cfg.vm.provision "shell", inline: "ansible-playbook ./install_ansible/configure_ssh.yaml", privileged: false
  end
end