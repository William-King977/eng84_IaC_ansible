# Infrastructure as Code (IaC) with Ansible

## Infrastructure as Code
IaC is a method of provision and manage IT infrastructure through the use of
machine-readable definition files (i.e. source code)

### Two parts of IaC
- Configuration Management
  CM tools are responsible for provisioning and maintaining the state of your systems. Some tools include:
  - Chef
  - Puppet
  - Ansible
- Orchestration
  Once the templates for all parts of the system are created, we need orchestration tools and scripts that talks to the cloud to pull them together into the architecture
  - CloudFormation (AWS)
  - Ansible (can do this too)
  - Terraform

### Benefits
- Speed and simplicity
- Configuration consistency
- Minimisation of risk
- increase efficiency in software development
- Cost savings

### Best practices
- Codify everything
- Document as little as possible
- Maintain version control
- Continuously test, integrate and deploy
- Make your infrastructure code modular
- Make your infrastructure immutable (when possible)

### Types of Infrastructure
- Mutable or Immutable
- On Premises or Cloud

## Ansible
- Simple, uses Yamal - human readable
- Agentless - makes it lightweight on agent nodes, no need to install in agent notes
- SSH to connect to other servers