# The Moving Parts

Incus looks like one command, but it is a
small cast of parts with distinct jobs.
Knowing which part holds what turns later
confusion into a named problem, so here
is the cast before anything is installed.

## The daemon: `incusd`

The daemon is the half that does. It runs
on the host machine, as root, and it:

- creates, starts, stops and destroys
  instances, through **LXC** for
  containers and **QEMU** for virtual
  machines;
- owns the **storage pools** the instances
  live on, the **networks** they join,
  and the **images** they are born from;
- keeps the bookkeeping — every instance's
  configuration, snapshots and state — in
  a database in its state directory (by
  default `/var/lib/incus`);
- serves everything through one **REST
  API**, the same API whether the request
  arrives from the command line beside it
  or from a client across the internet.

The lab met this part for real (chapter
3): started from the project's official
packages, it created its state directory
in scratch, generated its own server
certificate there, and opened its socket,
`unix.socket`, in that directory. The
first lines of `incus info` against it
(transcript `incus-lab/transcripts/t13-server-info.txt`)
show how much a running daemon will tell
you about itself:

```text
api_extensions_count: 562
api_status: stable
api_version: "1.0"
...
  driver: lxc
  driver_version: 7.0.0
  firewall: nftables
```

Five hundred sixty-two API extensions is
the daemon's way of saying the API has
grown by accretion for a decade without
breaking old clients — a fact chapter 13
leans on when one client drives several
servers of different vintages.

## The client: `incus`

The client is the half you touch. It is a
single standalone program — the lab's
copy was one 26-megabyte file that ran
with no installation at all — and it is
deliberately thin: it holds no instance
state, only a list of **remotes** (servers
it knows) and your preferences, in
`~/.config/incus/config.yml`. Everything
it shows you, it asked a daemon for;
everything it changes, it asked a daemon
to change. That thinness is why the
client runs on Linux, macOS, Windows and
FreeBSD (the releases ship all four) while
the daemon runs only on Linux: the client
is a remote control, and a remote control
can be made of anything.

The client's command tree is the map of
the whole product — the lab captured it
in transcript t01, and every chapter of
this guide lives somewhere in it:

```text
Available Commands:
  admin        Manage incus daemon
  cluster      Manage cluster members
  config       Manage instance and server configuration options
  console      Attach to instance consoles
  copy         Copy instances within or in between servers
  create       Create instances from images
  delete       Delete instances
  exec         Execute commands in instances
  ...
  launch       Create and start instances from images
  list         List instances
  ...
  profile      Manage profiles
  project      Manage projects
  publish      Publish instances as images
  remote       Manage the list of remote servers
  snapshot     Manage instance snapshots
  storage      Manage storage pools and volumes
  ...
  webui        Open the web interface
```

## Remotes and the `images:` server

A **remote** is a named server in the
client's configuration. Two are born with
every client (chapter 4 verifies this
against a brand-new configuration):

- **`local`** — the daemon on this very
  machine, reached through its unix
  socket. This is the default remote: a
  bare `incus list` means `local`.
- **`images`** — the project's public
  image server, `images.linuxcontainers.org`,
  which is not a daemon at all but a
  catalogue (**simplestreams** protocol):
  the official starting points — Debian,
  Ubuntu, Alpine, Fedora, Arch and more,
  as containers and as virtual machines.

Nearly every interesting name in Incus
can carry a remote prefix: `images:debian/13`
is an image on the catalogue, `bigserver:web`
would be an instance on a remote called
`bigserver`. When chapter 5 writes
`incus launch images:debian/13 workbench`,
the sentence parses as: *on my current
server, create an instance named
`workbench` from the catalogue's Debian 13
image* — and cause the server to fetch
that image if it does not already keep a
copy.

## Images

An **image** is a frozen starting point:
a root filesystem (and, for VMs, a disk)
plus metadata saying what it needs. A
daemon downloads images into its own
cache and keeps them until told otherwise;
an instance is, at birth, a writable copy
of an image arranged so the image itself
stays pristine. You can also freeze one of
your own instances back into an image
(`incus publish`, chapter 8) — the
project's catalogue is a convenience, not
a gate.

## Profiles

A **profile** is a named bundle of
configuration and devices that instances
inherit. Every instance is born wearing
the `default` profile (and any others you
name), and a profile carries the things
you would otherwise repeat a hundred
times: the root disk on a given pool, a
network connection, resource limits. The
lab read a real one back from its daemon
(transcript t16); chapter 9 explains how
to shape your own. Editing a profile
edits every instance that wears it — a
power to handle with respect.

## Storage pools and networks

A **storage pool** is a named expanse of
disk, managed by a **driver** that decides
how instances are stored on it: a plain
directory, a Btrfs or ZFS filesystem with
copy-on-write snapshots, an LVM volume
group, or a Ceph cluster spread across
machines. Instances live in pools; so do
**custom volumes** — disks that exist
independently of any one instance, to be
attached where needed (the storage
command tree the lab captured shows how
seriously this layer is taken: pools,
volumes, volume snapshots, even
S3-compatible buckets). Chapter 10.

A **network** in Incus is usually a
managed **bridge**: a private little LAN
the daemon runs on the host, complete with
addresses, DNS and NAT out to the world.
The standard initialisation builds one
called `incusbr0`. Instances connect to
networks through their profile, and
chapter 11 is about the choices.

## Projects, the agent, and the web face

Three smaller parts deserve names now.
**Projects** partition one daemon into
named compartments — instances, images
and networks kept apart per project — for
sharing a server tidily. The **Incus
agent** is a small program that runs
inside virtual machines (the releases
ship it for every guest platform) so a VM
can offer the courtesies containers get
for free: `exec`, file transfer, graceful
signals. And the **web interface**: the
client's `webui` command (its help is
transcript t09) opens a browser front-end
served by the daemon, for people and
moments that prefer one. Underneath all
three sits the same API the `incus`
command speaks — nothing in this guide's
later chapters is a private trick.
