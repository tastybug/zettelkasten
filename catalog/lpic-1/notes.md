# LPIC-1 Notes

## Basics

### Working with Text Files

* `tr`: replace or delete single characters from STDIN: `tr ':' '$' /etc/passwd`
*  `split`: split a file by number of lines into more files (for further parallel processing): `split -l5 my-large-file.txt`
*  `paste`: merge 2 files and concatinate the line of the 2nd file to the lines of the first file `paste names.txt addresses.txt`
*  `sed`: stream editor to replace or change lines of text (ie. a more powerful `tr`): `sed 's/:/$/g' /etc/passwd`
*  `tee`: it takes what comes from STDIN and writes it to a file AND to STDOUT, allowing: `grep bash /etc/passwd | tee results.txt | less`

## Networking

### Hostnames

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

### Changing the Order of DNS Resolution and the `dig`, `getent` and `nslookup` Commands

There are different commands to query DNS data:

* `dig` and `nslookup` query DNS servers for records.
* `gentent` does system level name resultion using the system database (passwd, hosts, DNS), using Name Service Switch (NSS).

> `nslookup` and `dig` do not look at `/etc/hosts` when resolving a query. `entent` however does so.

#### Changing the Order of Resolution

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

### Local Resolution with LLMNR - Link Local Multicast Name Resolution

LLMNR is a protocol for name resolution on local networks. It allows devices on the same local network or link to resolve each other's hostnames to IP addresses without the help of a DNS server.

> LLMNR works by sending multicast queries to all devices on the local network, asking for IP addresses associated with particular hostnames.

LLMNR is usually switched __off__ by default.

