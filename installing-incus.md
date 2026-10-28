# Installing Incus

"Installing Incus" means two different
things, and keeping them apart saves the
classic first-day confusion:

- **A server** needs the **daemon**
  (`incusd`) plus its entourage — LXC,
  QEMU for virtual machines, storage and
  network plumbing. That is an operating
  system package job, on a Linux machine,
  with root. This is the install that can
  host instances.
- **A client** is one standalone program.
  It needs no installation in any serious
  sense — download, make executable, run —
  and it works on Linux, macOS, Windows
  and FreeBSD, because it hosts nothing;
  it only talks.

This guide's lab verified the client half
with its own hands and exercised the
daemon half from the project's real
packages; the tidy package-manager
installs below are the project's
documented routes, labelled as such.

## The client, from the project's releases

The project's GitHub releases
(`github.com/lxc/incus/releases`) publish
ready-made client binaries per platform
(`bin.linux.incus.x86_64`, a macOS `.pkg`,
a Windows `.msi`, FreeBSD binaries) next
to a `checksums.txt`. The lab fetched the
Linux one for **Incus 7.0.2** — the release
GitHub itself flags as latest, from the
7.0 long-term-support series — and checked
it before running anything:

```text
$ sha256sum incus
b007fbfc29695e03bc3e66bbfb5fa3132bb791daa765e2385ac8f7cd1b9eb1d8  incus
$ ./incus --version
7.0.2
```

The checksum matched the release's own
`checksums.txt` exactly (transcript
`incus-lab/transcripts/t00-version.txt`
keeps the version half). Do that check
every time: this binary is a skeleton key
to every server you will ever point it
at, and "it downloaded fine" is not
verification. On a machine where the
client will live, put it somewhere on
your path (`/usr/local/bin` is the usual
place) — and note what you did *not* need:
no installer, no runtime, no daemon, no
root.

Check: `incus --version` prints the bare
number. `incus version` is more talkative
and previews chapter 4:

```text
$ ./incus version
Client version: 7.0.2
Server version: unreachable
```

(transcript t10.) Two halves, two version
lines; the second will say *unreachable*
until a daemon exists to answer.

## The server, from packages

A daemon is installed by the operating
system, because it *is* operating-system
infrastructure. The documented routes:

- **Debian and Ubuntu:** the project
  recommends the **Zabbly** repository,
  which packages Incus (and its matching
  LXC) for current Debian and Ubuntu
  releases, in a *stable* flavour tracking
  the feature series and an *LTS* flavour
  tracking the long-term-support series.
  Adding the repository and installing the
  `incus` package brings the daemon, the
  client, and the supporting cast.
- **Other distributions:** Fedora (via a
  COPR repository), Arch, Gentoo, NixOS
  and others carry Incus in their own
  packaging; the project's installation
  document is the authority for the list
  and the commands.
- **IncusOS**, the project's own appliance
  operating system — a whole machine
  dedicated to being an Incus host, which
  the client's `admin os` commands manage
  (the lab saw them in the command tree,
  transcript t08) — is installed as an OS,
  not as a package.

What the package route sets up for you,
beyond binaries: a system service for the
daemon, an `incus-admin` group whose
members may give it orders (and an
`incus` group with a deliberately poorer
seat — chapter 15 explains why the
distinction is nearly theoretical), the
**subuid/subgid** ranges that let
unprivileged containers map their users
(those ranges are quiet, essential, and
the cause of one of the lab's startup
complaints below), and bridge plumbing
permissions. This entourage is exactly why
there is no standalone daemon download:
the GitHub releases do not even ship one.

## What the lab did instead, and why it counts

The lab did not install those packages —
its sandbox is not a machine it may
rearrange — so it did the next most
honest thing: it downloaded the real
packages (daemon 7.5.1, stable flavour,
built for this Ubuntu) and **unpacked**
them into a scratch folder, the way you
might unpack a suitcase without moving
in. Two findings fell out that a tidy
install would have hidden:

First, the daemon audits its helpers
before it serves. Started bare, it stopped
immediately:

```text
Error: exec: "unsquashfs": executable file not found in $PATH
```

`unsquashfs` (from the `squashfs-tools`
package) is how the daemon opens the
squashfs images containers arrive as; a
packaged install simply brings it along.
The lab unpacked that package too, and
the daemon started.

Second, a started daemon is not a
*capable* daemon — it reports the gaps
itself. From the lab's startup log
(transcript t12), verbatim:

```text
level=warning msg="Instance type not operational" driver=qemu err="KVM support is missing (no /dev/kvm)" type=virtual-machine
level=warning msg=" - Couldn't find the CGroup memory controller, memory limits will be ignored"
level=error msg="Unable to parse system idmap" err="No map found for user"
```

Read those as a pre-flight checklist the
daemon performs on any host: *Can I run
virtual machines? Can I enforce limits?
Can I map users for unprivileged
containers?* On a normal, packaged install
on real hardware, all three pass quietly.
In this sandbox all three failed, for
reasons belonging to the sandbox, not to
Incus — a distinction the guide will keep
making, because a reader on real hardware
should expect better and a reader in a
nested container somewhere should expect
exactly this.

## After installing: not yet initialised

Whether packaged or unpacked, a fresh
daemon has no storage pool, no network,
and nothing to launch into. It must be
**initialised** — interactively, with
`incus admin init`, or from a written
configuration (a **preseed**) — which is
chapter 5's opening move, against the
lab's real daemon. Installation also
decides nothing about *access*: the
daemon serves its socket to root and the
`incus-admin` group, and serves the
network only once you tell it to (chapter
13). Installed, initialised, and reachable
are three separate states; this guide
keeps them separate on purpose.
