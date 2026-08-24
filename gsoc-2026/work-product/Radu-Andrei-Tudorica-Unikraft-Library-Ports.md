# GSoC'26: Upgrading Application Library Ports for Unikraft

<img src="https://developers.google.com/open-source/gsoc/resources/downloads/GSoC-Horizontal.svg"/>

## Summary

Unikraft ships **library ports**: recipes that build upstream applications such as Nginx, Redis and Python as unikernels.
Several of these had fallen years behind their upstream releases, and one could no longer be built at all on a current distribution.

My GSoC project brought six of these ports up to date and validated each one end to end, fixing whatever the platform was missing underneath along the way.

**Goals:**

- Upgrade the **Redis**, **Nginx** and **Python3** ports to current upstream releases.
- Extend to other ports in the set that had fallen behind.
- Validate each port by **running** the resulting unikernel, not just building it.
- Fix any gaps in Unikraft itself that the newer upstream versions expose.

**Delivered:**

- Upgraded **six ports**: SQLite, Lua, Nginx, Redis, Python3 and libgo.
- Completed the **Nginx STREAM module**, which had never linked successfully.
- Upstreamed **four fixes to the Unikraft core**, one of which was preventing every Python application from starting on the platform.
- Regenerated the **stale generated files** carried by the Python and libgo ports, some of which were six releases behind.
- Wrote a **testing document** with per-PR reproduction commands, and three blog posts.

## GSoC Contributor

**Name:** Radu Andrei Tudorica
**Email:** raduandreitudorica3@gmail.com
**GitHub profile:** https://github.com/RaduAndreiTudorica

## Mentors

