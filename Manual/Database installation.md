# SNOW Database Prerequisites #

## Hardware and OS - Before installation day ##

* 1 server (or more, depending on HA scenario) of RHEL 8.x or later.
* The server should have a dedicated volume of at least 500GB ( or more, depending on organization requirements) 
* A glide base file (glide-base-VER.tar.gz)

## Installation day ##

### Step 1 - glide partition: ###

The glide partition is the partition where the database will reside. In this step we will run the following commands to initialize the empty partition and format it using the XFS file system, we 
will do so using LVM to create a logical volume.

1. First we'll create the LVM partition
```sh
sudo pvcreate /dev/sdb
sudo vgcreate snow_vg  /dev/sdb
sudo lvcreate -L <Size-of-LV> -n snow_lv   snow_vg
```

2.Format it with the XFS filesystem and mount it to /glide
```sh
sudo mkfs.xfs /dev/snow_vg/snow_lv
sudo mkdir /glide
sudo mount /dev/snow_vg/snow_lv /glide
df -Th /glide
```

### Step 2. OS Settings ###

1. Make sure that SELinux and firewalld is disabled:
```sh
setenforce 0
sed -i 's/enforcing/disabled/' /etc/selinux/config
systemctl systemctl stop firewal
systemctl disable firewalld
```
2. 


