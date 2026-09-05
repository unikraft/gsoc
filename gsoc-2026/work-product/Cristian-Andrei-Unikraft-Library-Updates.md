# GSoC'26: Expanding the Unikraft Software Support Ecosystem

<img src="https://developers.google.com/open-source/gsoc/resources/downloads/GSoC-Horizontal.svg"/>

## Summary

Unikraft turns an ordinary application into a tiny, single-purpose virtual machine named a unikernel.
It does this through **library ports**: build recipes that download an upstream project, apply the patches it needs to run without a normal operating system and compile it into the image.
A port is only useful if it stays close to upstream, but some of the most important ones had fallen years behind: the C library, the network stack, the compiler runtime etc. The C++ / LLVM toolchain ports were worse off: they no longer built with any recent compiler.

This project updated Unikraft's **core** and **toolchain** library ports to current upstream versions.
Each update adds a menuconfig option, so the old and new versions can both be selected and nothing that relied on the old one breaks.
In this project, I tested every port by **building** and **booting** the resulting unikernel, and in doing so it exposed several bugs in Unikraft's core itself, which I fixed and sent upstream.

The proposal committed to the core C and networking toolchain: **musl**, **lwIP** and **lib-gcc** together with integration testing in `catalog-core`.
All of that was delivered, and the **LLVM / C++ toolchain**, which the proposal had listed as follow-on work after the program, was completed within GSoC as well.

**Goals:**

- Update the core library ports to current upstream releases: **musl**, **lwIP** and **lib-gcc**, each selectable from menuconfig.
- Update the **LLVM toolchain** ports **compiler-rt**, **libunwind**, **libc++abi**, **libc++** and **intel-intrinsics** to a single, matching LLVM release.
- Testing every port properly by building and booting it on both **ARM** and **x86** architectures.
- Fix anything the newer versions break in the Unikraft core itself.
- Add automated testing so that opening a pull request on any of these libraries runs a real test.

**Delivered:**

- Updated the three **core libraries**: **musl** `1.2.3 -> 1.2.5`, **lwIP** `2.1.x -> 2.2.1`, **lib-gcc** `7.3.0 -> 14.2.0`.
- Updated the five **LLVM toolchain libraries** to **LLVM 21.1.8**, moving forward by seven major releases.
- Re-ported two more libraries as mentor follow-up: **OpenSSL** `1.1.1c -> 3.5.7` and **lib-arm-intrinsics**, refreshed to GCC 14.2.0 headers.
- Sent **three fixes to the Unikraft core** that were blocking C++ applications from building on arm64 and Xen, plus a merged fix to the `catalog-core` test runner.
- Wrote a reusable **runtime test app** that exercises the whole C++/LLVM stack and ran it across every compiler-architecture combination.
- Set up **CI integration**: a workflow on each library that kicks off a test run in `catalog-core` whenever a pull request is opened.
- Wrote **two blog posts** and a testing document with step-by-step commands for each PR.

## GSoC Contributor

**Name:** Cristian Andrei
**Email:** cristian.andrei1423@gmail.com
**GitHub profile:** https://github.com/CristianAndrei1423

## Mentors

