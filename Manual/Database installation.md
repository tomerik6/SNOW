# SNOW Database Prerequisites #

## Hardware and OS - Before installation day ##

* 1 server (or more, depending on HA scenario) of RHEL 8.x or later.
* The server should have a dedicated volume of at least 500GB ( or more, depending on organization requirements) 
* A glide base file (glide-base-VER.tar.gz)
* (For MariaDB installation - a mariadb custom installation package and a custom my.cnf file

## Installation day ##

### Step 1 - glide partition: ###

The glide partition is the partition where the database will reside. In this step we will run the following commands to initialize the empty partition and format it using the XFS file system, we 
will do so using LVM to create a logical volume.

1. First we'll create the LVM partition
```sh
sudo pvcreate /dev/sdb -ffy
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
2. Set our timezone to UTC
```sh
sudo timedatectl set-timezone UTC
```
3. Disable THP 
4. Install the following RPM packages:
```sh
sudo yum install libaio
sudo yum install glibc
sudo yum install perl 
```
5. Kernel settings 
```sh
echo "* soft nproc 10240" | sudo tee -a /etc/security/limits.conf
echo "vm.swappiness=1" | sudo tee -a /etc/sysctl.conf
```

### Step 3. Deploy the database (MariaDB) ###
1. Extract the base file
```sh
tar zxf glide-base-<date>.tar.gz --exclude='logs' --exclude='nodes' --exclude='temp' -C /glide
```
2. Add service users and groups
```sh
groupadd mysql
useradd mysql -g mysql
```
3.Extract the custom cnf file (Edit the content of innodb_buffer_pool and innodb_log_size to 70% of your servers memory)
~~~sh
tar zxvf ServiceNow_mariadb10.4_my.cnf-20220429-512GB.tar.gz -C .
cp ServiceNow_mariadb10.4_my.cnf-20220429-512GB.tar.gz /etc/my.cnf
~~~
4. Deploy the DB
~~~sh
cd /glide
tar -zxvpf mariadb-10.11.2-linux-systemd-x86_64.tar.gz
ln -s mariadb-10.11.2-linux-systemd-x86_64 mysql
cd mysql
mkdir temp
chown -HR mysql:mysql /glide/mysql
./scripts/mysql_install_db --user=mysql
chown -R root
chown -R mysql data temp
ln -s /glide/bin/mysql.server-glide.sh /etc/init.d/mysql
ln -s /etc/init.d/mysql /etc/rc3.d/S99mysql
ln -s /etc/init.d/mysql /etc/rc3.d/K01mysql
ln -s /etc/init.d/mysql /etc/rc5.d/S99mysql
ln -s /etc/init.d/mysql /etc/rc5.d/K01mysql
~~~

5.

