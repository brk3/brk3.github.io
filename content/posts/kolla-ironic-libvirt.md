+++
categories = ["virtualisation", "openstack", "howto"]
date = "2018-04-19T12:46:12+01:00"
title = "Deploying virtual baremetal with Ironic & Kolla"

+++

A common barrier to entry when looking to try out OpenStack Ironic is that you need a baremetal
machine to use with it. It turns out that it is possible to use a virtual machine in lieu of
baremetal, and this is exactly what tools such as devstack and tripleo allow for.
This guide details how the same setup can be done using Kolla. Given the complexity of Ironic and
the variety of configurations possible, almost every step will have a multitude of variations and
possibilities, I just detail what worked for me.

Setup
=====

libvirt
-------
My host node is using the default libvirt network, which is 192.168.0.0/24. Attached to virbr0, I
have the standard set of nodes required to deploy a multinode OpenStack using Kolla. In addition, I
have created an extra guest to act as the baremetal, named 'virtual-baremetal'. There's nothing
special about the VM but the XML is posted [here][0] for reference.

```
$ virsh list
 Id    Name                  State
------------------------------------
 95    operator              running
 96    control01             running
 97    control02             running
 98    control03             running
 101   network01             running
 102   network02             running
 103   compute01             running
 104   compute02             running
 105   storage01             running
 106   storage02             running
 123   storage03             running
 128   virtual-baremetal     shut off
```

Each node has 4 NICs, I also grabbed the following snippet from the Ironic devstack scripts to allow
VNC access to the console:

```
<graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0'>
  <listen type='address' address='0.0.0.0'/>
</graphics>
<video>
  <model type='cirrus' vram='16384' heads='1' primary='yes'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
</video>
```

This proved to be valuable when debugging as the PXE boot process doesn't appear to attach to the
same console as exposed by 'virsh console'.

Because I'm using flat networking, I needed to disable DHCP from the libvirt network to prevent it
from trying to intercept boot requests from the Ironic target. To do this you can either remove the
\<dhcp\> element from the network XML, or in my case, I just killed the libvirt managed dnsmasq
processes completely and ran my own for extra control.

vbmc
----
On the host node we need to setup a virtual baseboard management controller (BMC) to allow Ironic to
control the node. This can be accomplished using vbmc:

```
yum install python-pip python-devel libvirt-devel gcc
pip install virtualbmc
vbmc add virtual-baremetal --port 6230 --username admin --password password --address 0.0.0.0
vbmc start --libvirt-uri=qemu:///session virtual-baremetal
```

You can use ipmitool to verify it's working:

```
ipmitool -I lanplus -H <host node ip> -L ADMINISTRATOR -p 6230 \
  -U admin -R 12 -N 5 -P password power status
```

Deploy
======
At this point you can follow the standard Ironic steps as outlined in the Kolla [documentation][1]
to create kernel/ramdisk images, associate a port, and finally boot the baremetal node.

[0]: https://gist.github.com/brk3/7fbb76f727de6bf7faeb2be1c1c5766f
[1]: https://docs.openstack.org/kolla-ansible/latest/reference/ironic-guide.html
