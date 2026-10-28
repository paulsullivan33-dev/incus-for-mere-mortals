# Incus for Mere Mortals

Most tools that run other computers make
you choose a kind. Docker and Podman run
**containers**: programs wrapped up with
their files, sharing the host's kernel.
Virtual machine tools run whole simulated
computers, each with its own kernel.
**Incus** refuses the choice. One daemon
and one command manage both kinds side by
side — **system containers** that behave
like complete Linux machines (services,
logins, init and all) and **virtual
machines** proper — and treat them the
same: launch, list, configure, snapshot,
copy, move. If you have ever wanted a
spare Linux box to try something on,
without buying one or rebooting anything,
this is the tool that conjures it.

This guide explains how Incus is put
together, how to install it, how to talk
to it when there is not yet anything to
talk to, how a first container would be
launched and used, and how the same verbs
drive virtual machines, images, storage
pools, networks and snapshots. The limits
are stated plainly, including the biggest
one: the daemon this guide teaches you to
run holds **root-level power** over its
host, and every convenience in the later
chapters flows from that fact.

The rule that governs everything here is
stated in chapter 1 and repeated in
chapter 15: **run instances on machines
you own, or machines you have explicit
permission to administer.** The lab obeyed
it strictly. Every command in this guide
ran on one borrowed sandbox machine,
against a daemon whose entire world —
its state, its storage pool, its one
attempted download — lived in a scratch
folder that was deleted afterwards.

The lab behind this guide ran the real
thing, in two halves. The **client** half
is the official standalone binary of
**Incus 7.0.2** (the current long-term
support release), downloaded from the
project's own GitHub releases with its
SHA-256 checked against the release's
checksum file. With it the lab read the
full command tree, listed the built-in
remotes, watched the client write its
first configuration file, and met — in
three different wordings — the error
every new user meets first: *the daemon
doesn't appear to be started*. The
**server** half is **incusd 7.5.1** from
the project's stable packages for Ubuntu
(the Zabbly repository the documentation
recommends), unpacked into scratch rather
than installed. The daemon genuinely
started, made its state and its socket in
the scratch folder, answered questions,
accepted an initial configuration from a
preseed file, and created a real storage
pool — and its startup log is this
guide's most honest page, because it
lists what this sandbox could not give
it: no KVM device, so virtual machines
were declared *not operational* by the
daemon itself; no cgroup controllers, so
resource limits would be ignored; and no
working route to the image server, so
the first launch waited on a download
that never arrived.

What the lab could not do, it did not
fake. No instance ever ran: the image
download died on this sandbox's network,
and the chapters that drive instances —
launching, shells, snapshots, copies —
are taught from the tool's own help text
and the project's documentation, and are
labelled documented-only wherever they
appear. `criteria.md` keeps the full
ledger of what was run and what was only
read.

## What this guide covers

- [**What Incus is**](what-incus-is.md) —
  the mental model: whole computers as
  things you can launch from a command
  line, two kinds of instance, and the
  responsibility that is part of the tool
- [**The moving parts**](the-moving-parts.md) —
  the daemon, the client, remotes, images,
  profiles, storage pools and networks,
  and which part holds what
- [**Installing Incus**](installing-incus.md) —
  the package route for a server, the
  standalone client for any machine,
  and what "installed" does and doesn't
  mean until the daemon is initialised
- [**First contact with a server**](first-contact.md) —
  the client with no server: the remotes
  it is born knowing, the config file it
  writes, and the not-running error in
  its several wordings
- [**Your first container**](your-first-container.md) —
  initialising the daemon, launching a
  container from an image, and the
  everyday verbs: list, info, stop, delete
- [**Working inside an instance**](working-inside.md) —
  shells and commands with `exec`, files
  in and out, the console, and forwarding
  a port to your own desk
- [**Virtual machines**](virtual-machines.md) —
  the `--vm` flag, the agent that makes a
  VM behave, graphical consoles, and how
  a VM differs from a container here
- [**Images**](images.md) — where images
  come from, the `images:` remote, caching,
  aliases, and publishing an instance as
  an image of your own
- [**Shaping instances**](shaping-instances.md) —
  configuration keys, resource limits,
  devices (shared folders, forwarded
  ports, GPUs), and profiles: settings
  with a name
- [**Storage pools and volumes**](storage.md) —
  where instances live: pool drivers from
  plain directories to ZFS, custom
  volumes, and what the choice costs
- [**Networking**](networking.md) — the
  managed bridge, addresses and NAT,
  private networks of your own, and
  letting the outside world reach an
  instance on purpose
- [**Snapshots, copies and backups**](snapshots-copies-and-backups.md) —
  freezing a moment, restoring it, cloning
  an instance, moving one, and the export
  file that is a real backup
- [**Remotes and other servers**](remotes-and-servers.md) —
  one client, many daemons: adding a
  server, trust and tokens, projects,
  the web interface, and clustering
  pointed at rather than taught
- [**The neighbours**](the-neighbours.md) —
  Docker and Podman, plain LXC, LXD,
  libvirt and Proxmox: what each does
  that Incus doesn't, and vice versa
- [**Limits**](limits.md) — root in a
  trench coat, containers that are not
  a security boundary, the kernel and
  hardware the tool quietly requires,
  and the honest ceiling of this guide's
  own lab
- [**Troubleshooting**](troubleshooting.md) —
  the daemon that isn't running, the
  image that won't download, the pool
  that refuses, and the warnings this
  guide's lab met so you can name yours
- [**Glossary**](glossary.md) — the
  terms, collected in one place

## What it doesn't cover

Administering machines you do not own or
have no permission to manage — that is
not a harder chapter of this book, and
chapter 15 explains why the line sits
where it does. Running instances for
other people as a hosting business —
accounts, billing, abuse desks,
isolation audits — is pointed at, not
taught; this guide stays with one person
and their own machines. High-availability
clustering across many servers is
introduced and left to the project's
documentation. Building your own images
from scratch (the project's distrobuilder
tool) and IncusOS, the project's own
appliance operating system, are named
and described, not walked through.

## How to use this guide

Read chapter 1 for the model and the
rule, and chapter 2 before installing
anything — knowing which part holds what
turns later confusion into a named
problem. Chapter 4 is deliberately
written for the moment *before* anything
works: if your first commands meet the
not-running error, that chapter is the
map. Chapter 5 initialises a daemon and
launches; do it first on a machine you
can afford to reformat, because a storage
pool is a real commitment of real disk.
Chapter 9 is the one that turns a running
instance into a *fitted* one — limits,
shared folders, profiles — and chapter 12
is the one that makes experiments safe,
because a snapshot you trust changes what
you dare to try. The lab's transcripts
live in `incus-lab/transcripts/`; its
command list is `examples/commands.txt`
and its notes are `examples/lab-notes.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Virtual Machines for Mere Mortals](https://paulsullivan33-dev.github.io/virtual-machines-for-mere-mortals/)
- [Podman for Mere Mortals](https://paulsullivan33-dev.github.io/podman-for-mere-mortals/)
- [Docker for Mere Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/incus-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/incus-for-mere-mortals/blob/main/LICENSE-CODE).
