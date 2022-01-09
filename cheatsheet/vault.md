# Vault

## Starting a CLI session

1. Have `vault` installed to have access to the CLI tools
2. Point `vault` to the server: `export VAULT_ADDR='http://127.0.0.1:8200'`
3. TODO we need a token to access data, how do we get that in reality?

## Working with K/V stores

* Check under which path the KV store is: `vault secrets list`
* Add value: `vault kv put secret/database 'password=2hard4u'`
* Get value: `vault kv get secret/databse`

## Admin

* list all peers when using raft as backend: `vault operator raft list-peers`
