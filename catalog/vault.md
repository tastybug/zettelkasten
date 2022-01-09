# Vault

Is a Hashicorp tool that offers management of certificates and secrets.

## Thinking like Vault


### Engines and Paths
Vault comes with different "engines" that you make available under paths. E.g. you can run a K/V store under `secret/`.

### K/V Model

The K/V engine is available under a path, say `secret/`. You are supposed to add nodes under the root, to which you attach k/v pairs.

Example: `vault kv put secret/nonprod/database 'password=bla'; vault kv put secret/nonprod/database 'username=dba'`

This results in:
* secret/nonprod/database // <- the node
    * username=dba // 1st k/v
	* password=bla // 2nd k/v

## Backends, Scaling Vault

It can be run stand-alone, in clusters and with multi-cluster setup, where the clusters stay in sync.
A cluster requires a backend that supports high availability. A cluster has a leader, the rest is standby. You can also have 2 backends, one for the leader-lock (stanza `ha_storage`) and the other for the actual data (stanza `storage`).

Calling a standby gets you redirected to the leader - this means that HA does not increase scalability. Scaling Vault requires scaling the backend, since you always use a single Vault node.

More on this here: https://www.vaultproject.io/docs/concepts/ha

## Dev Server

You can run Vault in dev server mode (`vault server -dev`), which runs in-memory. To access it with the CLI, run `export VAULT_ADDR='http://127.0.0.1:8200'` first, then verify with `vault secrets list`.

Dev server comes with a K/V store under `/secret` (verify this with `vault secrets list`).

## Productive Server

Usually you can use Packer to create a VM image with Vault or Terraform to setup Vault in a VM.

Running the server requires a configur

## Dynamic Secrets a.k.a Dynamic Credentials

Explain this closer with an example of AWS.
