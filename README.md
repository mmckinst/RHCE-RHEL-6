# Misc
## Test taking tips
* http://blog.remibergsma.com/2013/10/07/my-tips-for-the-red-hat-rhcsa-rhce-exam/
## Study material
* http://davidpint.org/rhce/index.php?title=Main_Page
* https://github.com/bnugent/rhce-study
* http://b.joaoubaldo.com/linux/rhce-my-study-guide/
* https://github.com/texastwister/OpenRHCE

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
```
yum install httpd

# open up port 80 and 443
system-config-firewall-tui
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT

# copy the vhost example from /etc/httpd/conf/httpd.conf and in /etc/httpd/conf.d/foo.conf
# add the AllowOverride thing in, copied from elsewhere in httpd.conf
[root@localhost httpd]# cat /etc/httpd/conf.d/foo.conf
NameVirtualHost *:80
<VirtualHost *:80>
    ServerAdmin webmaster@foo.com
    DocumentRoot /www/docs/foo.com
    ServerName foo.com
    ErrorLog logs/foo.com-error_log
    CustomLog logs/foo.com-access_log common
<Directory />
    AllowOverride All
</Directory>

</VirtualHost>
[root@localhost httpd]# 

# configure selinx
chcon -Rv --reference /var/www/html/ /www/docs/foo.com/

# block access to certain IPs
echo -e 'Order deny,allow\nDeny from 192.168.56.6\n' >> /www/docs/foo.com/.htaccess

chkconfig httpd on
service httpd start
```
* Configure private directories.
```
# same as above

# install httpd-manual package
# rpm -ql httpd-manual | xargs -n1 grep AuthType
# should grep /var/www/manual/howto/auth.html
# open file and find the code to copy and put in vhost
# change the Require user line to 'Require valid-user', also documented in the
# same config file
   AuthType Basic
   AuthName "Restricted Files"
   # (Following line optional)
   AuthBasicProvider file
   AuthUserFile /www/docs/foo.com/.htpasswd
   Require valid-user

# restart httpd because of the new stuff added
service httpd restart

# create the file for the user foo
htpasswd -c /www/docs/foo.com/.htpasswd foo

# fix selinux
chcon -Rv --reference /var/www/html/ /www/docs/foo.com/
```
* Deploy a basic CGI application.
```
# Same as above

# open up httpd.conf and search for ExecCGI to find the part about AddHandler
# pay attention to the line above it that says you need to add 'Options ExecCGI'
# add to vhost like follows
Options ExecCGI
AddHandler cgi-script .cgi
AddHandler cgi-script .pl

# restart apache to re-read the vhost
service httpd restart

# copy hello world example from /var/www/manual/howto/cgi.html
[root@localhost httpd]# cat /www/docs/foo.com/hello.pl
#!/usr/bin/perl
print "Content-type: text/html\n\n";
print "Hello, World.";
[root@localhost httpd]# 

# fix perms and adjust selinux
chmod 755 /www/docs/foo.com/hello.pl
chcon -Rv --reference /var/www/html/ /www/docs/foo.com/
```
* Configure group-managed content.
```
# this means to allow a group of users to manage the content of a website (eg
# members of webdevs group can change the website but members of dba group
# can't). it doesn't mean to restrict a certain group to a website.

# create group
groupadd webdevs

# put users in the webdev group
usermod -a -G webdevs alice
usermod -a -G webdevs bob

# set owner and group
chown -Rv apache:webdevs /www/docs/foo.com/

# set the sticky bit for the group
find /www/docs/foo.com/ -type d -print0 | xargs -0 chmod -v 2775
```
## DNS
* Configure a caching-only name server.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-BIND.html
yum install bind

# add/change the following in /etc/named.conf
# any will listen/respond on all ports
# allow-query-cache is not in the default config, that line has to be added

listen-on port 53 { any; };
listen-on-v6 port 53 { any; };
allow-query { any; };
allow-query-cache { any; };

listen-on port 53 { 127.0.0.1; 192.168.122.0/24; };
listen-on-v6 port 53 { ::1; fc00::/7; };
allow-query { 127.0.0.1; 192.168.122.0/24; };
allow-query-cache { 127.0.0.1; 192.168.122.0/24; };

# you can use an ACL too
acl good_servers { 127.0.0.1; 10.0.0.0/8; };
acl bad_servers { 192.168.0.0/16; };

allow-query { good_servers; };
allow-query-cache { good_servers; };
blackhole { bad_servers; };

# edit the firewall or write iptables rules to /etc/sysconfig/iptables to allow port 53 
system-config-firewall-tui
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 53 -j ACCEPT
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 53 -j ACCEPT

