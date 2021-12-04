# Ansible Recipes


## Get facts for a given host

Let's assume you are on the host already and ansible is installed:

`ansible -i,localhost -e ansible_connection=local all -m setup`

If the host is remote instead but you are able to ssh to it:

`ansible -i,192.168.178.30 all -m setup`

If there are a number of remote hosts and you want all vars collected and dumped locally, use this playbook:

```
tasks:
  - name: using template, create a remote file with all the vars in it
    template:
         src: templates/dump_variables
         des: /tmp/ansible_variables
  - name: fetch the files, copy them back to the control host
    fetch:
      src: /tmp/ansible_variables
      dest: "captures/{{ ansible_hostname }}"
      flat: yes 
```

with templates/dump_variables:
```
Playbook VARS:

{{ vars | to_nice_yaml }}
```

## Run a one-off command on one or more hosts

This is not going through a shell but executed directly (shell variables are thus not available):

`ansible -i,somehost all -m command -a 'cat /etc/os-release'`

If you want it done with a shell: 

`ansible -i,somehost all -m shell -a 'echo ${SHELL} > /tmp/blabla'`

What variables are available on a target during execution?

This prints all the facts:
`ansible -i,localhost -e ansible_connection=local all -m setup`

This prints hostvars, groupvars and so on:
`ansible -i localhost, -m debug -a 'var=hostvars[inventory_hostname]'`

And there is also this (untested): https://github.com/f500/ansible-dumpall

### Waiting for certain conditions to be true before continuing

To wait for a particular condition to be true before executing an action, there are a number of modules that help:

* wait_for: check for files or ports to be open/closed/present/absent
* service: is not a check, but you can ensure that a OS level service (systemd, initV, ..) is started/stopped/reloaded/restarted

## Complex conditionals

Problem: you want to decide whether to execute a task on a condition that is not visible via a fact. But there is a command that you can run which will give you parsable output that you can decide on?

Solution: use "registers". When a task is run, you can add a "register: some_var_name". This "some_var_name" will then contain a json dictionary with some stats, like stdout output, exit code, host it ran on and so on. Example:

```
ok: [ubuntu5] => {
    "some_var_name": {
        "changed": true,
        "cmd": [
            "hostname",
            "-s"
        ],
        "delta": "0:00:00.003245",
        "end": "2021-09-17 15:10:58.819339",
        "failed": false,
        "rc": 0,
	"stdout": "ubuntu5"
	// .. and so on
```

In a task dependent on this information, you can add a when block that checks the register:

```
    - name: some dependent task
      debug:
        msg: "I'm only supposed to show up when the register fits the given regex!"
      when:
        some_var_name.stdout is regex("[a-z]*5")

```

### Simple special case: only run a task, if another task is in state "changed"

Here you can either:
1. use "notify" to call a handler, which in itself can then execute some task
2. use "register" and then check the register variable like this: ` when: myregistervar is changed` or `when: myregistervar.changed`

## Mini Project

Have a hosts file:

```
[servers]
ubuntu1 // this is a host name
```

Have a ansible.cfg in either: /etc/ansible/ansible.cfg or ~/.ansible.cfg or in ./ansible.cfg

```
[defaults]
inventory = hosts
```

Then run the following module to see what facts ansible sees for the target:

`ansible servers -m setup -o | cut -d'>' -f2 | jq . | less` 

