# Vault

## Starting a CLI session

1. Have `vault` installed to have access to the CLI tools
2. Point `vault` to the server: `export VAULT_ADDR='http://127.0.0.1:8200'`
4. If you have a token, just go with `vault login` or specify any enabled auth method, e.g. userpass: `vault login -method=userpass username=pbartsch`
5. Alternatively, with a token already at hand, set a `VAULT_TOKEN=asd123` env variable.
6. There is now a session, further commands don't have to be authenticated (the token is stored in `~/.vault-token` and automatically read from there)

## Admin work: configuring auth methods

* `vault auth list` what auth methods are active
* `vault auth enable [-path=foo] [-description='some description'] approle`: enabled `approle` method under path `foo`
* `vault auth tune -default-lease-ttl=12h foo` edit the `approle` method under path `foo`
* `vault auth disable foo` disable the `approle` method under path `foo`

### Auth method `userpass` configuration

1. `vault auth enable -path=up userpass`
2. `vault auth tune -default-lease-ttl=24h up/`
3. `vault write auth/up/users/pbartsch password=2hard4u policies=pbartsch`, this creates a user `pbartsch` with a password
4. `vault list auth/userpass/users` what users are there?
5. `vault read auth/userpass/users/pbartsch` show the properties of this user
6. `vault login -method=userpass username=pbartsch`, then enter password. That will also show the token that could be used in API calls and so on.

### Auth method `approle` configuration and use

1. `vault auth enable approle`
2. `vault write auth/approle/role/bryan policies=bryan token_ttl=20m`: set up a role and link it to a policy
3. `vault list auth/approle/role`: verify
4. `vault read auth/approle/role/bryan/role-id` this shows a stable id for the bryan role. `role_id` is the credential name.
5. `vault write -force auth/approle/role/bryan/secret-id`: this creates a secret-id (this is different with each write). The `secret_id` is the credential secret!
6. `vault write auth/approle/login role_id=foo secret_id=bar`: does the login; this prints a token.
7. `vault login MY_NEW_TOKEN` do this if you want to use the token for the current CLI session

### Log in, put token into a variable

1. Do the userpass setup from above
2. `export VAULT_FORMAT=json`
3. `OUTPUT=$(vault login -method userpass username=pbartsch password=2hard4u) 
4. `VAULT_TOKEN=$(echo $OUTPUT | jq '.auth.client_token' -j)`
5. `vault login $VAULT_TOKEN`

### Working with policies (= RBAC)

1. `vault policy list` gives you existing policies
2. `vault policy read default` describes what the `default` policy is allowing. The format given is HCL.
3. `vault policy read root` fails funnily enough, even though it exists.
4. `vault policy fmt FILENAME` formats a policy file
5. `vault policy write funky-policy /tmp/funky.hcl`
6. `vault token create -policy="funky-policy"` to create a token to test things out

## Working with K/V stores

* Check under which path the KV store is: `vault secrets list`
* Add value: `vault kv put secret/database 'password=2hard4u'`
* Get value: `vault kv get secret/databse`

