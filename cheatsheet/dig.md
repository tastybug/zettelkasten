# DIG

Find A record of a domain (“forward lookup”)
* `dig apple.com`

Resolve an IP into a name / FQDN (“reverse lookup”)
* `dig -x 17.151.62.68`

Get the mail service for a domain
* `dig mx apple.de`

Get all records for the FQDN
* `dig apple.com ANY`

Zone transfer (must be supported by the name server; can be limited to certain IPs only)
* `dig AXFR somedomain.com @nameserver.somedomain.com`
* `dig zonetransfer.me ns; dig AXFR zonetransfer.me @nsztm1.digi.ninja`
