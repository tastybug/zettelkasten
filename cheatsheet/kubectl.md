# Kubectl

Check out the link, it has really good example one liners: <https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands>.

## General Structure

### Basics
* `create`: a resource
* `get`: a resource
* `run`: some image (k run -it toolbox --image=ubuntu)
* `delete`: a resource
* `expose`: a pod, creating a service on the fly

### App Management
* `apply`
* `scale`
* `set`: some value in some resource (k set env deploy/bla FOO=bar)
* `exec`: a command on a running container
* `logs`
* `top`: a node or a pod
* `cp`: data into a container
* `describe`
* `attach`: to a running process in some container

## One-Liners

### Namespaces and Contexts Configuration

* `kubectl config get-contexts`: what contexts are there?
* `kubectl config current-context`: what am I using right now?
* `kubectl config use-context CONTEXT`: switch context
* `kubectl config set-context --current --namespace NS` switch namespace for current context

### port forwarding
`kubectl port-forward service/move-listing-guardian 60113:80 --namespace nonprod-seller-app --context move-nonprod-dev-nl`

### Spawn Pod
`kubectl run $USER-debugmobile --context=move-prod-uwe --namespace=prod-seller-app --rm -ti --image=gcr.io/ecg-move-host/tooling -- bash`

`kubectl run curlpod --image=yauritux/busybox-curl -ti --rm --restart=Never -- curl heise.de`

### Create Shell in Running Pod
`kubectl exec seller-listing-enrichment-9 --container seller-listing-enrichment -it -- bash`

### Check Logs, Events
Check logs in a running container (most pods run istio and the actual service in parallel, so you need to specify the container:
`kubectl logs --container seller-listing-enrichment POD-NAME`

See general events in a context+namespace (-w follows)
* `kubectl get events --sort-by=.metadata.creationTimestamp [-w]`
* `kubectl get events -n nonprod-seller-kfk --context move-nonprod-dev-nl`

### Delete Multiple Resources In One Go
* `kubectl delete pods,services,deployments move-listing-guardian`

### Access K8S Secrets
* `kubectl get secret writer --namespace nonprod-seller-kfk -o jsonpath="{.data.password}" | base64 --decode`
* `kubectl get secret reader --namespace nonprod-seller-kfk -o jsonpath="{.data.password}" | base64 --decode`

### copy a file to a pod
  * `kubectl cp /tmp/foo dev-pbartsch/pbartsch-debugmobile:/root [-c container-name]`

### Get Cronjobs (a job is just another pod)

`kubectl get jobs -n nonprod-seller-kfk --context move-nonprod-uwe`

### Restart a deployment
* `kubectl rollout restart deployment move-event-dispatcher --context move-nonprod-dev-nl --namespace nonprod-seller-app`

### Scaling a Stateful Set
* `kubectl scale statefulset seller-listing-enrichment --replicas 0`

