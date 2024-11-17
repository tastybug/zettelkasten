# Working with YAML

## Command Line: `yq`

`docker run --rm   -v "$PWD:$PWD" -w="$PWD" mikefarah/yq:4.13.0 e '.somekey' example.yml`

## Verification, Linting

`yamllint example.yml`; return value is `1` on problems.

`docker run --rm -it -v "$PWD:/data" cytopia/yamllint invalid.yml`

## Multiple Documents in the Same File
A YML file can have multiple documents, separated by `---`. The documents are independent of each other and can be accessed by the respective index:

```
# example.yml
name: Alice
age: 30
---
name: Bob
age: 25
---
name: Charlie
age: 35
```

Then, when using v4 of `yq`:

```
yq e 'select(documentIndex == 1) | .name' example.yml
```
