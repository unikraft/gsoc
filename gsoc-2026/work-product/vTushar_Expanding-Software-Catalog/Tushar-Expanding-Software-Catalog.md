# GSoC'26: Expanding the Unikraft Software Catalog

<img src="https://developers.google.com/open-source/gsoc/resources/downloads/GSoC-Horizontal.svg"/>

## Project Overview

For Unikraft to be a credible alternative to a container runtime, it has to run the software people actually deploy.
At the start of this project the [catalog](https://github.com/unikraft/catalog) and [catalog-core](https://github.com/unikraft/catalog-core) covered the basics — Nginx, Redis, SQLite — but an engineer evaluating Unikraft for their stack could not find a DNS server, a time-synchronisation daemon, or a self-hosted Git host, and would move on.

Unikraft runs an application either in [**binary-compatibility mode**](https://unikraft.org/docs/concepts/compatibility), where `app-elfloader` maps an unmodified Linux ELF binary and a syscall shim routes each system call to a Unikraft handler, or as a **source port**, compiled against Unikraft's ports of libc (e.g. musl) and lwIP and linked directly into the unikernel — a smaller, single-purpose image with control over what is included.

The goal was to close those application gaps using both routes, and to fix the layers underneath — outdated libraries, missing syscalls, broken platform support — so that the applications which follow meet less friction.
Every application also ships a github actions workflow that builds it and boots it, so a regression shows up as a red run rather than as a surprise later on.

## GSoC Contributor

Name: Tushar Verma

Email: tusharVermaiota@proton.me

Github profile: [vTusharr](https://github.com/vTusharr)

## Mentors

* [Răzvan Deaconescu](https://github.com/razvand)
* [Răzvan Vîrtan](https://github.com/razvanvirtan)

## Contributions

### Applications added

**Dnsmasq** — a DNSSEC-validating DNS forwarder and cache, added as a **source port**.
The largest piece of work in the project: it builds on both x86_64 and arm64, and `app-dnsmasq.yml` runs a full DNS and DNSSEC suite (`test.sh`) against a booted image on both.
Getting there required work in three separate layers, described under *Workarounds* below.

- [lib-dnsmasq#2](https://github.com/unikraft/lib-dnsmasq/pull/2), [catalog-core#107](https://github.com/unikraft/catalog-core/pull/107), [lib-nettle#2](https://github.com/unikraft/lib-nettle/pull/2)

**Lighttpd** — a lightweight web server, ported **both ways**: first in bin-compatibility mode, then as a native source port against musl and lwIP.
The native port meant taking over the build system: lighttpd generates a `config.h` of feature probes and a config parser (via SQLite's `lemon`) before compiling, and Unikraft's build is plain make over a fetched tarball, so neither happens — both artifacts ship pre-generated, and TLS is a single toggle (`LIBLIGHTTPD_OPENSSL`) rather than a hard dependency.

- [catalog#282](https://github.com/unikraft/catalog/pull/282) (binary compatibility), [lib-lighttpd#1](https://github.com/unikraft/lib-lighttpd/pull/1) + [catalog-core#116](https://github.com/unikraft/catalog-core/pull/116) (source port)

**Chronyd** — the time-synchronisation daemon, in binary-compatibility mode, and one of my favourites.
A unikernel cannot adjust its own clock (`settimeofday`/`clock_settime` are stubs), but chronyd's `-x` mode tracks the offset to upstream servers in software and serves *corrected* time without touching the system clock.
The result is an NTP appliance: it boots, syncs to the pool, and the rest of the network gets accurate time from it, under **16 MB** of RAM.
It ships as two images — `chronyd/4.8`, and `chronyd/4.8-nts` which adds Network Time Security but pulls in gnutls and a CA bundle, taking the rootfs from ~2 MB to ~10 MB.
Each has a workflow that boots the image and polls it with a NTP client until the stratum is non-zero  so "it answers" and "it is synchronised" are checked separately.

- [catalog#288](https://github.com/unikraft/catalog/pull/288)

**Nginx 1.31** — a newer release added to `catalog/library`, which addresses [CVE-2026-42945](https://nvd.nist.gov/vuln/detail/CVE-2026-42945), with a matching build workflow.

- [catalog#280](https://github.com/unikraft/catalog/pull/280)

**Odin hello-world** — first example for the [Odin](https://odin-lang.org/) language,
Odin's `core:fmt` and `core:os` issue **raw Linux syscalls** rather than going through libc, so the example needs `CONFIG_LIBSYSCALL_SHIM_HANDLER=y` (the binary syscall trap); it also demonstrates Odin's C interoperability, which is why musl is in the example.

- [catalog-core#115](https://github.com/unikraft/catalog-core/pull/115)

### Library and platform fixes

Native ports kept tripping over the same class of outdated library, so several fixes landed in the libraries themselves.

- **[lib-nettle#2](https://github.com/unikraft/lib-nettle/pull/2)** — the existing port was tied to hand-written x86_64 assembly and a full GMP, so it only ever built on x86. Rebuilt on a current nettle over mini-gmp, generating the elliptic curve tables at build time instead of shipping assembly. This is what made DNSSEC possible on arm64.
- **[lib-openssl#11](https://github.com/unikraft/lib-openssl/pull/11)** — `openssl.org` retired `/source/old/`, so the build failed fetching a tarball that no longer existed. Now points at the permanent GitHub release tag.
- **[lib-pcre#5](https://github.com/unikraft/lib-pcre/pull/5)** — the archive moved from pcre.org to SourceForge.
- **[lib-zlib#13](https://github.com/unikraft/lib-zlib/pull/13)** — its `Config.uk` hard-selected `vfscore`, breaking any image on the newer posix-vfs stack. The application now decides its own filesystem stack.
- **[unikraft#1870](https://github.com/unikraft/unikraft/pull/1870)** — the arm64 binary syscall handler did not compile: `struct ukarch_execenv` undefined in `lib/syscall_shim/arch/arm64/syscall_handler.c`. One missing include.

### Adding workarounds

Several applications needed a gap worked around before they would run at all.
Each is a candidate for a proper upstream fix.

**`IP_PKTINFO` is not implemented in lwIP.**
dnsmasq uses it to learn which local address a query arrived on; `setsockopt()` failed outright, and the missing control message then made dnsmasq silently drop the UDP reply. Three patches make the `setsockopt()` non-fatal and stop requiring the control message on send and receive.

**Netlink `MSG_PEEK` is ignored.**
`nl_recvmsg`/`nl_recvfrom` declare `flags` as `__unused`, so a peek **consumes** the message.
The standard Linux peek-then-resize-then-read pattern — used by dnsmasq, glibc's `getifaddrs` and iproute2 — loses every other message and hangs forever once a peek consumes `NLMSG_DONE`.
The port pre-grows the buffer to 8 KB and reads once instead: the hang is gone, at the cost of silent truncation above that size safe for interface dumps, but a ceiling worth removing.

**`recvmmsg` is missing.**
Replies arrived and `select()` reported the socket readable, but chronyd never read a packet: Alpine's chrony is compiled with `HAVE_RECVMMSG` and has no runtime fallback, while the syscall shim returns `-ENOSYS`.
I verified two fixes — an `LD_PRELOAD` shim rewriting it into a `recvmsg` loop, and building from source with `HAVE_RECVMMSG` removed. The second shipped, since building from source also lets me drop dead weights (cmdmon, refclocks, privilege dropping, seccomp).

**CI networking on GitHub-hosted runners.**
The catalog-core run scripts build an isolated bridge with `POSTROUTING MASQUERADE` so the guest gets outbound Internet.
GitHub runners ship Docker preinstalled, and Docker sets the iptables `FORWARD` policy to `DROP` — which *persists after `dockerd` stops* — so the bridge silently drops forwarded packets and dnsmasq's forwarding fails in CI while working perfectly locally. The fix inserts `FORWARD ACCEPT` rules scoped to the bridge subnet, guarded behind `GITHUB_ACTIONS=true`.

### Blockers for applications which could not be added

**Gitea and an update to last year's diagnosis.**
Gitea does not implement git; it runs the binary from many call sites, and `routers/init.go` aborts startup outright if `git version` fails. So the entire question is whether we can spawn a subprocess or not.

Last year's work product recorded Gitea as *"blocked by `clone()` without `CLONE_VM`"*. That no longer holds: **Go does not use plain `fork`** — `syscall/exec_linux.go` sets `CLONE_VFORK | CLONE_VM`, which is exactly the case Unikraft's `clone` accepts.
In practice half of it works: with `CONFIG_LIBPOSIX_PROCESS_MULTIPROCESS=y` a C probe using `vfork()` + `execv()` + `waitpid()` runs git inside the unikernel, while the Go probe never reaches `execve` at all.

So the blocker is Go's vfork path specifically, not subprocesses in general — much narrower than "no multiple address spaces".

Measuring the alternatives against "does it spawn subprocesses":

- `gitea` (Go) -> hundreds of call sites, plus a fatal `git version` check at startup — **blocked**
- `gogs` (Go) -> same architecture, plus `git-module` — **blocked**
- `soft-serve` (Go) -> execs `upload-pack` in `pkg/git/service.go` — **blocked**
- `fossil` (C) -> `fork()` per connection — **will need a patch**
- [legit](https://github.com/icyphox/legit) (Go) -> two functions, both replaceable by go-git's pure-Go `plumbing/transport/server/` — **the tractable one**

The full investigation, with the probes is in [Blog Post III](https://github.com/unikraft/docs/pull/578).

**Three further defects found along the way:**

- **An out-of-bounds read in netlink** — found while putting dnsmasq's interface enumeration on `NETLINK_ROUTE`. `nl_handle()` guards `nlmsg_len <= len` but not `NLMSG_ALIGN(nlmsg_len) <= len`, so an unaligned dump request (1-byte `rtgenmsg`, `nlmsg_len=17`, which Linux accepts) underflows a `size_t` in `NLMSG_NEXT` and reads past the buffer. Linux returns `-EINVAL`; Unikraft crashes. dnsmasq's own requests are correctly sized so it dodges this, but anything else probing netlink will not.
- **A dual-stack compile break in lib-lwip** — the netlink glue uses `netif->address.type`, a field lwIP calls `ip_addr`. It only breaks when netlink and IPv6 are both enabled, so the driver is effectively IPv4-only.
- **Interface enumeration is not usable end-to-end** — the raw `RTM_GETADDR`/`RTM_GETLINK` dumps succeed, but dnsmasq still cannot resolve an interface *name* (`interface=en1` exits with `unknown interface en1`). The shipped config uses `listen-address=0.0.0.0` and avoids `local-service`.

## Blog Posts

The project work is documented in a 3-part blog series, *GSoC'26: Expanding the Unikraft Software Support Ecosystem*:

- [Blog Post I](https://github.com/unikraft/docs/blob/main/content/blog/2026-06-18-unikraft-gsoc-xpand-1.mdx), [Blog Post II](https://github.com/unikraft/docs/pull/572), [Blog Post III](https://github.com/unikraft/docs/pull/578)

## Current Status

Four applications were completed and submitted during the program — dnsmasq, lighttpd, chronyd and nginx 1.31 — along with the Odin language example, each with a workflow that builds and boots it.
Severa library and platform fixes accompany them, most of which unblock work beyond this project rather than only my own.

## Future Work

**Upstream the `recvmmsg` syscall** in `lib/posix-socket` — an extension of `recvmsg()` that receives multiple messages in a single system call, with performance benefits for some applications, and adds a timeout on the receive operation.

**Investigate [lwIP's SNTP module](https://www.nongnu.org/lwip/2_0_x/group__sntp.html)** as a much smaller alternative to a full chrony image for guests that only need to *consume* time. It already exists there and just needs the appropriate syscalls un-stubbed, and it would suit the arm platform which currently as of writting does not seems to seed clock properly.

## Main takeaways

The part I am most pleased with is that these are **functional images**, and that CI keeps them checking, dnsmasq answering DNSSEC validated queries, chronyd serving corrected time to a whole network under 16 MB.

chronyd pulled me into kernel timekeeping for the first time, monotonic versus wall clock, jiffies and the tick, and how a unikernel that cannot set its own clock can still serve accurate time to everyone else. That paid off when the arm64 build failed DNSSEC for reasons that made no sense at first — `generic_timer_epochoffset()` returns `0`, so `CLOCK_REALTIME` on arm starts at 1970 and every signature looks long expired.

The other lesson proving that Gitea can't be ported yet, and why, was worth more than a another catalog entry. And verification is where the real work is.

## Acknowledgements

My GSoC journey has been full of invaluable learnings and experiences.
From the very beginning, I learned how to write and shape proper git commits, remain persistent when concepts were difficult to grasp, and celebrate the progress that followed.
Beyond technical growth, I also gained important skills in staying on track with timelines, clearly communicating ideas, and presenting code and work in a professional manner.

All of this was possible thanks to the Google Summer of Code program, the supportive Unikraft community, and especially my mentors [Razvan Virtan](https://github.com/razvanvirtan) and [Razvan Deaconescu](https://github.com/razvand).
I am deeply grateful for the opportunity to contribute to this project as this was something I had joy while working.
A special note of thanks goes to Razvan Deaconescu for his continuous guidance, both technical and non-technical, helping me adopt good practices and encouraging a long-term mindset toward building impactful open-source projects.
