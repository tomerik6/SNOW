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
```
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
**Reboot the server and ensure the service starts up**


>Note: This step onwards requires a database deployment, see [Database Deployment](/Manual/Database installation.md)
7. Configure database connection and properties:
 ```vi /glide/nodes/<NodeName>_<NodePort>/conf/glide.db.properties ```
 For MariaDB Deployments
 ```
glide.db.name = <DatabaseName>
glide.db.rdbms = mysql
glide.db.url = jdbc:mysql://<Remote DB host>:3306/
glide.db.user = <username>
glide.db.password = <password>
```
For oracle deployments
```
glide.db.name = <DatabaseName>
glide.db.rdbms = oracle
glide.db.url = jdbc:oracle:thin:@<DB hostname>:1521:<SID>
glide.db.user = <username>
glide.db.password = <password>
glide.db.truncate_utf8 = true
```
8.Configure other settings
```vi /glide/nodes/<Nodename>/conf/glide.properties```
```
glide.proxy.host = https://example.service-now.com (should be set to your URL)
glide.proxy.path = /
glide.servlet.port = <NodePort>
glide.cluster.node_name = <NodeName>
glide.db.pooler.connections = 32
glide.db.pooler.connections.max = 32
glide.monitor.url = localhost
glide.self.monitor.fast_stats = false
glide.self.monitor.checkin.interval = 86400000
glide.self.monitor.server_stats.interval = 86400000
glide.self.monitor.fast_server_stats.interval = 86400000
glide.installation.self_hosted=true
glide.usageanalytics.central_instance=https://disabled.service-now.com
glide.ua.downloader.central_instance=
```
9. Restart the snow service for all of these changes to take effect:
```sh
systemctl restart snc_<node>.service
```
10. Run the tlog.sh script, a small wrapper script that tails the current log, to verify that the database was created
and the schema and data are being populated correctly. This process can take up to a couple hours depending
on your hardware deployed.
```sh
/glide/nodes/<node>/tlog.sh
```
Once messages in the log file similar to: "worker.0" or "worker.1" are observed the ServiceNow now
zboot process should be complete. Use a web browser to connect directly to the application server with a URL
similar to the following ```http://<application server>:16000```

After everything is up and working, see [Post Deployment]
