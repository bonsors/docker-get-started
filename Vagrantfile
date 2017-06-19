Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/xenial64"
    master.vm.hostname = 'master'

    master.vm.network :private_network, ip: "192.168.56.101"
    master.vm.network "forwarded_port", guest: 80, host: 80, host_ip:"192.168.56.101"
    master.vm.network "forwarded_port", guest: 8080, host: 8080, host_ip:"192.168.56.101"
    master.vm.network "forwarded_port", guest: 8081, host: 8081, host_ip:"192.168.56.101"

    master.vm.network "forwarded_port", guest: 2377, host: 2377, host_ip:"192.168.56.101"
    master.vm.network "forwarded_port", guest: 7946, host: 7946, host_ip:"192.168.56.101"
    master.vm.network "forwarded_port", guest: 4789, host: 4789, host_ip:"192.168.56.101"
	
    master.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", 512]
      v.customize ["modifyvm", :id, "--name", "master"]
    end
    master.vm.provision "shell", inline: <<-SHELL
      apt-get update
      sudo apt-get -y install \
        apt-transport-https \
          ca-certificates \
          curl
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
         $(lsb_release -cs) \
         stable"	
      apt-get update
      sudo apt-get -y install docker-ce
      sudo groupadd docker
      sudo usermod -aG docker ubuntu
      sudo systemctl enable docker
      sudo docker swarm init --advertise-addr 192.168.56.101:2377
      sudo docker swarm join-token -q manager > /vagrant/tokenmaster
      sudo docker swarm join-token -q worker > /vagrant/tokenworker
    SHELL
  end

  (1..3).each do |machine_id|
	  config.vm.define "slave#{machine_id}" do |slave|
		slave.vm.box = "ubuntu/xenial64"
		slave.vm.hostname = "slave#{machine_id}"
		
		slave.vm.network :private_network, ip: "192.168.56.#{110+machine_id}"
		slave.vm.network "forwarded_port", guest: 2377, host: 2377, host_ip:"192.168.56.#{110+machine_id}"
		slave.vm.network "forwarded_port", guest: 7946, host: 7946, host_ip:"192.168.56.#{110+machine_id}"
		slave.vm.network "forwarded_port", guest: 4789, host: 4789, host_ip:"192.168.56.#{110+machine_id}"
		slave.vm.provider :virtualbox do |v|
		  v.customize ["modifyvm", :id, "--memory", 512]
		  v.customize ["modifyvm", :id, "--name", "slave#{machine_id}"]
		end
		slave.vm.provision "shell", inline: <<-SHELL
                  apt-get update
                  sudo apt-get -y install \
                    apt-transport-https \
                      ca-certificates \
                      curl
                  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                  sudo add-apt-repository \
                    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
                     $(lsb_release -cs) \
                     stable"	
                  apt-get update
                  sudo apt-get -y install docker-ce
                  sudo groupadd docker
                  sudo usermod -aG docker ubuntu
                  sudo systemctl enable docker
                  sudo docker swarm join --token `cat /vagrant/tokenworker` 192.168.56.101
		SHELL
	  end
	end
end
