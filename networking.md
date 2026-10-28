# Networking

*Documented-only chapter: the lab's
preseed deliberately created no networks
(its sandbox could not hand out bridges),
so the managed-network story is taught
from the project's documentation. The
verified pieces — the proxy device's
shape, the client's `port-forward`, the
daemon naming `nftables` as its firewall —
are called out where they appear.*

## The default: a bridge of one's own

A normal `incus admin init` offers to
create a managed bridge, and accepting is
the right first answer. What you get, in
the bridge conventionally named
`incusbr0`:

- a private IPv4 subnet (and IPv6, if you
  let it) that exists only on this host;
- the daemon handing out addresses by
  DHCP and answering DNS for instance
  names (so `workbench.incus` resolves
  from the host and from sibling
  instances);
- NAT at the edge (the daemon drives the
  host's nftables firewall — transcript
  t13's server report names it), so
  instances reach the internet but the
  internet cannot reach them, until you
  arrange otherwise below.

An instance joins through a `nic` device
naming the network — in the default
profile, `eth0` is already such a device,
which is why a launched container simply
*has* an address in `incus list`. It is
worth saying plainly what this network
is: a tiny LAN in software, with the host
as its router. Everything you know about
LANs applies; everything you fear about
them does too, and chapter 15 notes what
a hostile instance on the bridge can and
cannot attempt.

## More networks, shaped on purpose

```bash
incus network create backend ipv4.address=10.20.30.1/24 ipv4.nat=true
incus network list
incus network show incusbr0
```

Reasons a second managed network earns
its keep: a *backend* LAN whose instances
have no NAT and no way out (databases
talk, nothing phones home); a *staging*
copy of a production layout; separate
subnets per project so two experiments
cannot even see each other. Managed
networks can also be physical in various
costumes — attached to a host interface
(`bridge` onto a real LAN), `macvlan` and
`sriov` hand-offs that give instances
first-class seats on the real network,
`ovn` networks that span a cluster. The
ladder is documented; climb it when a
need is named, not before. An instance's
side of the story is all in its nic
device (chapter 9): which network, a
fixed `ipv4.address` if you assign one,
`limits.ingress`/`limits.egress` if the
pipe needs narrowing.

## Letting the outside in, on purpose

Three ways, in increasing order of
commitment:

1. **`incus port-forward`** (chapter 6) —
   your client, your machine, for as long
   as the command runs. For *looking at*
   things.
2. A **proxy device** on the instance —
   `listen=tcp:0.0.0.0:8080
   connect=tcp:127.0.0.1:80` — the daemon
   listens on a host address and relays
   to the instance, persistently, across
   reboots of both. This is the standing
   answer for "the web app in the
   container is the site". The device
   type is the verified shape from
   chapter 9; its traffic flows through
   the daemon's user-space proxy (or the
   kernel's, for NAT mode on containers),
   which is a thought worth having about
   performance before you serve a crowd
   through it.
3. A **network forward** — the bridge
   itself claims a public address and
   forwards its ports to instances,
   the router-port-forward pattern at the
   managed-network level, with
   `incus network forward` commands to
   match.

Each step trades invisibility for
reachability. The bridge's default —
outbound yes, inbound no — is the safe
side of that trade, and every opening
above should be a decision, not a
default.

## Names, and knowing where you stand

On the managed bridge, instance names
resolve: from the host, from other
instances, `<name>.incus` (the domain is
the network's `dns.domain`, incus by
default). Two checks carry most debugging:
`incus list` shows the addresses the
daemon *believes* instances hold, and
`incus network show incusbr0` shows the
network the daemon believes it runs —
leases included under its `state`. When
an instance has no address, the fault is
almost always one of: no nic device
wearing the network (check the expanded
config, chapter 9), the instance's own
DHCP declined or its firewall sulking
(console time, chapter 6), or the host
bridge never created because init was
told not to — the lab's own preseed is
the worked example of the third, and its
instances, had any been born, would have
been networkless hermits by design.
