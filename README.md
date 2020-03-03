# K-OSINT.iso

## The plan 
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
### Packer
The VirtualBox Packer builder is able to create VirtualBox virtual machines and export them in the OVF format, starting from an ISO image. The builder builds a virtual machine by creating a new virtual machine from scratch, booting it, installing an OS, provisioning software within the OS, then shutting it down. The result of the VirtualBox builder is a directory containing all the files necessary to run the virtual machine portably.
- [packer-kali-linux/blob/master/kali.json](https://github.com/buffersandbeer/packer-kali-linux/blob/master/kali.json)
- [how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter](https://www.vlent.nl/weblog/2017/09/29/how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter/)
- [Kali-Packer repository](https://github.com/vortexau/Kali-Packer)
- [virtual-image-automation](https://blog.zaleos.net/virtual-image-automation/)
- [create-simple-centos-7-virtualbox-with-packer](https://softwaretester.info/create-simple-centos-7-virtualbox-with-packer/)
- [network_card_vbox](https://www.eanderalx.org/virtualization/8_network_card_vbox)
```
--nic<1-N> none|null|nat|natnetwork|bridged|intnet|hostonly|generic: Configures the type of networking for each of the VM's virtual network cards. Options are: not present (none), not connected to the host (null), use network address translation (nat), use the new network address translation engine (natnetwork), bridged networking (bridged), or use internal networking (intnet), host-only networking (hostonly), or access rarely used sub-modes (generic). These options correspond to the modes described in Section 6.2, “Introduction to Networking Modes”. 
```
```
Packer/
      |---<distribution>.json
      |---http/
      |       |--- preseed.cfg
      |---scripts/
              |--- init.sh
              |--- cleanup.sh
	      |--- ansible.sh
```
### tmp config file, below the packer verified one. 
```
{
  "variables": {
    "vm_name": "k-osint",
    "disk_size": 20480,
    "iso_checksum": "fe0fab66c49325c295a116cefd00ca94993efee0",
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
      "vboxmanage": [
        ["modifyvm","{{.Name}}","--memory","8000"],
        ["modifyvm","{{.Name}}","--cpus","3"], 
	["modifyvm","{{.Name}}","--audio","none"], 
	["modifyvm","{{.Name}}", "--nic1", "nat"]
	["modifyvm","{{.Name}}", "--nic2", "intnet"]
	["modifyvm","{{.Name}}", "--intnet2", "whonix"]

      ],
      "boot_command": [
        "<esc><wait>",
        "install preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg debian-installer=en_US auto locale=en_US kbd-chooser/method=us <wait>",
        "netcfg/get_hostname={{ .Name }} netcfg/get_domain=vagrantup.com fb=false debconf/frontend=noninteractive console-setup/ask_detect=false <wait>", 
	"console-keymaps-at/keymap=us keyboard-configuration/xkb-keymap=us <wait>",
        "<enter><wait>"
      ],
      "boot_wait": "10s",
      "guest_additions_path": "VBoxGuestAdditions_{{.Version}}.iso",
      "http_directory": "http",
      "shutdown_command": "echo 'vagrant' | sudo -S /sbin/shutdown -h 0",
      "ssh_password": "vagrant",
      "ssh_port": 22,
      "ssh_username": "vagrant",
      "ssh_wait_timeout": "10000s"
    }, 
    {
      "type": "qemu",
      "vm_name": "{{user `vm_name`}}",
      "iso_url": "{{user `iso_url`}}",
      "iso_checksum": "{{user `iso_checksum`}}",
      "iso_checksum_type": "{{user `iso_checksum_type`}}",
      "disk_size": "{{user `disk_size`}}",
      "ssh_wait_timeout": "30s",
      "shutdown_command": "shutdown -P now",
      "disk_size": "{{user `disk_size`}}",
      "format": "qcow2",
      "headless": false,
      "accelerator": "kvm",
      "http_directory": "http",
      "ssh_username": "vagrant",
      "ssh_password": "vagrant",
      "net_device": "virtio-net",
      "disk_interface": "virtio",
      "boot_command": [
        "<esc><wait>",
        "install <wait>",
        "preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg <wait>",
        "debian-installer=en_US <wait>",
        "auto <wait>",
        "locale=en_US <wait>",
        "kbd-chooser/method=us <wait>",
        "netcfg/get_hostname={{ .Name }} <wait>",
        "netcfg/get_domain=vagrantup.com <wait>",
        "fb=false <wait>",
        "debconf/frontend=noninteractive <wait>",
        "console-setup/ask_detect=false <wait>",
        "console-keymaps-at/keymap=us <wait>",
        "keyboard-configuration/xkb-keymap=us <wait>",
        "<enter><wait>"
      ], "qemuargs": [ 
           [ "-m", "512m" ],
           [ "--no-acpi", "" ]
      ]
    } 
  ],
  "provisioners": [
    {
      "type": "shell",
      "scripts": [
        "scripts/base.sh",
        "scripts/virtualbox.sh",
        "scripts/vagrant.sh",
        "scripts/ruby.sh",
        "scripts/puppet.sh",
        "scripts/chef.sh",
        "scripts/cleanup.sh",
        "scripts/zerodisk.sh"
      ],
      "pause_before": "10s",
      "override": {
        "virtualbox-iso": {
          "execute_command": "echo 'vagrant'|sudo -S bash '{{.Path}}'"
        }
      }
    }
  ],
  "post-processors": [  {
    "type": "vagrant",
    "keep_input_artifact": true,
    "output" : "{{user `box_name`}}",
    "only": ["virtualbox-iso"]

  }
  ]
}
```
### Packer verified and working
```
{
  "variables": {
    "vm_name": "k-osint",
    "disk_size": "50480",
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
      "vboxmanage": [
        ["modifyvm","{{.Name}}","--memory","8000"],
        ["modifyvm","{{.Name}}","--cpus","3"], 
	["modifyvm","{{.Name}}","--audio","none"], 
	["modifyvm","{{.Name}}", "--nic1", "nat"],
	["modifyvm","{{.Name}}", "--nic2", "intnet"],
	["modifyvm","{{.Name}}", "--intnet2", "whonix"]

      ],
      "boot_command": [
        "<esc><wait>",
        "install preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg debian-installer=en_US auto locale=en_US kbd-chooser/method=us <wait>",
        "netcfg/get_hostname={{ .Name }} netcfg/get_domain=vagrantup.com fb=false debconf/frontend=noninteractive console-setup/ask_detect=false <wait>", 
	"console-keymaps-at/keymap=us keyboard-configuration/xkb-keymap=us <wait>",
        "<enter><wait>"
      ],
      "boot_wait": "10s",
      "guest_additions_path": "VBoxGuestAdditions_{{.Version}}.iso",
      "http_directory": "http",
      "shutdown_command": "echo 'vagrant' | sudo -S /sbin/shutdown -h 0",
      "ssh_password": "vagrant",
      "ssh_port": 22,
      "ssh_username": "vagrant",
      "ssh_wait_timeout": "10000s"
    }
],
  "post-processors": [  {
    "type": "vagrant",
    "keep_input_artifact": true,
    "output" : "{{user `box_name`}}",
    "only": ["virtualbox-iso"]

  }
  ]
}
```

### Preceed
Preseeding provides a way to set answers to questions asked during the installation process, without having to manually enter the answers while the installation is running. This makes it possible to fully automate most types of installation and even offers some features not available during normal installations. If you are installing the operating system from a mounted iso as part of your Packer build, you will need to use a preseed file. [Example](https://www.debian.org/releases/stable/example-preseed.txt) 
- https://www.kali.org/dojo/preseed.cfg
- https://kali.training/topic/unattended-installations/
- [Full tutorial](https://www.debian.org/releases/stable/amd64/apb.en.html)
- [Automated Debian Install with Preseeding](https://www.youtube.com/watch?v=ndHi1sQWuH4)
- [preseed-kali-linux-from-a-mini-iso](https://medium.com/@honze_net/preseed-kali-linux-from-a-mini-iso-9ad622617241)


# Provisioning with ansible playbook
### How to create a playbook
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

