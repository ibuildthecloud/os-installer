#os-installer: Installs RancherOS
-----

##Purpose:

This repository provides scripts for the container used to install RancherOS.

The container can be used directly, but there is a wrapper in the Operating system `/usr/sbin/rancheros-install` that handles calling things in the right order. If you just want to disect it, start with the rancheros-install script and look in here to see what is being called.

##Basics

When booting from the ISO file RancherOS runs completely from memory. In order to run more containers, and save state between reboots, you need to persist and run from disk. 

When booting, RancherOS looks for a device labeled "RANCHER_STATE". If it finds a volume with that labeled the OS will mount the device and use it to store state. 

The scripts in this container will create a device labeled RANCHER_STATE and make it bootable. The two supported methods, are virtualbox-iso-vagrant and amazon-ebs. The approach can be translated to suit different needs.

The virtualbox-iso install type follows these steps:

1. ) partition device with a single partition the size of the disk.
2. ) format ext4 and label partition as RANCHER_STATE
3. ) Install grub2 on device
4. ) Place Kernel/Initrd and grub.cfg inside /boot on the device.
5. ) Seeds the cloud-init data so that the vagrant insecure key can be used.

The amazon-ebs approach follows these steps:

1. ) format the device (Ext4) and label RANCHER_STATE
2. ) Add PV-GRUB configuration (menu.lst)
3. ) Add Kernel and Initrd
4. ) Sets Rancher to look for EC2 cloud-init data.

The upgrade process looks like:

Assumes the device is bootable.

1. ) Mount RANCHER_BOOT if found, else RANCHER_STATE.
2. ) Clobber the grub2 or menu.lst config with current version and the rollback version if provided. 

## Usage

**Warning:** Using this container direclty can be like running with scissors...

```
 # Partition disk without prompting of any sort:
 sudo system-docker run --privileged --net=host -it --entrypoint=/scripts/set-disk-partitions rancher/os-installer:<version> <device>


 # install 
 sudo system-docker run --privileged --net=host -it --volumes-from=user-volumes rancher/os-installer:<version> -d <device> -t <install_type> -c <cloud file>
 
 #Upgrade
 sudo system-docker run --privileged --net=host -it rancher/os-installer:<version> -t rancher-upgrade -r <rollback_version>
 
```

The installation process requires a cloud config file. It needs to be placed in either /home/rancher/ or /opt/. The installer make use of the user-volumes to facilitate files being available between system containers.

 
## License
Copyright (c) 2014-2015 [Rancher Labs, Inc.](http://rancher.com)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.








