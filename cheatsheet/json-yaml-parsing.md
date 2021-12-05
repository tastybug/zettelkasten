# Working with JSON, YAML with yq, jq and python

## YQ

Filter like jq

`cat pod.yml | yq e '.kind'`

Convert to Json

```
cat pod.yml | yq -t=json e '.' -
cat pod.yml | yq -t=json e '.' - | jq '.kind'
# this produces a one-liner
cat pod.yml | yq e -j -I 2
```

## Plain Python

`python3 -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin, Loader=yaml.FullLoader), sys.stdout, indent=4)' < input.yaml > output.json`

## JQ

Filter Array Of Objects by Property Value
To filter the content of an array of objects by property value, use this:

```
# given json
{
  array: [
	{name: “front-end”, secret: “bla”},
	{name: “back-end”, secret: “blubb”}
  ]
}

# this will give you "blubb"
cat bla.json | jq '.array[] | select(.name == "front-end") | .secret'
```

