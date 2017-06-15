+++
categories = ["openstack"]
date = "2017-06-15T16:31:32+01:00"
title = "Debugging Murano with Kolla"

+++

The [kolla-ansible][1] project recently gained support for hacking on the
OpenStack services it deploys, much the same as devstack.

Services are being added as we go, so it's unlikely all will be supported for
the Pike release. That said it's quite easy to add support for new projects, so
if the one you're interested in isn't there yet have a go!

I've been using this for the past month or so to submit patches into the Murano
project, and wanted to give a short overview of how Kolla can be used to do
this.

Enabling
--------
Start by enabling dev mode for the project(s) you're interested in within
/etc/kolla/globals.yml:

```
murano_dev_mode: true
```

Then either deploy the project:

```
kolla-ansible deploy --tags murano
```

or reconfigure it if it's already been deployed:

```
kolla-ansible reconfigure --tags murano
```

Kolla will clone the source code for the project under *'/opt/stack'*, and bind
mount it into the container. The benefit of this is that you can use local
development tools, your favorite editor, etc. right from the node instead of
having to install them all into the container. Also if the container happens to
go into an error state or gets destroyed you don't lose your changes.

Debugging
---------
Working on the code is as easy as making some changes to the local code, and
restarting the container. Logs are available under
*'/var/lib/docker/volumes/kolla\_logs/\_data/murano'*, and can be
tailed/grepped from a separate terminal.

To actually debug the code, I've found [remote\_pdb][2] to be very effective.
We currently don't include this in the Docker images but it can be installed
easily:

```
docker exec -it -u root murano_api pip install remote_pdb
```

Set a break point as follows:

```
from remote_pdb import RemotePdb
RemotePdb('127.0.0.1', 4444).set_trace()
```

Restart the container, and attach to pdb using:

```
socat readline tcp:127.0.0.1:4444
```

Summary
-------
One of the best things about using Kolla is it's "don't touch the host where at
all possible" mantra. This is one area where I find it really surpasses
devstack, as deploys can quickly be torn down with confidence that nothing will
be left behind. This leads to more repeatable patches, testing, and less "works
on my machine" code.

Give it a try, and feel free to ask questions in [#openstack-kolla][3] on freenode.

[1]: https://docs.openstack.org/developer/kolla-ansible/
[2]: https://pypi.python.org/pypi/remote-pdb
[3]: https://kiwiirc.com/client/irc.freenode.net/#openstack-kolla
