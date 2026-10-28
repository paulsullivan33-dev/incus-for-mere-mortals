# Snapshots, Copies and Backups

*Documented-only chapter: no instance ran
in the lab, so no snapshot was taken.
Everything here follows the project's
documentation and the verified command
tree; the distinctions the chapter insists
on — snapshot, copy, backup — are the
parts readers most often blur, and the
blurring is where data is lost.*

## Snapshots: freezing a moment

```bash
incus snapshot create workbench before-upgrade
incus snapshot list workbench
incus snapshot restore workbench before-upgrade
incus snapshot delete workbench before-upgrade
```

A **snapshot** is the instance as it was:
its disk state, named, kept inside the
same pool, one `restore` away from being
the present again. Restore is total and
unapologetic — everything since the
snapshot is discarded — which is the
point and the danger in a single fact.
Snapshots can be **scheduled** rather
than remembered (the `snapshots.schedule`
and `snapshots.expiry` instance keys of
chapter 9: daily, keep a week, and the
chore disappears), and a snapshot of a
*running* container can include its
memory state (`--stateful`, where the
host supports it) — a pause button with a
rewind, as against the disk-only
photograph a default snapshot takes.

Snapshots are cheap exactly as far as
your pool driver makes them (chapter 10):
a reference on ZFS, a full copy on dir.
Cheap or not, hold this firmly: **a
snapshot is not a backup.** It lives in
the same pool, on the same disks, under
the same daemon as the thing it protects.
It defends against bad upgrades and
regretted commands — the disasters of
*judgement*. Against the disasters of
*hardware* you need the third section of
this chapter.

## Copies: the instance, again

```bash
incus copy workbench workbench2
incus copy bigserver:workbench workbench --refresh
incus move workbench oldbench
```

`copy` clones an instance — same server,
or across remotes (chapter 13), where it
becomes the migration verb: the source
keeps running (stop it first if the copy
must be perfectly consistent), the target
receives disk, snapshots and config, and
`--refresh` makes a repeated copy send
only the difference. `move` is copy plus
delete: the relocation. Together with
snapshots they make the chapter 5 promise
practical — an instance is a row in a
table, and rows can be duplicated,
renamed (`incus rename`), shelved and
shipped.

## Export: the backup that leaves the building

```bash
incus export workbench workbench-backup.tar.gz
incus import workbench-backup.tar.gz
```

`export` writes an instance — disk,
configuration, snapshots if asked — to a
single archive file, anywhere you like:
another disk, another machine, the backup
shelf. `import` resurrects it, on the
same server or a different one, possibly
under a new name. This is the backup of
the chapter's title: self-contained,
daemon-independent (a tarball plus
metadata, readable by future Incus even
if the pool is long gone), and only as
fresh as the last time you ran it — a
chore to schedule, not a spell to cast
once. Storage volumes export the same way
(chapter 10), and a server's *whole*
estate wants its state directory and
database backed up by the operating
system's own tools, a disaster-recovery
topic the project's documentation covers
and this guide deliberately does not
half-cover.

## The discipline that ties it together

Three sentences to keep. Snapshots are
for *before*: take one before every
change you cannot undo, and let expiry
prune them. Copies are for *experiments
and moves*: never rehearse on the only
instance of anything. Exports are for
*loss*: if losing the host tonight would
lose the work, the export cadence is the
measure of how much work you are willing
to redo. A reader who runs all three can
treat every other chapter of this guide
as a playground — which, on machines that
are yours (chapter 1), is exactly what
they are for.
