# -*- mode: ruby -*-
# vi: set ft=ruby :
$mem = 1024
$cpu = 1
$master = 1
$worker = 1
$masterIP = "192.168.1."
$workerIP = "192.168.2."
$image = "centos/7"
$scripts = "scripts/install.sh"

# for i in (1..$master)
#     puts "name: m#{i}"
#     ip = "%03d" % i
#     puts "ip: #$masterIP#{ip}"
#     puts "mem: #$mem"
#     puts "cpu: #$cpu"
# end

for i in (1..$master)
    name = "m#{i}"
    ip = "%03d" % i
    getvm(name, ip)
end

for i in (1..$worker)
    name = "s#{i}"
    ip = "%03d" % i
    getvm(name, ip)
end

def getvm (name, ip)
    Vagrant.configure(2) do |config|
        config.vm.box = $image
        config.vm.define name do |config|
            config.vm.network "private_network", ip: ip
            config.vm.hostname = name
            config.vm.provision "shell", path: $scripts
            config.vm.provider "virtualbox" do |v|
                v.memory = $mem
                v.cpus = $cpu
            end
        end
    end
end

