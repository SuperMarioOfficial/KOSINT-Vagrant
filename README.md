# The K-OSINT.iso automation project 
#### Virtualbox + Kali Linux personalized ISO + Packer + Ansible + Vagrant + Docker
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


![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)


# Build the ISO
## steps:
#### 1 Clone the live-build
``` bash
sudo apt update
sudo apt install git live-build cdebootstrap devscripts -y
git clone git://gitlab.com/kalilinux/build-scripts/live-build-config.git
cd live-build-config
```
#### 2 Select the packages [Full List](https://tools.kali.org/kali-metapackages)
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

![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)


# Automate the creation of a Vbox machine with Packer configuration files
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
### What is Packer?
The VirtualBox Packer builder is able to create VirtualBox virtual machines and export them in the OVF format, starting from an ISO image. The builder builds a virtual machine by creating a new virtual machine from scratch, booting it, installing an OS, provisioning software within the OS, then shutting it down. The result of the VirtualBox builder is a directory containing all the files necessary to run the virtual machine portably. [To know more visit learn.hashicorp.com](https://learn.hashicorp.com/packer#operations-and-development)

### Pakcer configuration examples
- [bento/example1.json](https://github.com/chef/bento/blob/master/packer_templates/debian/debian-10.2-amd64.json)
- [buffersandbeer/example2.json](https://github.com/buffersandbeer/packer-kali-linux/blob/master/kali.json)
- [elreydetoda/example3.json](https://github.com/elreydetoda/packer-kali_linux/blob/master/templates/template.json)
- [quarkslab/example4.json](https://github.com/quarkslab/packer-debian/blob/master/debian.json)
- [studentota2lvl/packer-Windows-Server/example5.json](https://github.com/studentota2lvl/packer-Windows-Server-2016/blob/1b9d4c975a1449f67a94911ae233e75fb48a3101/windows_2016.json)
- [geerlingguy/exaple6.json](https://github.com/geerlingguy/packer-boxes/blob/master/debian10/box-config.json)
- [capistrano/example7.json](https://github.com/capistrano/packer/blob/master/capistrano-Debian_7.4_64.json)
- [stefco/geco_vm.json](https://github.com/stefco/geco_vm/tree/51b80576ed37fd8a53cac6e05db232c1bf1e6f70)
- [xuxiaodong/.json](https://github.com/xuxiaodong/kvm-example/blob/df0bbad6b0071bdd29d83ad4a5ee965fcd71e819/archlinux-2020.02.01-amd64.json)

### References:
- [how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter](https://www.vlent.nl/weblog/2017/09/29/how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter/)
- [Kali-Packer repository](https://github.com/vortexau/Kali-Packer)
- [virtual-image-automation](https://blog.zaleos.net/virtual-image-automation/)
- [create-simple-centos-7-virtualbox-with-packer](https://softwaretester.info/create-simple-centos-7-virtualbox-with-packer/)
- [eanderalx.org/network_card_vbox](https://www.eanderalx.org/virtualization/8_network_card_vbox)
- [frankietyrine/packer-kali_linux](https://github.com/frankietyrine/packer-kali_linux)
- [automating-red-team-homelabs-part-2-build-pentest-destroy-and-repeat](https://blog.secureideas.com/2019/05/automating-red-team-homelabs-part-2-build-pentest-destroy-and-repeat.html)
- [bento/packer_templates](https://github.com/chef/bento/tree/master/packer_templates)
- [Automated Install Kali Linux (Packer) youtube](https://www.youtube.com/watch?v=uDLC2JMCLek)
- [packer.io/docs/templates/provisioners](https://packer.io/docs/templates/provisioners.html)
- - [gwagner/packer-centos/virtualbox-guest-additions.sh](https://github.com/gwagner/packer-centos/blob/master/provisioners/install-virtualbox-guest-additions.sh)
- [riywo/packer-example/virtualbox.sh](https://github.com/riywo/packer-example/blob/master/scripts/virtualbox.sh)
- [run provisioner on specidfic build ](https://packer.io/docs/templates/provisioners.html)
- [how-to-use-packer-to-create-ubuntu-18-04-vagrant-boxes](https://www.serverlab.ca/tutorials/dev-ops/automation/how-to-use-packer-to-create-ubuntu-18-04-vagrant-boxes/)
- [provisioning-development-environment](https://www.endpoint.com/blog/2014/03/14/provisioning-development-environment_14)

- [search?q=%22Delete+X11+libraries%22&type=Code](https://github.com/search?q=%22Delete+X11+libraries%22&type=Code)
- ["dpkg --list | awk '{ print $2 }' | grep linux-source | xargs apt-get -y purge"](https://github.com/search?l=Shell&q=%22dpkg+--list+%7C+awk+%27%7B+print+%242+%7D%27+%7C+grep+linux-source+%7C+xargs+apt-get+-y+purge%22&type=Code)
 
### Packer configuration file k-osint.json
```
{
  {
  "variables": {
    "vm_name": "k-osint",
    "disk_size": "80000",
    "iso_checksum": "93ea9f00a60551412f20186cb7ba7d1ff3bebf73",
    "iso_checksum_type": "sha1",
    "iso_url": "k-osint.iso",
    "box_name" : "k-osint", 
    "ssh_username": "vagrant",
    "ssh_password": "vagrant", 
    
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
      "shutdown_command": "echo '{{user `ssh_password`}}' | sudo -S /sbin/shutdown -hP now",
      "communicator": "ssh",
      "ssh_username": "vagrant",
      "ssh_password": "vagrant", 
      "ssh_port": 22,
      "ssh_wait_timeout": "60m",
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
   "provisioners": [
    		{
      "type": "shell",
      "execute_command": "echo '{{user `ssh_password`}}' | {{ .Vars }} sudo -S -E sh -eux '{{ .Path }}'",
      "scripts": [ "{{template_dir}}/scripts/cleanup.sh","{{template_dir}}/scripts/virtualbox.sh"],
      "expect_disconnect": true
      }]

}
```
### Notes packer configuration file
#### variables:
- W10, find SHA1 and SHA256
```
certutil -hashfile k-osint.iso SHA1
certutil -hashfile VBoxGuestAdditions.iso SHA256
```
#### boot_command:
- ```/install/vmlinuz noapic``` it tells the [kernel](https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html) to not make use of any [IOAPICs](https://wiki.osdev.org/IOAPIC) that may be present in the system.
#### provisioners:
Be careful to set ssh username and password to the same username/password of the preceed or it won't work.
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
- Remember to provide the **sudo rights to your scripts**. Most of the examples echo <something>, that probably is not the right password, and if you are doing it for the first time it is easier to overlook that you are piping the wrong password.Although, a better way to do this is not to hardcode the password, but to echo the ssh_pass variable, lastly do not forget to add the ssh_password to the list of variables otherwise it will fail.
```
"execute_command": "echo '{{user `ssh_password`}}' | {{ .Vars }} sudo -E -S sh '{{ .Path }}'",
```
	
---

![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)


## Preceed configuration file
Preseeding provides a way to set answers to questions asked during the installation process, without having to manually enter the answers while the installation is running. This makes it possible to fully automate most types of installation and even offers some features not available during normal installations. If you are installing the operating system from a mounted iso as part of your Packer build, you will need to use a preseed file. [Example](https://www.debian.org/releases/stable/example-preseed.txt) 

### References:
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
### Preseed configuration file
#### [source](https://gitlab.com/kalilinux/build-scripts/kali-vagrant/-/blob/master/http/preseed.cfg)
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

# root
d-i passwd/root-password password vagrant
d-i passwd/root-password-again password vagrant

d-i apt-setup/use_mirror boolean true
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean false
d-i grub-installer/bootdev string /dev/sda
d-i finish-install/reboot_in_progress note

# Enable SSH
d-i preseed/late_command string in-target systemctl enable ssh
```
### Notes preseed file
- **root password** do not forget to check if the root password has been set, for instance these lines were missing from the original file. Nb: some preseed files disable normal account creation, but in this preseed file there is a vagrant login account.
```# root
d-i passwd/root-password password vagrant
d-i passwd/root-password-again password vagrant
```

![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)


## Provisioners Scripts
Provisioning can be done in many stages and not only here, and in different ways. [Provisioners](https://packer.io/docs/provisioners/shell.html) use builtin and third-party software to install and configure the machine image after booting. Provisioners prepare the system for use, so common use cases for provisioners include:
- installing packages
- patching the kernel
- creating users
- downloading application code
#### It can happen in different ways such as
- inline
- shell
- ansible
- vagrant
```
   "provisioners": [
    		{
      "type": "shell",
      "execute_command": "echo '{{user `ssh_password`}}' | {{ .Vars }} sudo -S -E sh -eux '{{ .Path }}'",
      "scripts": [ "{{template_dir}}/scripts/cleanup.sh"],
      "expect_disconnect": true
      }]
```

```
{
  "type": "shell",
  "inline": [
    "sudo apt-get install -y git",
    "ssh-keyscan github.com >> ~/.ssh/known_hosts",
    "git clone git@github.com:exampleorg/myprivaterepo.git"
  ]
}
```
### Scripts: 
- [bonzofenix/scripts](https://github.com/bonzofenix/trainings/tree/master/bosh-lite/scripts)
- [xuxiaodong/scripts](https://github.com/xuxiaodong/kvm-example/tree/df0bbad6b0071bdd29d83ad4a5ee965fcd71e819/scripts)

![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)

## script cleaup.sh 
``` bash
#!/bin/sh -eux
logz='cleanup.log'

echo "##############################################################################"
echo "# 01_Update System                                                           #" | tee -a $logz
echo "##############################################################################"
apt-get -y -qq update | tee -a $logz
apt-get update --fix-missing | tee -a $logz
DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -o Dpkg::Options::='--force-confnew' | tee -a $logz
DEBIAN_FRONTEND=noninteractive apt-get dist-upgrade -y -o Dpkg::Options::='--force-confnew' | tee -a $logz
DEBIAN_FRONTEND=noninteractive apt-get autoremove -y -o Dpkg::Options::='--force-confnew' | tee -a $logz
DEBIAN_FRONTEND=noninteractive apt-get -y -qq dist-upgrade | tee -a $logz
DEBIAN_FRONTEND=noninteractive apt-get -y -qq install linux-headers-"$(uname -r)" | tee -a $logz

echo "##############################################################################"
echo "# 02_Cleaning                                                                #" | tee -a $logz
echo "##############################################################################"
echo"-----Delete linux-headers-----" | tee -a $logz
dpkg --list | awk '{ print $2 }' | grep 'linux-headers' | grep -vF "$(uname -r)" | xargs apt-get -y purge | tee -a $logz

echo "-----Delete linux-image-----" | tee -a $logz
dpkg --list | awk '{ print $2 }' | grep 'linux-image' | grep -vF "$(uname -r)" | xargs apt-get -y purge | tee -a $logz

echo "-----Delete residuals-----" | tee -a $logz
dpkg -l | grep '^rc' | awk '{print $2}' | xargs dpkg --purge| tee -a $logz

echo "-----Delete linux firmaware-----" | tee -a $logz
apt-get -y purge linux-firmware | tee -a $logz

echo "-----Delete X11 libraries-----" | tee -a $logz
apt-get -y purge libx11-data xauth libxmuu1 libxcb1 libx11-6 libxext6 | tee -a $logz

echo "-----Delete obsolete networking-----" | tee -a $logz
apt-get -y purge ppp pppconfig pppoeconf | tee -a $logz

echo "-----Delete oddities-----" | tee -a $logz
apt-get -y purge popularity-contest | tee -a $logz
apt-get -y purge installation-report | tee -a $logz

echo "-----Delete Packages-----" | tee -a $logz
apt-get -y autoremove | tee -a $logz
apt-get -y clean | tee -a $logz

echo "-----Truncate any logs that have built up during the install-----" | tee -a $logz
find /var/log -type f -exec truncate --size=0 {} \;| tee -a $logz
find /var/log/ -name "*.log" -exec rm -f {} \;  | tee -a $logz

echo "-----Cleaning up dpkg backup files-----" | tee -a $logz
find /var/cache/debconf -type f -print0 '*-old' | xargs rm -f | tee -a $logz

echo "-----Whiteout root-----" | tee -a $logz
count=$(df --sync -kP / | tail -n1  | awk -F ' ' '{print $4}') | tee -a $logz
count=$(($count-1)) | tee -a $logz
dd if=/dev/zero of=/tmp/whitespace bs=1M count=$count | cat "dd exit code $? is suppressed" | tee -a $logz
rm /tmp/whitespace | tee -a $logz

echo "-----Whiteout /boot-----" | tee -a $logz
count=$(df --sync -kP /boot | tail -n1 | awk -F ' ' '{print $4}')| tee -a $logz
count=$(($count-1)) | tee -a $logz

dd if=/dev/zero of=/boot/whitespace bs=1M count=$count | cat "dd exit code $? is suppressed" | tee -a $logz
rm /boot/whitespace | tee -a $logz

echo "-----Blank netplan machine-id (DUID) so machines get unique ID generated on boot.-----" | tee -a $logz
truncate -s 0 /etc/machine-id | tee -a $logz

echo "-----Zero out the free space to save space in the final image-----" | tee -a $logz
dd if=/dev/zero of=/EMPTY bs=1M || true | tee -a $logz
rm -f /EMPTY | tee -a $logz

echo "-----clear the history so our install isn't there-----" | tee -a $logz
export HISTSIZE=0 | tee -a $logz
rm -f /root/.wget-hsts | tee -a $logz

echo "##############################################################################"
echo "# 04_Reboot                                                                  #"| tee -a $logz
echo "##############################################################################"
shutdown -r now | tee -a $logz

```


### Provisioning with ansible playbook
#### How to create a playbook
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

![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)


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

![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)

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
![](https://raw.githubusercontent.com/frankietyrine/K-OSINT.iso/master/unnamed.png)

# FAQ
### Sudo Issues
- [packer-cant-execute-shell-provisioner-as-sudo](https://stackoverflow.com/questions/48537171/packer-cant-execute-shell-provisioner-as-sudo)
- [packer-build-fails-due-to-tty-needed-for-sudo](https://stackoverflow.com/questions/31788902/packer-build-fails-due-to-tty-needed-for-sudo)
- [sudo issues](https://stackoverflow.com/questions/34706972/simple-shell-inline-provisionning)
