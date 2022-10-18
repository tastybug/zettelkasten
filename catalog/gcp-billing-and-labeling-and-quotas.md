# GCP Billing and Quotas

Resources are generally all the services that GCP offers. Each resource is part of a project, which in turn is associated with a folder and an organization. Resources are furthermore either global (like images, snapshots, networks) or regional (external ips) or zonal (instances, disks).

> Since every resource belongs to a project, billing and reporting is happening on a project level.

Quotas are also a project level mechanism. They control:

1. how many resources one can create in a project: e.g. 15 VPC networks, 24 CPUs
2. the limit of a usage, e.g. rate limits of APIs

Quotas can be altered using the Cloud Console or via a support ticket.

> Quotas prevent runaway costs, malicious attacks, billing surprises and force users to consider sizing

## Labels

Can be attached to any resource: VMs, disks, snapshots, images, SQL servers and so on. Use it to tag resources to filter, attribution (cost center, ownership, environment, state, ..) and run bulk operations on multiple resources.

__Labels propagate through billing!__

> There is a limit of 64 labels per resource.

### How Tags Differ from Labels

Tags work like labels, but can only be set to VM instances. Tags can be used for firewalling rules to select what instance can do what.
