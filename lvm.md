
## Expand existing file system 

In order to expand a file system that runs LVM, you need to expand the virtual 
disk and the logical disk. 

After that, you're able to expand the file system.
 
There are 2 ways of expanding a disk: 

[*] Expand an existing disk

[*] Add a new disk, and add it to the volume group 
  
###   Expand an existing disk (VMware)

(This is the preferred way)

[1] Expand the disk in VMware 
![vmware disk][logo]

[logo]: https://github.com/M00kaw/m00kaw.github.com/blob/master/vmware_disk_00.png

Linux then needs to scan for the expanded disk, in order to register that the size has increased: 

`echo 1 | sudo tee /sys/class/scsi_device/<scsidevice>/device/rescan`

e.g: 

`echo 1 | sudo tee /sys/class/scsi_device/32:0:0:0/device/rescan`

The lazy person would just scan all devices: 

`for device in /sys/class/scsi_device/*/device/rescan ; do echo 1 | sudo tee $device ; done`

Now all we need, is to make LVM expand the physical volume: 

`sudo pvresize <device>`

e.g:

`sudo pvresize /dev/sdb`


### Add a new disk 

Add a new disk in VMware, and rescan the scsi controller: 

`echo "- - -" | sudo tee /sys/class/scsi_host/<controllerid>/scan`

e.g:

`echo "- - -" | sudo tee /sys/class/scsi_host/host32/scan`

(pre-requisite: /dev/sdb --> storagevg)


Add the new disk to existing disk group (in this case "storagevg"): 

`sudo vgextend storagevg /dev/sdc`

### Expand the logical volume 

We almost always want to use all available space: 

`sudo lvextend -l +100%FREE <vg>/<lv>`

e.g: 

`sudo lvextend -l +100%FREE storagevg/storagelv`

### expand the file system 

`sudo resize2 <lvmdevice>`

e.g:

`sudo resize2fs /dev/mapper/storagevg-storagelv`

