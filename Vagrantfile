$install_docker_script = <<SCRIPT
echo Installing Docker...
curl -sSL https://get.docker.com/ | sh
usermod -aG docker ubuntu
cat /vagrant/id_rsa.pub >> /home/ubuntu/.ssh/authorized_keys
SCRIPT
 
$manager_script = <<SCRIPT
echo Swarm Init...
docker swarm init --listen-addr 10.100.199.200:2377 --advertise-addr 10.100.199.200:2377
docker swarm join-token --quiet worker > /vagrant/worker_token
curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
docker stack deploy -c portainer-agent-stack.yml portainer
SCRIPT
 
$worker_script = <<SCRIPT
echo Swarm Join...
docker swarm join --token $(cat /vagrant/worker_token) 10.100.199.200:2377
SCRIPT
 
Vagrant.configure('2') do |config|
 
  vm_box = 'ubuntu/focal64'
 
 
  config.vm.define :manager, primary: true  do |manager|
    manager.vm.box = vm_box
    manager.vm.box_check_update = true
	manager.vm.disk :disk, size: "100GB", primary: true
    manager.vm.network :private_network, ip: "10.100.199.200"
    manager.vm.network :forwarded_port, guest: 8080, host: 8080
    manager.vm.network :forwarded_port, guest: 9443, host: 9443
	manager.vm.network :forwarded_port, guest: 9000, host: 9000
    manager.vm.hostname = "manager"
    manager.vm.synced_folder ".", "/vagrant"
    manager.vm.provision "shell", inline: $install_docker_script, privileged: true
    manager.vm.provision "shell", inline: $manager_script, privileged: true
    manager.vm.provider "virtualbox" do |vb|
      vb.name = "manager"
      vb.memory = "2048"
    end
  end
 
  (1..3).each do |i|
    config.vm.define "worker0#{i}" do |worker|
      worker.vm.box = vm_box
      worker.vm.box_check_update = true
	  worker.vm.disk :disk, size: "50GB", primary: true
      worker.vm.network :private_network, ip: "10.100.199.20#{i}"
      worker.vm.hostname = "worker0#{i}"
      worker.vm.synced_folder ".", "/vagrant"
      worker.vm.provision "shell", inline: $install_docker_script, privileged: true
      worker.vm.provision "shell", inline: $worker_script, privileged: true
      worker.vm.provider "virtualbox" do |vb|
        vb.name = "worker0#{i}"
        vb.memory = "2048"
      end
    end
  end
 
end