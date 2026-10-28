# The Neighbours

Incus sits in a crowded street. Each
neighbour below does something Incus
doesn't, or refuses something Incus does;
knowing the boundaries keeps you from
forcing the wrong tool.

## Docker and Podman: application containers

The famous containers are **application
containers**: one program, packaged with
its files, built from a Dockerfile,
pulled from a registry, run a thousand
times identically. Incus's containers are
**system containers**: a whole Linux with
init, cron, ssh and a package manager,
meant to be lived in and administered
like a machine. Choose Docker/Podman when
the unit of work is a program to deploy;
choose Incus when the unit of work is a
computer to use. The two coexist happily
on one host (and people run Docker
*inside* Incus containers, ideally
unprivileged ones, as a way of giving
Docker a fence). This series' own Docker
and Podman guides — named in the footer —
teach those tools on their own terms.

## Plain LXC: the engine room

Incus's containers *are* LXC containers,
driven through LXC's library; `lxc-*`
tools manage the same technology by hand,
one container per config file, no daemon,
no API, no images service. You would go
there to learn how the sausage is made,
or on a system too small for a daemon.
Incus exists precisely because running a
fleet by config file stops being fun —
everything in chapters 5–13 is the delta.

## LXD: the parent

LXD is Incus's ancestor and nearest
relative (chapter 1 tells the fork
story): same architecture, same command
shape, different stewardship and licence
— LXD is Canonical's product now; Incus
is the Linux Containers community's.
Servers and clients are *not*
interchangeable, but the project's
`lxd-to-incus` migration tool (shipped in
the same releases the lab downloaded)
converts a host in place. Advice written
for one generally translates to the other
with names changed; allegiance is a
matter of which project's hands you want
on your infrastructure.

## libvirt, QEMU by hand, VirtualBox

QEMU is the machine under Incus's VMs;
**libvirt** (with `virt-manager` as its
friendly face) is the older, broader
virtualisation manager — more guest
variety, more knobs, a GUI tradition, and
none of Incus's container side. VirtualBox
is the desktop-first cousin: excellent
for a Windows or desktop guest on your
desk, not a headless-server tool. Choose
libvirt when the workload is *virtual
machines as a discipline*; choose Incus
when VMs are one kind of instance among
two and the command table is the
interface you want. The series' Virtual
Machines guide (also in the footer) maps
this territory tool by tool.

## Proxmox VE: the appliance

Proxmox is a whole *platform*: a Debian
host pre-assembled with a web UI,
KVM virtual machines and — note — LXC
system containers, plus clustering,
backup tooling and storage integration.
If you want a dedicated virtualisation
*server* with a browser console and a
vendor's opinion about how it all fits,
Proxmox is the neighbour to beat. Incus
is the lighter claim: a daemon and a
command you add to a Linux you already
run, scripted from the terminal, with
your own choices for the rest. IncusOS
(chapter 13) is the project's own step
toward the appliance shape, from the
other direction.

## systemd-nspawn, Vagrant, and the rest

**systemd-nspawn** runs a directory as a
simple machine with almost no ceremony —
lovely for a quick chroot-with-benefits,
no image service, no remote API.
**Vagrant** is a workflow tool for
*development environments* across many
backends (VirtualBox, libvirt, cloud) —
it would happily treat an Incus host as
one of its providers' neighbourhoods.
**Firecracker** and the microVM world
optimise VM startup for serverless
density — an industry tool, not a
homelab one. And at the far end of the
street, **Kubernetes** orchestrates
application containers across fleets: a
different question (which copies of my
program run where) from Incus's question
(here is a computer; do something), with
this series' K3s and Kubernetes guides
posted at that corner.
