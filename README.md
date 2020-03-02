# K-OSINT.iso

## The plan:
- build the iso
- configure virtualbox with packer
- provisioning with ansible playbook
### maybe ... 
- build the iso
- create a vagrant box
- provisioning with ansible playbook


## Build the ISO
### steps:
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

## Automatic installation process
### Packer
The VirtualBox Packer builder is able to create VirtualBox virtual machines and export them in the OVF format, starting from an ISO image. The builder builds a virtual machine by creating a new virtual machine from scratch, booting it, installing an OS, provisioning software within the OS, then shutting it down. The result of the VirtualBox builder is a directory containing all the files necessary to run the virtual machine portably.
- [packer-kali-linux/blob/master/kali.json](https://github.com/buffersandbeer/packer-kali-linux/blob/master/kali.json)
- [how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter](https://www.vlent.nl/weblog/2017/09/29/how-to-create-a-debian-virtualbox-machine-with-packer-with-an-additional-host-only-adapter/)

```
Packer/
      |---ubuntu1804.json
      |---http/
      |       |--- preseed.cfg
      |---scripts/
              |--- init.sh
              |--- cleanup.sh
```

```
{
    "builders": [
        {
            "type": "virtualbox-iso",
            "boot_command": [
            "",
            "",
            "",
            "/install/vmlinuz",
            " auto",
            " console-setup/ask_detect=false",
            " console-setup/layoutcode=us",
            " console-setup/modelcode=pc105",
            " debconf/frontend=noninteractive",
            " debian-installer=en_US",
            " fb=false",
            " initrd=/install/initrd.gz",
            " kbd-chooser/method=us",
            " keyboard-configuration/layout=USA",
            " keyboard-configuration/variant=USA",
            " locale=en_US",
            " netcfg/get_domain=vm",
            " netcfg/get_hostname=vagrant",
            " grub-installer/bootdev=/dev/sda",
            " noapic",
            " preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg",
            " -- ",
            ""
            ],
            "boot_wait": "10s",
            "disk_size": 81920,
            "guest_os_type": "Ubuntu_64",
            "headless": false,
            "http_directory": "http",
            "iso_urls": [
            "iso/ubuntu-18.04-server-amd64.iso",
            "http://cdimage.ubuntu.com/ubuntu/releases/bionic/release/ubuntu-18.04-server-amd64.iso"
            ],
            "iso_checksum_type": "sha256",
            "iso_checksum": "a7f5c7b0cdd0e9560d78f1e47660e066353bb8a79eb78d1fc3f4ea62a07e6cbc",
            "ssh_username": "vagrant",
            "ssh_password": "vagrant",
            "ssh_port": 22,
            "ssh_wait_timeout": "10000s",
            "shutdown_command": "echo 'vagrant'|sudo -S shutdown -P now",
            "guest_additions_path": "VBoxGuestAdditions_{{.Version}}.iso",
            "virtualbox_version_file": ".vbox_version",
            "vm_name": "packer-ubuntu-18.04-amd64",
            "vboxmanage": [
            [
                "modifyvm",
                "{{.Name}}",
                "--memory",
                "1024"
            ],
            [
                "modifyvm",
                "{{.Name}}",
                "--cpus",
                "1"
            ]
            ]
        }
    ],
    "provisioners": [{
      "type": "shell",
      "scripts": [
        "scripts/init.sh",
        "scripts/cleanup.sh"
      ]
    }],
    "post-processors": [{
      "type": "vagrant",
      "compression_level": "8",
      "output": "ubuntu-18.04-{{.Provider}}.box"
    }]
  }
```

### Preceed
Preseeding provides a way to set answers to questions asked during the installation process, without having to manually enter the answers while the installation is running. This makes it possible to fully automate most types of installation and even offers some features not available during normal installations. If you are installing the operating system from a mounted iso as part of your Packer build, you will need to use a preseed file. [Example](https://www.debian.org/releases/stable/example-preseed.txt) 
- https://www.kali.org/dojo/preseed.cfg
- https://kali.training/topic/unattended-installations/
- [Full tutorial](https://www.debian.org/releases/stable/amd64/apb.en.html)
- [Automated Debian Install with Preseeding](https://www.youtube.com/watch?v=ndHi1sQWuH4)
- [preseed-kali-linux-from-a-mini-iso](https://medium.com/@honze_net/preseed-kali-linux-from-a-mini-iso-9ad622617241)


## provisioning
### Ansible
### Vagrant

### XFCE GUI
![](https://i.ytimg.com/vi/Twcm18oFmqs/maxresdefault.jpg)
After a research done, I came to the conclusion that xfce is probably the lightest and fasted gui. 
- [linux_distros/GUI_ram_consumption_comparison_updated](https://www.reddit.com/r/linux/comments/5l39tz/linux_distros_ram_consumption_comparison_updated/)
- [Power Use, RAM + Boot Times With Unity, Xfce, GNOME, LXDE, Budgie & KDE Plasma](https://www.phoronix.com/scan.php?page=article&item=ubu-1704-desktops&num=2)
- [Kali xfce](https://www.youtube.com/watch?v=Twcm18oFmqs)
- [Change from Gnome to xfce](https://www.youtube.com/watch?v=uIbG3SI5hd8)
- [How To Install Xfce4 & MATE Desktop Environments On Kali Linux](https://www.youtube.com/watch?v=hy9F87basCI)

## Links
- [create-kali-linux-iso](https://www.cybrary.it/blog/0p3n/create-kali-linux-iso/)
- [live-build-config-examples](https://gitlab.com/kalilinux/recipes/live-build-config-examples/-/blob/master/kali-linux-mate-top10-nonroot.sh)
- [dojo-mastering-live-build/](https://www.kali.org/docs/development/dojo-mastering-live-build/)
- [kali-metapackages](https://tools.kali.org/kali-metapackages)
- [creating-an-Ubuntu-VM-with-packer](https://kappataumu.com/articles/creating-an-Ubuntu-VM-with-packer.html)
- [how-to-use-packer-to-create-ubuntu-18-04-vagrant-boxes](https://www.serverlab.ca/tutorials/dev-ops/automation/how-to-use-packer-to-create-ubuntu-18-04-vagrant-boxes/)
- [whonix-kali](https://github.com/j7k6/vagrant-whonix-kali)
