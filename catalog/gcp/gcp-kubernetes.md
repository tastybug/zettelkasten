# GCP Kubernetes (GKE)

GKE offers managed K8S clusters with the following properties:

* auto-upgrade
* auto node healing (not sure what that is)
* nodes are based on GCP Compute; new nodes join the pool in a managed manner

## Zonal vs Regional Clusters

By default, a cluster has 3 nodes and a control plane in a single zone. Nodes and control plane can however span across multiple zones in a region, increasing availability.

When creating a cluster, one has to decide whether it's meant to be zonal or regional. Migrations between the modes are not possible.

## Mode of Operation: Autopilot vs Standard Mode

Clusters are by default created in `Autopilot Mode`, which lessens the degree of administrative work and influence. An `Autopilot` cluster is always regional, uses the K8S release channel version and so on.
`Standard Mode` otoh allow for regional or zonal clusters and bleeding edge K8S versions.

## Costs

This is pretty complex and comes down to the number of nodes and commitment. Control plane costs are currently as such:

>The cluster management fee of $0.10 per cluster per hour (charged in 1 second increments) applies to all GKE clusters irrespective of the mode of operation, cluster size or topology.

The free tier gives you currently enough credit to make the control plane of a single cluster free.

## Anthos

Anthos is described as a migration tool that turns an existing VM landscape into a K8s setup. It is described as automatically creating containers from existing VMs and then deploying those contains into a ready GKE cluster.
Apparently the migratation source can be manifold: 
* GCP Cloud Compute
* Azure
* AWS
* On Premise VMWare

The "processor" of this migration is a "GKE processing cluster" running a migration tools. The output goes into 2 places: Cloud Storage will contain the YAML and Dockerfiles and a target GKE cluster (NOT the processing cluster) will be running the pods with the migrated containers.
Once the migration is done, the processing cluster can be deleted.
