VAGRANT_IMAGE_NAME = "ubuntu/focal64"

K8S_MASTER_NODES = 2
K8S_WORKER_NODES = 2

K8S_MASTER_IP_START = 10
K8S_WORKER_IP_START = 20
K8S_LB_IP_START = 30
DB_IP_START = 50
NGINX_IP_START = 80

PRIVATE_IP_NW = "10.10.10."

DOMAIN_NAME = "guestbook.shongololo.xyz"

Vagrant.configure("2") do |config|
    config.vm.box = VAGRANT_IMAGE_NAME
    config.vm.box_check_update = false
    config.ssh.insert_key = false

    # Provision Load Balancer to make Master Nodes Highly Available
    config.vm.define "k8s-lb" do |lb|
        lb.vm.provider "virtualbox" do |vb|
            vb.name = "k8s-lb"
            vb.memory = 512
            vb.cpus = 1
            vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
            vb.customize ["modifyvm", :id, "--nested-hw-virt","on"]
            vb.customize ["modifyvm", :id, "--cpuhotplug","on"]
            vb.customize ["modifyvm", :id, "--audio", "none"]
            vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
            vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
            vb.customize ["modifyvm", :id, "--nictype3", "virtio"]
        end
        lb.vm.hostname = "k8s-lb"
        lb.vm.network :private_network, ip: PRIVATE_IP_NW + "#{K8S_LB_IP_START}"
        lb.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/playbooks/k8s_lb.yml"
            ansible.extra_vars = {
                node_ip: PRIVATE_IP_NW + "#{K8S_LB_IP_START}",
            }
        end
    end

    # Provision Master Nodes
    (1..K8S_MASTER_NODES).each do |i|
        config.vm.define "k8s-master-#{i}" do |node|
            # Name shown in the GUI
            node.vm.provider "virtualbox" do |vb|
                vb.name = "k8s-master-#{i}"
                vb.memory = 2048
                vb.cpus = 2
                vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
                vb.customize ["modifyvm", :id, "--nested-hw-virt","on"]
                vb.customize ["modifyvm", :id, "--cpuhotplug","on"]
                vb.customize ["modifyvm", :id, "--audio", "none"]
                vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
                vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
                vb.customize ["modifyvm", :id, "--nictype3", "virtio"]
            end
            node.vm.hostname = "k8s-master-#{i}"
            node.vm.network :private_network, ip: PRIVATE_IP_NW + "#{K8S_MASTER_IP_START + i}"
            node.vm.provision "ansible" do |ansible|
                if i == 1
                    ansible.playbook = "ansible/playbooks/k8s_master_primary.yml"
                else
                    ansible.playbook = "ansible/playbooks/k8s_master_secondary.yml"
                end
                ansible.extra_vars = {
                    node_ip: PRIVATE_IP_NW + "#{K8S_MASTER_IP_START + i}",
                }
            end
        end
    end

    # Provision Worker Nodes
    (1..K8S_WORKER_NODES).each do |i|
        config.vm.define "k8s-worker-#{i}" do |node|
            node.vm.provider "virtualbox" do |vb|
                vb.name = "k8s-worker-#{i}"
                vb.memory = 512
                vb.cpus = 1
                vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
                vb.customize ["modifyvm", :id, "--nested-hw-virt","on"]
                vb.customize ["modifyvm", :id, "--cpuhotplug","on"]
                vb.customize ["modifyvm", :id, "--audio", "none"]
                vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
                vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
                vb.customize ["modifyvm", :id, "--nictype3", "virtio"]
            end
            node.vm.hostname = "k8s-worker-#{i}"
            node.vm.network :private_network, ip: PRIVATE_IP_NW + "#{K8S_WORKER_IP_START + i}"
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "ansible/playbooks/k8s_worker.yml"
                ansible.extra_vars = {
                    node_ip: PRIVATE_IP_NW + "#{K8S_WORKER_IP_START + i}",
                }
            end
        end
    end

    # Provision Database server to persist data
    config.vm.define "db" do |db|
        db.vm.provider "virtualbox" do |vb|
            vb.name = "db"
            vb.memory = 512
            vb.cpus = 1
            vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
            vb.customize ["modifyvm", :id, "--nested-hw-virt","on"]
            vb.customize ["modifyvm", :id, "--cpuhotplug","on"]
            vb.customize ["modifyvm", :id, "--audio", "none"]
            vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
            vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
            vb.customize ["modifyvm", :id, "--nictype3", "virtio"]
        end
        db.vm.hostname = "db"
        db.vm.network :private_network, ip: PRIVATE_IP_NW + "#{DB_IP_START}"
        db.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/playbooks/db.yml"
            ansible.extra_vars = {
                node_ip: PRIVATE_IP_NW + "#{DB_IP_START}",
            }
        end
    end

    # Provision nginx reverse proxy to do SSL termination
    config.vm.define "nginx" do |nginx|
        nginx.vm.provider "virtualbox" do |vb|
            vb.name = "nginx"
            vb.memory = 512
            vb.cpus = 1
            vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
            vb.customize ["modifyvm", :id, "--nested-hw-virt","on"]
            vb.customize ["modifyvm", :id, "--cpuhotplug","on"]
            vb.customize ["modifyvm", :id, "--audio", "none"]
            vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
            vb.customize ["modifyvm", :id, "--nictype2", "virtio"]
            vb.customize ["modifyvm", :id, "--nictype3", "virtio"]
        end
        nginx.vm.hostname = "nginx"
        nginx.vm.network :private_network, ip: PRIVATE_IP_NW + "#{NGINX_IP_START}"
        nginx.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/playbooks/nginx.yml"
            ansible.extra_vars = {
                node_ip: PRIVATE_IP_NW + "#{NGINX_IP_START}",
                domain_name: DOMAIN_NAME,
            }
        end
    end

end
