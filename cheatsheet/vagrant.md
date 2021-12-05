# Vagrant

Docker for virtual machines. Utilizes Virtualbox and other providers.
Available base images (called boxes) can be browsed at https://app.vagrantup.com/boxes/search.

## Prerequisites

Install virtualbox first and make sure, the kernel modules are “approved” by macos.

## Commands

Create a default Vagrantfile with the specified box as the base image.
* `vagrant init jasonc/centos8`

Start, restart, stop, destroy the box.
* `vagrant [up|reload|halt|destroy]`

Shows all non-destroyed boxes across all projects.
* `vagrant global-status`

Show boxes in the current project (where your Vagrantfile is).
* `vagrant status`
