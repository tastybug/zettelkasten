# Vault

## Starting a CLI session

1. Have `vault` installed to have access to the CLI tools
2. Point `vault` to the server: `export VAULT_ADDR='http://127.0.0.1:8200'`
3. TODO describe here how to get a token on the command line
4. Login to vault providing your token: `vault login`
5. Start working

## Admin work: configuring auth methods

* `vault auth list` what auth methods are active
* `vault auth enable [-path=foo] [-description='some description'] approle`: enabled `approle` method under path `foo`
* `vault auth tune -default-lease-ttl=12h foo` edit the `approle` method under path `foo`
* `vault auth disable foo` disable the `approle` method under path `foo`

### Auth method 'userpass' configration

1. `vault auth enable -path=up userpass`
2. `vault auth tune -default-lease-ttl=24h up/`
3. `vault write auth/up/users/pbartsch password=2hard4u policies=pbartsch`, this creates a user `pbartsch` with a password
4. `vault list auth/userpass/users` what users are there?
5. `vault read auth/userpass/users/pbartsch` show the properties of this user

## Working with K/V stores

* Check under which path the KV store is: `vault secrets list`
* Add value: `vault kv put secret/database 'password=2hard4u'`
* Get value: `vault kv get secret/databse`

## Admin

* list all peers when using raft as backend: `vault operator raft list-peers`
