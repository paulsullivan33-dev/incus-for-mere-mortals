# Limits

What Incus cannot do, and what it must
not be asked to do — including the limit
of this guide's own evidence, stated as
plainly as the others.

## The daemon is root in a trench coat

Everything convenient in this guide flows
from one uncomfortable fact: `incusd` runs
as root and can be *talked into* doing
anything root can do. Access to its unix
socket — root, or membership of the
`incus-admin` group — is therefore
**root-equivalent**, full stop. The
`incus` group exists as a restricted seat,
but the restriction is a courtesy, not a
wall; the project's own documentation is
clear that local access should be granted
like sudo, because it is sudo with more
steps. Remote access (chapter 13) is
guarded by mutual TLS and revocable
trust — real protection — and the
corresponding duty: a lost laptop with a
trusted client certificate is a lost
server, until the trust entry is removed.
Recent versions add finer-grained
authorization (external identity
providers, per-object permissions) for
installations that serve many people;
the homelab translation is unchanged:
**the client is a key; guard it like
one** — which is also why chapter 3
checked a checksum before running one.

## Containers are fences, not vaults

An unprivileged container is a serious
fence: its root is mapped to a nobody on
the host, its view is namespaced, its
resources are capped. It is still the
host's own kernel answering every system
call, and kernels have bugs; a container
escape is a category of security news,
not a hypothetical. A **privileged**
container (`security.privileged=true`)
removes the user mapping and should be
read as: *this instance is the host, for
security purposes*. Practical corollaries:

- Run other people's code in a VM (the
  stronger wall) or on a machine you can
  afford to lose — not in a privileged
  container beside your life.
- Never publish an image (chapter 8)
  without scrubbing it: SSH host keys,
  credentials, shell histories ride along
  in a frozen filesystem, gift-wrapped.
- An instance with a host directory
  attached (chapter 9's disk device) has
  exactly the access that path implies.
  Convenience mounts are decisions about
  trust, made in YAML.

## The host requirements are real

The lab's startup log is this chapter's
verified exhibit (transcript t12). No
`/dev/kvm`: virtual machines are simply
*not operational* — the daemon says so
itself. No visible cgroup controllers:
limits are configured, accepted, and
*ignored*, also by the daemon's own
admission. No subuid mapping for the
daemon's user: unprivileged containers
have no users to map. Add the documented
requirements — a current Linux kernel and
distro, storage you can dedicate, a
willingness to let the daemon drive your
firewall and bridges — and the shape is
clear: Incus is infrastructure, and it
behaves like infrastructure in places
that are not really hosts: nested
containers, rented slices with locked
kernels, Windows and macOS (clients only,
ever; there is no daemon port and the
project promises none).

## Shape limits of the tool itself

- **Guests are Linux-shaped.** Containers
  are Linux by construction. VMs run what
  QEMU runs, but the catalogue and the
  agent story are Linux and BSD; a
  Windows desktop is another tool's job
  (chapter 14).
- **A snapshot is not a backup** and a
  copy on the same pool is not either —
  chapter 12's sermon, repeated here
  because limits chapters are where
  sermons belong.
- **Stateful magic has edges.** Live
  migration and memory-state snapshots
  exist, bounded by kernel features,
  drivers and guest cooperation; the
  boring verbs (stop, copy, start) are
  the ones that always work.
- **One daemon's failure is total.** The
  database in the state directory is the
  kingdom; if it is lost, instances
  become orphaned directories. The
  project documents backup and recovery
  of the daemon itself; a guide this
  size points at that documentation
  rather than paraphrasing it badly.

## The legal and ethical edge

Chapter 1's rule — *your machines, or
explicit permission to administer them* —
has a shadow side worth naming in a tool
this capable: Incus makes it *easy* to
spin up a small internet of machines, and
ease is not permission. Running instances
on an employer's or school's hardware
"beyond" your role, hosting services for
others without their infrastructure
rules applying, filling a borrowed
machine with guests its owner never
agreed to: the tool will not stop you,
which is precisely why the rule is in
chapter 1 and not in the fine print of
this one. Within the line lies a decade
of legitimate, interesting work.

## This guide's own limit

Stated once, plainly, as chapter 5
promised: the lab verified the client
end to end and a real daemon through
initialisation and pool creation — and
never ran an instance, because its
sandbox's network would not fetch an
image and its kernel would not have run
a VM regardless. The instance-level
chapters rest on the tool's own help and
the project's documentation, are labelled
documented-only where they stand, and
should be re-checked against `incus
<command> --help` on your version, which
— chapter 4's lesson — is the
documentation that can never be out of
date for the program in your hand.
