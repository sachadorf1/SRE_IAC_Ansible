# Infrastructure As Code (IAC)
## Iac - Configuration Management and Orchestration
### Which tools are used for push config and pull config

- What is Ansible and benefits of it?
- Why should we use Ansible?
- Create a Diagram for Ansible on prem, Hybrid and public architecture
- Step by step guide for installation and setting up Ansible controller with 2 agent nodes - include commands in your README.md
- What is the default Directory structure for Ansible
- What is the Inventory/hosts file and the purpose of it?
- What should be added to hosts file to establish secure connection between ANsible controller and agent nodes? (include code block)
- What are Ansible `Ad-hoc commands`?
- Add a structure of creating adhoc commands `ansible all -m ping` (m module)
- include all the adhoc commands we have used today in this documentation


## We will use 18.04 ubuntu for ansible controller and agent nodes set up 
### Please ensure to refer back to your vagrant documentation

- **You may need to reinstall plugins or dependencies required depending on the OS you are using.**

```vagrant 
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what

# MULTI SERVER/VMs environment 
#
Vagrant.configure("2") do |config|

# creating first VM called web  
  config.vm.define "web" do |web|
    
    web.vm.box = "bento/ubuntu-18.04"
   # downloading ubuntu 18.04 image

    web.vm.hostname = 'web'
    # assigning host name to the VM
    
    web.vm.network :private_network, ip: "192.168.33.10"
    #   assigning private IP
    
    config.hostsupdater.aliases = ["development.web"]
    # creating a link called development.web so we can access web page with this link instread of an IP   
        
  end
  
# creating second VM called db
  config.vm.define "db" do |db|
    
    db.vm.box = "bento/ubuntu-18.04"
    
    db.vm.hostname = 'db'
    
    db.vm.network :private_network, ip: "192.168.33.11"
    
    config.hostsupdater.aliases = ["development.db"]     
  end

 # creating are Ansible controller
  config.vm.define "controller" do |controller|
    
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    config.hostsupdater.aliases = ["development.controller"] 
    
  end

end
```


## Ansible controller and agent nodes set up guide
- Clone this repo and run `vagrant up`
- `(double check syntax/intendation)`

- Comment out the first line
- Comment out this line: config.hostsupdater.aliases=... in each machine section of the Vagrantfile.
- Destroy any remaining machines that could be running and `vagrant up`
```
vagrant destroy
vagrant up
```

- SSH into the 3 machines and run update and upgrade on each of them
```
vagrant ssh app
sudo apt-get update -y
sudo apt-get upgrade -y

vagrant ssh app
sudo apt-get update -y
sudo apt-get upgrade -y

vagrant ssh app
sudo apt-get update -y
sudo apt-get upgrade -y
```

- SSH into the controller machine

```
vagrant ssh controller
```

Run the following commands:
```
sudo apt-get update
sudo apt-get install software-properties-common -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible -y
```
- Check that it has been installed properly
```
`ansible --version`
```
`ssh vagrant@192.168.33.10`
- yes
- enter password (it will not show you what you enter)

`sudo apt-get update -y`

`exit`

`ssh vagrant@192.168.33.11`
-yes
- enter password

`sudo apt-get update -y`

exit

- Will take you back to the controller machine

ansible all -m ping

(all is for all machines, can change for web or db)

cd /etc/ansible/

sudo nano hosts
```
[web]
192.168.33.10
```

`ansible all -m ping`

`sudo ansible all -m ping`

`sudo nano hosts`
```
[web]
192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
```

`ansible all -m ping`

- Should have "ping": "pong"

`sudo ansible web -m ping`

- Should have "ping": "pong"

`sudo ansible db -m ping`

- Should say cannot match

`sudo nano hosts`
```
[web]
192.168.33.10 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
[db]
192.168.33.11 ansible_connection=ssh ansible_user=vagrant ansible_ssh_pass=vagrant
```

`ansible db -m ping`
- (We don't need sudo)
- Should have "ping": "pong"

`ansible web -a "uname -a"`
- Gives info about machine

`ansible db -a "uname -a"`

`ansible all -a "uname -a"`

`ansible all -a "date"`

`ansible all -a "free"`
- Shows us the free space inside our machines

`ansible all -a "pwd"`
`ansible all -a "ls -a"`


### Ansible playbooks
- Ansible playbooks are a completely different way to use Ansible than ad-hoc task execution
- Yet Another Markup Language (YMAL)

- playbooks start with `---` 3 dashes


### YAML

Create file nginx_playbook.yml

sudo nano nginx_playbook.yml

```
# Create a playbook to install nginx web server on web machine
# web 192.168.33.10
# Let's add the 3 dashes to start the YAML
---
# add the name of the host
- hosts: web

# gather facts about the installation steps
  gather_facts: yes
# we need admin access
  become: true
# add instructions to install nginx on web machine
  tasks:
  - name: Install Nginx
    apt: pkg=nginx state=present
# ensure the nginx server is running
```

ansible-playbook nginx_playbook.yml

- Let's check if the playbook worked for us
`ansible web -a "sudo systemctl status nginx"`
- Enter the web ip in your browser and you should see nginx default page!

Task:
- create a playbook to install/configure node on the web machine
- create a playbook to install/configure mongodb in the db machine
- get the nodeapp to work on web up with /posts
- HINT: ansible official documentation available
- Youtube
- Stackover flow
- configure nginx reverse proxy