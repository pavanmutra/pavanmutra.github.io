## Ansible tutorial
### prerequesites
install vagrant and virtualbox

create Vagrant file with below content it will create 2 VM's  one ansible master and one node
```
vagrant plugin install vagrant-hostsupdater
```

```
Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/xenial64"
    master.vm.network "private_network", ip: "192.168.0.1"
  end
 
  config.vm.define "slave" do |slave|
    slave.vm.box = "ubuntu/xenial64"
    slave.vm.network "private_network", ip: "192.168.0.2"
    slave.vm.network "forwarded_port", guest: 80, host: 8080
   end
 
end
```

check status using vagrant status 

```
vagrant status
Current machine states:

master                    running (virtualbox)
slave                     running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
```
