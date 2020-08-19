<<<<<<< HEAD
IMAGE_NAME = "ubuntu/xenial64"
N = 3
=======
PROVIDER = "virtualbox"
BOX_OS = "ubuntu/xenial64"
NODE_COUNT = 3
DISK_COUNT = 2
DISK_SIZE_GB = 10   # in GB
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
                (1..DISK_COUNT).each do |diskID|
                    diskPath = "./tmp/node-#{i}-disk-#{diskID}.vdi"
                    if !File.exists?(diskPath)
                        provider.customize ['createhd', '--filename', diskPath, '--variant', 'Standard', '--size', DISK_SIZE_GB * 1024]
                        provider.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', diskID-1, '--device', 0, '--type', 'hdd', '--medium', diskPath]
                    end
                end
            end

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
