# First Contact With a Server

Every Incus user meets the same first
moment: a client, no server, and an error
message. This chapter makes that moment
boring on purpose — everything in it was
run by the lab against a client with
nothing to talk to — because the client
with no server is not a dead end. It is
half the product, working as designed.

## The client is born knowing two remotes

Ask a brand-new client what servers it
knows. The lab pointed its client at an
empty configuration folder (the
`INCUS_CONF` environment variable chooses
where the client keeps its affairs) and
asked:

```text
$ ./incus remote list
┌─────────────────┬────────────────────────────────────┬───────────────┬─────────────┬────────┬────────┬────────┐
│      NAME       │                URL                 │   PROTOCOL    │  AUTH TYPE  │ PUBLIC │ STATIC │ GLOBAL │
├─────────────────┼────────────────────────────────────┼───────────────┼─────────────┼────────┼────────┼────────┤
│ images          │ https://images.linuxcontainers.org │ simplestreams │ none        │ YES    │ NO     │ NO     │
├─────────────────┼────────────────────────────────────┼───────────────┼─────────────┼────────┼────────┼────────┤
│ local (current) │ unix://                            │ incus         │ file access │ NO     │ YES    │ NO     │
└─────────────────┴────────────────────────────────────┴───────────────┴─────────────┴────────┴────────┴────────┘
```

(transcript `incus-lab/transcripts/t03-remote-list.txt`;
the box-drawing is the client's own.)
Read the table like a sentence. `images`
is a **public** catalogue speaking
**simplestreams** with no authentication:
a book of starting points, not a server.
`local` is the daemon on this machine,
spoken to through the unix socket with
**file access** as its credential — if
you can open the socket, you may command
it, which is why the socket's permissions
are the security model (chapter 15). The
`(current)` marker names the default
remote: commands without a prefix go to
`local`.

Merely asking wrote the client's
configuration file. The lab read it back
(transcript t03-config-yml.txt):

```yaml
default-remote: local
remotes:
  images:
    protocol: simplestreams
    public: true
    addr: https://images.linuxcontainers.org
  local:
    protocol: incus
    public: false
    addr: unix://
aliases: {}
defaults:
  list_format: ""
  console_type: ""
  console_spice_command: ""
  no_color: false
```

That file — `~/.config/incus/config.yml`
at its usual home — is the *entire* client
side of Incus. Everything else you will
ever read with this tool is fetched live
from a daemon. Back this file up and your
client is restored; lose it and you have
lost a list of addresses, nothing more.

## The error, in its several wordings

Now ask for something only a daemon can
give. `incus list` is the polite version
(transcript t02):

```text
$ ./incus list
Usage: incus list ([<remote>:]|--all-remotes) [<filter>...]
                  ┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅

Error: The incus daemon doesn't appear to be started (socket path: /var/lib/incus/unix.socket)
```

Three lessons in one scolding. The client
prints the command's usage first (its habit
when a command fails at the door), then the
diagnosis, then — the part to hold onto —
**the exact socket path it tried**:
`/var/lib/incus/unix.socket`. "The daemon
doesn't appear to be started" therefore
means something precise: *nothing is
listening at the address I was configured
to use.* The causes branch from there —
daemon not installed, installed but not
started, running with a different state
directory — and troubleshooting (chapter
16) walks them in order. `incus info`
fails identically (transcript t05).

Other wordings exist deeper in. Ask the
catalogue a question that needs a local
daemon's help — listing images *with a
filter* — and the courtesy layer falls
away to the plumbing (transcript t07b):

```text
$ incus image list images:debian
Error: Get "http://unix.socket/1.0": dial unix /var/lib/incus/unix.socket: connect: no such file or directory
```

Same fact, no makeup: the API is HTTP,
the socket is a file, the file is not
there. Meeting both forms early means
neither will alarm you later.

## Asking the catalogue directly

An unfiltered question to a public remote
is the one thing the client does entirely
by itself — it fetched the catalogue's
index without troubling any daemon. In
the lab's sandbox the conversation got
exactly one step before the building's
network intervened (transcript t07):

```text
$ incus image list images:
Error: Failed parsing stream: Get "https://images.linuxcontainers.org/streams/v1/index.json": net/http: TLS handshake timeout
```

Recorded here, unhidden, because it is
the shape of a real and common failure:
the client *did* set out for
`images.linuxcontainers.org` on its own,
and the encrypted handshake never
completed. On an ordinary connection this
command answers with the full catalogue
— hundreds of images — and chapter 8
works with that list. The lab's sandbox
allowed it exactly one external
conversation (the release download of
chapter 3) and choked this one; the guide
says so rather than pasting in a list it
did not see. Hold the general lesson,
though, because chapter 16 needs it: an
Incus failure that names a URL and a
network error is about the road, not the
tool.

## Small rooms that work with no server

Two more rooms of the client are furnished
before any daemon exists. **Aliases** —
your own shortcuts — are stored locally,
and every client is born with one worth
knowing (transcript t06):

```text
$ ./incus alias list
┌───────┬──────────────────────┐
│ ALIAS │        TARGET        │
├───────┼──────────────────────┤
│ shell │ exec @ARGS@ -- su -l │
└───────┴──────────────────────┘
```

That is why `incus shell workbench` will
work in chapter 6: it is not a command,
it is this alias, expanding to an `exec`
that asks for a login shell. And **help**
is local and complete — every command
documents itself. The lab's favourite
page is the one chapter 5 lives on
(transcript t04), because its examples
are the tool's own:

```text
  incus launch images:debian/12 c1
      Create and start a container named "c1"

  incus launch images:debian/12 v1 --vm -c limits.cpu=4 -c limits.memory=4GiB
      Create and start a virtual machine named "v1" with 4 vCPUs and 4GiB of RAM
```

When this guide and `incus <command>
--help` disagree, believe the help: it
was generated by the program you are
actually running.
