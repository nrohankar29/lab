# Vagrant RHCE lab setup

Vagrant files for RHCE exam preparation. <br/>
Learn more about vagrant at: https://www.vagrantup.com/ <br/>
Sahara vagrant plugin used for snapshots: https://github.com/jedi4ever/sahara


## Installation

### Ubuntu 16.04.1 LTS

#### Provider: virtualbox

1. Update repos: `sudo apt update && sudo apt upgrade`
2. Install vagrant and dependencies: `sudo apt -y install vagrant && sudo apt -y install zlib1g-dev`
3. Install virtualbox: `sudo apt -y install virtualbox`
4. Patch a file if vagrant version is 1.8.1 (If you manually installed latest vagrant, no need to patch) link: http://stackoverflow.com/questions/36811863/cant-install-vagrant-plugins-in-ubuntu-16-04/36991648
5. Install required vagrant plugins: `vagrant plugin install vagrant-vbguest && vagrant plugin install sahara`
6. Download centos vagrant box:	`vagrant box add centos/7` If asked for provider select virtualbox.
7. Clone this and run `vagrant up` inside lab/virtualbox

###CentOS 7.2/Fedora 24

#### Provider: libvirt

1. Update: `sudo yum update`
2. Install dependencies and vagrant: `sudo yum -y install vagrant redhat-rpm-config vagrant-libvirt vagrant-libvirt-doc libvirt-devel libxslt-devel libxml2-devel virt-manager`
3. Install required vagrant plugins: `vagrant plugin install vagrant-libvirt && vagrant plugin install fog && vagrant plugin install sahara`
4. Download centos 7 vagrant box: `vagrant box add centos/7` If asked for provider select libvirt.
5. Clone this and run `vagrant up` inside lab/libvirt


##### If using fedora replace yum by dnf
##### run `vagrant up --no-parallel` to prevent bringing up all VM's together when using libvirt

### Run these commands if you do not have a local repo

#### Ubuntu 16.04.1 LTS

```
sed -i '7,13s/^/#/g' virtualbox/scripts/classroom/classroom.sh
sed -i '12,18s/^/#/g' virtualbox/scripts/server/server.sh
sed -i '12,18s/^/#/g' virtualbox/scripts/desktop/desktop.sh
```

#### CentOS 7.2/Fedora 24

```
sed -i '7,13s/^/#/g' libvirt/scripts/classroom/classroom.sh
sed -i '12,18s/^/#/g' libvirt/scripts/server/server.sh
sed -i '12,18s/^/#/g' libvirt/scripts/desktop/desktop.sh
```

## Recommendations

If possible clone the official centos mirrors so that the packages are available locally. Find the nearest mirros from: https://www.centos.org/download/mirrors/ and change in scripts/repoupdate 

### Ubuntu 16.04.1 LTS

1. Run a cron job daily using the script, scripts/repoupdate 
2. Install apache webserver: `sudo apt -y install apache2 && sudo rm -f /var/www/html/index.html && sudo systemctl enable apache2 && sudo systemctl restart apache2'
3. Modify the ip address in the virtualbox/scripts folder to the ip of your base machine: 
```
find scripts/ -type f -name "*.sh" -exec sed -i 's/172.16.0.143/<your-ip>/g' {} \;
```

### CentOS 7.2/Fedora 24

1. Run a cron job daily using the script, scripts/repoupdate
2. Install apache webserver: `sudo yum -y install httpd && sudo sed -i s/^/#/g /etc/httpd/conf.d/welcome.conf && sudo systemctl enable httpd && sudo systemctl restart httpd`
3. Modify the ip address in the libvirt/scripts folder to the ip of your base machine: 
```
find scripts/ -type f -name "*.sh" -exec sed -i 's/172.16.0.143/<your-ip>/g' {} \;
```

#### Modify the scripts in scripts/,virtualbox/scripts,libvirt/scripts folders as per need

## Lab system info

domain: example.com <br/>
hosts: classroom.example.com, server1.example.com, desktop1.example.com <br/>
services: DNS, NTP, Kerberos, LDAP <br/><br/>
To access run:
```
vagrant ssh classroom
vagrant ssh server
vagrant ssh desktop
```
##### It is recommended not to modify any settings on classroom server unless required

#### classroom

hostname: classroom.example.com <br/>
ipv4: 192.168.33.254 <br/>
ipv6: 2000::254/64 <br/>
kdc server: classroom.example.com <br/>
REALM: EXAMPLE.COM <br/>
ldap server: classroom.example.com <br/>
DN: dc=example,dc=com <br>

users: <br/>
* user: vagrant, password: vagrant <br/>
* user: root, password: centos <br/>

ldap users:
* ldapuser1 <br/>
* ldapuser2 <br/>
* ldapuser3 <br/>

ldap user home directory: /home/guests <br/>
ldap authentication password for ldap users: password <br/>
kerberos authentication password for ldap users: kerberos <br/>
keytab files: <br/>

* server: http://classroom.example.com/keytab/server1.keytab <br/>
* desktop: http://classroom.example.com/keytab/desktop1.keytab <br/>

CA certificate: http://classroom.example.com/pki/example_ca.crt <br/>

For httpd: <br/><br/>
server1 key file: http://classroom.example.com/pki/tls/private/server1.key <br/>
server1 crt file: http://classroom.example.com/pki/tls/certs/server1.crt <br/>

Sample WSGI script: http://classroom.example.com/scripts/epoch.py <br/>

#### server1

hostname: server.example.com <br/>
ipv4: 192.168.33.11 <br/>
ipv6: set one in 2000::/64 range <br/>

users: <br/>
* user: vagrant, password: vagrant <br/>
* user: root, password: centos <br/>

script for checking teaming: /usr/local/scripts/teambridge.sh  <br/>
Make the script executable: `chmod u+x /usr/local/scripts/teambridge.sh` <br/>
and run `./usr/local/scripts/teambridge.sh` <br/>
##### teaming interfaces
* eno1 <br/>
* eno2 <br/><br/>
bridge ip: 10.10.0.254 <br/>
set any ip in the 10.10.0.0/24 network for the teamed interface <br/><br/>

secondary disk: /dev/sdb if using virtualbox, otherwise /dev/vdb <br/>

#### desktop1

hostname: desktop1.example.com <br/>
ipv4: 192.168.33.10 <br/>
ipv6: set one in 2000::/64 range <br/><br/>
users: <br/>

* user: vagrant, password: vagrant <br/>
* user: root, password: centos <br/>

secondary disk: /dev/sdb if using virtualbox, otherwise /dev/vdb <br/>


