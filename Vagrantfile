Vagrant.configure("2") do |config|
	# Set vagrant box
	config.vm.box = "ubuntu/bionic64"

	# Build 2 backends
	(1..2).each do |be|
		config.vm.define "backend_#{be}" do |backend|
			backend.vm.provider :virtualbox do |vb|
				vb.name = "backend_#{be}"
			end
			backend.vm.network :private_network, ip: "10.99.25.2#{be}"
			backend.vm.hostname = "backend-#{be}"
		end
	end

	# Build loadbalancer
	config.vm.define "loadbalancer" do |loadbalancer|
		loadbalancer.vm.provider :virtualbox do |vb|
			vb.name = "loadbalancer"
		end
		loadbalancer.vm.network :private_network, ip: "10.99.25.10"
		loadbalancer.vm.hostname = "loadbalancer"
	end

	# Run Ansible
	config.vm.provision "ansible" do |ansible|
		ansible.playbook = "ansible/autoweb.yml"
		#ansible.verbose = true
		ansible.groups = {
			"backends" => ["backend_[1:2]"],
			"loadbalancers" => ["loadbalancer"],
			"all" => ["backend_[1:2]", "loadbalancer"]
		}
	end
end