- [Razvan Deaconescu](https://github.com/razvand)
- [Stefan Jumarea](https://github.com/StefanJum)
- [Sriprad Potukuchi](https://github.com/procub3r)

## Contributions

### 1) Library port upgrades

Each upgrade meant rewriting patches against restructured upstream code, tracking down build and link failures across hundreds of sources, and updating source lists for files that moved or disappeared upstream.

- **Key Pull Requests:**
  - **[SQLite 3.40.1 -> 3.53.1](https://github.com/unikraft/lib-sqlite/pull/12)**
  - **[Lua 5.4.4 -> 5.4.8](https://github.com/unikraft/lib-lua/pull/12)**
  - **[Nginx 1.15.6 -> 1.30.0](https://github.com/unikraft/lib-nginx/pull/22)**
  - **[Redis 7.0.11 -> 8.0.2](https://github.com/unikraft/lib-redis/pull/17)**
  - **[Python 3.10.11 -> 3.13.14](https://github.com/unikraft/lib-python3/pull/25)**
  - **[libgo, GCC 12.1.0 -> 14.4.0](https://github.com/unikraft/lib-libgo/pull/10)**

Highlights per port:

- **Nginx** crossed eight years of upstream changes.
  Sources moved, HTTP/2 proxy support was split into a new module the existing one calls into, and a pre-existing bug turned up: `ngx_http_limit_req_module.c` was listed twice, once under the wrong config option.
- **Redis 8** introduces `fast_float`, a C++ implementation of `strtod`.
  Rather than pulling a C++ toolchain into the unikernel, the port now provides a small `strtod`-based implementation.
  Redis already falls back to `strtod` for inputs `fast_float` cannot handle.
- **Python 3.13** needed four of its five patches rewritten or dropped, a new build rule to generate CPython's frozen module headers, and a regenerated `_sysconfigdata.py` that had been produced by a **3.7** build.
- **libgo** could not be built on any current distribution: GCC 12.1.0's own i386 intrinsics headers reference built-ins renamed in later releases, so they fail against a GCC 13+ host.

### 2) Completing the Nginx STREAM module

The STREAM section of the port listed plenty of feature modules but none of the core sources they depend on, so raw TCP and UDP proxying had never linked.
Adding the five missing sources was not enough: `NGX_STREAM_UPSTREAM_ZONE`, which upstream's `configure` defines and Unikraft's static `ngx_auto_config.h` did not, had to be added as well.

Validated with a real TCP proxy forwarding to a backend on the host.

- **Key Pull Request:**
  - **[Complete the STREAM module](https://github.com/unikraft/lib-nginx/pull/23)**

### 3) Core fixes

- **`_SC_CLK_TCK` in `sysconf()`** - CPython 3.13 reads this while initialising the `posix` module and treats a non-positive result as fatal, so **no Python application could start**.
  In 3.10 the value was read lazily, which is why the gap went unnoticed.
- **`extern "C"` guard in `ectx.h`** - the `uk_pal_ectx_*` functions were `static inline` without a guard, giving them C++ linkage when pulled into C++ code and breaking the build for any C++ application.
- **`_SC_CLK_TCK` in nolibc** - follow-up: the constant was undefined in nolibc's `unistd.h`, so applications building against it failed to compile once `sysconf()` handled the name.
- **Missing `<uk/assert.h>` in `lib/ukpod`**.

- **Key Pull Requests:**
  - **[Handle `_SC_CLK_TCK` in `sysconf()`](https://github.com/unikraft/unikraft/pull/1877)** (merged)
  - **[`extern "C"` guard in `ectx.h`](https://github.com/unikraft/unikraft/pull/1832)**
  - **[Define `_SC_CLK_TCK` in nolibc](https://github.com/unikraft/unikraft/pull/1887)**
  - **[Enable `LIBPOSIX_TTY_STDOUT_SERIAL` in the SQLite defconfig](https://github.com/unikraft/catalog-core/pull/106)**

### 4) Blog posts

- **[SQLite, Lua and Redis](https://github.com/unikraft/docs/pull/564)** (merged)
- **[Nginx, the STREAM module and Python3](https://github.com/unikraft/docs/pull/570)**
- **[Finishing Python3, reworking Redis, and libgo](https://github.com/unikraft/docs/pull/576)**

## Things worth passing on

**Generated files go stale silently.**
Both the Python and libgo ports carry files that a normal upstream build produces: `_sysconfigdata.py` for Python, and `sysinfo.go`, `libcalls.go` and the `linknames` files for libgo.
Nothing about them changes during an upgrade, so they keep working until something references a symbol only a newer generator would have emitted.
The error points at the source that wanted the symbol, never at the stale file.
Regenerating libgo's set meant building GCC 14.4.0 on the host.

**When a program prints nothing, check whether printing works at all.**
The Python port booted and went quiet, and I spent hours inside the interpreter's startup path before noticing the ukboot banner was missing too.
The banner is written with `fprintf`, so its absence meant userspace stdout was not reaching the console and the problem had nothing to do with Python.
The interpreter had been reporting the real error all along.

**A build that works in your directory proves little.**
Redis built fine for me and failed from a clean checkout; libgo's own baseline turned out to be unbuildable on any modern distribution.
Writing the testing document, running every command from scratch rather than reconstructing it from memory, is what surfaced both.

## Future Work

- **A Go application in catalog-core.** libgo is tested against `app-helloworld-go`, which has no defconfig; getting it running takes four config options that are easy to miss.
- **The core does not build with GCC 15.** It defaults to `-std=gnu23`, where an empty parameter list means zero parameters, so `(*ctorfn)(argc, argv)` in `lib/ukboot/boot.c` fails and nothing builds on a fresh Ubuntu 26.04 install.
- **Generating the generated files during the build.** Python's frozen module headers are now produced at build time; doing the same for libgo would mean invoking `mksysinfo.sh` and friends from the port.
- **CI for fork pull requests.** The `static-analysis` workflow now fails on every external PR, since `actions/checkout` refuses to check out fork code in a `workflow_run` context.

## Acknowledgements

Thanks to my mentors, Razvan Deaconescu, Stefan Jumarea and Sriprad Potukuchi, for their guidance and for consistently quick feedback, and in particular for the suggestion to write up the testing instructions, which found more problems than it documented.
Thanks also to the reviewers whose comments improved the work: fetching tarballs over HTTPS, questioning the value returned for `_SC_CLK_TCK` (which turned out to be wrong by a factor of 10000), and catching the nolibc build breakage.