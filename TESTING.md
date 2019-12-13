###Local test bed architecture / specs

***posmom101.localdomain.local***  
Puppet Open Source Master of Masters - MoM  
1 vCPU / 1024MB / 16GB  

***poscmprxy111.localdomain.local***  
Puppet Open Source Compile Master Reverse Proxy  
1 vCPU / 512MB / 16GB  

***poscm121.localdomain.local***  
Puppet Open Source Compile Master - CM  
1 vCPU / 512MB / 16GB  

***poscm122.localdomain.local***  
Puppet Open Source Compile Master - CM  
1 vCPU / 512MB / 16GB  

***posagent131.localdomain.local***  
Puppet Open Source Agent - production  
1 vCPU / 512MB / 8GB  

***posagent132.localdomain.local***  
Puppet Open Source Agent - development  
1 vCPU / 512MB / 8GB  


###Test bed systems CentOS 7.x Kickstart template
*substitute jinja tags with real values*

```
#version=DEVEL
# System authorization information
auth --enableshadow --passalgo=sha512
# Use network installation
url --url="https://mirrors.kernel.org/centos/7/os/x86_64"
# Use graphical install
install
graphical
# Run the Setup Agent on first boot
firstboot --disable
ignoredisk --only-use=sda
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8

# Network information
network  --bootproto=static --device={{ network.device }} --gateway={{ network.gateway }} --ip={{ network.ip }} --nameserver={{ network.nameserver }} --netmask={{ network.netmask }} --ipv6=auto --activate
network  --hostname={{ network.hostname }}.{{ network.domainname }}

# Root password
rootpw --plaintext puppet
# System services
services --disabled="chronyd"
# System timezone
timezone America/Los_Angeles --isUtc --nontp
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
# Partition clearing information
clearpart --all --initlabel
zerombr
# Disk partitioning information
part /boot --fstype="xfs" --ondisk=sda --size=512
part pv.157 --fstype="lvmpv" --ondisk=sda --size=1024 --grow
part /boot/efi --fstype="efi" --ondisk=sda --size=256 --fsoptions="umask=0077,shortname=winnt"
volgroup centos_{{ network.hostname }} --pesize=4096 pv.157
logvol /  --fstype="xfs" --size=1024 --name=root --vgname=centos_{{ network.hostname }} --grow
logvol swap  --fstype="swap" --size=1024 --name=swap --vgname=centos_{{ network.hostname }}

%packages
@^minimal
@core
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
```