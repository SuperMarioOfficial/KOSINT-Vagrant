# Simple steps from Packer to Vagrant

### Packer
- adjust the vm settings in the packer file
- change the passwords in the preseed and ssh of the packer file
- run the command ```packer build --force <name of the file>.json```
### Vagrant
- add or change the password for the ssh into the vagrantfile
- make sure that the ansible file is in the same folder
#### copy paste the following commands
```
    vagrant box add kosint k-osint-virtualbox.box --name kosint
    vagrant init kosint
    vagrant up
```

### What next?
- ssh into the machine and use it
- upload it to the vagrant cloud
