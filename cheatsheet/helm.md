# Helm

It's declarative configuration management for Kubernetes, analogous to Puppet. You declare a bunch of resources and Helm makes sure it's there. A Chart is like a puppet resource declaration. A release is a concrete deployment of a chart. There can me multiple releases of a Chart of course with different names.

Helm has a central, public chart repository at <https://artifacthub.io>. In helm commands this is called "hub".

## Use Cases

`helm repo list` lists repos.

`helm search repo guardian` does a free text search for guardian in charts in all repos

`helm search hub nginx` free text search ("nginx") in charts on the public artifacthub.io or the on-premise version of it

`helm show all move/move-listing-guardian` display chart, values of a chart

`helm list` list all releases in current context + namespace. (change with kubectx and kubens or run `helm list --kube-context move-nonprod-dev-nl --namespace nonprod-seller-app`)

`helm rollback move-listing-guardian` rollback a release to a previous version (in current context + namespace or run `helm rollback move-listing-guardian --kube-context move-nonprod-dev-nl --namespace nonprod-seller-app`)

`helm rollback move-listing-guardian` rollback a release to a previous version (in current context + namespace or run `helm rollback move-listing-guardian --kube-context move-nonprod-dev-nl --namespace nonprod-seller-app`)

`helm uninstall move-listing-guardian` uninstall a release (in current context + namespace or run `helm uninstall move-listing-guardian --kube-context move-nonprod-dev-nl --namespace nonprod-seller-app`)
