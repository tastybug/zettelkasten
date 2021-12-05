# DNS

## Forward zones, Reverse Zones

Zone information contains resources (A, MX, TXT, AAAA etc). A Forward Zone allows forward lookups, meaning lookup by name. A Reverse Zone allows lookups by IP, resulting in a name.

## Resource Records

### A
An Address record. Consisting of a name (“www”) which is relative to the zone (“example.com”) and an IP address.

### CNAME
Canonical Name, an alias record. You point a name to an FQDN (e.g. ww IN CNAME www.example.com. to catch typos)

### SOA
Yields primary DNS server for a zone (“Start of Authority”). When looking at the answer for a SOA request, you see 2 hosts: the first is the actual host (e.g. master.bla.com), the second is actually the email address of the admin (e.g. hans\.gruber.bla.com - here the first none-escaped dot is the separator of the email address local part)

### NS
A name server of a zone (dig de. NS). SOAs _can_ also be NS but can also be just used as a source of truth for all NS servers handling public queries.

### MX
Point to the mail server for a given zone. It has no key/name and points to an FQDN (“@ IN MX 0 mail.example.com”). MX is the only type of record that comes with a “priority” property. The priority is higher the lower the number is with 0 being the limit.

### AAAA
Like A it’s a Name/IP record, but for IPv6.

### TXT
Is a general purpose record. It has no name and is attached to the apex of the zone. There can be multiple TXT records (check “dig google.com TXT” for examples). The value size  is limited to 255 bytes. If the value exceeds the limit, help yourself with creating multiple TXT records and concatenate them on the client side.

### SRV
Service Record. This announces to the world what server applications you run via what protocol (TCP, UDP) and on which host. Using this info, a client (e.g. Thunderbird) can build its connection settings by just pointing it to a domain.

Example: `_ldap._tcp IN SRV 0 5 389 ldap.example.com.`. For this setting there is a priority of 0 (lower is better) and a weight of 5 (higher is better here).  The priority is not the same priority as in MX, it’s part of the value string. The problem it solves is the same though: there can be multiple hosts offering the service with differing ports. A client is supposed to cycle by priority.
There is a registry (“Service Name and Transport Protocol Port Number Registry”) for appname/protocol pairs. You cannot just make them up.

### SPF
Sender Policy Framework. Fairly complex stuff around preventing spammers from sending mails from impersonating sites.

* Example 1: `@ IN SPF "V=SPF1 -all"` -> this domain does not send any mail
* Example 2: `@ IN SPF "V=SPF1 MX A A:mail.example.com -all"` -> only the server mail.example.com will send mails from this example.com domain.
* Example 3: `@ IN SPF "V=SPF1 MX A A:mail.relay.com -all"` -> only the server mail.relay.com (that my ISP runs for me) will send mails from this example.com domain.

### PTR
Pointer record, this allows for reverse lookups from IP to name. You must be the owner of an IP to have this record. If your ISP owns, it you need to ask the ISP to create such a record.
PTR records are in a separate zone, a reverse zone.

Example: you own the IP block 192.168.1.X. The zone is then to be named “1.168.192.IN-ADDR.ARPA”. In the zone, you have a number of PTR records:
* `12 IN PTR @` -> 192.168.1.12 is the host for the base domain
* `15 IN PTR mail.example.com.` -> 192.168.1.15 is the IP of mail.example.com

## Other Concepts

### Primary Zone vs Secondary Zone

Servers mentioned in NS hold the secondary zone which is a cache of the primary zone, provided by SOA. The secondary zone can be stale, depending on the refresh configuration between NS and SOA.
Resolvers see the secondary zone.

### Zone Transfer
A zone transfer happens when a secondary syncs with the primary and receives a full “zone dump”.
To zone transfer, you need to ask the primary. Primary might be configured to not allow everyone a zone transfer. A primary might even be configured to limit who can query.

DNS servers for a bigger zone are usually setup redundantly: there is a primary and some secondary servers. The secondary servers ping the primary and look at a “serial number field” for a zone record. If that number increased (works like an “etag” header), the secondary will request a full zone dump from the primary to ensure its state is in sync.
The primary DNS server is held in a “SOA” record. Example: `dig . SOA` yields the primary root server, which is “a.root-servers.net”.

## Server Types

### Resolver

Resolver = DNS client. Submits queries, receives responses. If you query a DNS server and it does not have an answer but it setup for recursion, it itself acts as a resolver by doing a recursive lookup starting at the root servers all the way down to the authoritative DNS server for the host that you asked for.

### Forwarder Server

Is a proxy for another server that does the resolution.

### Caching Server

Has no zone authority, it either forwards or resolves recursively itself. It’s what a home router usually does for you.

## How failsafe is DNS?

DNS is a primary (SOA)/secondary (NS) hierarchy. Secondaries check the primary (see SOA record) in configured intervals (“refresh”). If the primary is not available, a retry will be attempted which again is determined by the zone configuration (“retry” in seconds). If the primary cannot be reached for a certain amount of seconds (“expire” in seconds), the secondary declares the zone as inactive and stops answering requests for that zone.
A primary can act as an NS but can also act solely as a SOA. In very simple zones you likely only have a single name server, acting as SOA and NS.

