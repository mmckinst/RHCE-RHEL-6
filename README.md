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
* Use /proc/sys and sysctl to modify and set kernel runtime parameters.
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
