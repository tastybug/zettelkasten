# GCP as a Application Platform

There are a number of options to run an application on GCP.

![choosing application platform](gcp-choosing-a-deployment-type.png)

## VM based (Cloud Compute)

You run your own VM, the rest is up to you. It's possible to specify a VM to run a specific container from the container registry but I'm not aware of means to update that container without re-creating the VM.

## Containerized Application

### GKE a.k.a. Run your own K8S cluster

You're the admin of a GKE cluster. You decide the region, number of nodes and so on.

### Cloud Run

This option allows you to run a given container image (it must be present in GCP Container Registry they say) in a managed GKE cluster.
So almost as GKE, but no administrative worries.

> The application must be stateless!

You pick a region for your image.

TODO:
* scaling?
* delivery? rolling updates?

## Non-Containerized Application

### Cloud Functions (Event Driven!)

Functions are exactly like Lambda. Inexpensive (pay my 100ms of runtime), short lived and event driven. The following events are supported:
* Pub/Sub
* web requests
* changes in a storage bucket

TODO:
* What is your deliverable? 
* How to update it?
* scalability limits

### App Engine

This is a bit weird: each GCP project can contain a single App Engine application. 

An application is WHAT? An application contains 1 or more services, which itself have 1 or more versions.

TODO:
* What is your deliverable? 
* delivery
* scaling
