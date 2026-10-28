# Shaping Instances — Config, Devices and Profiles

*Documented-only chapter, except for its
one solid anchor: the lab created a real
default profile on a real daemon and read
it back (transcript t16). The keys and
devices below are the project's
documented vocabulary, not the lab's
invention.*

An instance fresh from an image is
generic: default CPU access, default
memory manners, one disk, one network
card. Shaping is how it becomes *yours*,
and Incus gives it a grammar with three
nouns: **config** (settings on the
instance), **devices** (things attached
to it), and **profiles** (both, bundled
and named, ready to reuse).

## Config: keys on the instance

```bash
incus config set workbench limits.memory 2GiB
incus config get workbench limits.memory
incus config show workbench
incus config edit workbench
```

Settings are key/value pairs. The keys
you will meet first:

- `limits.cpu` — how many CPUs (a count,
  or a set like `0-3`), and
  `limits.cpu.allowance` for shares;
- `limits.memory` — a ceiling like
  `2GiB`, or a percentage of the host;
  `limits.memory.enforce` decides whether
  the ceiling is a wall or a strong hint;
- `limits.processes` — a cap on process
  count, the fork-bomb fence;
- `limits.disk.priority` and the
  `limits.network.*` family — shares of
  the contended resources;
- `boot.autostart` — should this instance
  rise with the daemon (and
  `boot.autostart.priority` / `.delay`
  to order a household);
- `security.privileged` — the loud one.
  Default `false`: the container's users
  are mapped to unprivileged ranges on
  the host (chapter 3's idmap). Setting
  it `true` makes container root into
  host root's neighbour and removes the
  mapping protection — occasionally
  necessary for exotic workloads, never a
  casual switch, and chapter 15 explains
  why in full;
- `environment.<NAME>` — environment
  variables handed to the instance's
  init;
- `snapshots.schedule`, `snapshots.expiry`
  and friends — chapter 12's automation,
  which lives here;
- `user.<anything>` — your own keys, for
  your own notes and tooling, ignored by
  Incus itself.

Changes to a running instance apply as
the key allows — limits generally do,
live; a changed root-disk size wants a
restart's cooperation. `incus launch -c
limits.memory=4GiB` (chapter 5's help
transcript shows the flag) sets keys at
birth; `config set` adjusts them after.

## Devices: things attached

```bash
incus config device add workbench share disk source=/srv/data path=/data
incus config device add workbench web proxy listen=tcp:0.0.0.0:8080 connect=tcp:127.0.0.1:80
incus config device remove workbench share
```

A **device** is an attachment with a type
and options. The working set:

- **disk** — a directory (containers: a
  host path appears inside; VMs: shared
  over a filesystem protocol by the
  agent) or the instance's `root` disk
  itself, whose `size` and `pool` live
  here;
- **nic** — a network card, joined to a
  managed network (`network=`) or a host
  bridge, with optional fixed addresses
  and rate limits;
- **proxy** — a standing port forward:
  the host listens, the instance answers.
  Chapter 11 contrasts it with the
  client's temporary `port-forward`;
- **gpu**, **usb**, **unix-char** /
  **unix-block** — passing real hardware
  or device nodes through, the way a
  media box gets its tuner;
- **tpm**, **pci** — a virtual TPM for a
  VM, a whole PCI device surrendered to
  one.

Devices are where instances touch the
host's world, which is why they are
spelled out in configuration rather than
discovered by poking around — and why
`config show --expanded` (which prints
the instance's own settings *plus* what
its profiles contribute) is the view to
trust when something behaves as if a
setting came from nowhere.

## Profiles: settings with a name

A **profile** is config and devices,
bundled under a name and applied to
instances by list:

```bash
incus profile create small
incus profile set small limits.cpu 2
incus profile assign workbench default,small
incus launch images:debian/13 tiny --profile default --profile small
```

Every instance wears the `default`
profile from birth. On a normally
initialised server, `default` carries the
root disk and an `eth0` on the bridge;
the lab's scratch init made a plainer one
(transcript `incus-lab/transcripts/t16-profile-show.txt`
— the real profile of chapter 5's
preseed: a root disk on the `default`
pool, nothing else). Profiles stack in
order, later ones overriding earlier ones
key by key, and editing a profile
re-shapes every instance wearing it on
their next start — a fleet-wide change
from one edit, which is either the appeal
or the hazard, depending on the day.

The pattern that emerges in real use: one
profile per *role* (`web`, `database`,
`buildbox`), a `default` kept boring, and
sizes kept as profiles too, so "give it
the medium shape" is `--profile medium`
rather than a paragraph of flags nobody
remembers the same way twice.
