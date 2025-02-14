---
title: Google Summer of Code 2025 Ideas List
---

## Unikraft Project Ideas

Thank you for your interest in participating in [Google Summer of Code 2025 (GSoC25)](https://summerofcode.withgoogle.com/programs/2025)!

Unikernels are a novel Operating System (OS) model providing unprecedented optimization for software services.
The technology offers a clean slate OS design which improves the efficiency of cloud services, IoT and embedded system applications by removing unnecessary software layers by specializing the OS image.
One unikernel framework which provides minimal runtime footprint and fast (millisecond-level) boot times is Unikraft, and aims as a means to reduce operating costs for all services that utilize it as a runtime mechanism.

Unikraft is a Unikernel Development Kit and consists of an extensive build system in addition to core and external library ecosystem which facilitate the underlying functionality of a unikernel.

## Mentors of the projects

Mentors will be assigned when the project is initiated.  Please feel free to reach out beforehand to discuss the project.

| Mentor | Email |
|--------|-------|
| [Razvan Deaconescu](https://github.com/razvand) | razvan.deaconescu@upb.ro |
| [Alexander Jung](https://github.com/nderjung) | alex@unikraft.io |
| [Cezar Crăciunoiu](https://github.com/craciunoiuc) | cezar@unikraft.io |
| [Michalis Pappas](https://github.com/michpappas) | michalis@unikraft.io |
| [Ștefan Jumărea](https://github.com/StefanJum) | stefanjumarea02@gmail.com |
| [Răzvan Vîrtan](https://github.com/razvanvirtan) | virtanrazvan@gmail.com |
| [Hugo Lefeuvre](https://github.com/hlef) | hugo.lefeuvre@ubc.ca |

Below are a list of open projects for Unikraft which can be developed as part of GSoC25.

---

### Expanding the Unikraft Software Support Ecosystem

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 2 |
| **Constraints/requirements** | Basic OS concepts, familiarity with POSIX and system calls, build systems and tool stacks. |

#### Description

One of the weak points of most unikernel projects has always been application support, often requiring that applications be ported to the target unikernel OS.
With Unikraft we have been making a strong push towards POSIX compatibility so that applications can run unmodified on Unikraft.
We have been doing this in two different ways:

1. by adding support for the Musl libc library such that applications can be compiled against it, using their native build systems, and then linked into Unikraft
1. through [binary-compatibility mode](https://unikraft.org/docs/concepts/compatibility), where unmodified ELFs are directly executed in Unikraft and the resulting syscalls trapped and redirected to the Unikraft core, via the [`app-elfloader`](https://github.com/unikraft/app-elfloader).

This has lead to the creation of the [application `catalog` repository](https://github.com/unikraft/catalog) where running applications and examples are brought together.

This project focuses on expanding Unikraft's software support ecosystem by [adding new applications](https://unikraft.org/docs/contributing/adding-to-the-app-catalog) to the [application `catalog` repository](https://github.com/unikraft/catalog), primarily in binary-compatibility mode.
While doing this, you will also:

1. implement and extend system calls
1. add extensive testing for the application or framework that is to be included in the catalog
1. add benchmarking scripts to measure the performance and resource consumption of the application running with Unikraft
1. conduct synthetic tests using tools such as [the Linux Test Project](https://linux-test-project.github.io/)

The success of this project will directly impact Unikraft adoption.
The project length can be varied depending on which of these items are covered by the project.

#### Reading & Related Material

* https://www.musl-libc.org/
* https://unikraft.org/guides/using-the-app-catalog
* https://github.com/unikraft/catalog
* https://unikraft.org/docs/contributing/adding-to-the-app-catalog

---

### Software Quality Assurance of Unikraft Codebase

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 2 |
| **Constraints/requirements** | C programming skills, Linux command-line experience, build tools |

#### Description

During its 6 years of existence, Unikraft, now at version 0.16.1, has grown in features, application support and codebase.
As it matures, a high quality of the code and robust behavior are a must to provide a stable solution for its user base.

The aim of this project is to assist in the software quality assurance of the Unikraft codebase, by tackling one of the following ideas:

1. The use of the [`uktest` framework](https://github.com/unikraft/unikraft/tree/staging/lib/uktest) to create unit tests for [internal libraries](https://github.com/unikraft/unikraft/tree/staging/lib/) and [external libraries](https://github.com/search?q=topic%3Alibrary+org%3Aunikraft+fork%3Atrue&type=repositories).
   Not many libraries have unit tests, those that do are mostly exceptions.
   This will directly impact the stability of the code base and allow quick validation of new features that may break existing functionality.

1. Inclusion of static and dynamic analysis tools that highlight potential spots of faulty or undefined behavior.

1. The use of compiler builtins and compiler flags providing constraints on the code to increase its resilience to faulty behavior.

1. Augmenting the CI/CD system used by Unikraft (based on [GitHub Actions](https://github.com/features/actions)) to feature automatic testing, validation and vetting of contributions to Unikraft repositories: core, libraries, applications.
   Potential items are:
   1. handling running of unikernels instead of simple builds
   1. static analysis of images to be delivered as reports to GitHub pull requests
   1. regression checks on performance (delivered as % change from the current upstream version)

Any other project that is targeted towards increasing the robustness of Unikraft source code is welcome.
These will both increase the viability of Unikraft as a stable solution and increase the quality of future contributions, by enforcing good practices on submitted code.

#### Reading & Related Material

* [Writing Tests in Unikraft](https://unikraft.org/docs/develop/writing-tests/)
* https://www.guru99.com/unit-testing-guide.html
* https://docs.kernel.org/dev-tools/kunit/index.html
* https://github.com/features/actions
* https://unikraft.org/docs/contributing/review-process/

---

### Supporting macOS networking (medium-large, 175-350hrs)

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | Good Go skills, familiarity with virtualization, macOS and networking, good OS knowledge |

#### Description

[KraftKit](https://github.com/unikraft/kraftkit), the supporting codebase for the modular library operating system Unikraft designed for cloud native applications, provides users with the ability to build, package and run unikernels.
As a Swiss-army-knife of unikernel development, it eases both the construction and deployment of unikernels.
To this end, supporting diverse user environments and their ability to run unikernels locally supports the ultimate goal of the project.  One such environment which requires more attention is macOS.

Towards better facilitating the execution of unikernel virtual machine images on macOS, this project aims to introduce new packages which interface directly with macOS environments by interfacing natively with the local networking environment such that the execution of unikernels is accessible through a more direct communication mechanisms of the host.
Until now, the project only supports Linux bridge networking with accommodation (albeit "stubs") in the codebase for Darwin.

#### Reading & Related Material

* https://github.com/unikraft/kraftkit/issues/841

---

### Converting the eroFS library to Golang and testing it

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | Good Go skills, decent C skills, familiarity with file systems, basic testing knowledge |

#### Description

[EROFS](https://docs.kernel.org/filesystems/erofs.html) (Enhanced Read-Only File System) is a lightweight, high-performance read-only filesystem tailored for Linux environments.
It is designed to provide fast and efficient access to data while supporting built-in transparent compression, which helps reduce storage overhead.
Currently, Golang has support through [libraries](https://pkg.go.dev/gvisor.dev/gvisor/pkg/erofs) only for reading EROFS files and no support for creating them.

Towards better support in [KraftKit](https://github.com/unikraft/kraftkit/pull/2007), this project aims to introduce a new library that reimplements the `mkfs.erofs` command with all its functionality.
This is [currently](https://github.com/erofs/erofs-utils/blob/dev/mkfs/main.c) implemented in C which can only be imported into Golang with C to Go bindings.
Some [attempts](https://github.com/dpeckett/archivefs/tree/main/erofs) have been made to implement this, but are incomplete and do not offer all arguments, of which some we need.
Finally, at all steps tests should be implemented that compare original functionality to the ported library functionality.

#### Reading & Related Material

* https://docs.kernel.org/filesystems/erofs.html
* https://github.com/erofs/erofs-utils/blob/dev/mkfs/main.c
* https://github.com/unikraft/kraftkit/pull/2007
* https://pkg.go.dev/gvisor.dev/gvisor/pkg/erofs
* https://github.com/dpeckett/archivefs/tree/main/erofs

---

### Fine-Tuning Unikraft's Performance

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | Good C skills, familiarity with general operating system concepts, good testing knowledge |

#### Description

Over the past releases the development focus of Unikraft has been set on improving its compatibility with existing code bases and adding missing operating system features.
This means that less efforts were dedicated to performance-testing Unikraft, resulting in a potential loss of performance in recent releases.
Now that Unikraft is reaching the desired level of maturity and compatibility, it is time to go back to evaluating and fine-tuning its performance.

The aim of this project is to 1) evaluate the current performance of Unikraft, 2) identify potential performance bottlenecks, and 3) address these bottlenecks through targeted patches.
- To evaluate the performance of Unikraft, this project will base on the evaluation of the Unikraft EuroSys paper, re-running experiments with the latest release of Unikraft.
  The first phase of the project will be to create a new repository with updated experiments that can easily be run in a push-button manner (deliverable 1).
- Following this, bottlenecks will be identified.
  Performance bottlenecks may lie in any Unikraft component: this will be a unique opportunity to touch on many operating system concepts.
  Performance bottlenecks will be reported in the form of GitHub issues (deliverable 2).
- Finally, the project will aim to provide self-contained, targeted fixes for these bottlenecks in the form of GitHub Pull-Requests (deliverable 3).

This project is a unique opportunity to learn about performance evaluation and optimization in a production-grade operating system.
It is also an opportunity to participate in a potential academic journal submission of Unikraft by refreshing its evaluation.

#### Reading & Related Material

* The Unikraft EuroSys 2021 paper (see the Evaluation, Section 5): https://dl.acm.org/doi/10.1145/3447786.3456248
* The EuroSys 2021 evaluation repository: https://github.com/unikraft/eurosys21-artifacts

---

### Testing Framework for Unikraft Builds

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | Python knowledge, Linux CLI |

#### Description

We are currently developing a [testing framework](https://github.com/unikraft-upb/catalog/tree/razvand/generator/new-design/utils/new-design) that is able to multiplex the variety of configuration options, VMMs, hypervisors, architectures, boot protocols, to validate the successful building and running of unikernel images.
This framework is able to configure, build, run and test the variety of Unikraft builds.
It is written in Python and is subject to improvements and refactoring.

We are looking to augment the testing infrastructure to make it seamless to be used by Unikraft developers and users.
To this end we aim to:

- Consolidate the testing framework as a separate project inside its own repository.
- Have the testing framework work out-of-the-box with the [`catalog`](https://github.com/unikraft/catalog) and [`catalog-core`](https://github.com/unikraft/catalog-core) repositories.
- Integrate the testing framework with the CI/CD system used in the [Unikraft organization repositories](https://github.com/unikraft) to automatically validate builds for contributions.
  Tests are to be triggered each time a pull request is open in the [`unikraft`](https://github.com/unikraft/unikraft) and in core library repositories.

#### Reading & Related Material

* https://github.com/unikraft/catalog
* https://github.com/unikraft/catalog-core
* https://github.com/unikraft-upb/catalog/tree/razvand/generator/new-design/utils/new-design

---

### Update Newlib and Pthread-embedded Libraries

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | C, assembly, Linux CLI, GNU build tools |

#### Description

The default Unikraft standard C library (libc) is [Musl](https://github.com/unikraft/lib-musl), a lightweight libc providing a POSIX interface.
Up until 2022, the default libc was [Newlib](https://github.com/unikraft/lib-newlib).
Starting with [release 0.11.0](https://unikraft.org/blog/2022-12-02-unikraft-releases-janus) the default libc switched to Musl.

Ever since that point, Newlib supported hasn't been updated to keep up with the recent version of Unikraft.

The goal of this project is to update [Newlib](https://github.com/unikraft/lib-newlib) and [`pthread-embedded`](https://github.com/unikraft/lib-pthread-embedded) support to the recent Unikraft versions.
Such as current builds would work out-of-the-box with Newlib and pthread-embedded as well as Musl.

The steps to be done are:

1. Update Newlib and pthread-embedded to build with the most recent Unikraft version.
1. Update Newlib version to the [most recent upstream version](https://sourceware.org/newlib/).
1. Build and run applications on the [`catalog-core`](https://github.com/unikraft/catalog-core) and [`catalog`](https://github.com/unikraft/catalog) repositories.
1. (Optionally) Add CI pipelines to work with Newlib and pthread-embedded.

#### Reading & Related Material

* https://github.com/unikraft/lib-newlib
* https://github.com/unikraft/lib-pthread-embedded
* https://github.com/RWTH-OS/pthread-embedded
* https://sourceware.org/newlib/

---

### Update Unikraft Core External Libraries

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 2 |
| **Constraints/requirements** | C, assembly, Linux CLI, GNU build tools |

#### Description

The Unikraft core external libraries haven't been updated in the past 2 years.
We aim to update them to their latest version.
That means:

- Update [`lib-musl`](https://github.com/unikraft/lib-musl) from 1.2.3 to 1.2.5 (the most recent [upstream Musl](https://musl.libc.org/) version).
- Update [`lib-lwip`](https://github.com/unikraft/lib-lwip) from 2.1.2 to 2.2.1 (the most recent [upstream LWIP](https://savannah.nongnu.org/projects/lwip/) version).
- Update [`lib-gcc`](https://github.com/unikraft/lib-gcc) from 7.3.0 to 14.2.0 (the most recent [upstream GCC](https://ftp.gnu.org/gnu/gcc/) version).
- Update [`lib-libcxx`](https://github.com/unikraft/lib-libcxx) from 14.0.6 to 19.1.7 (the most recent [upstream LLVM](https://github.com/llvm/llvm-project/releases) version).
- Update [`lib-libcxxabi`](https://github.com/unikraft/lib-libcxxabi) from 14.0.6 to 19.1.7 (the most recent [upstream LLVM](https://github.com/llvm/llvm-project/releases) version).
- Update [`lib-compiler-rt`](https://github.com/unikraft/lib-compiler-rt) from 14.0.6 to 19.1.7 (the most recent [upstream LLVM](https://github.com/llvm/llvm-project/releases) version).
- Update [`lib-libunwind`](https://github.com/unikraft/lib-libunwind) from 14.0.6 to 19.1.7 (the most recent [upstream LLVM](https://github.com/llvm/llvm-project/releases) version).

The update is aimed to use the [workflow for Unikraft microlibrary version](https://docs.google.com/document/d/1A-CAss5RvgYapg3YO8GNCdMki6cgq_7XG5om8nVWWGk/edit?usp=sharing).
As part of the update effort, we aim to also test and validate builds for the [`catalog-core`](https://github.com/unikraft/catalog-core) and [`catalog`](https://github.com/unikraft/catalog) repositories.

#### Reading & Related Material

* [RFC: Unikraft Microlibrary Versioning](https://docs.google.com/document/d/1A-CAss5RvgYapg3YO8GNCdMki6cgq_7XG5om8nVWWGk/edit?usp=sharing)

---

### Update Unikraft Application Libraries

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 2 |
| **Constraints/requirements** | C, assembly, Linux CLI, GNU build tools |

#### Description

The Unikraft application libraries haven't been updated in the past 2 years.
We aim to update them to their latest upstream version.
Target libraries / applications are:

- [`lib-nginx`](https://github.com/unikraft/lib-nginx)
- [`lib-redis`](https://github.com/unikraft/lib-redis)
- [`lib-sqlite`](https://github.com/unikraft/lib-sqlite)
- [`lib-python3`](https://github.com/unikraft/lib-python3)
- [`lib-libgo`](https://github.com/unikraft/lib-libgo)
- [`lib-lua`](https://github.com/unikraft/lib-lua)

The update is aimed to use the [workflow for Unikraft microlibrary version](https://docs.google.com/document/d/1A-CAss5RvgYapg3YO8GNCdMki6cgq_7XG5om8nVWWGk/edit?usp=sharing).
As part of the update effort, we aim to also test and validate builds for the [`catalog-core`](https://github.com/unikraft/catalog-core) and [`catalog`](https://github.com/unikraft/catalog) repositories.

#### Reading & Related Material

* [RFC: Unikraft Microlibrary Versioning](https://docs.google.com/document/d/1A-CAss5RvgYapg3YO8GNCdMki6cgq_7XG5om8nVWWGk/edit?usp=sharing)

---

### Add FreeBSD Libc as Unikraft External Library

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | C, assembly, Linux CLI, GNU build tools |

#### Description

The default Unikraft standard C library (libc) is [Musl](https://github.com/unikraft/lib-musl), a lightweight libc providing a POSIX interface.
[FreeBSD Libc](https://github.com/freebsd/freebsd-src/tree/main/lib/libc) is the default libc used by default by FreeBSD, with a compatible license with Unikraft.

The goal of this project is have a FreeBSD libc build repository for Unikraft and build existing applications against it.
In the end, you would be able to build and run applications on the [`catalog-core`](https://github.com/unikraft/catalog-core) and [`catalog`](https://github.com/unikraft/catalog) repositories using the FreeBSD libc variant.

#### Reading & Related Material

* https://github.com/freebsd/freebsd-src/tree/main/lib/libc

---
