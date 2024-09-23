# envsubst

Basic templating on the command line. Takes a template file and replaces all mentioned env variables.

## Examples

With a template file for complex cases:
```
$ echo “Greetings to $TO from $FROM.” > template.txt
$ TO=John envsubst < template.txt
> Greetings to John from
$ TO=John FROM=Jane envsubst < template.txt
> Greetings to John from Jane
```

Without template file, with piping right into `envsubst`:
```
$ echo "Hi $TO" | envsubst
> Hi
$ export TO=Phil
$ echo "Hi $TO" | envsubst
> Hi Phil
```

If you want to only expand certain variables and leave others, mention those that you want expanded:

```
$ TO=John FROM=Jane envsubst '$TO' < template.txt
> Greetings to John from $FROM.
```

### Alternatives

<https://github.com/a8m/envsubst>
<https://github.com/drone/envsubst>

Both offer improvements like default values, pattern matching etc.
