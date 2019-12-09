## Ansible tutorial
### prerequesites
install vagrant and virtualbox

create Vagrant file with below content it will create 2 VM's  one ansible master and one node
```
vagrant plugin install vagrant-hostsupdater
```

```go
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

## Ansible Adhoc commands
```
ansible host-pattern -m module [-a 'module arguments'] [-i inventory]
```
## Ansible commanline options

```
inventory 			-i
remote_user			-u
become 				--become, -b
become_method		--become-method
become_user			-u
become_ask_password	--ask-become-pass, -K
```
## Yaml 
YAML(YAML Ain’t Markup Language) gives a even simpler way than JSON to represent the data. It removes curly brackets({}) and squard brackets([]) except for inline collections. In addition, vertical alignment is used to show the structure. Technically, YAML is a superset of JSON. In most cases, it is possible to convert YAML to JSON and JSON to YAML. YAML also has extra features, which are not in JSON.

* Commenting: Adding additional information as a comment
* Aliasing and Anchoring: Giving an alias to an item and referring it instead of typing/repeating many times
* Merging: Merging aliased item into a current map

```yaml
---
- foo
- bar
-
  - baz
  - qux

```
or
```yaml

---
- foo
- bar : [baz, qux]
```

```json
[
  "foo",
  "bar",
  [
    "baz",
    "qux"
  ]
]
```

## Ansible playboooks

### Ansible modules 

modules are ready to use tools designed to do specific operations

when we run modules are copied to managed hosts and executed there

to get list of modules 
```
ansible-doc -l
```
to get sample snippets
```
ansible-doc -s yum
```
sample for adding ansible user in all hosts

```yaml
- name: add ansible user
  hosts: all
  become: true
  become_method: sudo
  become_user: root
  tasks:
    - user:
        groups: ansible
        name: ansible
```
```
```
```
```
