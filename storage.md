# Storage Pools and Volumes

Storage is where Incus stops being an
abstraction: every instance is files and
blocks on a real disk, and the **storage
pool** is the arrangement. The lab
created a real pool on a real daemon —
the plainest kind, on purpose — so this
chapter's core is verified; the richer
drivers are described from the project's
documentation and labelled accordingly.

## One pool, created for real

From chapter 5 (transcripts
`incus-lab/transcripts/t14-admin-init.txt`
and t15): the lab's preseed declared a
pool named `default` on the **dir**
driver with a folder as its source; the
daemon refused until the folder existed,
then created the pool and reported it
`CREATED`, already *USED BY 1* — the
default profile's root disk. A pool,
once made, is mostly visited by three
commands:

```bash
incus storage list          # the pools
incus storage info default  # what lives there, and how much room is left
incus storage show default  # its configuration
```

`incus storage create` makes more pools
after init, and an instance chooses its
pool through its root disk device
(chapter 9) — so "put the database
instances on the fast pool" is a profile
edit, not a migration project, until the
data itself has to move (`incus move
--storage`, chapter 12).

## The drivers, honestly compared

The driver decides what a pool *is*
underneath, and the choice trades
features against requirements:

- **dir** — a directory on whatever
  filesystem you already have. Works
  anywhere, needs nothing, and the lab
  could prove it end to end. Its costs:
  instance copies and snapshots are full
  copies (slow, and they eat the space
  they save you from thinking about), and
  there are no quotas beyond the disk's
  own edge. The honest default for a
  first install on a machine you are
  still learning.
- **btrfs** — a Btrfs filesystem as the
  pool. Snapshots and copies become cheap
  copy-on-write references; quotas exist.
  Wants the pool's home to be a Btrfs
  filesystem (or a loop file on one,
  which init will offer to create).
- **zfs** — a ZFS pool. The heavyweight
  favourite for serious hosts: datasets
  per instance, instant snapshots,
  compression, the works — at the price of
  ZFS's own learning curve and appetite.
- **lvm** — an LVM volume group, with thin
  provisioning if you ask (`lvm` and its
  thin pool do the copy-on-write). The
  traditional enterprise answer.
- **ceph** and **cephfs** — storage spread
  across a Ceph cluster: the multi-server
  story, where the pool is a network and
  instances can outlive any one host.
  Real infrastructure, out of this
  guide's depth by design.
- **truenas** — pools backed by a TrueNAS
  system over its API, new enough that
  the lab met it only as a binary riding
  in the package (`truenas_incus_ctl` sat
  in the daemon's own folder). Named so
  you recognise it; not taught.

The pattern: **dir teaches, btrfs/zfs
serve**. Nothing forces a choice forever
— pools coexist, and instances move
between them.

## Custom volumes: disks with their own lives

An instance's root disk dies with the
instance. A **custom volume** is a disk
that belongs to a pool and outlives any
attachment:

```bash
incus storage volume create default photos
incus storage volume attach default photos workbench /photos
incus storage volume detach default photos workbench
```

Volumes hold the data you would grieve:
attach to one instance at a time (or
share carefully, read-only where offered),
snapshot them independently (`incus
storage volume snapshot ...` — the lab
captured the whole volume command family
in the storage help tree), and back them
up by export. The discipline they enable
is the point: *instances are cattle,
volumes are pets* — rebuild the instance
from an image, reattach the volume, and
the data never noticed.

## What the pool keeps, and the space truth

A pool's population is threefold: cached
**images** (chapter 8), **instances**
(roots and their snapshots), and
**custom volumes**. `incus storage info`
reports usage per pool, but the honest
space arithmetic depends on the driver:
on dir, a snapshot is a second copy of
everything; on btrfs/zfs, it is a receipt
for blocks not yet changed. When a disk
fills anyway, the suspects in order are
old snapshots, an image cache nobody
pruned, and a volume someone forgot —
chapter 16 keeps the checklist.
