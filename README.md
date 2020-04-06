
# How to access this machine
```
    vagrant box add mindwarelab/kosint
    vagrant up
``` 
# Instructions to rebuild the box
This repository contain the basic packer instruction to create a vagrant box. The key points of the configuration of the box as explained below. 
### Packer [Virtual machine settings]
- adjust the vm settings in the packer file
- change the passwords in the preseed and ssh of the packer file
- run the command ```packer build --force <name of the file>.json```
### Ansible [Software provisioning]
- check the ansible playbook file to add or remove packages
### Vagrant [Manage the virtual machine]
- add or change the password for the ssh into the vagrantfile
- make sure that the ansible playbook is in the right folder
