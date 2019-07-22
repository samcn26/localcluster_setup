# -*- mode: ruby -*-
# vi: set ft=ruby :
$mem = 1024
$cpu = 1
$master = 3
$worker = 2
$masterBaseName = "m"
$workerBaseName = "s"
$masterIP = "192.168.1."
$workerIP = "192.168.2."
$image = "centos/7"
$scripts = "scripts/install.sh"
$timezone = "Asia/Tokyo"

def getvm (name, ip)
    Vagrant.configure(2) do |config|
        config.vm.box = $image
        config.vm.define name do |config|
            config.vm.network "private_network", ip: ip
            config.vm.hostname = name
            config.vm.provision "shell", inline: "mkdir -p /root/.ssh /home/vagrant/.ssh"
            config.vm.provision "shell", run:"always" do |s|
              s.inline = <<-SHELL
                case #{$worker} in
                  0)
                    exit
                    ;;
                  1)
                    grep "#{$workerIP}101" /etc/hosts
                    if [ ! $? -eq 0 ]; then
                      echo #{$workerIP}101 #{$workerBaseName}1 | tee -a /etc/hosts
                    fi
                    ;;
                  *)
                    for x in $(seq 1 #{$worker}); do
                      ip=$(printf "%02d" ${x})
                      grep "#{$workerIP}1${ip}" /etc/hosts
                      if [ ! $? -eq 0 ]; then
                        echo #{$workerIP}1${ip} #{$workerBaseName}${x} | tee -a /etc/hosts
                      fi
                    done
                    ;;
                esac
                case #{$master} in
                  0)
                    exit
                    ;;
                  1)
                    grep "#{$masterIP}101" /etc/hosts
                    if [ ! $? -eq 0 ]; then
                      echo #{$masterIP}101 #{$masterBaseName}1 | tee -a /etc/hosts
                    fi
                    ;;
                  *)
                    for x in $(seq 1 #{$master}); do
                      ip=$(printf "%02d" ${x})
                      grep "#{$masterIP}1${ip}" /etc/hosts
                      if [ ! $? -eq 0 ]; then
                        echo #{$masterIP}1${ip} #{$masterBaseName}${x} | tee -a /etc/hosts
                      fi                      
                    done
                    ;;
                esac
              SHELL
            end
            if name[0,$masterBaseName.length] === $masterBaseName
                config.vm.provision "file", source: "~/.ssh/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
                config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"
                config.vm.provision "shell" do |s|
                    s.inline = <<-SHELL
                      chmod 400 /home/vagrant/.ssh/id_rsa
                    SHELL
                end
            end           
            config.vm.provision "shell" do |s|
                ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip               
                s.inline = <<-SHELL
                    # set timezone
                    timedatectl set-timezone  #{$timezone}
                    # allow ssh from host
                    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                    systemctl restart sshd
                    echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
                    echo #{ssh_pub_key} >> /root/.ssh/authorized_keys                                        
                SHELL
            end
            config.vm.provider "virtualbox" do |v|
                v.memory = $mem
                v.cpus = $cpu
            end
            config.vm.provision "shell", path: $scripts
        end
    end
end

for i in (1..$master)
    name = "#{$masterBaseName}#{i}"
    ip = "#{$masterIP}1%02d" % i
    getvm(name, ip)
end

for i in (1..$worker)
    name = "#{$workerBaseName}#{i}"
    ip = "#{$workerIP}1%02d" % i
    getvm(name, ip)
end