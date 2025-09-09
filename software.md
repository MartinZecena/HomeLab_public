# Software

## Proxmox

- convert from vms from VirtualBox to Proxmox 
  - convert .vdi into .img
 ```
vboxmanage clonehd --format RAW ./<source>.vdi ./<dest>.img
 ```
  - convert .img to .qcow2
```
qemu-img convert -f raw -O qcow2 ~/<source>.img ~/<dest>.qcow2
```
- upload the file with scp to proxmox server

### Metasploitable3 

- install Packer
//download and install hashicorp GPG key
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-			keyring.gpg

//add hashicorp repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -		cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

//install
sudo apt install packer
	
- install vagrant
sudo apt install vagrant
vagrant plugin install vagrant-reload

- download and start Metasploitable3
//clone repository
git clone https://github.com/rapid7/metasploitable3.git
cd ./metasploitable3
//repeat for windows2008 
packer build --only=virtualbox-iso packer/templates/ubuntu_1404.json
./build.sh ubuntu1404
wget -c http://old-releases.ubuntu.com/releases/14.04.0/ubuntu-14.04-server-amd64.iso //if previous command gets stuck
./build.sh ubuntu1404 windows2008

### Websploit
- create a kali vm
* 8 GB ram
* 3 vCPU
* 50 GB Disk

- run the following command inside the vm
curl -sSL https://websploit.org/install.sh | sudo bash

### ReverseExploitation VM
- create a debian 12 vm minimal installation
- separate partition for home/reverse3xploit/

- configure package manager 
```
 sudo tee /etc/apt/sources.list <<EOF
deb http://deb.debian.org/debian bookworm main contrib non-free
deb http://deb.debian.org/debian-security bookworm-security main contrib non-free
deb http://deb.debian.org/debian bookworm-updates main contrib non-free
EOF
sudo apt update

```
- Add DNS to /etc/resolv.conf
```
nameserver 1.1.1.1
```

- installed packages
  gdb, GEF,pwndbg, Ghidra(and dependencies), radare2, cutter(and dependencies), git, binutils, elfutils, openjdk-21-jdk
    build-essential, cmake, make, pip, python3-pwntools, ROPgadget,ropper,binwalk, ruby-full, one_gadget, hexedit, xxd, libc6-dbg, strace, ltrace, patchelf,  openssh-server, qemu-user, gdb-multiarch, man, 
    xorg fluxbox xterm , tmux
- add environment variables for ghidra and cutter
    
- remove internet access bridge, attach VLAN access brige
- activate ssh, generate keys, only allow public key authentification
