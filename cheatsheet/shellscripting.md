# Shell Scripting Notes

## Safety flags

```
set -o errexit
set -o pipefail
set -o nounset
```

Errexit makes sure that the script execution stops as soon as a command is not successful. Remember that “a command” can consist of multiple single commands linked with pipes, ampersands etc.
For exceptional situations where it would be ok to fail, use `command || true`.

Pipefail supports errexit to catch situations where a failing command is before a pipe, which errexit does not catch (e.g. `cat /doesnotexist | wc -l`).

Nounset prevents execution if any variable is not set. Here it helps to use default values for variables, e.g.
* `${VARIABLE:-}` this defaults to empty value
* `${VARIABLE-cool}` this defaults to `cool`
