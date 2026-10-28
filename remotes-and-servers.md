# Remotes and Other Servers

The client's configuration file (chapter
4) is a list of **remotes**, and so far
only two names have been in it. This
chapter adds the rest of the world: your
own servers, on other machines, driven by
the same client, the same commands, the
same API the lab verified — 562 extensions
of it — on its scratch daemon. The
mechanics of adding and trusting a remote
are documented here; the lab had exactly
one daemon and it lived for an afternoon,
so multi-server life is described, and
labelled as such in `criteria.md`.

## Adding a server

A daemon serves the network only when
told: on the server,

```bash
incus config set core.https_address :8443
```

opens its HTTPS door (8443 is its
customary port). Then trust must cross
between you and it, and Incus's ceremony
is deliberately physical the first time:

1. On the server, mint a **token**:
   `incus config trust add --name laptop`
   prints a one-time secret.
2. On the client, `incus remote add
   bigserver https://bigserver.example:8443`
   and give the token when asked. (Without
   a token, the client instead shows you
   the server's certificate fingerprint to
   confirm out loud — the SSH
   first-connection ritual, and to be
   taken as seriously.)

After that, the client's certificate is
known to the server, the server's to the
client, and `bigserver:` becomes a prefix
like `images:`: `incus list bigserver:`,
`incus launch bigserver:images:debian/13
web`, `incus copy workbench
bigserver:workbench`. `incus remote
switch bigserver` changes the default for
a sitting; `incus remote list` keeps the
census (the table shape is chapter 4's
verified one, with `tls` where `file
access` stood). A token is spent once and
a trust entry can be revoked on the
server (`incus config trust remove`) —
the pair of facts to remember when a
laptop goes missing, which is chapter
15's concern.

## Projects: compartments on one server

A **project** partitions a daemon:
instances, images, networks and profiles
kept per project, switched with `incus
project switch books` or per command with
`--project`. The uses are tidiness with
teeth — a project per course, per client,
per experiment with its own resource
quotas where configured — and the honest
limit is in chapter 15: projects organise
the trusted, they do not wall off the
untrusted, because the daemon's socket
still commands all of them.

## The web interface

`incus webui bigserver:` (the command's
own help is transcript t09) opens the
daemon's browser front-end — instances,
images, networks and profiles as pages
instead of tables, served by the daemon
itself over the same HTTPS and the same
trust. Whether a given server offers it
depends on how it was packaged (the UI is
a separate component in the project's
packages). Treat it as a convenience for
browsing and the occasional button; the
command line remains the complete tool,
and everything the UI can do, it does
through the same API — nothing to learn
twice.

## When servers multiply: clustering, briefly

Several daemons can join into a
**cluster** (`incus cluster`): one API,
instances placed across members, networks
and Ceph storage spanning the group,
members evacuable for maintenance. It is
the project's answer to "my one host
became five", and it is *pointed at, not
taught*, as the README promised — the
concepts above (remotes, trust, projects)
are its alphabet, and the project's
documentation walks the rest. The same
restraint applies to **IncusOS**
(chapter 3): a dedicated appliance OS
whose whole job is to be an Incus host,
managed through the `admin os` commands
the lab saw in the command tree — real,
current, and a chapter of somebody
else's guide.