- [Razvan Deaconescu](https://github.com/razvand)
- [Shashank Srivastava](https://github.com/shank250)

## Contributions

### 1) Core library updates

All three follow the **Microlibrary Versioning RFC** pattern: a menuconfig option lets the user pick the version and the old version keeps working, so nothing downstream breaks.
The existing patches were moved into per-version folders and rewritten to fit the reorganised code.

- **Key Pull Requests:**
  - **[musl 1.2.3 -> 1.2.5](https://github.com/unikraft/lib-musl/pull/98)**
  - **[lwIP 2.1.x -> 2.2.1](https://github.com/unikraft/lib-lwip/pull/81)**
  - **[lib-gcc 7.3.0 -> 14.2.0](https://github.com/unikraft/lib-gcc/pull/5)**

What each one needed:

- **musl:** re-ported all 20 compatibility patches to 1.2.5 and split them into separate versioned sub-folders.
In musl 1.2.5 a preprocessor guard was hiding the `dirent64` type alias from Unikraft's virtual file system, so it no longer compiled.
Removing that guard in patch `0014` fixes it.

- **lwIP:** re-ported 14 patches to 2.2.1. Along the way I registered a new module so the build would link, adapted the socket code to a renamed struct field and stopped a patch from stripping out a socket type the new version needs.
Every 2.2.1-only line is guarded by a config flag, so 2.1.x is untouched.

- **lib-gcc:** GCC 14.2.0 ships a newer **libffi** (3.4), but the port carried hand-made headers from the old libffi (3.2) that were picked up instead.
I regenerated the three libffi headers from the 14.2.0 sources and added the two x86-64 files the new version needs.
Confirmed through a small C program that makes a real libffi call and boots on QEMU, Firecracker and Xen.

### 2) LLVM toolchain library updates

These five libraries make up the C++ runtime stack. The first four are all one LLVM release and have to move together, whereas intel-intrinsics is just header files.
I chose **21.1.8** on purpose: it is the last release that still ships the small per-project source archives the ports download.
From version 22 onwards, LLVM only publishes a monorepo, which would break how the ports fetch their code.

- **Key Pull Requests:**
  - **[libunwind -> 21.1.8](https://github.com/unikraft/lib-libunwind/pull/13)**
  - **[libc++abi -> 21.1.8](https://github.com/unikraft/lib-libcxxabi/pull/8)**
  - **[libc++ -> 21.1.8](https://github.com/unikraft/lib-libcxx/pull/38)**
  - **[compiler-rt -> 21.1.8](https://github.com/unikraft/lib-compiler-rt/pull/22)**
  - **[intel-intrinsics -> 21.1.8 headers](https://github.com/unikraft/lib-intel-intrinsics/pull/6)**

What each library needed:

- **compiler-rt:** followed sources that had been moved or renamed between the two releases, and marked the 80-bit floating point helpers as x86-only, since they do not exist on arm64.
Version 21 also newly needs the `cpuid.h` header, so on x86-64 the port now pulls in intel-intrinsics to provide it.

- **libc++abi:** the patches ended up different enough between the two versions that they now live in separate per-version folders.
Version 21 also needs a newer C++ standard flag.

- **libc++:** rewrote its build-time config file for version 21's new format, made the threading sources conditional on whether threads are enabled, and added a small `aligned_alloc` helper the version expects.
Moreover, version 21's number-parsing code now pulls in LLVM's own libc, so the port had to **bundle 120 of LLVM-libc's headers**, as there is no separate download for them.

- **intel-intrinsics:** worked out the exact set of x86 header files needed from clang 21.1.8 and refreshed them.

### 3) OpenSSL and arm-intrinsics

- **[OpenSSL 1.1.1c -> 3.5.7](https://github.com/unikraft/lib-openssl/pull/12):** a full rewrite of the port, not a version bump, since OpenSSL 3.x is built completely differently (its new "provider" system).
I re-derived the whole build from OpenSSL's own build output: 981 C sources plus 39 assembly files for libcrypto and 94 for libssl.
Also, I replaced ~90 hand-written code-generation rules with a step that runs OpenSSL's own generator.

- **[lib-arm-intrinsics -> GCC 14.2.0](https://github.com/unikraft/lib-arm-intrinsics/pull/3):** this library just ships GCC's own ARM header files, unchanged, so the unikernel behaves like a normal GCC build.
I refreshed both headers from the old GCC-7-era to the new official GCC 14.2.0 ones and checked they were identical to the compiler's own copy.

### 4) Core fixes

Building the C++ and arm64 code with newer compilers turned up several bugs in the Unikraft core itself.
These are not library problems, the core had simply never been built this way before.
Several of them date back to a platform refactor and had gone unnoticed until the whole stack was tested with both Clang and GCC across every target.

- **arm64 C++ exception path:** the exception headers did an implicit `int -> enum` conversion that C++ rejects, and the fix is an explicit cast.
The same bug was also present on the Xen platform, where it had never been reported.

- **arm64 under Clang:** the floating-point save code was compiled with the floating-point registers switched off, and Clang's assembler then rejected the floating-point instructions it contained.
This turned out to be a regression of a previously-closed issue.

- **C++ on Xen:** the C++ include path was missing one entry, so `libc++abi` could not find the Xen platform's exception header and the build failed.
Adding that entry fixes it.

- **QEMU hanging during tests:** the test runner started QEMU in the background, where it froze trying to read the terminal.
Redirecting its input from `/dev/null` in all 25 run scripts fixed it.

Together with a separate `extern "C"` core fix contributed by another GSoC participant (which these PRs depend on), these are what finally let C++ applications build on arm64 and Xen.

- **Key Pull Requests:**
  - **[plat/native: fix arm64 build for C++ apps (int -> enum cast)](https://github.com/unikraft/unikraft/pull/1876)**
  - **[plat/native: fix arm64 build under Clang](https://github.com/unikraft/unikraft/pull/1875)**
  - **[plat/xen: fix Xen build for C++ apps](https://github.com/unikraft/unikraft/pull/1879)** (merged)
  - **[catalog-core: redirect QEMU stdin to /dev/null](https://github.com/unikraft/catalog-core/pull/108)** (merged)

### 5) Integration testing and CI

- **Runtime test app:** a small standalone app with 14 checks that each genuinely use one of the libraries: throwing and catching exceptions (libunwind and libc++abi), runtime type checks, 128-bit and extended-precision math (compiler-rt) and libc++ containers and algorithms.

- **CI dispatch:** a workflow added to each toolchain library that, when a pull request is opened, tells `catalog-core` to boot the relevant apps against that PR's code and report back.
The sending half is open as a PR on all five libraries, the receiving half is built and tested against `catalog-core` [PR #118](https://github.com/unikraft/catalog-core/pull/118) and is on hold until that PR is merged and the maintainers install the required GitHub app.

- Sender PRs: [compiler-rt#23](https://github.com/unikraft/lib-compiler-rt/pull/23),
    [libunwind#15](https://github.com/unikraft/lib-libunwind/pull/15),
    [libc++#39](https://github.com/unikraft/lib-libcxx/pull/39),
    [libc++abi#9](https://github.com/unikraft/lib-libcxxabi/pull/9),
    [intel-intrinsics#7](https://github.com/unikraft/lib-intel-intrinsics/pull/7).

### 6) Blog posts

- **[Expanding Unikraft's Software Support Ecosystem - Blog post 1](https://github.com/unikraft/docs/pull/568)**
- **[Expanding Unikraft's Software Support Ecosystem — Blog post 2](https://github.com/unikraft/docs/pull/577)**

## Testing / current state

Everything was checked by building **and** booting, never by building alone.

- **musl / lwIP:** the full `catalog-core` test suite passes with **both clang and gcc** on the fixed core, across QEMU, Firecracker and Xen wherever those can run locally.
The networking apps were confirmed on both architectures.

- **lib-gcc:** the libffi update is proven by real libffi call running inside the unikernel on x86-64.

- **LLVM 21.1.8:** the `llvm-stress` app passes **14/14** in every combination.
A useful finding: libc++ 21 needs a fairly new compiler, **clang 19 or newer** on x86, **gcc 15 or newer** on arm64.

- **OpenSSL 3.5.7:** a small in-unikernel test reproduces a host SHA-256 hash exactly, a real **nginx HTTPS** server booted and served a page.

All the library PRs are open and tested.
The arm64 build path relies on the core fixes above, which is what the `Depends-On:` line in a PR is for.
The core-library and mentor follow-up PRs are in review.

## Things worth passing on

- **Newer libc++ needs a newer compiler than Ubuntu ships.** libc++ 21 uses compiler features that only exist in clang 19 and gcc 15.
The distribution's gcc-14 cross-compiler simply fails on arm64.
The arm64 cases only became testable after I downloaded the newer GCC.

- **"Use clang" does not mean clang on every architecture.** Clang can cross-compile to any architecture on its own, plain gcc cannot, and the arm64 build sets no cross-compiler by default.
That means a run meant to use gcc can silently fall back to the wrong compiler and pass for the wrong reason.
The safe habit is to name the compiler explicitly for every target.

## Future work

- **Merge the core fixes** which unblock the arm64 build for every C++ library PR.
- **Finish the CI receiving side** once the depended-on PR is merged.
- **Generate lib-gcc's generated files during the build** instead of carrying regenerated copies, the way other ports in the catalog do.

## Acknowledgements

Thanks to my mentors, Razvan Deaconescu and Shashank Srivastava, for their guidance and quick feedback throughout, and to the reviewers whose comments improved the work, especially the push to test each port by booting it, which is what turned up most of the core bugs in the first place.