# Shell Scripting Notes

### System Shell
`/bin/sh` is what defines the _system shell_, which is the shell that is picked up in scripts via the shebang. It is usually a symbolic link to bash, dash or something else.

### Login Shell
The login shell of a user depends on the setting in `/etc/passwd` and can be altered via `usermod --shell /bin/bash USER`

### Available Shells
A list of avaiable shells and pseudo-shells can be listed with `cat /etc/shells`.

