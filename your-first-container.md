# Your First Container

**A note before the chapter begins.**
This is the chapter where instances start
running, and it is the chapter this
guide's lab could only begin. The daemon
initialisation in the first half below is
real — run, answered, and kept in
transcripts. The launch in the second
half failed in the lab for a network
reason the transcript shows plainly, and
everything after it (the running
container, the verbs at work) is taught
from the tool's own help and the
project's documentation, and marked as
documented-only here and in
`criteria.md`. The guide would rather
hand you an honest map than a forged
photograph.

## Initialising the daemon

A fresh daemon knows nothing: no storage
pool, no network, no default profile
worth the name. `incus admin init` sets
that up. Run bare, it interviews you —
storage backend and size, a bridge
network or none, whether to serve the
network — and sensible answers are the
defaults. Run with `--preseed`, it reads
the whole conversation from a YAML file
instead, which is what you want the
second time and what the lab used, so
that its choices are on record.

The lab's preseed was deliberately
minimal: a storage pool on the plain
**dir** driver (chapter 10 explains the
drivers), rooted in the scratch folder,
an empty list of networks — the sandbox
was in no position to hand out bridges —
and a default profile carrying just a
root disk on that pool:

```yaml
storage_pools:
- config:
    source: /path/to/scratch/pool-data
  driver: dir
  name: default
profiles:
- devices:
    root:
      path: /
      pool: default
      type: disk
  name: default
```

(Condensed from the lab's file; a normal
init also gives the default profile an
`eth0` device on the bridge network.)
The first attempt failed, usefully
(transcript `incus-lab/transcripts/t14-admin-init.txt`):

```text
$ incus admin init --preseed < preseed.yaml
Error: Failed to create storage pool "default": Source path '/home/hatch/workspace/build-check/incus-scratch/pool-data' doesn't exist
```

A `dir` pool's source folder must exist
before the daemon will adopt it — the
daemon creates pools' *contents*, not
their homes. One `mkdir` later, the same
command exited silently with success, and
the pool stood in the list, already worn
by the default profile (transcript t15):

```text
$ ./incus storage list
┌─────────┬────────┬─────────────┬─────────┬─────────┐
│  NAME   │ DRIVER │ DESCRIPTION │ USED BY │  STATE  │
├─────────┼────────┼─────────────┼─────────┼─────────┤
│ default │ dir    │             │ 1       │ CREATED │
└─────────┴────────┴─────────────┴─────────┴─────────┘
```

Check, on your own machine: `incus
storage list` shows your pool `CREATED`,
`incus profile show default` shows a root
disk pointing at it (and normally an
`eth0` on your bridge), and `incus list`
prints an empty table — a tidy workshop
with no work in it yet. The lab reached
exactly this point, and its final list is
transcript t19.

## Launching

Here is the whole trick, one line
(documented — the lab's attempt follows):

```bash
incus launch images:debian/13 workbench
```

`launch` is `create` plus `start`: the
daemon fetches the image from the
`images:` catalogue if its cache lacks
it, copies it into the pool as a new
instance named `workbench`, applies the
default profile and any flags, and starts
it. A sibling command, `incus create`,
stops after the copying so you can adjust
a not-yet-running instance; `incus init`
is `launch`'s other name (the help page
in transcript t04 keeps both, with the
project's own examples, including
variants with `-c limits.memory=4GiB`
and instance-type shortcuts borrowed
from cloud sizing, `-t aws:t2.micro`).

The lab ran the Debian version of this
command against its initialised daemon.
The client printed nothing and waited;
after 75 seconds the lab stopped it
(transcript t18). The daemon's side of
the story is in its log, the same failure
the client met in chapter 4: the road to
`images.linuxcontainers.org` ends in a
TLS handshake timeout in that sandbox, so
no image ever arrived and no instance was
ever born. On your network the same
command's evidence of success is
different and plain: a progress report of
the download, a line like `Creating
workbench` / `Starting workbench`, and
then this chapter's real check —

```bash
incus list
```

— showing `workbench`, `RUNNING`, with an
IPv4 address from your bridge. If your
first launch instead fails, it will fail
where the lab's did or one step later,
and chapter 16 takes the failures in
order.

## The everyday verbs

With an instance running — this is the
documented half of the chapter — the
daily vocabulary is small:

- `incus info workbench` — the instance's
  report card: state, addresses, resource
  use, snapshots. (Against the *server*,
  bare `incus info` is the daemon's own
  report, the one transcripts t05 and t13
  show failing and then succeeding.)
- `incus stop workbench` / `incus start
  workbench` / `incus restart workbench` —
  ask politely; add `--force` when polite
  has failed and you mean it.
- `incus pause workbench` / `incus resume
  workbench` — freeze a container in
  place, memory and all, and thaw it
  later (containers only; a VM is
  stopped, not frozen).
- `incus delete workbench` — destroy it.
  A running instance refuses; stop it
  first or `delete --force`. Deletion is
  the point of no return in a tool whose
  other verbs are so reversible, and
  chapter 12's snapshots are the
  counterweight.
- `incus list` itself grows teeth worth
  knowing early: `incus list -c ns4`
  chooses columns, `incus list
  status=RUNNING` filters, and the list
  is the same table the lab kept meeting,
  finally with rows in it.

Chapter 6 steps inside the instance;
chapter 9 covers the flags and settings
that shape one at birth (`-c`, `-p`, `-d`,
`-s` for pool, `-t` for instance type);
and the whole life cycle — snapshot,
copy, move, publish, export — waits in
chapters 8 and 12.
