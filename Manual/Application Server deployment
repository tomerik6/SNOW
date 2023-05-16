# SNOW Application node deployment #

## Hardware and OS Requirements ##

* 1 server (or more, depending on HA scenario) of RHEL 9.x or later.
* A glide base file (glide-base-VER.tar.gz)
* The glide orbit package
* A JDK 1.8 / 11 (32bit) package

## Deployment Steps ##

### Step 1. OS Settings ###

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
3. Install the following RPM packages:
```sh
sudo yum install libgcc libgcc.i686
sudo yum install glibc glibc.i686
sudo yum install rng-tools
```
5. Kernel settings 
```sh
echo "* soft nproc 10240" | sudo tee -a /etc/security/limits.conf
echo "* soft nofile 16000" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 16000" | sudo tee -a /etc/security/limits.conf
echo "vm.swappiness=1" | sudo tee -a /etc/sysctl.conf
```

### Step 2. Deployment ###
1. Extract the base file
```sh
mkdir /glide
tar zxf glide-base-<date>.tar.gz -C /glide
```
2. Add service user
```sh
useradd servicenow
```
3. Install JDK
```sh
tar zxf jdk-<ver>.tar.gz -C /glide
ln -s /glide/jdk-<ver> /glide/java
export PATH=$PATH:/glide/java/bin
~~~
4. Extract the orbit package using java
> Note that 'node name' should be something similiar to prod1 or dev1
```sh
java -jar /<path to>/glide-<build-version>.zip --dst-dir /glide/nodes/<node-name>_16000 install -n <node-name> -p 16000
```
5. Then we'll change the ownership for the service account
```sh
chown -R servicenow:servicenow /glide/nodes/<NodeName>_<NodePort>/
```
6. Create a service unit file for the deployed scripts
```sh
vi /etc/systemd/system/snc_<NodeName>.service
```
```
[Unit]
Description=ServiceNow Tomcat Container
After=syslog.target
[Service]
Type=forking
ExecStart=/glide/nodes/<NodeName>_<NodePort>/startup.sh
ExecStop=/glide/nodes/<NodeName>_<NodePort>/shutdown.sh
User=servicenow
Group=servicenow
UMask=0007
LimitNOFILE=16000
[Install]
WantedBy=multi-user.target
```
```sh
systemctl daemon-reload
systemctl enable snc_<nodename>.service
```
Reboot the server and ensure the service starts up.

7.

