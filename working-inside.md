# Working Inside an Instance

*Documented-only chapter: the lab's
daemon never ran an instance (chapter 5
tells why), so everything here is taught
from the tool's help and the project's
documentation, and labelled as such in
`criteria.md`. The one verified artefact
is small and real: the `shell` alias,
below, which the lab read from a fresh
client's own configuration.*

An instance you cannot get inside is an
aquarium. Incus gives four doors, each
with a different temperament.

## `exec`: commands without ceremony

```bash
incus exec workbench -- apt update
```

runs one command inside the instance and
returns its output and its exit code, as
if it had run on your own desk. The `--`
matters: everything after it belongs to
the instance, flags and all. `exec` is
the scripting door — loops over ten
instances, health checks, configuration
management — and it works on virtual
machines too, through the agent of
chapter 7.

## `shell`: the login you actually wanted

```bash
incus shell workbench
```

drops you at an interactive login shell
as root inside the instance. It is not a
command but an **alias**, and the lab
verified its definition in a brand-new
client before any server existed
(transcript `incus-lab/transcripts/t06-alias-list.txt`):

```text
│ shell │ exec @ARGS@ -- su -l │
```

`shell` expands to `exec <instance> --
su -l`: a real login shell, with your
instance's root environment, no password
asked — the daemon vouches for you. That
vouching is the convenience and the
warning in one: anyone who can run this
client against this daemon is root in
every instance on it. Chapter 15 develops
the point. `incus console workbench` is
the fourth door: the instance's own
console, the screen it would show a
monitor — the way in when networking is
broken or the system is still booting,
and for VMs optionally a graphical
console (`--type=vga`, chapter 7).

## Files in and out: `incus file`

```bash
incus file push ./notes.txt workbench/root/notes.txt
incus file pull workbench/var/log/app.log ./app.log
incus file edit workbench/etc/hosts
```

`push` and `pull` copy files and whole
directory trees between your machine and
the instance; `edit` opens the instance's
file in your editor and writes it back on
save. Ownership and permissions are
options (`--uid`, `--gid`, `--mode`), and
for containers the copies do not need the
instance running — a stopped container's
files are reachable, which rescues many a
broken boot. For VMs the agent carries
the traffic, and the instance must be up.

## Reaching a service: `port-forward`

An instance on the daemon's bridge has an
address on the host's private network,
not on yours — when the client is on a
different machine, its services can feel
far away. Chapter 11 covers the standing
answers (proxy devices, networks you
design). The quick, personal answer is in
the client's command list:

```bash
incus port-forward bigserver:workbench 8080
```

forwards a port on *your* machine through
the daemon to the instance, for as long as
the command runs — an SSH tunnel's
convenience without the SSH. It is the
right shape for a look at a web interface
or a database port: explicit, temporary,
and gone when you stop it.

## House rules for guests

Three habits the documentation and
experience agree on. First, prefer `exec`
and configuration (chapter 9) to hand
edits you will forget; an instance you
built by commands can be rebuilt by
commands. Second, treat `shell` the way
you treat root anywhere — because it is.
Third, remember the doors have a pecking
order when things break: `exec` needs a
healthy instance and (for VMs) a healthy
agent; `console` needs neither network
nor agent, and is the door that still
opens during a bad boot. Chapter 16's
troubleshooting leans on exactly that
order.
