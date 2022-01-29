# Vault

## What it is
Is a Hashicorp tool that offers management of certificates and secrets. It comes in 3 flavors:

* open source: self hosted, free, dynamic secrets, ACL, key rolling, encryption as a service, aws/azure/gcp auto unseal, local clustering in a single DC (here I'm not sure why this cannot span different DCs)
* enterprise: self hosted, non-free, all of open source plus disaster recovery, namespaces, replication (aka multi-cluster setups), read-only nodes, HSM auto-unseal, MFA, Sentinel, FIPS 140-2 & Seal Wrap (some gov standard)
* vault on HCP: hosted, non-free, fully managed, scalable, pay by the hour

## Vault Concepts

### Auth Methods: Different Ways of Authenticating to Get a Token

Vault supports different means of authentication to then allow access to the secrets, engines and so on. Auth methods are optimized for different human or system to system use cases. The purpose of each auth method is to ultimately give you a token. With the token you perform your actions using any of the interfaces, be it CLI, Web or API.

Initially, the only method is the "token method", which simply expects a token being given (e.g. the `root token` that will be pre-generated for you when starting the server in dev mode).

Each auth method is enabled under a customizable path. `vault auth list` prints all enabled auth methods and their paths.

Some available auth methods:

* `userpass`: user/password manually set up
* `okta`, `github`: human-based, centralized OAuth providers
* tls certificates, k8s, azure, kerberos, `approle` and others are means for S2S authentication

### Entities and Aliases

An entity is something that authenticats with Vault, could be a system or a person. Each entity has 0+ aliases. An alias links an entity to an auth method (e.g. entity "Jane" has alias `jsmith` with auth method `userpass` and `funky22` with auth method `github`). 

Entities and Aliases come with a bunch of metadata and 1+ policies. The policies of each can differ - the token generated when logging in will contain the policies from the entity plus the one from the alias.

It's a bit counterintuitive that the method of authentication brings different permissions, but this is how it works according to [this](https://www.udemy.com/course/hashicorp-vault/learn/lecture/27039872#overview). Example:

1. Entity `jane` has policy `management`.
2. Alias for `github` has policy `HR`.
3. Alias for `userpass` has policy `finance`.

When logging in via github, Jane will receive a token with policies `management` and `HR`. When logging in via `userpass`, it's `management` plus `finance`.

### Vault Groups

You can group entities into Vault Groups. A group comes with a policy. When logging in an entity in a group, you receive a token with 3 policies attached to it: group, entity and alias.

Some auth methods come with external groups, like scopes in JWT or groups in LDAP. You can map those external groups in Vault to internal groups or policy names.

### Policies: Defining What a Token Can Do

By default, you have access to nothing. Only by being associated with a policy, you get access.
Policies are cumulative, meaning that you can be subject to multiple policies and you will have access to the superset of all permissions.

Initially, there are 2 policies: `root` and `default`. The first one is added to all root tokens, the second to non-root tokens. Both policies cannot be deleted and only `default` can be changed. `default` provides "common permissions".

A policy covers paths and defines the allowed verbs on that path (`read`, `create`, `update`, `delete`, `list`, `sudo` and `deny`). For example, run `vault policy read default` to see what the `default` policy allows. Each policy is described in HCL, e.g.:

```
path "kv/database/nonprod" {
	capabilities = ["read"]
}

path "sys/policies/*" {
	capabilities = ["create", "list", "delete"]
}
```

Some paths cover administrative functions, e.g. `sys/seal` to seal the vault instance. These special paths require `sudo` capability.

#### Test Driving Policies

Let's say you created a policy and want to try out whether it works: `vault token create -policy="newpolicyname"` will give you a token with that ability.

Next, simply try a few things out that the token is supposed to support. Read a path, write to it and so on. This can be done on a dev server instance.

#### `*` Wildscards in Paths

Policies can contain wildcards in paths (see example above). A wildcard is only allowed at the end of a path. A wildcard matches paths of any depth.

Example: `secret/app/f*` matches:

* `secret/app/foo`
* `secret/app/foo/bar/baz`
* but not `secret/app`

#### `+` Wildcards in Paths

A `+` in a patch supports wildcard matching for a single directory in the path. 

Example: `secret/+/+/db` matches:

* `secret/app/nonprod/db`
* `secret/app/prod/db`
* but not `secret/prod/db
* and not `secret/app/prod`

#### Templating

There is variable replacement in some policy strings with values available to the token.
Here is an example for a generic policy statement that can apply to many users sharing a policy. It allows every user to list and create and read data under `secret/data/USERNAME/*`:

```
path "secret/data/{{identity.entity.name}}/*" {
	capabilities = ["list", "create", "read"]
}
```

The list of variables isn't long, here's a few examples:

* `identity.entity.id` entity id
* `identity.entity.name` entity name
* `identity.entity.metadata.$METADATAKEY` Metadata associated with the entity.

### Tokens

As described a token is associated with policies which describe what you are allowed to do. `vault token lookup $token` will additionally tell you about metadata that applies to the token:

* Accessor: ?
* TTL: a default lifespan, which might be extendable via renewals; once reached, the token is revoked
* Policies: the superset of what you can do
* Max TTL: the max lifespan; renewals in sum cannot exceed this
* Number of uses left: 0 means unlimited uses
* Renewable: whether it's renewable
* Orphaned Token: if true, parental revokation does not propagate


#### Assessing Tokens

* check metadata and capabilities of a token: `vault token lookup $TOKEN`
* check what capabilities a token has at a path: `vault token capabilties $TOKEN secrts/data/prod/db`

#### Creating Child Token
* `vault token create [-policy=bla] [-period=10m]`

#### Token Hierarchy

Some token (not batch, not use-limit and maybe others) are able to create child tokens. Both can have their individual TTL, but keep in mind that when a parent token is revoked, the child tokens are revoked too, even if they still have TTL left.

#### Service Tokens (OAuth Access Token like)

Service tokens are the default kind of token. They are persisted to the storage backend. They can be renewed, revoked and you can create child tokens.

This is how you create one (`-type=service` is the default and can be left out):

```
~ # vault token create -policy=default [-type=service]
Key                  Value
---                  -----
token                s.703mZQRwzr178Oii5QjBraab
token_accessor       XAlpRAtF097z6WOLegdZGnBr
token_duration       768h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]
```

As you can see, service tokens start with `s.`.

#### Batch Tokens (JWT like)

Batch tokens are encrypted blobs. They are not stored to the backend and not replicated across clusters.

They are limited in their abilities: they can't be renwed, they can't be root tokens, create child tokens and a few more things.

It is possible to configure an auth method as such that it produces batch tokens by default.

The vault documentation proposes to use them when many clients (e.g. 100.000 containers) would request service tokens at once, where the backend (which _also_ syncs service tokens across clusters) would be a bottleneck. I'm getting the impression thatbatch tokens are like JWT: self contained, locally usable.

This is how you create one:

```
~ # vault token create -policy=default -type=batch
Key                  Value
---                  -----
token                b.AAAAAQLsVGFi5IrPAlUQiGOXUw6HTvFMHOxZ-HANvk-t0XvyDzTnENO5CAK9Xl7LSJC7D-VrvW3dWuTFNJrDof8XxaBVeAGGFy1u-d3H4UeX0Y4YpTKkrgebNpZaYPm17pjzGVM
token_accessor       n/a
token_duration       768h
token_renewable      false
token_policies       ["default"]
identity_policies    []
policies             ["default"]
```
As can be seen, batch tokens start with a `b.`.

### Engines and Paths
Vault comes with different "engines" that you make available under paths. E.g. you can run a K/V store under `secret/`.

### K/V Engine

The K/V engine is available under a path, say `secret/`. You are supposed to add nodes under the root, to which you attach k/v pair(s).

Example: `vault kv put secret/nonprod/database 'password=bla' 'username=dba'`

This results in:

```
* secret/nonprod/database // <- the node
	* password=bla
	* username=dba
```

#### KV `v1` vs `v2`: don't trip over `/data/`
Watch out: there is a v1 KV engine and a v2 KV engine. They differ insofar the v2 allows for more finegrained ACLs. It is however confusing that when configuring paths for a v2, you need to remember to add a `/path` in between:

```
path "secret/data/dev/foo" {  
   // whatever
}
```
 
When accessing the path, you _omit_ mention the `/data`: `vault kv get secret/dev/foo`.

## Backends

There are many backend implementations, some community owned. They either support HA or not (see next section).

Popular backends are cloud based like S3, Consul and the internal backend, called `raft`, where each node has it's own FS based persistency, which is synced automatically.

* Consul: can be scaled independently, increasing throughput
* Raft: is a file on the FS of each Vault node (the node is stateful!), no extra network hop necessary; sync is done transparently between the nodes

### How Backends Influence Scaling

It can be run stand-alone, in clusters and with multi-cluster setup, where the clusters stay in sync.
A cluster requires a backend that supports high availability. A cluster has a leader, the rest is standby. You can also have 2 backends, one for the leader-lock (stanza `ha_storage`) and the other for the actual data (stanza `storage`).

Calling a standby gets you redirected to the leader - this means that HA does not increase scalability. Scaling Vault requires scaling the backend, since you always use a single Vault node.

More on this here: https://www.vaultproject.io/docs/concepts/ha

## Run Modes

### Dev Server

You can run Vault in dev server mode (`vault server -dev`), which runs in-memory. To access it with the CLI, run `export VAULT_ADDR='http://127.0.0.1:8200'` first, then verify with `vault secrets list`.

Dev server comes with a K/V store under `/secret` (verify this with `vault secrets list`).

### Productive Server

Usually you can use Packer to create a VM image with Vault or Terraform to setup Vault in a VM.

Running the server requires a configuration file: `vault server -config=<file>`, in a persistent VM one also needs systemd configuration.

#### Configuration with Internal Storage and no TLS (Example)

```
storage "raft" {
	path = "/opt/vault2/data1"
	node_id = "node1"
    # this is cloud based auto-join configuration
    # alternatively, one can manually join each node: `vault operator raf join http://thisnode.com:8200`
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


