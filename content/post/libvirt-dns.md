+++
categories = ["howto"]
draft = false
date = "2017-01-09T12:26:42Z"
title = "Libvirt DNS"

+++

Here's some short notes on how to configure DNS for libvirt managed VMs.

* Create a libvirt network, and add the following to it's XML:

```xml
<network>
  ...
  <domain name='my.domain.local'/>
  ...
</network>
```

* Ensure the instance has DHCP\_HOSTNAME set in network-scripts. I use virt-edit to do this:

```bash
virt-edit \
    -d $name \
    /etc/sysconfig/network-scripts/ifcfg-eth0 \
    -e 's/^DHCP_HOSTNAME=.*/DHCP_HOSTNAME="'$name'"/'
```

(above is taken from https://gist.github.com/brk3/8d029adf03fbfcd5b69f473cea451355)

* Ensure the gateway is available as a nameserver in /etc/resolv.conf:

```
nameserver 192.168.4.1
```

That's it, VMs should be pingable/contactable via their hostnames from the host.
