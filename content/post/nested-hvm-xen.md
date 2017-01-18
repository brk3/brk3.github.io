+++
categories = ["virtualisation", "openstack", "howto"]
date = "2017-01-18T16:47:06Z"
title = "Nested Xen"

+++

As part of efforts to more regularly test the Xen hypervisor with Nova as part of OpenStack
development, I'm spending some time figuring out how to virtualise Xen on Xen, specifically Oracle's
[Oracle VM Server][0].

In virtualisation terms the different levels of hypervisor are referred to as _LN_, where N is the
[nesting level][1]. In our setup, Oracle VM Server 3.4.2 (OVM-S) is running on baremetal as _L0_,
another instance of OVM-S is running on that as _L1_, and the final OpenStack instantiated guest
would be _L2_.

There are not so many blogs or up to date howtos in this area, so most of the below comes direct
from the official [Xen documentation][2].

Also, we're using the xenlight (xl) toolkit, as of yet I have not found how to enable the relevant
parameters in libvirt to make nesting work.

Starting from _L0_, here is an example _xl_ guest config file:


```
builder = "hvm"
name = "example"
memory = 4096
vcpus = 2
vif = [ 'bridge=xenbr0' ]
disk = [ 'format=raw, vdev=hda, access=rw, target=/root/disk.img', 'format=raw, vdev=hdc, access=ro, devtype=cdrom, target=/root/V789846-01.iso' ]
vnc = 1
vnclisten="0.0.0.0"
hap=1
nestedhvm=1
cpuid = ['0x1:ecx=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx']

```

This is copied directly from _/etc/xen/xlexample.hvm_, with minor tweaks for disks and networking.
The sample files are a good place to start as the above example will almost certainly be out of date
in time.

I'm using bridged [networking][3], _xenbr0_ was created in the usual way with the hosts default
interface added to it.

The last three [lines][4] are the important pieces, without these _L1_ will fail to install with the
following error:

```
libxl__domain_make domain creation fail: cannot make domain: -3
```

Once logged into _L1_, an image can be booted for _L2_. In an OpenStack environment Nova will be
managing this step. To verify it's working however, we can boot Cirros in Xen using the following
config:

```
builder = "hvm"
name = "example.hvm"
memory = 128
vcpus = 2
vif = [ 'xenbr0' ]
disk = [ '/root/cirros.img,qcow2,xvda' ]
vnc = 1
vnclisten="0.0.0.0"
```

TODO
----
I will continue to update this post in time as I make improvements to this process. In particular, I
would like to solve the following:

* Set the required nesting params via libvirt, so that we aren't forced to use xl.
* Install using console rather than VNC.

[0]: https://www.oracle.com/virtualization/vm-server-for-x86/index.html
[1]: https://wiki.xenproject.org/wiki/Nested_Virtualization_in_Xen#Introduction
[2]: https://xenbits.xen.org/docs/unstable/man/xl.cfg.5.html
[3]: https://wiki.xenproject.org/wiki/Xen_Networking
[4]: https://wiki.xenproject.org/wiki/Nested_Virtualization_in_Xen#Quick-start_guide
