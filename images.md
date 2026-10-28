# Images

*Mostly documented-only: the lab's client
reached for the catalogue and was stopped
by its sandbox's network (that failure is
verified, transcript t07), so the
catalogue's contents are described from
the project's documentation, while
everything about how images behave —
remotes, caching, aliases — follows the
tool's verified design and help.*

## Where instances come from

An image is a beginning: a complete root
filesystem, built by someone, frozen, and
signed. Launching copies the image into a
pool and lets the copy diverge; the image
itself is never dirtied by the instances
born from it, and twenty instances from
one image still cost the image's storage
once, plus their own changes (on the
copy-on-write pool drivers of chapter 10,
even the copies start as references).

## The catalogue: `images:`

The project's image server —
`images.linuxcontainers.org`, the remote
your client was born knowing (chapter 4)
— is built by the Linux Containers
project from the distributions' own
published material. Its habits:

- Names are `distribution/release`, often
  with a variant: `images:debian/13`,
  `images:ubuntu/24.04`, `images:alpine/3.21`,
  and for many, a cloud variant with the
  trimmings servers expect.
- Most entries exist as **both** a
  container and a virtual-machine image;
  the `--vm` flag picks the other twin.
- Architectures are in the name's shadow:
  ask on an ARM host and you get ARM
  images.
- `incus image list images:` browses it
  (the command the lab's sandbox refused
  to let finish), and `incus image info
  images:debian/13` reads one entry's
  papers: size, fingerprint, the dates it
  was built and expires.

The catalogue is a convenience with a
supply chain — images are signed by the
project and verified by your daemon on
download, which is why chapter 3 made a
small ceremony of checksums: this guide
trusts artifacts it can verify.

## The cache on your server

A daemon downloads an image the first
time an instance asks for it, keeps it in
its pool-side cache, and launches later
instances from the copy. Consequences
worth internalising:

- The first launch of a new image is the
  slow one — a few hundred megabytes down
  the wire. The tenth is local.
- `incus image list` (no remote) shows
  what your server keeps; `incus image
  delete <fingerprint>` evicts. Unused
  cached images expire on their own after
  a period the server configuration sets.
- An image in the cache is identified by
  its **fingerprint** — a hash of its
  content — and may wear **aliases**
  (`debian/13` and friends) that move as
  newer builds land. An alias is a
  nickname for "current"; a fingerprint
  is a promise about exact bytes. Builds
  you must reproduce want fingerprints.

## Your own images: publish, export, import

The catalogue is not a gate. Three verbs
make images of yours:

- `incus publish workbench --alias
  myimage` freezes a stopped instance
  into an image on the same server —
  configure once, launch forever. The
  project asks, sensibly, that you remove
  your secrets and host keys first;
  chapter 15 repeats the warning.
- `incus image export` / `incus image
  import` move an image between servers
  as a file (a tarball plus its metadata)
  — the sneakernet route, and the way to
  carry an image to a server whose
  network you do not trust with catalogue
  access.
- `incus copy` on an *instance* between
  remotes (chapter 12) often beats image
  juggling when the destination is one
  machine, not a fleet.

Building images from nothing — a
distribution assembled to your recipe —
is a different tool's job: the project's
**distrobuilder** takes a YAML definition
and produces Incus (and LXD) images. It
is named here so you know it exists, and
left to its own documentation, as the
README promised.

## A note on trust

An image is someone else's root
filesystem with your work about to live
in it. The catalogue's images are built
in the open from the distributions'
official material and signed; random
images from the internet deserve the
suspicion you would give a stranger's
operating system — because that is
exactly what one is. When in doubt,
launch it with no network worth the name,
look inside, and publish your cleaned
descendant as the image you actually use.
