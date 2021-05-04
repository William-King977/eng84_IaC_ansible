# Infrastructure as Code (IaC) with Ansible
## Infrastructure as Code
IaC is a method of provision and manage IT infrastructure through the use of machine-readable definition files (i.e. source code), rather than through old fashioned operating procedures and manual processes.

### Benefits
* Speed and simplicity
* Configuration consistency
* Minimisation of risk
* increase efficiency in software development
* Cost savings

### Best practices
* Codify everything
* Document as little as possible
* Maintain version control
* Continuously test, integrate and deploy
* Make your infrastructure code modular
* Make your infrastructure immutable (when possible)

### Two parts of IaC
**Configuration Management:**
* CM tools are responsible for provisioning and maintaining the state of your systems. Some tools include:
  * Chef
  * Puppet
  * Ansible

**Orchestration:**
* Once the templates for all parts of the system are created, we need orchestration tools and scripts that talks to the cloud to pull them together into the architecture.
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
