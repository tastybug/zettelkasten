# Golang Tooling

Build an execute main.go (assuming it contains main function). another.go is assumed to be used by main.go.
* `go run main.go another.go`

Adding a dependency. It will automatically be added to go.mod and go.sum.
* `go get rsc.io/quote`

List dependency tree
* `go list -m all`

## Updating a dependency

First list all deps
* `go list -m all`

Spot a dependency you want to update, e.g. rsc.io/sampler.
* `go get rsc.io/sampler`
