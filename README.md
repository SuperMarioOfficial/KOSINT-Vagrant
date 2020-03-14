
**This is not the only way of creating a fully functional vagrant box, but this is how I do it** 
### Packer [Virtual machine settings]
- adjust the vm settings in the packer file
- change the passwords in the preseed and ssh of the packer file
- run the command ```packer build --force <name of the file>.json```
### Ansible [Software provisioning]
- check the ansible playbook file to add or remove packages
### Vagrant [Manage the virtual machine]
- add or change the password for the ssh into the vagrantfile
- make sure that the ansible playbook is in the right folder
#### copy paste the following commands
```
    vagrant box add kosint k-osint-virtualbox.box --name kosint
    vagrant init kosint
    vagrant up
```

### What next?
- ssh into the machine and use it
- upload it to the vagrant cloud
- read the documentation 
