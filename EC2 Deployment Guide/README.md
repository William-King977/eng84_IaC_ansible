# EC2 Deployment with Playbooks
## Step 1: Create a Vault file
1. Go to the `/etc/ansible/` directory
2. `mkdir group_vars`, then go inside
3. `mkdir all`, then go inside
4. Create the file using `sudo ansible-vault create pass.yml` - then inside, add both keys:
   * `aws_access_key: place_here`
   * `aws_secret_key: place_here`
   * To exit: `Esc` > `:wq` > Enter

## Step 2: Launching EC2 Instances
1. On the `/etc/ansible` directory, modify the `hosts` file and add the following code to enable Python3:

   ```
   [local]
   localhost ansible_python_interpreter=/usr/bin/python3
   ```
2. Create a Playbook file (YAML) with contents similar to `launch_ec2_instances.yml`
3. Execute `sudo ansible-playbook playbook_name.yml --ask-vault-pass --tags launch_ec2` to launch both EC2 instances
4. If the above commands worked, both EC2 instances will be running on AWS

## Step 3: Running Commands from the Controller
1. Import the key (usually `.pem`):
   * **Vagrant:** `scp ~/.ssh/key_name vagrant@controller_ip:~/.ssh/key_name`
   * **EC2 AWS:** `scp -i ~/.ssh/aws_key -r ~/.ssh/key_name ubuntu@controller_ec2_public_ip:~/.ssh/key_name`
2. Modify the `hosts` file in `/etc/ansible`, and modify `web` and `db` with the following contents:

   ```
   [web]
   web_private_ip ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem

   [db]
   db_private_ip ansible_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem
   ```

3. NOTE: the IP must be public if your controller is running on Vagrant
4. The `web`, `db` security groups and NACLs need to be modified to allow the controller to SSH into them
5. Now, you can execute commands like this: `sudo ansible web -a "command" --ask-vault-pass`

## Step 4: Using Playbooks to Install Dependencies
Before carrying out these steps, ensure that `provision_mongodb.yml`, `provision_nginx_proxy.yml` and `provision_nodejs_web.yml` are created with the same contents (change where applicable).
1. Modify the database's security group (and NACL) to allow all traffic from the web app
2. Run the nodejs playbook using `sudo ansible-playbook provision_nodejs_web.yml --ask-vault-pass`. This playbook also executes the other two playbooks mentioned previously. If your playbook execution gets stuck at **Upgrade Linux**, see **Linux Upgrade GRUB Issue**.
3. After running the playbook, the web app will be running on its public IP and the other features will be working as well (posts and Fibonacci)

## Linux Upgrade GRUB Issue
What happens here is, after performing `sudo apt-get update -y`, the GRUB configuration file somehow gets modified. As a result, the GRUB GUI will display during `sudo apt-get upgrade -y`, requiring user input, and the process gets *stuck* unless one interrupts it. To solve this, follow the steps below:
1. Interrupt the process with `Ctrl + C`
2. SSH into the instance being provisioned
3. Run `sudo killall apt apt-get`
4. Run `sudo dpkg --configure -a`
5. When the GUI displays, press `Enter` (keep the local version)
6. Return to your controller and re-run the playbook

As of writing, this issue is known to happen in Ubuntu 16.04.
