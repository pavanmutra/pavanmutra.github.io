## Ansible tutorial
### prerequesites
install vagrant and virtualbox

create Vagrant file with below content it will create 2 VM's  one ansible master and one node
```
vagrant plugin install vagrant-hostsupdater
```

```
Vagrant.configure("2") do |config|
  servers=[
    {
      :hostname => "database",
      :box => "centos/7",
      :ip => "192.168.56.101",
      :ssh_port => '2210'
    },
    {
      :hostname => "webserver-1",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.102",
      :ssh_port => '2211'
    },
    {
      :hostname => "webserver-2",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.103",
      :ssh_port => '2212'
    },
    {
      :hostname => "loadbalancer",
      :box => "bento/ubuntu-18.04",
      :ip => "192.168.56.104",
      :ssh_port => '2213'
    },
    {
      :hostname => "queue",
      :box => "centos/7",
      :ip => "192.168.56.105",
      :ssh_port => '2214'
    },
    {
      :hostname => "control",
      :box => "centos/7",
      :ip => "192.168.56.106",
      :ssh_port => '2215'
    }

  ]

  servers.each do |machine|

    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.hostname = machine[:hostname]
    
      node.vm.network :private_network, ip: machine[:ip]
      node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"

      node.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--memory", 512]
        v.customize ["modifyvm", :id, "--name", machine[:hostname]]
      end
    end
  end

  id_rsa_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))

  config.vm.provision :shell,
        :inline => "echo 'appending SSH public key to ~vagrant/.ssh/authorized_keys' && echo '#{id_rsa_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"

  config.ssh.insert_key = false
end
```

check status using vagrant status 

```
vagrant status
Current machine states:

database                  running (virtualbox)
webserver-1               running (virtualbox)
webserver-2               running (virtualbox)
loadbalancer              running (virtualbox)
queue                     running (virtualbox)
control                   running (virtualbox)
This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
```
