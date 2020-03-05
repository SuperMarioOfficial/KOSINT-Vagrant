# K-OSINT.iso

## The plan 
![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/idea.PNG)
![](https://blog.secureideas.com/wp-content/uploads/2018/09/packer_vagrant_eco.png)
- build the iso
    - kali-tools-information-gathering
    - kali-tools-reporting
    - kali-tools-social-engineering
- configure virtualbox with packer
- provisioning with ansible playbook
    - Install docker
    - pull docker images
- create a vagrant box
------


# Build the ISO
## steps:
#### 1
``` bash
sudo apt update
sudo apt install git live-build cdebootstrap devscripts -y
git clone git://gitlab.com/kalilinux/build-scripts/live-build-config.git
cd live-build-config
```
#### 2 https://tools.kali.org/kali-metapackages
***$vi*** ``` kali-config/variant-default/package-lists/kali.list.chroot```
```
# You always want those
kali-linux-core

# Kali applications
kali-tools-information-gathering
kali-tools-reporting

# Graphical desktop
kali-desktop-xfce
```
#### 3 preseed.cfg [1](https://www.kali.org/dojo/preseed.cfg)
***$vi*** ```kali-config/common/includes.binary/isolinux/install.cfg```
```
label install
    menu label ^Install Automated
    linux /install/vmlinuz
    initrd /install/initrd.gz
    append vga=788 -- quiet file=/cdrom/install/preseed.cfg locale=en_US keymap=us hostname=kali domain=local.lan
```

# configure virtualbox with packer
### Tree structure project
```
Packer/
      |---k-osint.json
      |---k-osint.iso
      |---http/
      |       |--- preseed.cfg
      |---scripts/
              |--- cleanup.sh
	      |--- virtualbox.sh
	      |--- ansible.sh
```
# Packer 
The VirtualBox Packer builder is able to create VirtualBox virtual machines and export them in the OVF format, starting from an ISO image. The builder builds a virtual machine by creating a new virtual machine from scratch, booting it, installing an OS, provisioning software within the OS, then shutting it down. The result of the VirtualBox builder is a directory containing all the files necessary to run the virtual machine portably.
- [how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter](https://www.vlent.nl/weblog/2017/09/29/how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter/)
- [Kali-Packer repository](https://github.com/vortexau/Kali-Packer)
- [virtual-image-automation](https://blog.zaleos.net/virtual-image-automation/)
- [create-simple-centos-7-virtualbox-with-packer](https://softwaretester.info/create-simple-centos-7-virtualbox-with-packer/)
- [eanderalx.org/network_card_vbox](https://www.eanderalx.org/virtualization/8_network_card_vbox)
- [frankietyrine/packer-kali_linux](https://github.com/frankietyrine/packer-kali_linux)
- [automating-red-team-homelabs-part-2-build-pentest-destroy-and-repeat](https://blog.secureideas.com/2019/05/automating-red-team-homelabs-part-2-build-pentest-destroy-and-repeat.html)
- [bento/packer_templates](https://github.com/chef/bento/tree/master/packer_templates)
- [Automated Install Kali Linux (Packer) youtube](https://www.youtube.com/watch?v=uDLC2JMCLek)
### Pakcer configuration examples
- [bento/example1.json](https://github.com/chef/bento/blob/master/packer_templates/debian/debian-10.2-amd64.json)
- [buffersandbeer/example2.json](https://github.com/buffersandbeer/packer-kali-linux/blob/master/kali.json)
- [elreydetoda/example3.json](https://github.com/elreydetoda/packer-kali_linux/blob/master/templates/template.json)
- [quarkslab/example4.json](https://github.com/quarkslab/packer-debian/blob/master/debian.json)
- [studentota2lvl/packer-Windows-Server/example5.json](https://github.com/studentota2lvl/packer-Windows-Server-2016/blob/1b9d4c975a1449f67a94911ae233e75fb48a3101/windows_2016.json)
- [geerlingguy/exaple6.json](https://github.com/geerlingguy/packer-boxes/blob/master/debian10/box-config.json)
- [capistrano/example7.json](https://github.com/capistrano/packer/blob/master/capistrano-Debian_7.4_64.json)

#### W10, find SHA1 and SHA256
```
certutil -hashfile k-osint.iso SHA1
certutil -hashfile VBoxGuestAdditions.iso SHA256
```

## k-osint.json
```
{
  "variables": {
    "vm_name": "k-osint",
    "disk_size": "80000",
    "iso_checksum": "93ea9f00a60551412f20186cb7ba7d1ff3bebf73",
    "iso_checksum_type": "sha1",
    "iso_url": "k-osint.iso",
    "box_name" : "k-osint", 
    "box_desc" : "Official Kali Linux OS distro of Tracelab"
  },
  "description": "{{user `box_desc`}}",
  "builders": [
	{ 
      "headless": false,
      "type": "virtualbox-iso",
      "virtualbox_version_file": ".vbox_version",
      "guest_os_type": "Debian_64",
      "vm_name": "{{user `vm_name`}}",
      "iso_url": "{{user `iso_url`}}",
      "iso_checksum": "{{user `iso_checksum`}}",
      "iso_checksum_type": "{{user `iso_checksum_type`}}",
      "disk_size": "{{user `disk_size`}}",
      "http_directory": "http",
      "shutdown_command": "echo 'vagrant' | sudo -S /sbin/shutdown -hP now",
      "communicator": "ssh",
      "ssh_username": "vagrant",
      "ssh_password": "vagrant",
      "ssh_port": 22,
      "ssh_wait_timeout": "10000s",
      "guest_additions_mode": "disable",
      "vboxmanage": [
        ["modifyvm","{{.Name}}","--memory","6000"],
        ["modifyvm","{{.Name}}","--cpus","3"], 
	["modifyvm","{{.Name}}","--audio","none"], 
	["modifyvm","{{.Name}}", "--nic1", "nat"],
	["modifyvm","{{.Name}}", "--nic2", "intnet"],
	["modifyvm","{{.Name}}", "--intnet2", "whonix"],
	["modifyvm", "{{.Name}}", "--accelerate3d", "on"],
        ["modifyvm", "{{.Name}}", "--usb", "on"],
        ["modifyvm", "{{.Name}}", "--graphicscontroller", "vboxsvga"],
	["modifyvm", "{{.Name}}", "--clipboard-mode", "bidirectional"],
        ["modifyvm", "{{.Name}}", "--draganddrop", "bidirectional"]

	],

	"boot_wait": "5s",
        "boot_command": [ 
         "<esc><wait>",
        "/install/vmlinuz noapic ",
        "preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg ",
        "hostname={{ .Name }} ",
        "auto=true ",
        "interface=auto ",
        "domain=vm ",
        "initrd=/install/initrd.gz -- <enter>"
      ]

    }
],
""provisioners": [
    		{
      "type": "shell",
      "execute_command": "echo '{{user `ssh_password`}}' | {{ .Vars }} sudo -S -E sh -eux '{{ .Path }}'",
      "scripts": [ "{{template_dir}}/scripts/cleanup.sh","{{template_dir}}/scripts/virtualbox.sh"],
      "expect_disconnect": true
      }]
}
```
## Notes
#### boot_command:
- ```/install/vmlinuz noapic``` it tells the [kernel](https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html) to not make use of any [IOAPICs](https://wiki.osdev.org/IOAPIC) that may be present in the system.
#### provisioners:
**Be careful to set ssh username and password to the same username/password of the preceed or it won't work.**
- k-osint.json
```
"ssh_username": "vagrant",
"ssh_password": "vagrant",
```
- preseed file 
```
d-i passwd/user-password password vagrant
d-i passwd/user-password-again password vagrant
```
**Important**, remember to provide the sudo rights to your scripts. Most of the examples echo <something>, that probably is not the right password, and if you are doing it for the first time it is easier to overlook that you are piping the wrong password.Although, a better way to do this is not to hardcode the password, but to echo the ssh_pass variable in this way:
```
"execute_command": "echo '{{user `ssh_password`}}' | {{ .Vars }} sudo -E -S sh '{{ .Path }}'",
```
- [packer.io/docs/templates/provisioners](https://packer.io/docs/templates/provisioners.html)
- - [gwagner/packer-centos/virtualbox-guest-additions.sh](https://github.com/gwagner/packer-centos/blob/master/provisioners/install-virtualbox-guest-additions.sh)
- [riywo/packer-example/virtualbox.sh](https://github.com/riywo/packer-example/blob/master/scripts/virtualbox.sh)

## Preceed
Preseeding provides a way to set answers to questions asked during the installation process, without having to manually enter the answers while the installation is running. This makes it possible to fully automate most types of installation and even offers some features not available during normal installations. If you are installing the operating system from a mounted iso as part of your Packer build, you will need to use a preseed file. [Example](https://www.debian.org/releases/stable/example-preseed.txt) 
- https://www.kali.org/dojo/preseed.cfg
- https://kali.training/topic/unattended-installations/
- [Full tutorial](https://www.debian.org/releases/stable/amd64/apb.en.html)
- [Automated Debian Install with Preseeding](https://www.youtube.com/watch?v=ndHi1sQWuH4)
- [preseed-kali-linux-from-a-mini-iso](https://medium.com/@honze_net/preseed-kali-linux-from-a-mini-iso-9ad622617241)
- [video kali-packer](https://www.youtube.com/watch?v=uDLC2JMCLek)
- [automating-red-team-homelabs-part-1-kali-automation](https://blog.secureideas.com/2018/09/automating-red-team-homelabs-part-1-kali-automation.html)
- [automating-red-team-homelabs-part-2-build-pentest-destroy-and-repeat](https://blog.secureideas.com/2019/05/automating-red-team-homelabs-part-2-build-pentest-destroy-and-repeat.html) 
### Examples:
- [/kalilinux/build-scripts/kali-vagrant/preseed.cfg](https://gitlab.com/kalilinux/build-scripts/kali-vagrant/-/blob/master/http/preseed.cfg)
- [kalilinux/recipes/kali-preseed-examples/preseed.cfg](https://gitlab.com/kalilinux/recipes/kali-preseed-examples/-/blob/master/kali-linux-rolling-preseed.cfg)
## preseed.cfg [source/preseed.cfg](https://gitlab.com/kalilinux/build-scripts/kali-vagrant/-/blob/master/http/preseed.cfg)
```
d-i debian-installer/locale string en_US.UTF-8
d-i console-keymaps-at/keymap select us
d-i mirror/country string enter information manually
d-i mirror/http/hostname string http.kali.org
d-i mirror/http/directory string /kali
d-i keyboard-configuration/xkb-keymap select us
d-i mirror/http/proxy string
d-i mirror/suite string kali-rolling
d-i mirror/codename string kali-rolling

d-i clock-setup/utc boolean true
d-i time/zone string US/Eastern

# Disable security, volatile and backports
d-i apt-setup/services-select multiselect

# Enable contrib and non-free
d-i apt-setup/non-free boolean true
d-i apt-setup/contrib boolean true

# Disable source repositories too
d-i apt-setup/enable-source-repositories boolean false

# Partitioning
d-i partman-auto/method string regular
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean true
d-i partman-auto/choose_recipe select atomic
d-i partman-auto/disk string /dev/sda
d-i partman/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman-partitioning/confirm_write_new_label boolean true

# Disable CDROM entries after install
d-i apt-setup/disable-cdrom-entries boolean true

# Install default packages
tasksel tasksel/first multiselect desktop-xfce, meta-default, standard

# Change default hostname
d-i netcfg/get_hostname string kali
d-i netcfg/get_domain string unassigned-domain
#d-i netcfg/choose_interface select auto
d-i netcfg/choose_interface select eth0
d-i netcfg/dhcp_timeout string 60

d-i hw-detect/load_firmware boolean false

# vagrant user account
d-i passwd/user-fullname string vagrant
d-i passwd/username string vagrant
d-i passwd/user-password password vagrant
d-i passwd/user-password-again password vagrant

d-i apt-setup/use_mirror boolean true
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean false
d-i grub-installer/bootdev string /dev/sda
d-i finish-install/reboot_in_progress note

# Enable SSH
d-i preseed/late_command string in-target systemctl enable ssh
```

# Provisioning with ansible playbook
## How to create a playbook
- [pedantically_commented_playbook.yml/playbook.yml ](https://github.com/ogratwicklcs/pedantically_commented_playbook.yml/blob/master/playbook.yml)
- [Using Ansible for system updates](https://www.redpill-linpro.com/sysadvent/2017/12/24/ansible-system-updates.html)
```
"provisioners": [
{
"type": "shell",
"script": "provision.sh"
}
]
```
```
#!/bin/bash
set -e
#provision.sh sudo apt-get update
 echo "apt-get update done."
 sudo apt-get -y upgrade
 sudo apt-get install -y python-dev python-pip
 sudo pip install ansible
 sudo timedatectl set-timezone Europe/Istanbul
 sudo localectl set-locale LANG=en_US.utf8
 sudo wget ‘https://s3.amazonaws.com/packeramidemo/i_playbook.yml'
 echo "Running build."
 sudo ansible-playbook i_playbook.yml
```

```
hosts: localhost
 connection: local
 sudo: yes
 tasks:
  — name: Install list of packages
  apt: name={{item}} state=installed
 with_items:
 — htop
 — ctop
 — screen
 — unzip
 — curl
 — sudo
 — mtr
 — bash-completion
 — tree
 — colordiff
 — ntp
 — bwm-ng
 — docker-compose
 — apt-transport-https- name: install docker
 
- name: Update and upgrade apt packages
  become: true
  apt:
    update_cache: yes
    upgrade: yes
    cache_valid_time: 86400 #One day
 
# do the actual apt-get dist-upgrade
 - name: apt-get dist-upgrade
   apt:
     upgrade: dist # upgrade all packages to latest version
    
 - name: apt-get autoremove
      command: apt-get -y autoremove
      args:
        warn: false

# check if we need a reboot
 - name: check if reboot needed
    stat: path=/var/run/reboot-required
    register: file_reboot_required
```


# Create Vagrant box
- [vagrant-whonix-kali](https://github.com/j7k6/vagrant-whonix-kali/blob/master/Vagrantfile)
### How to create a box


# Docker
```
FROM Xubuntu 18.04.4 LTS
RUN apt-get update && \
    apt-get -y dist-upgrade
RUN sudo add-apt-repository universe && sudo apt update
RUN sudo apt install torbrowser-launcher

RUN useradd -m -d /home/anon anon

WORKDIR /home/anon


RUN mkdir /home/anon/Downloads && \
    chown -R anon:anon /home/anon && \
    apt-get autoremove

USER anon

CMD /home/anon/tor-browser_en-US/Browser/start-tor-browser
```

# Other
### XFCE GUI
![](https://i.ytimg.com/vi/Twcm18oFmqs/maxresdefault.jpg)
After a research done, I came to the conclusion that xfce is probably the lightest and fasted gui. 
- [linux_distros/GUI_ram_consumption_comparison_updated](https://www.reddit.com/r/linux/comments/5l39tz/linux_distros_ram_consumption_comparison_updated/)
- [Power Use, RAM + Boot Times With Unity, Xfce, GNOME, LXDE, Budgie & KDE Plasma](https://www.phoronix.com/scan.php?page=article&item=ubu-1704-desktops&num=2)
- [Kali xfce](https://www.youtube.com/watch?v=Twcm18oFmqs)
- [Change from Gnome to xfce](https://www.youtube.com/watch?v=uIbG3SI5hd8)
- [How To Install Xfce4 & MATE Desktop Environments On Kali Linux](https://www.youtube.com/watch?v=hy9F87basCI)
### Links
- [create-kali-linux-iso](https://www.cybrary.it/blog/0p3n/create-kali-linux-iso/)
- [live-build-config-examples](https://gitlab.com/kalilinux/recipes/live-build-config-examples/-/blob/master/kali-linux-mate-top10-nonroot.sh)
- [dojo-mastering-live-build/](https://www.kali.org/docs/development/dojo-mastering-live-build/)
- [kali-metapackages](https://tools.kali.org/kali-metapackages)
- [creating-an-Ubuntu-VM-with-packer](https://kappataumu.com/articles/creating-an-Ubuntu-VM-with-packer.html)
- [how-to-use-packer-to-create-ubuntu-18-04-vagrant-boxes](https://www.serverlab.ca/tutorials/dev-ops/automation/how-to-use-packer-to-create-ubuntu-18-04-vagrant-boxes/)
- [whonix-kali](https://github.com/j7k6/vagrant-whonix-kali)
- [Wchecksum](https://answers.microsoft.com/en-us/insider/forum/insider_wintp-insider_update/is-there-built-in-checksum-for-win-10/8dba82be-f036-4460-b427-954e057b678a)
- [creating-an-Ubuntu-VM-with-packer](https://kappataumu.com/articles/creating-an-Ubuntu-VM-with-packer.html)
- https://www.perkin.org.uk/posts/create-virtualbox-vm-from-the-command-line.html
- https://www.andreafortuna.org/2019/10/24/how-to-create-a-virtualbox-vm-from-command-line/
- https://networking.ringofsaturn.com/Unix/Create_Virtual_Machine_VBoxManage.php
- [Chapter 6. Virtual Networking](https://www.virtualbox.org/manual/ch06.html)

### Hide and Sneak
This application assists in managing attack infrastructure for penetration testers by providing an interface to rapidly deploy, manage, and take down various cloud services.
- [link](https://github.com/rmikehodges/hideNsneak)

### Cloud-formation
- [automated-red-team-infrastructure-deployment-with-terraform](https://rastamouse.me/2017/08/automated-red-team-infrastructure-deployment-with-terraform---part-1/)
```
--nic<1-N> none|null|nat|natnetwork|bridged|intnet|hostonly|generic: Configures the type of networking for each of the VM's virtual network cards. Options are: not present (none), not connected to the host (null), use network address translation (nat), use the new network address translation engine (natnetwork), bridged networking (bridged), or use internal networking (intnet), host-only networking (hostonly), or access rarely used sub-modes (generic). These options correspond to the modes described in Section 6.2, “Introduction to Networking Modes”. 
```

