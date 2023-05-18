# SNOW Database Deployment Guide #

## Hardware and OS Requirements ##

* 1 server (or more, depending on HA scenario) of RHEL 8.x or later.
* The server should have a dedicated volume of at least 500GB ( or more, depending on organization requirements) 
* A glide base file (glide-base-VER.tar.gz)
* (For MariaDB installation - a mariadb custom installation package and a custom my.cnf file

## Deployment Steps ##

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
echo "/dev/snow_vg/snow_lv /glide xfs defaults 0 0" | tee -a /etc/fstab
```

### Step 2. OS Settings ###

1. Make sure that SELinux and firewalld is disabled:
```sh
setenforce 0
sed -i 's/enforcing/disabled/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```
2. Set our timezone to UTC
```sh
sudo timedatectl set-timezone UTC
```
3. Disable THP 
```sh
sudo sed -i '/^GRUB_CMDLINE_LINUX/s/"$/ transparent_hugepage=never"/' /etc/default/grub
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
5. Install the following RPM packages:
```sh
sudo yum -y install libaio
sudo yum -y install glibc
sudo yum -y install perl 
```
5. Kernel settings 
```sh
echo "* soft nproc 10240" | sudo tee -a /etc/security/limits.conf
echo "vm.swappiness=1" | sudo tee -a /etc/sysctl.conf
```
**Reboot the system**

### Step 3. Deploy the database (MariaDB) ###
1. Extract the base file
```sh
tar zxf glide-base-20220708.tar.gz --exclude='logs' --exclude='nodes' --exclude='temp' -C /glide
```
2. Add service users and groups
```sh
groupadd mysql
useradd mysql -g mysql
```
3.Extract the custom cnf file (Edit the content of innodb_buffer_pool and innodb_log_size to 70% of your servers memory)
~~~sh
tar zxvf ServiceNow_mariadb10.4_my.cnf-20220429-512GB.tar.gz -C .
cp ServiceNow_mariadb10.4_my.cnf-20220429-512GB /etc/my.cnf
~~~
4. Deploy the DB
~~~sh
cd /glide
tar -zxvpf mariadb-10.4.29-linux-systemd-x86_64.tar.gz
ln -s mariadb-10.4.29-linux-systemd-x86_64 mysql
cd mysql
mkdir temp
chown -HR mysql:mysql /glide/mysql
./scripts/mysql_install_db --user=mysql
chown -R root .
chown -R mysql data temp
ln -s /glide/bin/mysql.server-glide.sh /etc/init.d/mysql
ln -s /etc/init.d/mysql /etc/rc3.d/S99mysql
ln -s /etc/init.d/mysql /etc/rc3.d/K01mysql
~~~

5. Start the service, make sure its running and then enable it for system startup
~~~sh
sudo /etc/init.d/mysql start
sudo systemctl status mysql
^status^enable
~~~

6. Install mysql client if needed, and then login using root and add a username with the below command
   make sure to save this username, you will need it for the application deployment later on.
~~~sh
yum -y install mysql
~~~
~~~sql
mysql > GRANT ALL PRIVILEGES ON *.* TO <username>@'<source app IP>' IDENTIFIED BY '<some password>';
mysql> flush privileges;
~~~

