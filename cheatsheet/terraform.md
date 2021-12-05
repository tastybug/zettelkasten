# Terraform

Prepare local workspace by downloading providers
* `terraform init`

Refreshes the internal state (but does not overwrite the persisted state), prints actions to be done.
* `terraform plan`

Apply the changes
* `terraform apply`

Display current state, including logical resources like “tls_private_key”.
* `terraform show [-json]`

Display defined outputs according to the current state.
* `terraform output [var-name]`

Display the state.
* `terraform state`

Validates the configuration files.
* `terraform validate`

Format the configuration code.
* `terraform fmt`

List providers used and the binary source for each.
* `terraform providers`

Update the local state according to reality.
* `terraform refresh`
