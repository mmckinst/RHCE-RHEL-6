# Misc
## SELinux
```
# check and change SELinux mode
sestatus
setenforce enforcing
setenforce permissive
$VISUAL /etc/selinux/config

# check SELinux booleans
getsebool -a | grep http
getsebool httpd_enable_cgi

# change SELinux booleans, use -P to write the policy to the disk so it will persist across reboots
setsebool httpd_enable_cgi off
setsebool -P httpd_enable_cgi off

# show context of a file
ls -lZ

# set or copy the context of a file
# use semanage fcontext so it will persist across reboots
# use -R just like chown and chgrp to do it recursively
chcon -u system_u -t httpd_sys_content_t /var/www/html/index.html
chcon --reference /var/www/html/ /var/www/html/index.html
semanage fcontext -a -s system_u -t httpd_sys_content_t /var/www/html/

# restore context if you screw something up
# use -R just like chown and chgrp to do it recursively
restorecon /var/www/html/
```
# SYSTEM CONFIGURATION AND MANAGEMENT
* Route IP traffic and create static routes.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-networkscripts-static-routes.html

# show routes
ip r

# add a route
ip r add 192.168.10.0/24 via 10.0.0.1
ip r add 192.168.10.0/24 via 10.0.0.1 dev eth0
ip r add default via 10.0.0.1
ip r add default via 10.0.0.1 dev eth0

# delete a route
ip r del 192.168.10.0/24
ip r del default

# add a static route
echo '192.168.10.0/24 via 10.0.0.1' >> /etc/sysconfig/network-scripts/ifcfg-eth0
echo '192.168.10.0/24 via 10.0.0.1 dev eth0' >> /etc/sysconfig/network-scripts/ifcfg-eth0
```
* Use iptables to implement packet filtering and configure network address translation (NAT).
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Firewalls-Common_IPTables_Filtering.html
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Firewalls-FORWARD_and_NAT_Rules.html
# configure iptables to start on boot and start it
chkconfig iptables on
service iptables start

# show current packet filtering
iptables -nL

# add a new firewall rule using a TUI can be faster and less error/typo prone
# than doing by hand
system-config-firewall-tui

# add a new firewall rule using iptables
# just copy existing line and modify as necessary from /etc/sysconfig/iptables
# or write one like below
# iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
emacs /etc/sysconfig/iptables

# configure NAT
sysctl -w net.ipv4.ip_forward=1
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
INSERT MORE STUFF HERE
INSERT MORE STUFF HERE
INSERT MORE STUFF HERE
```

* Use /proc/sys and sysctl to modify and set kernel runtime parameters.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-proc-sysctl.html

# get runtime parameter
sysctl vm.swappiness
sysctl -a | grep vm.swappiness
cat /proc/sys/vm/swappiness

# modify runtime parameter
sysctl -w vm.swappiness=50
echo '50' > /proc/sys/vm/swappiness

# modify runtime parameter to persist across reboots
echo 'vm.swappiness=50' >> /etc/sysctl.conf
```
* Configure a system to authenticate using Kerberos.
* Configure a system as an iSCSI initiator that persistently mounts an iSCSI target.
* Produce and deliver reports on system utilization (processor, memory, disk, and network).
* Use shell scripting to automate system maintenance tasks.
```
man bash
```
* Configure a system to log to a remote system.
* Configure a system to accept logging from a remote system.

# NETWORK SERVICES

Network services are an important subset of the exam objectives. RHCE candidates should be
capable of meeting the following objectives for each of the network services listed below:

* Install the packages needed to provide the service.
* Configure SELinux to support the service.
* Configure the service to start when the system is booted.
* Configure the service for basic operation.
* Configure host-based and user-based security for the service.

## HTTP/HTTPS
* Configure a virtual host.
* Configure private directories.
* Deploy a basic CGI application.
* Configure group-managed content.

## DNS
* Configure a caching-only name server.
* Configure a caching-only name server to forward DNS queries.
*Note: Candidates are not expected to configure master or slave name servers.*

## FTP
* Configure anonymous-only download.

## NFS
* Provide network shares to specific clients.
* Provide network shares suitable for group collaboration.

## SMB
* Provide network shares to specific clients.
* Provide network shares suitable for group collaboration.

## SMTP
* Configure a mail transfer agent (MTA) to accept inbound email from other systems.
* Configure an MTA to forward (relay) email through a smart host.

## SSH
* Configure key-based authentication.
* Configure additional options described in documentation.

## NTP
* Synchronize time using other NTP peers.