# chkconfig the service on and restart it
chkconfig named on
service named start
```
* Configure a caching-only name server to forward DNS queries.
```
# same as above but add the following to the config file
forward only;
forwarders {8.8.8.8; 8.8.4.4;}
```

## FTP
* Configure anonymous-only download.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-ftp-servers-vsftpd.html
yum install vsftpd

# change/set the following in /etc/vsftpd/vsftpd.conf
#
# local_enable will forbid local users from logging in. RHCE says to allow
# anonymous-only which doesn't include local users
#
# write_enable will disable commands that can modify a file like DELE, RNFR, and STOR
anonymous_enable=YES
local_enable=NO
write_enable=NO

# adjust firewall as necessary
system-config-firewall-tui

# block access for IPs using tcpwrappers
echo 'vsftpd: ALL' >> /etc/hosts.deny
echo 'vsftpd: 192.168.56.5' >> /etc/hosts.allow

# put stuff in a different directory
mkdir /anon_ftp
usermod -d /anon_ftp/ ftp
chcon -Rv --reference /var/ftp/pub/ /anon_ftp

# chkconfig the service on and start it
chkconfig vsftpd on
service vsftpd start
```

## NFS
* Provide network shares to specific clients.
```
yum groupinstall "NFS file server"

# configure firewall
system-config-firewall-tui
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2049 -j ACCEPT

# add stuff to /etc/exports
/www	*(ro,sync) 192.168.56.5(rw,sync)

# chkconfig and start services
chkconfig rpcbind on
chkconfig nfs on
chkconfig nfslock on

service rpcbind start
service nfs start
service nfslock start
```
* Provide network shares suitable for group collaboration.

## SMB
* Provide network shares to specific clients.
* Provide network shares suitable for group collaboration.

## SMTP
* Configure a mail transfer agent (MTA) to accept inbound email from other systems.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-email-mta.html#s2-email-mta-postfix
# http://wiki.centos.org/HowTos/postfix
yum install postfix
chkconfig sendmail off
service sendmail stop
alternatives --config mta

# connfigure postfix and accept mail for certain domains
# in /etc/postfix/main.cf set the following
myhostname = herp.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain, foo.com, bar.com
mynetworks = 192.168.56.0/24, 127.0.0.0/8

# use 'postconf' to see the entire postfix config
# use 'postconf -n' to see only the parts changed from the default value

# configure the firewall
system-config-firewall-tui
-A INPUT -m state --state NEW -m tcp -p tcp --dport 25 -j ACCEPT

chkconfig postfix on
service postfix restart
```
* Configure an MTA to forward (relay) email through a smart host.

## SSH
* Configure key-based authentication.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-ssh-configuration-keypairs.html
yum install openssh-server

# disable password authentication
echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config

# generate and copy a key
ssh-keygen -b 8192
ssh-copy-id root@192.168.56.3

# add a user and put a key in place for them
adduser mmckinst
mkdir ~mmckinst/.ssh
echo 'key' ~mmckinst/.ssh/authorized_keys
chmod 700 ~mmckinst/.ssh
chmod 600 ~mmckinst/.ssh/authorized_keys
chown -Rv mmckinst:mmckinst ~mmckinst/.ssh*

# restrict access via IP
echo 'sshd: ALL' >> /etc/hosts.deny
echo 'sshd: 192.168.56.5' >> /etc/hosts.allow
iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -s 192.168.56.5 -j ACCEPT
# restrict access to certain users
echo 'AllowUsers mmckinst' >>  /etc/ssh/sshd_config

# chkconfig and start the service
chkconfig sshd on
service sshd start
```
* Configure additional options described in documentation.
```
# same as above, just edit /etc/ssh/sshd_config as necessary
```

## NTP
* Synchronize time using other NTP peers.
```
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2_Adding_a_Peer_Address.html
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2_Adding_a_Server_Address.html

# use 'server' if the server is a higher stratum. use 'peer' if the server is of the same stratum

yum install ntp

# remove the nopeer option in /etc/ntp.conf
restrict default kod nomodify notrap noquery

# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-Configure_NTP.html#s2-Configure_Access_Control_to_an_NTP_service
# restrict access to certain IP address
# copy from the default line but remove the 'noquery' option
restrict 192.168.56.0 mask 255.255.255.0 nomodify notrap

# open up udp 123
system-config-firewall-tui
iptables -A INPUT -m state --state NEW -m udp -p udp --dport 123 -j ACCEPT

# chkconfig the service and start it
chkconfig ntpd on
service ntpd start
```
