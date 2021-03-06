IMAGE_NAME = "centos/7"
IP_MASTER = "192.168.219.181"
IP_CLIENT = "192.168.219."
N = 1

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  # ansible-cleint001
  (1..N).each do |i|
    config.vm.define "ansible-client#{i}" do |cfg|
      cfg.vm.box = IMAGE_NAME
      cfg.vm.network "public_network", ip: IP_CLIENT + "#{i+10}" 
      cfg.vm.network :forwarded_port, guest: 8888, host: 8888, id: 'jupyternotebook'
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
    cfg.vm.hostname = "ansible-server-2"
    cfg.vm.network "public_network", ip: IP_MASTER

    cfg.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.name =  "ansible-server-2"
    end
    cfg.vm.provision  "shell", inline: <<-SCRIPT
      yum install epel-release -y
      yum install python36 libselinux-python3 -y 
      yum install sshpass -y
      sudo pip3 install ansible
    SCRIPT

    # copy ansible files
    cfg.vm.provision "file", source: "./install_ansible", destination: "install_ansible"
    cfg.vm.provision "shell", inline: "ansible-playbook ./install_ansible/add_hosts.yaml", privileged: false
    cfg.vm.provision "shell", inline: "ansible-playbook ./install_ansible/configure_ssh.yaml -i /home/vagrant/hosts", privileged: false

    # install a jupyter notebook
    cfg.vm.provision "file", source: "./juypternotebook", destination: "juypternotebook"
    cfg.vm.provision "shell", inline: "ansible-playbook ./juypternotebook/site.yaml -i /home/vagrant/hosts", privileged: false
  end
end