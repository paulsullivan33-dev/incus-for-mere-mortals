# Troubleshooting

Symptom first, cause second, in the order
the lab and the documentation suggest you
check them. The first three entries are
the lab's own afternoons, verbatim;
the rest are the documented classics.

## "The incus daemon doesn't appear to be started"

*The first error every user meets —
chapter 4's home ground.*

1. **There is no daemon.** The client is
   installed, the server is not — or this
   is a client-only machine pointing at
   its own `local`. Check
   `incus version`: `Server version:
   unreachable` agrees (transcript t10).
2. **The daemon is installed but not
   running.** On a packaged install:
   `systemctl status incus` (the service
   may be named `incus.service`; some
   setups socket-activate, and the daemon
   starts on first command — if commands
   still fail, the service's own log,
   `journalctl -u incus`, is the witness).
3. **It is running — somewhere else.** The
   error names the socket it tried:
   `/var/lib/incus/unix.socket`. A daemon
   started with a different state
   directory (the lab's scratch daemon
   used one, via `INCUS_DIR`) listens
   there instead; match the client's
   expectation to the daemon's reality.
4. **It is running and you may not ask.**
   Socket permissions: you need root or
   `incus-admin` membership (log out and
   back in after joining the group), and
   chapter 15 says what membership means.

## The launch hangs, or the image never arrives

*The lab's chapter 5 story: the client
waited, silent, for a download that could
not happen (transcript t18).*

1. **The road to the image server.** The
   daemon fetches from
   `images.linuxcontainers.org`. Its log
   names failures plainly — the lab's
   read `TLS handshake timeout` for the
   daemon's own metadata fetch, the same
   failure the client met alone in
   transcript t07. Firewalls, proxies and
   captive networks live here; test the
   address from the *server*, not from
   your desk.
2. **A different remote entirely.** A
   mistyped remote or image name fails
   faster and ruder (the project cannot
   find what never existed). `incus image
   list images:` distinguishes "catalogue
   unreachable" from "image unknown".
3. **Patience calibration.** A first
   launch downloads hundreds of megabytes
   to a cold cache. Silence for a minute
   is a download; silence for fifteen is
   this section.

## "Failed to create storage pool ... Source path doesn't exist"

*Verbatim from the lab's preseed
(transcript t14).* A `dir` pool's source
folder must exist before initialisation;
the daemon adopts folders, it does not
create their parents. `mkdir` the path,
re-run init. The same family, for other
drivers: a ZFS pool name that already
exists as a dataset, an LVM group that
was never made — the daemon reports the
driver's complaint in its own words, and
the fix lives on the host, not in Incus.

## The daemon starts with warnings

*The lab collected a full set (transcript
t12); on real hardware most should be
absent, and each that remains is a
diagnosis, not noise:*

- `Instance type not operational ...
  KVM support is missing (no /dev/kvm)` —
  virtualisation off in firmware, absent
  on the CPU, or not passed into this
  (virtual) host. Containers are
  unaffected.
- `Couldn't find the CGroup ... controller,
  ... limits will be ignored` — the host's
  cgroup tree is not visible or not
  delegated (nested containers, some
  containerised hosts). Limits are being
  *accepted and not enforced*: know it
  before you rely on them.
- `Unable to parse system idmap / No map
  found for user` — no subuid/subgid range
  configured for the daemon's user in
  `/etc/subuid` and `/etc/subgid`.
  Packaged installs arrange this; fix the
  ranges (a wide, unshared span per the
  documentation) before unprivileged
  containers will start.
- `AppArmor support has been disabled
  because of lack of kernel support` —
  the host kernel cannot confine
  containers that way. A posture note,
  not a failure.

## The instance starts but has no network

1. **No nic, or the wrong network.** The
   lab's own preseed is the type specimen:
   a default profile with only a root
   disk makes networkless instances *by
   design*. Read `incus config show
   <name> --expanded` and look for the
   `eth0` device and its `network=`.
2. **The bridge was never created** —
   init was told "no network", or its
   creation failed. `incus network list`
   tells the truth.
3. **The guest declined its address** —
   DHCP off in the image, a firewall in
   the guest sulking. The console
   (chapter 6) is the door that works
   when the network doesn't.

## The instance misbehaves under limits you set

Before blaming the guest: confirm the
limits are *real* on this host (the
cgroup warning above is the counterexample
the lab verified), read them back with
`incus config show --expanded` — a profile
may be overriding the instance, or the
instance the profile, in the order
chapter 9 describes — and remember that
several limits apply at *start*: a running
instance may be honouring yesterday's
configuration until restarted.
