# Virtual Machines

*Documented-only chapter — with one
verified exception, and it is the
chapter's first lesson: this guide's lab
daemon looked at its host, found no KVM,
and declared virtual machines "not
operational" in its own startup log. The
machinery below is taught from the tool's
help and the project's documentation;
that refusal was run for real.*

## The flag is the whole difference

```bash
incus launch images:debian/13 bigbox --vm
```

Same image catalogue (it carries VM
variants of the distributions), same
profile, same verbs afterwards — but what
is born is a QEMU virtual machine with
its own kernel, emulated hardware, and a
virtual disk, instead of a container
sharing the host's kernel. `incus create
--vm`, `incus init --vm`: the flag rides
on every creation command, and the lab's
capture of the tool's own help
(transcript `incus-lab/transcripts/t04-launch-help.txt`)
shows the project's examples leaning on
it:

```text
  incus launch images:debian/12 v1 --vm -c limits.cpu=4 -c limits.memory=4GiB
      Create and start a virtual machine named "v1" with 4 vCPUs and 4GiB of RAM

  incus launch images:debian/12 v2 --vm -d root,size=50GiB -d root,io.bus=nvme
      Create and start a virtual machine named "v2", overriding the disk size and bus
```

Note the device example: a VM's root is a
real disk with a size and a bus
(`nvme`, `scsi`), because a VM, unlike a
container, cannot simply *be* a directory
in the pool. In `incus list` the two kinds
sit in one table with a `TYPE` column —
`CONTAINER` or `VIRTUAL-MACHINE` — and
chapter 5's verbs do not distinguish
them. What distinguishes them is under
the floorboards:

| | Container | Virtual machine |
|---|---|---|
| Kernel | the host's, shared | its own |
| Start time | about a second | a boot: tens of seconds |
| Memory | soft limits, shares host RAM's slack | the RAM you assign is set aside |
| Isolation | kernel namespaces; strong, not absolute | hardware virtualisation; the stronger wall |
| Guests | Linux only | Linux and BSDs practically; other systems are not the catalogue's business |
| Needs | any modern Linux host | a host with KVM (`/dev/kvm`) |

## The agent: how a VM learns manners

A container needs no help to offer `exec`,
file transfer, and tidy shutdowns — the
daemon is already halfway inside it. A
VM is a stranger behind a wall, so Incus
runs a small program inside it, the
**Incus agent** (the releases ship it per
guest platform), and the catalogue's VM
images carry it. With the agent alive,
`incus exec`, `incus file`, `incus shell`
and graceful `incus stop` work on a VM
exactly as on a container; the daemon's
`incus info` also gains the VM's real
addresses and usage. Without it — a
foreign disk image, an agent stopped —
you are reduced to the console, which is
why the next section exists. The agent is
also the honest boundary of the VM
convenience story: everything pleasant
about driving a VM like a container flows
through a guest-side helper the guest can
switch off.

## Consoles, graphical and otherwise

`incus console <name>` attaches to the
VM's serial console — boot messages,
a login prompt, the door that works when
networking and agent are both dead.
`incus console --type=vga <name>` opens
the graphical display instead, handed to
a SPICE viewer (the client has a
`console_spice_command` setting for
choosing yours; it sat empty in the lab's
fresh configuration, chapter 4). A VM can
also carry a `tpm` device for a virtual
TPM, and Secure Boot is supported for the
catalogue images — the full PC costume,
for guests that expect one.

## When the host says no

Back to the lab's verified refusal
(transcript t12):

```text
level=warning msg="Instance type not operational" driver=qemu err="KVM support is missing (no /dev/kvm)" type=virtual-machine
```

The daemon did not crash, refuse to
start, or pretend: it started, marked the
whole `virtual-machine` type inoperative,
and served containers (as far as the
sandbox let it). That is the general
shape of a KVM-less host — an old machine
with virtualisation switched off in its
firmware, a cloud instance with no nested
virtualisation, a containerised sandbox
like this guide's. The lesson to carry:
Incus needs no KVM to be useful, but the
`--vm` half of its promise does, and
`ls -l /dev/kvm` on the host answers the
question before any guide has to.

Choose containers for density and speed —
dozens of Linux systems on one host,
started in a blink. Choose VMs when the
guest needs its own kernel (a different
one, a patched one, a BSD), when the
isolation must be hardware, or when the
workload is hostile enough that you want
the stronger wall — remembering chapter
15's companion truth, that neither kind
is a wall against the host's own root.
