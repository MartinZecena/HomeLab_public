# Storage
this file contains configuration details for apropiate file handling, and information of how i optimized my workflow with correct data handling

## workflow optimizations

 - shared folders between NAS, devices and vms are the key to make my setup so flexible and allow working with either my laptop or my vms on the same data at the same time, it also enables the posibility to work without laptop. This workflow is optimized to devide resources more efficiently and provide a higher level of fault tolerance.
 
 - workflow pessimization - security optimization   all backup data will be encrypted 

 A simple example to understand the workflow concept: run a reverse exploitation vm on proxmox and use it only to run the program and use gdb for testing, everything else ghidra, vscode, radare2 runs on laptop.
 vm has a shared folder in /mnt/ with /mnt/Storage in proxmox, proxmox at the same time has mounted the NAS folder Storage in /mnt/Storage, laptop then mounts NAS Storage folder in /mnt/Storage this way laptop and vm have a folder in which they can share data and at the same time this data is safely stored in our NAS, important data will also be stored on a separate 1TB SSD on laptop for isolation in case of catastrophe with NAS, in case of catastrophe with NAS and laptop all important data will also be uploaded to the cloud.

## Storage Structure and backup
 - new user will be created on NAS with only access to the Storage folder
 
 - Directory organization
 ```
 NAS
 /nasStorage
    /laptopStorage
        /lab
        /devices
    /pc1Storage
    /pc2Storage
    /pi5Storage

Laptop
/storage
    /laptopStorage
        /lab
        /devices
    /pc1Storage
    /pc2Storage
    /pi5Storage

PC1
/pc1Storage

pc2
/pc2Storage

pc3
/pc3Storage

pi5
/pi5Storage
```
- /nasStorage will be used for shared storage and backup of /laptopStorage
- /storage will be used for backup data of lab and local storage
- /devices contains all backup files and necessary data to restore devices to a initial configuration
- /lab contains all necessary data to modularly restore home lab to a initial configuration

 - to update copies all hosts will first upload the updates to NAS, copies in laptop are then going to be updated 
### setting up
- In laptop on file /etc/fstab to mount the 1TB SSD add following line:
```
<UUID> ext4 defaults 0 2
```
In each device follow these steps to mount the nas:

- for security create a credentials file with username and password in /etc/ and change permissions
- add following command on file /etc/fstab to mount nas on devices,
```
//<nas_ip>/nasStorage /mnt/Storage cifs credentials=/etc/<credentialsfile>,uid=1000,gid=1000,iocharset=utf8,file_mode=0777,dir_mode=0755,_netdev 0 0
```




## Shared Folders

## Encryption