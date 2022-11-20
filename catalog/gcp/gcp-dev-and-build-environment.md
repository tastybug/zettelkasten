# GCP Build Pipeline and SCM

GCP offers a handful of services that can resemble a CI/CD pipeline when combined.

## Cloud Source Repositories

This is like github: a place for git repositories. They offer to mirror upstream git repos hosted on Github and Bitbucket.
GCP offers strong search capabilities over these repos.

Pricing is tame and there is an okay free tier: up to 5 users, 50GB space and 50GB of egress are free.

## Cloud Build

This is an environment that uses VM instances to run either build instructions or Dockerfiles (acting as build containers).
It is also possible to use a private pool of VMs.

Pricing: you pay for build minutes, but you have control over the VM size and region. Everyone get 120mins per day for free.

## Artifact Registry (fka Container Registry)

Nexus/Artifactory like centralized artifact repository for container images, maven remote and for NPM.

Has some neat features: 
* cached worldwide for distributed teams
* scans for vulnerabilities
* prevent risky images from being used ("binary authorization")

## Cloud Workstations

On demand, consistent cloud dev environment. Specifically: a web application to manage VM images which are the basis for dedicated developer VMs. The VMs can be reached via SSH, HTTPS or TCP.
This is essentially like the cloud shell, but beefier.
