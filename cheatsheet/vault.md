# Vault Cheatsheet

## Starting a CLI session

1. Have `vault` installed to have access to the CLI tools
2. Point `vault` to the server: `export VAULT_ADDR='http://127.0.0.1:8200'`
4. If you have a token, just go with `vault login` or specify any enabled auth method, e.g. userpass: `vault login -method=userpass username=pbartsch`
5. Alternatively, with a token already at hand, set a `VAULT_TOKEN=asd123` env variable.
6. There is now a session, further commands don't have to be authenticated (the token is stored in `~/.vault-token` and automatically read from there)

## Admin work 

### Configuring Auth Methods

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

```
$ vault auth enable -path=approller approle
Success! Enabled approle auth method at: approller/

$ vault write auth/approller/role/move-listing-guardian policies=kv-wiz2 token_ttl=20m
Success! Data written to: auth/approller/role/move-listing-guardian

# the value of role_id is the username later on
vault read auth/approller/role/move-listing-guardian/role-id
Key        Value
---        -----
role_id    2acbb216-271a-6d58-7c60-73f2b3faeecb

# secret_id contains the password for the login
$ vault write -force auth/approller/role/move-listing-guardian/secret-id
Key                   Value
---                   -----
secret_id             bc848681-e013-63a5-0c83-30f8130af182
secret_id_accessor    13f37157-d4ce-3f16-2a59-b0973d6816f8
secret_id_ttl         0s

# produce a token using the "user" and "password"
$ vault write auth/approller/login role_id=2acbb216-271a-6d58-7c60-7
3f2b3faeecb secret_id=bc848681-e013-63a5-0c83-30f8130af182
Key                     Value
---                     -----
token                   s.WtBjMPdstV2CEVv28eAQQrrL
token_accessor          HE6tdJz1KSw5DhZ6diuHOMXM
token_duration          20m
token_renewable         true
token_policies          ["default" "kv-wiz2"]
identity_policies       []
policies                ["default" "kv-wiz2"]
token_meta_role_name    move-listing-guardian

# optionally log in now with the token
vault login s.WtBjMPdstV2CEVv28eAQQrrL
```

### Setting Up a New Policy

1. `vault policy list` gives you existing policies
2. `vault policy read default` describes what the `default` policy is allowing. The format given is HCL.
3. `vault policy read root` fails funnily enough, even though it exists.
4. `vault policy fmt FILENAME` formats a policy file
5. `vault policy write funky-policy /tmp/funky.hcl`
6. `vault token create -policy="funky-policy"` to create a token to test things out

## Working with K/V stores using CLI

* Check under which path the KV store is: `vault secrets list`
* Add value: `vault kv put secret/database 'password=2hard4u' 'user=dba'`
* Get value: `vault kv get secret/databse`


## Best Practices

### Dynamic Secrets a.k.a Dynamic Credentials

Explain this closer with an example of AWS.

### Periodic, Renewable Tokens

Imagine you have a docker image that will be in use for months or years. The image will pull vault secrets before starting another process. It's hard to create a new token each time the container starts.

Solution: create a periodic token. Periodic tokens have a TTL but no max TTL. They can live forever, as long as you renew them within their TTL.

Make sure that the token has a policy attached to it that allows it to renew itself. Creating a token with `period` requires `sudo` capability.

```
# (you logged in before)
vault token create -policy=some-policy -period=2m
# do some work
vault token renew $token

```

### I need a token with a use limit

You want a token that can only be used once. Watch out, logging in already seems to consume a use.

```
vault token create -use-limit=3
```

### Give me a token that does not expire due to its parent

Revokation of a parent token due to TTL or manual revokation always revokes all child tokens. You want a token that is not affected by this. This requires `sudo` capability.

```
vault token create [-policy=bla] -orphan
```

### Log in, put token into a variable

1. `export VAULT_FORMAT=json`
2. `OUTPUT=$(vault login -method userpass username=pbartsch password=2hard4u)`
3. `VAULT_TOKEN=$(echo $OUTPUT | jq '.auth.client_token' -j)`
4. `vault login $VAULT_TOKEN`

### Create & Use Token via API (example with `userpass`)

It's possible to remotely log in and create a token that way. This example assumes that there is a user `pbartsch` with password `123` set up.

```
# produce a token using userpass method
$ echo '{"password": "123"}' > payload.json
$ curl -X POST -d @payload.json http://127.0.0.1:8200/v1/auth/userpass/login/pbartsch | jq .auth.client_token
"s.U970zeIP86fkAKVbMCU7EQw3"

# let's try a call with it
$ curl -X POST -d '{"token": "s.U970zeIP86fkAKVbMCU7EQw3"}' -H 'Authorization: Bearer s.U970zeIP86fkAKVbMCU7EQw3' http://127.0.0.1:80/v1/auth/token/lookup | jq
```