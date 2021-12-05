# Terraform

Like puppet, it’s a declarative configuration management. It comes with many resource types, is idempotent. Unlike puppet, there is no agent deployed. You run your changes and push them out.

TF uses a syntax called HLC, Hashicorp Configuration Language.

TF has an internal state that it keeps after running. It contains logical resources like generated keys, random data as well as outputs that you defined in your code. It also contains the ids of all resources created and is thus connected to the real world state.

## Modules

Resources always part of a module, if no particular modules has been created, stuff is in the “root module”. Resources in a module do not leak outside, but by using “variables” and “output”, you can accept arguments or return values from and to the outside.

See <https://www.freecodecamp.org/news/terraform-modules-explained/> for some examples on how to achieve module parameters and return values.


## Simplest dockerized sample project

Create a main.tf with this content:

```
resource "local_file" "pets" {
  filename = "./pets.txt"
  content = "I like dogs."
}
```

Then execute this with `docker run -i -t -v $PWD:$PWD --rm -w $PWD hashicorp/terraform:light apply`


