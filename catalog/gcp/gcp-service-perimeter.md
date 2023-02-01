# Service Perimeter

When deciding whether one is able to perform an action, IAM is taken into account, which is an identity based decision. On top of that, GCP offers "Perimeters", which declare different cloud resources in one or more projects to be part of the same "context", so to say. Everything that is part of that context can interact with other resources in that context.

Example: A VM in `Project A` can copy data from `Bucket A` in Project A to a `Bucket B` in Project B, if both projects are part of the security perimeter.
The VM is not able to copy it to a `Bucket C` outside of the perimeter.

Service Perimeters can be found in the "Security" toolbox in Cloud Console, under `VPC Service Controls`. They require having an organization, which makes it hard to test for me :)
