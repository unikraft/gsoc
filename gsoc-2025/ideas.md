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

### Supporting macOS networking (medium-large, 175-350hrs)

| | |
|-|-|
| **Difficulty** | 3/5 |
| **Project Size** | Variable (175 or 350 hours) |
| **Maximum instances** | 1 |
| **Constraints/requirements** | Good Go skills, familiarity with virtualization, macOS and networking, good OS knowledge |

#### Description

[KraftKit](https://github.com/unikraft/kraftkit), the supporting codebase for the modular library operating system Unikraft designed for cloud native applications, provides users with the ability to build, package and run unikernels.
As a swiss-army-knife of unikernel development, it eases both the construction and deployment of unikernels.
To this end, supporting diverse user environments and their ability to run unikernels locally supports the ultimate goal of the project.  One such environment which requires more attention is macOS.

Towards better facilitating the execution of unikernel virtual machine images on macOS, this project aims to introduce new packages which interface directly with macOS environments by interfacing natively with the local networking environment such that the execution of unikernels is accessible through a more direct communication mechanisms of the host.
Until now, the project only supports Linux bridge networking with accommodation (albeit "stubs") in the codebase for Darwin.

#### Reading & Related Material

* https://github.com/unikraft/kraftkit/issues/841

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
