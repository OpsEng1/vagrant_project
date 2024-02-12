# Define VM configurations
box_name = "ubuntu/focal64"
host_memory = "512"
host_cpus = 1
controller_memory = "1024"
controller_cpus = 1
disk_size = "1GB"
controller_disk_size = "2GB"
ssh_pub_key = "/home/vagrant/.ssh/id_rsa.pub"
host1_ip = "192.168.59.11"
host2_ip = "192.168.59.12"
controller_ip = "192.168.59.10"
host1_name = "ansible-host1"
host2_name = "ansible-host2"
controller_name = "ansible-controller1"
host1_defined_name = "ansible-host1"
host2_defined_name = "ansible-host2"
controller_defined_name = "ansible-controller1"
# export VAGRANT_SSH_PASSWORD="XXXXXX" in .bashrc
ssh_password = ENV['VAGRANT_SSH_PASSWORD'] || ""

Vagrant.configure("2") do |config|
  # Define the base box and its URL
  config.vm.box = box_name
  config.vm.box_check_update = false

  # Define VM configurations for ansible-host1
  config.vm.define host1_defined_name do |host1|
    host1.vm.hostname = host1_name
    host1.vm.network "private_network", ip: host1_ip
    host1.vm.provider "virtualbox" do |vb|
      vb.name = host1_defined_name
      vb.memory = host_memory
      vb.cpus = host_cpus
    end
    host1.vm.disk :disk, name: "disk", size: disk_size
    host1.vm.provision "shell", inline: <<-SHELL
      # Enable PubkeyAuthentication and PasswordAuthentication in sshd_config
      sudo sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
      sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

      # Restart SSH service to apply changes
      sudo systemctl restart sshd
    SHELL
  end

  # Define VM configurations for ansible-host2
  config.vm.define host2_defined_name do |host2|
    host2.vm.hostname = host2_name
    host2.vm.network "private_network", ip: host2_ip
    host2.vm.provider "virtualbox" do |vb|
      vb.name = host2_defined_name
      vb.memory = host_memory
      vb.cpus = host_cpus
    end
    host2.vm.disk :disk, name: "disk", size: disk_size
    host2.vm.provision "shell", inline: <<-SHELL
      # Enable PubkeyAuthentication and PasswordAuthentication in sshd_config
      sudo sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
      sudo sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

      # Restart SSH service to apply changes
      sudo systemctl restart sshd
    SHELL
  end

  # Define VM configurations for ansible-controller1
  config.vm.define controller_defined_name do |ansible|
    ansible.vm.hostname = controller_name
    ansible.vm.network "private_network", ip: controller_ip
    ansible.vm.provider "virtualbox" do |vb|
      vb.name = controller_defined_name # What is displayed on Virtualbox
      vb.memory = controller_memory
      vb.cpus = controller_cpus
    end
    ansible.vm.disk :disk, name: "disk", size: controller_disk_size

    ansible.vm.provision "shell", inline: <<-SHELL
      # Update apt package index
      sudo apt-get update

      # Install dependencies
      sudo apt-get install -y software-properties-common

      # Add Ansible PPA
      sudo apt-add-repository --yes --update ppa:ansible/ansible

      # Install Ansible
      sudo apt-get install -y ansible

      ssh-keygen -t rsa -N "" -f /home/vagrant/.ssh/id_rsa
      sudo chmod 644 /home/vagrant/.ssh/id_rsa
      sudo chmod 644 /home/vagrant/.ssh/id_rsa.pub
      sshpass -p "#{ssh_password}" scp -o StrictHostKeyChecking=no #{ssh_pub_key} vagrant@#{host1_ip}:/home/vagrant/id_rsa.pub
      sshpass -p "#{ssh_password}" scp -o StrictHostKeyChecking=no #{ssh_pub_key} vagrant@#{host2_ip}:/home/vagrant/id_rsa.pub
      sshpass -p "#{ssh_password}" ssh -o StrictHostKeyChecking=no vagrant@#{host1_ip} 'cat /home/vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys'
      sshpass -p "#{ssh_password}" ssh -o StrictHostKeyChecking=no vagrant@#{host2_ip} 'cat /home/vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys'

      # Automatically add hosts to known_hosts
      sudo ssh-keyscan -H #{host1_ip} >> /home/vagrant/.ssh/known_hosts
      sudo ssh-keyscan -H #{host2_ip} >> /home/vagrant/.ssh/known_hosts
    SHELL
  end
end
