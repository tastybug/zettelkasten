# Connecting to GCP

There are a number of options to integrate on-premise networks with the GCP infrastructure to increase bandwith between both worlds.

>Before starting, you need to figure out what you aim for:
>* you only need access to Google Workspace (fka. G Suite) and Google APIs, in which case check
>  * Direct Peering to directly peer at a Google Edge Location
>  * Carrier Peering to connect through a support provider to Google, if you are too far away from a Google Edge location
> * asd


## Direct Peering (Layer 2)

As the name implies, you directly peer with Google at one of over 100 locations in 33 countries.
This traffic does include GCP traffic, it is however counted as egress traffic because Direct Peering exists outside of Google Cloud.

> Direct Peering can be used by GCP, but does not require it.


# Connecting to GCP and/or the Google Network

Broadly speaking you might have the following challenges:

* have 2 organizations in GCP with infrastructure that you want to connect (layer 2)
* increase your network throughput between Google and own services (layer 3)
* have GCP resources and the own on-prem services on the same private network (layer 2)

## Layer 2, Option 1: IPSec VPN Tunnel (also with HA option)

This is realistic for low-ish data volume. VPN SLA is given as 99,9%.

HA VPN is a 2-tunnel flavor of this with an SLA of 99,99%.

> Expected throughput is 1.5 - 3 Gbps per tunnel

![classic vpn topology](./pics/gcp-classic-vpn-topology.png)

## Layer 2, Option 2: VPC Peering between different GCP VPCs

You have different VPCs, possibly belonging to different GCP organizations.
The subnet range must not overlap in this case.

## Layer 2, Option 3: Cloud Interconnect

This achieves the same result as a VPN, but allows for more throughput due to a physical connection. It requires co-location of at least the router of the customer in one of the supported data centers (they are spread all around the world).
SLA is given as 99,9% up to 99,99%.

> Expected throughout 10 Gbps (100 Gbps in beta) per link

## Layer 2, Option 4: Partner Interconnect

If `Cloud Interconnect` is unrealistic, peering via a service provider is possible (they might be closer to your physical location). This likely means shared infrastructure in between GCP and the onprem network and thus lower throughput.

> Expected throughput is 50 Mbps - 10 Gbps per connection

## Layer 3, Options 1&2: Direct or Carrier Peering

One is able to connect directly to Google Edge “Points of Presence” (PoPs). Alternatively, this can also be done via a Carrier inbetween, giving you more geographical flexibility (Carrier has alternative PoPs). The result of this is an increase in network throughput.

Example: Video Content Provider that wants to upload massive amounts of content on a regular basis.

> There are no SLAs for Peering. Throughput is given as 10 Gbps for direct peering and “depends on carrier” for Carrier peering
