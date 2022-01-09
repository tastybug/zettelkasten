# Vault

Is a Hashicorp tool that offers management of certificates and secrets.
It comes with 3 flavors:

* open source: self hosted, free, dynamic secrets, ACL, key rolling, encryption as a service, aws/azure/gcp auto unseal, local clustering in a single DC (here I'm not sure why this cannot span different DCs)
* enterprise: self hosted, non-free, all of open source plus disaster recovery, namespaces, replication (aka multi-cluster setups), read-only nodes, HSM auto-unseal, MFA, Sentinel, FIPS 140-2 & Seal Wrap (some gov standard)
* vault on HCP: hosted, non-free, fully managed, scalable, pay by the hour

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

## Backends

There are many backend implementations, some community owned. They either support HA or not (see next section).

Popular backends are cloud based like S3, Consul and the internal backend, called `raft`, where each node has it's own FS based persistency, which is synced automatically.

* Consul: can be scaled independently, increasing throughput
* Raft: is a file on the FS of each Vault node, no extra network hop necessary; sync is done transparently between the nodes

## How Backends Influence Scaling

It can be run stand-alone, in clusters and with multi-cluster setup, where the clusters stay in sync.
A cluster requires a backend that supports high availability. A cluster has a leader, the rest is standby. You can also have 2 backends, one for the leader-lock (stanza `ha_storage`) and the other for the actual data (stanza `storage`).

Calling a standby gets you redirected to the leader - this means that HA does not increase scalability. Scaling Vault requires scaling the backend, since you always use a single Vault node.

More on this here: https://www.vaultproject.io/docs/concepts/ha

## Dev Server

You can run Vault in dev server mode (`vault server -dev`), which runs in-memory. To access it with the CLI, run `export VAULT_ADDR='http://127.0.0.1:8200'` first, then verify with `vault secrets list`.

Dev server comes with a K/V store under `/secret` (verify this with `vault secrets list`).

## Productive Server

Usually you can use Packer to create a VM image with Vault or Terraform to setup Vault in a VM.

Running the server requires a configuration file: `vault server -config=<file>`, in a persistent VM one also needs systemd configuration.

### Configuration with Internal Storage and no TLS (Example)

```
storage "raft" {
	path = "/opt/vault2/data1"
	node_id = "node1"
    # this is cloud based auto-join configuration, other options exist
	retry_join {
	auto_join = "provider=aws region=us-east-1 tag_key=vault tag_value=us-east-1"
	}
}

# use AWS KMS to store unseal keys
seal "awskms" {
	region = "us-east-1"
	kms_key_id = "arn:aws:kms:us-east-1:826734782634:key/some-key-here"
}

listener "tcp" {
	address = "0.0.0.0:8200"
	cluster_address = "0.0.0.0:8200"
	tls_disable = 1
}

api_addr = "http://10.0.0.37:8200"
# this is only for cluster setups (which is an enterprise feature)
cluster_addr = "http://10.0.0.37:8201"
cluster_name = "vault-prod-us-east-1
ui = true
log_level = INFO"
```

## Dynamic Secrets a.k.a Dynamic Credentials

Explain this closer with an example of AWS.
