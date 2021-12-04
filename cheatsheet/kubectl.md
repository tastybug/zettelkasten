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

### Oneliners

* port forwarding
  * `kubectl port-forward service/move-listing-guardian 60113:80 --namespace nonprod-seller-app --context move-nonprod-dev-nl`
* spawn pod:
  * `kubectl run $USER-debugmobile --context=move-prod-uwe --namespace=prod-seller-app --rm -ti --image=gcr.io/ecg-move-host/tooling -- bash`
  * `kubectl run curlpod --image=yauritux/busybox-curl -ti --rm --restart=Never -- curl heise.de`: another example
* get shell in a running pod
  * `kubectl exec seller-listing-enrichment-9 --container seller-listing-enrichment -it -- bash`
* check logs in a running container (most pods run istio and the actual service in parallel, so you need to specify the container
  * kubectl logs —container seller-listing-enrichment POD-NAME`
* see general events in a context+namespace (-w follows)
  * `kubectl get events --sort-by=.metadata.creationTimestamp [-w]`
  * `kubectl get events -n nonprod-seller-kfk --context move-nonprod-dev-nl`
* delete resources
  * `kubectl delete pods,services,deployments move-listing-guardian`
* access k8s secrets
  * `kubectl get secret writer --namespace nonprod-seller-kfk -o jsonpath="{.data.password}" | base64 --decode`
  * `kubectl get secret reader --namespace nonprod-seller-kfk -o jsonpath="{.data.password}" | base64 --decode`
* switch "context", which is cluster + namespace + user:
  * `kubectl config get-contexts; kubectl config set-context XYZ`
* see current cluster, namespace (the active one is marked)
  * `kubectl config get-contexts`
* copy a file to a pod
  * `kubectl cp /tmp/foo dev-pbartsch/pbartsch-debugmobile:/root [-c container-name]`
* get cronjobs (a job is just another pod)
  * `kubectl get jobs -n nonprod-seller-kfk --context move-nonprod-uwe`
* restart a deployment
  * `kubectl rollout restart deployment move-event-dispatcher --context move-nonprod-dev-nl --namespace nonprod-seller-app`
* scale a stateful set
  * `kubectl scale statefulset seller-listing-enrichment --replicas 0`

