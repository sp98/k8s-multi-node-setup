<<<<<<< HEAD
IMAGE_NAME = "ubuntu/xenial64"
N = 3
=======
PROVIDER = "virtualbox"
BOX_OS = "centos/7"
NODE_COUNT = 3
DISK_COUNT = 1
DISK_SIZE = 10   # in GB
MASTER_HOST_NAME = "master"
NODE_HOST_NAME = "worker"
DISK = "disk"

#PROVIDER = ENV['PROVIDER'].to_s.strip.empty? ? 'virtualbox'.freeze : ENV['PROVIDER']
#BOX_OS = ENV['BOX_OS'].to_s.strip.empty? ? 'ubuntu/xenial64'.freeze : ENV['BOX_OS']
>>>>>>> first commit

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
    end

    # Configure master
    config.vm.define MASTER_HOST_NAME do |master|
        master.vm.box = BOX_OS
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = MASTER_HOST_NAME
        master.vm.provider "virtualbox" do |v|
            v.memory = 1024
            v.cpus = 2
        end
        master.vm.provision "ansible" do |ansible|
            ansible.playbook = "master-playbook.yaml"
            ansible.extra_vars = {
                node_ip: "192.168.50.10",
                master_host: MASTER_HOST_NAME
            }
        end
    end

    # Configure Nodes
    (1..NODE_COUNT).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = BOX_OS
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "node-#{i}"

            #adding disks
            node.vm.provider "virtualbox" do |provider|
                if File.exist?(".vagrant/node#{i}-disk-1.vdi")
                    provider.customize ['storagectl', :id, '--name', 'SATAController', '--remove']
                end
                provider.customize ['storagectl', :id, '--name', 'SATAController', '--add', 'sata']

                (1..DISK_COUNT).each do |diskID|
                    if !File.exists?(".vagrant/node#{i}-disk-#{diskID}.vdi")
                        provider.customize ['createhd', '--filename', ".vagrant/node#{i}-disk-#{diskID}.vdi",  '--variant', 'Standard', '--size', DISK_SIZE * 1024]
                    end
                        provider.customize ['storageattach', :id,  '--storagectl', 'SATAController', '--port', diskID-1, '--device', 0, '--type', 'hdd', '--medium', ".vagrant/node#{i}-disk-#{diskID}.vdi"]
                end
            end

            # provision nodes
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "node-playbook.yaml"
                ansible.extra_vars = {
                    node_ip: "192.168.50.#{i + 10}",
                    node_host: node.vm.hostname
                }
            end
        end
    end
end
