# GCP Billing, Quotas and Labels

## What are Resources?

Resources are generally all the services that GCP offers. Each resource is part of a project, which in turn is associated with a folder and an organization. Resources are furthermore either global (like images, snapshots, networks) or regional (external ips) or zonal (instances, disks).

> Since every resource belongs to a project, billing and reporting is happening on a project level.

## What are Quotas?
Quotas are also a project level mechanism. They control:

1. how many resources one can create in a project: e.g. 15 VPC networks, 24 CPUs
2. the limit of a usage, e.g. rate limits of APIs

Quotas can be altered using the Cloud Console or via a support ticket.

> Quotas prevent runaway costs, malicious attacks, billing surprises and force users to consider sizing

## What are Labels

Can be attached to any resource: VMs, disks, snapshots, images, SQL servers and so on. Use it to tag resources to filter, attribution (cost center, ownership, environment, state, ..) and run bulk operations on multiple resources.

__Labels propagate through billing!__

> There is a limit of 64 labels per resource.

### How Tags Differ from Labels

Tags work like labels, but can only be set to VM instances. Tags can be used for firewalling rules to select what instance can do what.

## Billing

It is possible to create budgets, which is a specified target amount and bind this to one or more _projects_. Once thresholds of that budgets are either reached or projected to be reached, GCP will:

* send an e-mail
* optionally write to a pub/sub topic for programmatic consumption ("Programmatic Budgeting")

> With Programmatic Budgeting one would be able to alter resources automatically, scale resources down or delete those resources that have a particular labelling configuration. Pretty powerful.

### Slicing and Dicing Spend Data

It is possible to export spend data to GBQ, where it can be queried. With the help of labelling of resources, this is a very powerful approach to get insights by running queries.
Alternatively, data can be exported as a file (CSV, JSON) into a storage bucket.

Furthermore, Google Data Studio can be used to visualize spend data and to allow data to be sliced and diced.
