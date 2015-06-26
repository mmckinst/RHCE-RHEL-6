# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|

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
  config.vm.provision "shell",
  inline: 'setenforce Enforcing',
  inline: 'echo "H4sIAGGtjFUCA5WQQWuDQBCF7/6KgdxKp4QcvVndUEuMwWzIofRgdK0r7q7sjkj662tiCiX1ktMb5s33ZpgFrKUVQ962UBhdya/e5iSNhsFKIqHhdAZ3diQUTj5Wt3lvAUmu+3wEe0dGye8JNBVQLR1UshUwqjYEVhRGKaFLUb54T6NDwnp+vN0dOARhyHYcPpb+8tPz12l2DLLorpse+L9RDGAKQAWOchKAOCnb8+B1E+/fWPScsU3AWQTY3Og/XAeyUN2sJaE1s8b9ri07XppUdJe8q2DZGUuwWs0GNJCxdxaOFVrRiIJwkFRfL8HaOMLOmlqeJInyQv3+4yEuTJMk5t4PKjhOtNwBAAA=" | base64 -d | gunzip > /etc/sysconfig/iptables; service iptables restart'


  # test.example.com is used to set up the service you're practicing
  config.vm.define "test" do |test|
    test.vm.hostname = 'test.example.com'
    test.vm.network :private_network, ip: "10.0.0.5", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end

  # allowed.example.com is used to test the service you set up on
  # test.example.com. If you're suppoed to configure a service to block/allow
  # certain IPs, allowed.example.com should be able to access the service
  config.vm.define "allowed" do |allowed|
    allowed.vm.hostname = 'allowed.example.com'
    allowed.vm.network :private_network, ip: "10.0.0.6", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end

  # blocked.example.com is used to test the service you set up on
  # test.example.com. If you're suppoed to configure a service to block/allow
  # certain IPs, blocked.example.com should NOT be able to access the service
  config.vm.define "blocked" do |blocked|
    blocked.vm.hostname = 'blocked.example.com'
    blocked.vm.network :private_network, ip: "10.0.0.7", :netmask => "255.255.255.0",
    virtualbox__intnet: true
  end
end
