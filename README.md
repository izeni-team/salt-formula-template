# Salt Template #

A starting point to express testing tools and best practices for
custom salt formula.

## Structure ##

A changelog should be maintained to express clearly high level changes,
as well as pinned versions to those releases.

README.rst is a template file for the instanciation of a formula that
should replace this readme.

Salt installes the directory according to a namespace declared in the
root project directory. `./{{ template }}` should be named
according to what the state will be called when installed to salt.
Be careful to avoid naming collisions with the installed server.

## Usage ##

Provided for your convenience are a series of docker-compose tools
to aid in the rapid development. These require docker and docker-compose
to be used.

### Aliases ###
`source aliases` will provide the ability to pass commands to the master
or minion by title (e.g. `master salt-key -L` or `minion salt-call state.sls foo`).

### Cookbook ###

#### Start a Test Cluster ####

```
> docker-compose up -d
> master salt-key -A  # Accepts minion
```

#### Increase Number of Minions ####

```
> docker-compose scale ubuntu=2
> master salt-key -A  # Accepts new minions
```

#### Test the Formula ####

```
> master salt '*' state.sls formula
```


## Init System Testing ##

E.g. upstart, systemd, etc.

Init systems cannot be readily tested in a docker environment and thus
require a fully fledged VM host to test against. I've documented the
process of setting this up, because it is not officially documented as
of Jan 31, 2017.


* Install virtualbox
* Create a virtualbox base image as root -- Base image must offer sshd with user/pass or user/key
* Register runner -- See below for sample config.


__Runner Config Sample__
```
[[runners]]
  name = "$RUNNER_NAME"
  url = "$FQDN/ci"
  token = "$TOKEN"
  executor = "virtualbox"
  [runners.ssh]
    user = "$GUEST_USER"
    password = "$GUEST_PASSWORD"
  [runners.virtualbox]
    base_name = "$VM_TO_CLONE"
    disable_snapshots = false
  [runners.cache]
```

### Overview of How Virtualbox Executor Works ###
* Runner attempts to find $base_name VM.
* Runner attempts to find $base_name VM's $base_snapshot or default ('Base State').
* Runner attempts to find or clones $base_snapshot into $base_name-$key-concurrent-$id.
  - If it did not previously exist, runner snapshots as 'Started'.
  - If it previously existed, the runner reverts to 'Started'.
* Runner connects to clone via SSH and runs the scripts in your job.

### Gotchas ###
* VM must be stopped before executor uses it.
* Virtualbox image must be run as root to setup daemon.
* When requiring sudo access, you will want to exclude sudo password and set a SUDO_ASKPASS env -- `apt install ssh-askpass`
* Runner creates a snapshot point when it is run the first time. If you need to change the base, delete the snapshot.
* VM snapshot must be removed to update from your base_name.
