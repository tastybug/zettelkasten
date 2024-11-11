# LPIC-1 Notes

## Hostnames

There are 3 kinds of hostnames: transient and static and pretty hostname. While static hostnames are used for non-temporary systems, the transient hostname is for short-lived systems (e.g. cloud environments) or a name coming from DHCP. The pretty name is for display purposes only, can contain special characters and has no functional impact.

> A host can have all hostname types at the same time. A static hostname functionally has higher precendence than a transient one.

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

## Changing the Order of DNS Resolution and the `dig`, `getent` and `nslookup` Commands

There are different commands to query DNS data:

* `dig` and `nslookup` query DNS servers for records.
* `gentent` does system level name resultion using the system database (passwd, hosts, DNS), using Name Service Switch (NSS).

> `nslookup` and `dig` do not look at `/etc/hosts` when resolving a query. `entent` however does so.

### Changing the Order of Resolution

`sudo vi /etc/nsswitch.conf`: here you can change the sources and orders of resolution of different types like hosts, users, groups and more:

```
# In order of likelihood of use to accelerate lookup.
passwd:     sss files systemd
shadow:     files
group:      sss files systemd
hosts:      files dns myhostname
services:   files sss
netgroup:   sss
automount:  files sss

aliases:    files
ethers:     files
// snip
```
