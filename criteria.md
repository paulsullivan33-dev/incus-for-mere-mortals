# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's daemon ran in a scratch folder, kept every byte of
   its state there, and administered nothing but itself.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 15 names what the tool
   cannot do: it cannot make access to its socket anything
   less than root-equivalent, cannot make a privileged
   container a security boundary, cannot run virtual machines
   on a host without KVM or honour limits that a hidden cgroup
   tree will not enforce, cannot run its daemon anywhere but
   Linux, and cannot call a snapshot a backup. It also names
   this guide's own ceiling: the client was verified end to
   end and a real daemon was run through initialisation and
   storage-pool creation, but no instance ever ran in the lab,
   so the instance-level chapters are documented-only.
8. **The ethics rule is in the text, early.** Chapter 1 and the
   README state the rule plainly — *your machines, or machines
   you have explicit permission to administer* — before any
   procedure depends on it, and later chapters point back to
   it where the tooling would make crossing the line easy:
   publishing an image (chapter 8), attaching a host directory
   to an instance (chapter 9), and trusting a new remote
   client (chapter 13). The verification route belongs to the
   same honesty: the daemon ran only from the project's
   official packages, unpacked into a scratch folder with its
   state confined there, and was shut down and deleted
   afterwards.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a 2-CPU, 7.7 GB Ubuntu 24.04 sandbox.
Two halves of Incus were exercised, from two different official
sources, and the guide keeps them distinct wherever it matters:
the **client** is the standalone binary of Incus **7.0.2**
(the 7.0 LTS series) from the project's GitHub releases, its
SHA-256 verified against the release's `checksums.txt` before
first run; the **daemon** is **incusd 7.5.1** from the project's
official stable packages for Ubuntu 24.04 (the Zabbly
repository the documentation recommends), downloaded as .deb
packages and **unpacked into a scratch folder, never
installed**. The daemon needed one helper those packages
normally bring along — `unsquashfs`, whose absence it reported
verbatim — supplied the same way from Ubuntu's squashfs-tools
package. All state (client config via `INCUS_CONF`, daemon
state via `INCUS_DIR`) was confined to scratch, and the lab
ended with `incus admin shutdown` and a verified clean slate:
no daemon processes, no `/var/lib/incus`, no new interfaces or
mounts. Transcripts live in `incus-lab/transcripts/`.

Run for real, with transcripts as evidence:

- Client version and the full command tree; help for `launch`,
  `admin`, `storage` and `webui` (t00, t01, t04, t08, t11,
  t09).
- The no-server error in its wordings: the polite socket-path
  message from `list` and `info`, `Server version:
  unreachable` from `incus version`, and the raw unix-socket
  dial error behind a filtered image list (t02, t05, t10,
  t07b).
- A fresh client's `remote list` and the `config.yml` it
  writes on first use; the built-in `shell` alias (t03, t06).
- The daemon starting from the unpacked packages — after the
  verbatim `unsquashfs` refusal — and logging its environment
  audit: no `/dev/kvm` (virtual machines declared not
  operational), missing cgroup controllers (limits to be
  ignored), an unparseable system idmap, AppArmor disabled
  (t12).
- The running daemon answering: `incus info` (LXC driver
  7.0.0, 562 API extensions, nftables firewall), server
  config, storage and profile queries (t13, t17, t15, t16).
- `incus admin init --preseed` twice: the verbatim failure
  when the dir pool's source folder does not exist, then
  success after creating it — a `dir` storage pool created
  and worn by a real default profile (t14, t15).

Documented only — described from the tool's own help and the
project's documentation, not run, and the text does not claim
otherwise:

- Everything an instance does once it exists: launching to
  completion, start/stop/pause, `exec` and `shell` sessions,
  file push/pull, consoles. The lab's one launch attempt is
  itself a transcript (t18): the client waited silently for
  75 seconds while the daemon tried to fetch the image, and
  was stopped; the sandbox's network ends connections to
  `images.linuxcontainers.org` in a TLS handshake timeout —
  the same wall the client met alone when asking the
  catalogue for its index (t07), and the daemon met fetching
  its instance-types metadata (t12).
- Virtual machines beyond the daemon's own declaration that
  they were not operational without KVM (t12).
- The full image catalogue, publishing images, and image
  export/import.
- Managed bridge networking (the lab's preseed deliberately
  created no network), proxy devices in traffic, and
  additional networks.
- Snapshots, copies and moves between servers, exports as
  backups, scheduled snapshots.
- Adding HTTPS remotes, trust tokens, the web interface in a
  browser, projects in use, and clustering.
- The packaged install routes (Zabbly repository setup, other
  distributions' packages) — the lab consumed the packages'
  contents directly, without the package manager's service,
  group and subuid setup, and chapter 3 says which parts of a
  real install that setup provides.
- The comparison tools in chapter 14 are characterised, not
  benchmarked.

Fixtures are this guide's own and deliberately generic: one
scratch folder, one throwaway daemon with an empty table, one
`dir` pool with nothing in it, and instance names like
`workbench` that belong to no one. No example is modelled on
any real person, network, service or organisation, and the
daemon administered nothing but the sandbox it briefly lived
in.
