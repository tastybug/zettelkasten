# LPIC-1 Notes

## Hostnames

There are 3 kinds of hostnames: transient and static and pretty hostname. While static hostnames are used for non-temporary systems, the transient hostname is for short-lived systems (e.g. cloud environments) or a name coming from DHCP. The pretty name is for display purposes only, can contain special characters and has no functional impact.

> A host can have all hostname types at the same time!

```shell
[root@rocky ~]# hostnamectl
 Static hostname: rocky
       Icon name: computer-vm
// snip
[root@rocky ~]# hostnamectl set-hostname "philipp's vm"
[root@rocky ~]# hostname bla 
[root@rocky ~]# hostnamectl
   Static hostname: philippsvm
   Pretty hostname: philipp's vm
Transient hostname: bla
```

> Transient hostnames do not survive a restart.

There are different tools and locations (the list is incomplete) to influence the hostname.
* `hostname` sets the transient hostname
* `/etc/hostname` canonical source of the static hostname
* `hostnamectl` systemd tool to influence the various hostname types
  * `hostnamectl set-hostname --static foo`
  * `hostnamectl set-hostname --transient bar`
  * `hostnamectl set-hostname --pretty baz`
