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
![image](https://user-images.githubusercontent.com/44005332/117135325-b0f1b280-ad9e-11eb-913d-b0bd08c951fa.png)

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
* `ansible-playbook install_nginx.yml` - run the YAML file

### Ansible Vault
Dependencies:
* sudo apt-add-repository --yes --update ppa:ansible/ansible
* sudo apt install ansible -y
* sudo apt install python3-pip -y
* pip3 install awscli
* pip3 install boto boto3
* sudo apt-get upgrade -y

#### Create a Vault file
* Go to the `/etc/ansible/` directory
* `mkdir group_vars`, then go inside
* `mkdir all`, then go inside
* Create the file using `sudo ansible-vault create pass.yml` - then inside:
  * `aws_access_key: place_here`
  * `aws_secret_key: place_here`
  * To exit: `esc` > `:wq` > Enter
* The contents of `pass.yml` is encrypted

To run YAML files with the vault: `sudo ansible-playbook install_nginx.yml --ask-vault-pass`

#### Removing the Vault
Do the following in both the `web` and `db` machines:
* `cd /etc/ssh/`
* `sudo nano sshd_config`
* Inside, ensure that `PermitRootLogin yes` and `PasswordAuthentication yes` are set to `yes` or uncommented.
* Restart SSH using `sudo systemctl restart ssh`

In the `controller`, do the following:
* Delete the vault file
* `sudo ansible-playbook install_nginx.yml` or other YAML files should work now

### Launching EC2 Instances with a Playbook
1. Configure a vault as shown in **Create a Vault file**
2. On the `/etc/ansible` directory, modify the `hosts` file and add the following code to enable Python3:
   ```
   [local]
   localhost ansible_python_interpreter=/usr/bin/python3

   ```
3. Create a Playbook file (YAML) with the following contents:
   ```
   ---
   - hosts: localhost
     connection: local
     gather_facts: True

     vars:
       key_name: your_key_name
       region: eu-west-1

     # Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
     image: ami-038d7b856fe7557b3
     subnet_id: your_subnet_id

     tasks:
      - name: Facts
        block:
        - name: Get instances facts
          ec2_instance_facts:
            aws_access_key: "{{aws_access_key}}"
            aws_secret_key: "{{aws_secret_key}}"
            region: "{{ region }}"
          register: result
        tags: always

      - name: Provisioning EC2 instances
        block:
        - name: Creating Web Instance
          # Web instance
          ec2:
            aws_access_key: "{{aws_access_key}}"
            aws_secret_key: "{{aws_secret_key}}"
            key_name: "{{ key_name }}"
            instance_tags:
              Name: eng84_william_ansible_web
            id: eng84_william_ansible_web
            image: "{{ image }}"
            vpc_subnet_id: "{{ subnet_id }}"
            group_id: web_security_group_id
            instance_type: t2.micro
            region: "{{ region }}"
            wait: true
            count: 1
            assign_public_ip: yes

        - name: Creating Database Instance
          # DB instance
          ec2:
            aws_access_key: "{{aws_access_key}}"
            aws_secret_key: "{{aws_secret_key}}"
            key_name: "{{ key_name }}"
            instance_tags:
              Name: eng84_william_ansible_db
            id: eng84_william_ansible_db
            image: "{{ image }}"
            vpc_subnet_id: "{{ subnet_id }}"
            group_id: db_security_group_id
            instance_type: t2.micro
            region: "{{ region }}"
            wait: true
            count: 1
            assign_public_ip: yes
        tags: ['never', 'create_ec2']
   ```
4. Execute `sudo ansible-playbook playbook_name.yml --ask-vault-pass` to run the file without launching the instances
5. Execute `sudo ansible-playbook playbook_name.yml --ask-vault-pass --tags create_ec2` to launch both EC2 instances
6. If the above commands worked, both EC2 instances will be running on AWS

### Running Commands from the Controller
1. Import the key (usually `.pem`):
   * **Vagrant:** `scp ~/.ssh/key_name vagrant@controller_ip:~/key_name`
   * **EC2 AWS:** `scp -i ~/.ssh/DevOpsStudent.pem -r ~/.ssh/key_name ubuntu@controller_ec2_public_ip:~/key_name`
   * Then move it to the `~/.ssh` folder.
2. Use `chmod 400` to ensure it's readable for YOU only
3. Modify the `hosts` file in `/etc/ansible`, and modify `web` and `db` with the following contents:
   ```
   [web]
   web_public_ip ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem

   [db]
   db_public_ip ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem
   ```
4. The `web`, `db` security groups and NACLs need to be modified to allow the Controller to SSH into them
5. Now, you can execute commands like this: `sudo ansible web -a "command" --ask-vault-pass`
