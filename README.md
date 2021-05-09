# Infrastructure as Code (IaC) with Ansible
## Infrastructure as Code
IaC is a method to provision and manage IT infrastructure through the use of machine-readable definition files (i.e. source code), rather than through old fashioned operating procedures and manual processes.

### Benefits
* Speed and simplicity
* Configuration consistency
* Minimisation of risk
* Increase efficiency in software development
* Cost savings

### Best practices
* Codify everything
* Document as little as possible
* Maintain version control
* Continuously test, integrate and deploy
* Make your infrastructure code modular
* Make your infrastructure immutable (when possible)

### Two parts of IaC
**Configuration Management:**<br />
CM tools are responsible for provisioning and maintaining the state of your systems. Some tools include:
* Chef
* Puppet
* Ansible

**Orchestration:**<br />
Once the templates for all parts of the system are created, we need orchestration tools and scripts that talks to the cloud to pull them together into the architecture.
* CloudFormation (AWS)
* Ansible (can do this too)
* Terraform

### Types of Infrastructure
* Mutable or Immutable
* On Premises or Cloud

## Ansible
Ansible is an IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration and more. It works only on Mac and Linux.

### Benefits
* **Simple** - uses YAML (Yet Another Markup Language), so it's human readable
* **Agentless** - makes it lightweight on agent nodes. No need to install in agent notes, only the master node.
* **Secure** - uses SSH to connect to other servers
* **Integration** - can be integrated with other tools, such as Docker and Jenkins

### Ansible Architecture
![image](Diagrams/EC2%20Deployment%20with%20Ansible.png)

### Adhoc commands
Adhoc commands can be used to run commands in other servers from the Ansible controller.

* `ssh vagrant@server_ip` - SSH into another vagrant machine
* `hosts` file holds info to configure with the other servers (stored in `/etc/ansible` directory)

Adding a web host in the `hosts` file:
```
[web]
192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```

Commands with the web server:
* `ansible web -m ping` - ping into the web server
* `ansible all -m ping` - ping all the machines
* `ansible web -a "free -m"` - display available memory in web server
* `ansible web -a "ls -a"` - display file structure in web server
* `ansible web -a "command"` - can be done with any command
* `ansible web -m shell -a "command"` - add `-m shell` if necessary
* `-m` - specifies a module
* `-a` - an adhoc

Uptime, Update and Upgrade commands:
* `ansible all -a "uptime"` - uptime for all machines
* `ansible db -a "uptime"` - uptime for the database
* `ansible all -a "sudo apt-get update -y"` - update Linux on all machines
* `ansible all -a "sudo apt-get upgrade -y"` - upgrade Linux on all machines

### Playbooks
* Playbooks are written in YAML (.yml/yaml) with a set of instructions/tasks to do the configuration management on the hosts/agent nodes.
* Same location as the `hosts` file (`/etc/ansible`)
* Indentation is important! It must be 2 spaces, no tabs.

Making playbook files:
* `sudo nano install_nginx.yml` - creating a YAML file to install Nginx
* `sudo ansible-playbook install_nginx.yml` - run the YAML file

## Ansible Vault
Ansible Vault allows you to keep sensitive data, such as passwords or keys in encrypted file, rather than as plaintext in playbooks. The data inside can be referenced inside playbooks when they need to be used.

### Dependencies
```
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt-get update -y

sudo apt install ansible -y
sudo apt install python3-pip -y
pip3 install awscli
pip3 install boto boto3
```

### Create a Vault file
1. Go to the `/etc/ansible/` directory
2. `mkdir group_vars`, then go inside
3. `mkdir all`, then go inside
4. Create the file using `sudo ansible-vault create pass.yml` - then inside, add both keys:
   * `aws_access_key: place_here`
   * `aws_secret_key: place_here`
   * To exit: `Esc` > `:wq` > Enter
* The contents of `pass.yml` is encrypted
* To run YAML files with the vault: `sudo ansible-playbook install_nginx.yml --ask-vault-pass`
* It can only be edited with `sudo ansible-vault edit pass.yml`

### Removing the Vault
Do the following in both the `web` and `db` machines:
* `cd /etc/ssh/`
* `sudo nano sshd_config`
* Inside, ensure that `PermitRootLogin yes` and `PasswordAuthentication yes` are set to `yes` or uncommented.
* Restart SSH using `sudo systemctl restart ssh`

In the `controller`, do the following:
* Delete the vault file
* `sudo ansible-playbook install_nginx.yml` or other YAML files should work now
