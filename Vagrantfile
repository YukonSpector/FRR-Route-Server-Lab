VAGRANTFILE_API_VERSION = "2"
INTERNAL_NET_ONE = "100.100.100."
DOMAIN = ".sample.com"
BOX = "ubuntu/bionic64"
HWVIRT = "on"
CLIENT_COUNT = 4
CLIENT_RAM = "512"
CLIENT_CPU_COUNT = "1"
CLIENT_CPU_CAP = "50"
CLIENT_ANSIBLE = "route-client.yaml"

servers=[
  {
    :hostname => "routeServer1",
    :peer_ip => INTERNAL_NET_ONE + "100",
    :ram => "512",
    :cpu_count => "1",
    :cpu_cap => "50",
    :playbook => "route-server-1.yaml"
  }
]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.box = BOX
            node.vm.usable_port_range = (2200..2210)
            node.vm.hostname = machine[:hostname] + DOMAIN
            node.vm.network "private_network", ip: machine[:peer_ip], virtualbox__intnet: "peering_fabric"
            node.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
                vb.customize ["modifyvm", :id, "--cpus", machine[:cpu_count]]
                vb.customize ["modifyvm", :id, "--cpuexecutioncap", machine[:cpu_cap]]
                vb.customize ["modifyvm", :id, "--hwvirtex", HWVIRT]
            end
            if (!machine[:playbook].nil?)
                node.vm.provision "ansible" do |ansible|
                    ansible.playbook = machine[:playbook]
                end
            end
        end
    end
    (1..CLIENT_COUNT).each do |i|
        config.vm.define "routeClient#{i}" do |client|
            client.vm.box = BOX
            client.vm.usable_port_range = (2211..2220)
            client.vm.hostname = "routeClient" + "routeClient#{i}" + DOMAIN
            client.vm.network "private_network", ip: INTERNAL_NET_ONE + "#{i}", virtualbox__intnet: "peering_fabric"
            client.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory", CLIENT_RAM]
                vb.customize ["modifyvm", :id, "--cpus", CLIENT_CPU_COUNT]
                vb.customize ["modifyvm", :id, "--cpuexecutioncap", CLIENT_CPU_CAP]
                vb.customize ["modifyvm", :id, "--hwvirtex", HWVIRT]
            end
            client.vm.provision "ansible" do |ansible|
                ansible.playbook = CLIENT_ANSIBLE
            end
        end
    end
end