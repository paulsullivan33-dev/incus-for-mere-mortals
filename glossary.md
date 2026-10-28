# Glossary

**Agent** — the Incus agent, a small
program running inside a virtual machine
so the daemon can offer it `exec`, file
transfer and graceful shutdown (chapter
7).

**Alias** — a client-side shortcut that
expands to other commands (`shell` is the
built-in one, chapter 6) — or, for an
image, a movable nickname like
`debian/13` pointing at a current build
(chapter 8).

**Bridge** — a software LAN on the host.
The daemon's managed bridges give
instances addresses, DNS and NAT (chapter
11).

**Cluster** — several daemons joined into
one logical server, driven through a
single API (chapter 13).

**Config** — the key/value settings on an
instance, a profile, or the server itself
(`incus config ...`, chapter 9).

**Container** — in this guide, always a
*system* container: a complete Linux
system sharing the host's kernel,
isolated by namespaces and cgroups
(chapter 1). Not an application container
of the Docker kind (chapter 14).

**Daemon** — `incusd`, the server half of
Incus: root-owned, stateful, the program
that creates and keeps instances
(chapter 2).

**Device** — a thing attached to an
instance through configuration: a disk, a
network card, a proxy forward, a GPU
(chapter 9).

**Driver** — the implementation behind a
storage pool (dir, btrfs, zfs, lvm, ceph)
deciding how instances are stored and
what snapshots cost (chapter 10).

**Fingerprint** — the hash that names an
image's exact content, or a certificate
for the humans to compare out loud
(chapters 8 and 13).

**Idmap** — the mapping between a
container's user IDs and unprivileged
ranges on the host (`/etc/subuid`,
`/etc/subgid`); what makes an unprivileged
container's root a nobody outside
(chapters 3 and 15).

**Image** — a frozen, signed root
filesystem (plus metadata) from which
instances are born (chapter 8).

**Instance** — one thing Incus manages: a
container or a virtual machine, with its
config, disks, snapshots and state.

**IncusOS** — the project's own appliance
operating system: a machine whose whole
job is to be an Incus host (chapters 3
and 13).

**KVM** — the Linux kernel's hardware
virtualisation support, exposed as
`/dev/kvm`; the gatekeeper for all
virtual machines (chapter 7).

**LXC** — Linux Containers: the underlying
container technology and the project
(Linux Containers) that maintains both
LXC and Incus.

**Preseed** — the YAML file that answers
`incus admin init`'s questions in advance
(chapter 5).

**Profile** — a named bundle of config
and devices that instances inherit;
`default` is the one every instance is
born wearing (chapter 9).

**Project** — a named compartment on a
server, partitioning instances, images
and networks (chapter 13).

**Proxy device** — a device that makes
the daemon listen on a host address and
relay to an instance's port (chapters 9
and 11).

**QEMU** — the machine emulator that runs
Incus's virtual machines, on KVM where
available (chapter 7).

**Remote** — a named server in the
client's configuration; `local` and
`images` are the two every client is born
with (chapter 4).

**Simplestreams** — the catalogue
protocol the `images:` remote speaks: an
index of signed images, fetched by the
client or daemon (chapters 2 and 8).

**Snapshot** — a named, restorable record
of an instance's state inside its pool.
Not a backup (chapter 12).

**Storage pool** — a named expanse of
disk under one driver, holding images,
instance roots and custom volumes
(chapter 10).

**Trust token** — the one-time secret a
server mints (`incus config trust add`)
so a new client may join its list of
trusted certificates (chapter 13).

**Volume** — a custom storage volume: a
disk in a pool with a life of its own,
attachable to instances one at a time
(chapter 10).
