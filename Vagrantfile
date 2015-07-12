# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|

  iptables_config = <<END
# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
END

  # I was unable to find an existing vagrant box that was close to a default
  # minimal CentOS 6 install. They either had selinux and/or the firewall
  # disabled. If selinux is disabled (not in permissive mode) you have to do a
  # reboot to put it in permissive mode which rules out most boxes out there.
  #
  # The box from chef is the best I could find, selinux is in permissive mode
  # and going from permissive to enforcing is just one command. Likewise,
  # enabling the firewall is just a matter of putting a config file in place and
  # restarting the firewall.
  config.vm.box = "chef/centos-6.6"
  config.vm.provision 'set_selinux_enforcing', type: 'shell', inline: 'setenforce enforcing;sed -i "/SELINUX=/cSELINUX=enforcing" /etc/selinux/config'
  config.vm.provision 'complicate_selinux', type: 'shell', inline: '/usr/sbin/setsebool httpd_enable_cgi=off httpd_use_cifs=off httpd_use_nfs=off allow_postfix_local_write_mail_spool=off'
  config.vm.provision 'install_and_enable_dbus', type: 'shell', inline: 'yum -q -y install dbus dbus-python; service messagebus start >/dev/null;chkconfig messagebus on'
  config.vm.provision 'configure_iptables', type: 'shell', inline: "echo '#{iptables_config}' > /etc/sysconfig/iptables; service iptables restart >/dev/null;chkconfig iptables on"

  # test1.example.com is used to set up the service you're practicing
  config.vm.define "test1" do |test1|
    test1.vm.hostname = 'test1.example.com'
    test1.vm.network :private_network, ip: "10.0.0.5", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end

  # test2.example.com is used to set up the service you're practicing
  config.vm.define "test2" do |test2|
    test2.vm.hostname = 'test2.example.com'
    test2.vm.network :private_network, ip: "10.0.0.6", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end

  # allowed.example.com is used to test the service you set up on
  # test.example.com. If you're suppoed to configure a service to block/allow
  # certain IPs, allowed.example.com should be able to access the service
  config.vm.define "allowed" do |allowed|
    allowed.vm.hostname = 'allowed.example.com'
    allowed.vm.network :private_network, ip: "10.0.0.7", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end

  # blocked.example.com is used to test the service you set up on
  # test.example.com. If you're suppoed to configure a service to block/allow
  # certain IPs, blocked.example.com should NOT be able to access the service
  config.vm.define "blocked" do |blocked|
    blocked.vm.hostname = 'blocked.example.com'
    blocked.vm.network :private_network, ip: "10.0.0.8", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end
end
