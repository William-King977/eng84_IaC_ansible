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
* **Agentless** - makes it lightweight on agent nodes, no need to install in agent notes
* **Secure** - uses SSH to connect to other servers

### Adhoc commands
Ansible adhoc commands can be used to find out the configurations/run commands for other servers from the Ansible controller.

* `ssh vagrant@server_ip` - SSH into another vagrant machine
* `ansible web -m ping` - ping into the web server
* `ansible all -m ping` - ping all the machines
* `ansible web -a "free -m"` - display available memory in web server
* `ansible web -m shell -a "ls -a"` - display file structure in web server
* `-m` - specifies a module
* `-a` - an adhoc


* `ansible all -m shell -a "uptime"` - uptime for all machines
* `ansible db -m shell -a "uptime"` - uptime for the database


* `ansible all -m shell -a "sudo apt-get update -y"` - update Linux on all machines
* `ansible all -m shell -a "sudo apt-get upgrade -y"` - upgrade Linux on all machines


* `hosts` file holds info to configure with the other servers (stored in `/etc/ansible` directory)
* `ansible web -a "uname -a"` - view info about the web server
