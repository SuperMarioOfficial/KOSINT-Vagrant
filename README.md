# K-OSINT.iso

### Light waight GUI
![](https://i.ytimg.com/vi/Twcm18oFmqs/maxresdefault.jpg)
After a research done, I came to the conclusion that xfce is probably the lightest and fasted gui. 
- [linux_distros/GUI_ram_consumption_comparison_updated](https://www.reddit.com/r/linux/comments/5l39tz/linux_distros_ram_consumption_comparison_updated/)
- [Power Use, RAM + Boot Times With Unity, Xfce, GNOME, LXDE, Budgie & KDE Plasma](https://www.phoronix.com/scan.php?page=article&item=ubu-1704-desktops&num=2)
- [Kali xfce](https://www.youtube.com/watch?v=Twcm18oFmqs)
- [Change from Gnome to xfce](https://www.youtube.com/watch?v=uIbG3SI5hd8)

### Packages
***$vi*** ``` kali-config/variant-default/package-lists/kali.list.chroot```
```
# You always want those
kali-linux-core

# Kali applications
#<package>

# Graphical desktop
kali-desktop-xfce
```
#### kali-linux-core: Base Kali Linux System – core items that are always included
```
    cifs-utils
    ftp
    gdisk
    iw
    lvm2
    mlocate
    netcat-traditional
    nfs-common

    openssh-server
    openvpn
    p7zip-full
    parted
    rfkill
    samba
    snmp

    sudo
    tcpdump
    testdisk
    tftp
    tightvncserver
    tmux
    unrar
    vim
    whois
```
#### kali-tools-information-gathering: Used for Open Source Intelligence (OSINT) & information gathering
```
    0trace
    arping
    automater
    braa
    cdpsnarf
    dmitry
    dnmap
    dnsenum
    dnsmap
    dnsrecon
    dnstracer
    dnswalk
    enum4linux
    fierce
    firewalk
    fping
    fragroute
    fragrouter
    ftester

    hping3
    ike-scan
    intrace
    irpas
    lbd
    maltego
    masscan
    metagoofil
    miranda
    nbtscan
    ncat
    netdiscover
    netmask
    nmap
    onesixtyone
    p0f
    recon-ng
    smbmap
    smtp-user-enum

    snmpcheck
    sparta
    sslcaudit
    ssldump
    sslh
    sslscan
    sslyze
    swaks
    thc-ipv6
    theharvester
    tlssled
    twofi
    unicornscan
    urlcrazy
    wafw00f
    wol-e
    xprobe
    zenmap
```
#### kali-tools-reporting: Reporting tools
```
maltego
```
