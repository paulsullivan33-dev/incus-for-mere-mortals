# What Incus Is

You know the feeling of wanting a second
computer for ten minutes: to try an
upgrade before it touches the real
machine, to run a stranger's install
script somewhere it can do no harm, to
hand a student or a script a whole Linux
box and say *break this one, it doesn't
matter*. **Incus is the program that
makes that second computer from a
command.** You name an operating system
and a name of your own —

```bash
incus launch images:debian/13 workbench
```

— and a moment later there is a machine
called `workbench`, running, with its own
files, its own network address, its own
users and services, that you can log into,
stop, snapshot, copy, and delete like a
row in a table. (That command is chapter
5's, and this guide's lab never got to
run it — the reason is a good story, and
it is told honestly in chapters 3 and 5.)

The things Incus makes are called
**instances**, and they come in two kinds:

- **System containers.** A complete Linux
  system — init, services, logins, the
  works — running on the host's own
  kernel, fenced off by the kernel's
  namespaces and control groups. This is
  the technology of **LXC**, the Linux
  Containers project that also maintains
  Incus. A system container is *not* the
  one-program container of Docker fame
  (chapter 14 draws that line carefully);
  it is a machine in every way that
  matters day to day, minus a kernel of
  its own.
- **Virtual machines.** A simulated
  computer with its own kernel, run by
  **QEMU** on the host's **KVM**
  virtualisation support. Heavier, slower
  to start, and isolated by hardware
  rather than by convention.

The Incus move — the reason for this
guide — is that both kinds are driven by
the *same* commands, the same
configuration, the same snapshots. A flag
(`--vm`) chooses the kind at birth; after
that, `list`, `exec`, `snapshot`, `copy`
and friends do not care which they are
talking to.

## Where it came from

Incus is the community continuation of
**LXD**. LXD began at Canonical in 2014
as a friendlier manager for LXC. In 2023
Canonical took LXD's development in-house
under its own licence terms, and the
project's outside contributors — gathered
around the Linux Containers project,
where the code had always lived — forked
the last community release and renamed
it. That fork is Incus: same lineage,
same command shape (the `lxc` command
became `incus`, the daemon `incusd`), and
an unbroken line of releases since,
currently the **7.0 long-term-support
series** and the **7.5 feature series**.
There is even an official migration tool,
`lxd-to-incus`, in the project's releases
(chapter 14). If you already know LXD,
you know most of this guide; if you meet
advice written for LXD, it usually applies
with the names changed.

## The rule, stated before everything else

Incus's daemon runs as **root**, and it
has to: creating containers and virtual
machines *is* rearranging the machine at
the lowest level. Anyone who can give
the daemon orders can, with a little
ingenuity, own the host outright — this
is not a bug in the tool, it is what the
tool *is*. So this guide has one rule,
and it is not fine print:

> **Run and administer instances only on
> machines that are yours — machines you
> own, or where the owner has given you
> explicit permission to administer them.**

Your own homelab box: yes. A spare
machine at work that your team has handed
you for exactly this: yes. A friend's
server you "just quickly" put a container
on, a school or employer machine you were
given an ordinary login on, a cloud
instance someone else is paying for: no.
The daemon's power also means the *client*
is a skeleton key: whoever holds an
account that can talk to it holds the
machine (chapter 15 returns to this, and
chapter 13 explains the trust ceremony
that guards remote access). This guide's
lab went further than most readers will
need to: the daemon it ran was unpacked
into a scratch folder, kept every byte of
its state there, and was shut down and
deleted when the evidence was gathered.

## A first taste of the shape

Two programs do the work (chapter 2 has
the full cast). The **daemon**, `incusd`,
runs on the machine that will host the
instances. The **client**, `incus`, runs
anywhere — the same machine, your laptop,
a different continent — and sends the
daemon orders. Before any server exists,
the client is already informative:

```text
$ ./incus --version
7.0.2
```

(transcript `incus-lab/transcripts/t00-version.txt`,
from the lab's checksum-verified copy of
the official release binary). And it is
honest about the missing half:

```text
$ ./incus list
Error: The incus daemon doesn't appear to be started (socket path: /var/lib/incus/unix.socket)
```

(transcript t02.) Chapter 4 is devoted to
that moment, because every reader will
meet it, and because a client that works
with no server at all turns out to be
half the product.

## Who it is for

The homelabber who wants a throwaway
Linux box per experiment instead of one
precious server. The developer who needs
*that* distribution, *that* version, for
an afternoon. The teacher who wants a
roomful of identical machines that reset
between classes. The cautious, who would
like to rehearse the scary upgrade on a
copy of the machine first. The common
thread is one person, their own hardware,
and work that deserves a whole computer
but not a whole purchase.

## What it is not

Incus is not Docker: its containers are
systems, not application packages, and
chapter 14 explains when each is right.
It is not a security boundary against the
host's administrator, and a privileged
container is barely a boundary at all —
chapter 15 is blunt about this. It is not
a way to run Windows or macOS guests in
any practical sense: its world is Linux
(and some BSD) guests on a Linux host. It
does not run on a Windows or macOS host —
the *client* happily does, driving a
server elsewhere. And it is not light
magic: a virtual machine still costs the
RAM you give it, and a storage pool is
real disk, spent for real.
