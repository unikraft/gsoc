# Unikraft GSoC'23: Expanding binary compatibility mode

<img width="100px" src="https://summerofcode.withgoogle.com/assets/media/gsoc-2023-badge.svg" align="right" />

## Summary

One of the weak points of most unikernel projects has always been application support, often requiring that applications be ported to the target unikernel OS.
In this area, Unikraft has made significant progress by prioritizing POSIX compatibility.
Through its binary compatibility mode, Unikraft allows unmodified ELFs to be executed, with resulting syscalls wrapped and redirected to the Unikraft core via the [`app-elfloader`](https://github.com/unikraft/app-elfloader).
Currently, Unikraft supports loading both statically and dynamically linked ELFs in binary compatibility mode.
However, Unikraft has not been able to fully support all behaviors of glibc and user applications, which causes certain programs to fail in initializing or crash.
Additionally, running applications in binary compatibility mode incurs performance loss compared to ported programs, due to the TLS switching and system interrupts caused by binary syscalls.
To address the issues mentioned above, this project focuses on expanding Unikraft's binary compatibility mode by: 

* Use [`vDSO`](https://man7.org/linux/man-pages/man7/vdso.7.html) (_virtual dynamic shared object_) and `vsyscall` (_virtual system call_) to bypass binary syscalls.

* Expanding supported syscalls to provide compatibility for a wider range of applications.

* Adding AArch64 architecture support for `app-elfloader` and `syscall-shim` layer.

* Implement a faster way to port new applications by externally compiling.

* Establishing a CI/CD system to ensure that new commits do not break libc compatibility.

## GSoC contributor

Name: `Tianyi Liu`

Email: `i.pear@outlook.com`

Github profile: [i-Pear](https://github.com/i-Pear/)

## Mentors

[Razvan Deaconescu](https://github.com/razvand)

[Simon Kuenzer](https://github.com/skuenzer)

[Stefan Jumarea](https://github.com/StefanJum)

## vDSO and vsyscall support for binary compatibility

The `vsyscall` (_virtual system call_) is the first and oldest mechanism used to accelerate system calls.
Due to security concerns, it has been deprecated, and `vDSO` (_virtual dynamic shared object_) serves as its successor.
However, in the context of unikernels, kernel isolation is not a concern, allowing us to utilize both of them.
The [`app-elfloader`](https://github.com/unikraft/app-elfloader) now supports both of these features.

If users don't have the source code of their application but desire the performance of Unikraft, [`app-elfloader`](https://github.com/unikraft/app-elfloader) can load unmodified ELF files.
However, there will be some performance overhead compared to a native build, while the two newly introduced features can help mitigate this difference.

The `vDSO` is available for both dynamically linked applications and statically linked ones, to accelerate certain commonly used time-related system calls (`clock_gettime`, `gettimeofday`, `time`, `clock_getres`).
As a result, some server programs can experience significant performance improvements.

And if the application is dynamically linked, it can further utilize `vsyscall` to avoid costly binary syscalls (since `vsyscall` has been deprecated in the latest version of libc, we provide patched dynamic runtime libraries for [`glibc`](https://github.com/unikraft/fork-glibc) and [`musl`](https://github.com/unikraft/fork-musl)).

## Adapting to the .NET runtime

`.Net` is a widely used framework.
According to TIOBE's rankings, `C#` and `Visual Basic`, both based on `.NET`, have market shares of `6.87%` and `2.90%` respectively.
Therefore, adapting to the `.Net` runtime can significantly increase the number of applications supported by Unikraft.

As a complex dynamic language virtual machine, the `.NET` runtime requires a lot of configuration in the environment, involving many system calls that require support from the operating system.
However, one advantage is that the `.NET` runtime is implemented in C++ and directly uses `glibc`, so our previous support for `glibc` can be reused, including `VDSO`.

For the `posix-mmap` module, we provided the `mlock` and `msync` system calls.

For the `uksched` module, we provided the `sched_getaffinity` and `sched_setaffinity` system calls.

## External compiling for various applications

Unikraft currently supports many native applications, but most of these applications are not very complex.
This is because developers have to understand the application's build process and port it into the Unikraft build system.
However, more complex applications like `Node.js`, for example, come with a large number of libraries, and their build processes are also quite intricate, making porting such applications a time-consuming task.

Our insight is that the Unikraft build system only needs to perform the final linking step without requiring full knowledge of the entire build process.
Therefore, compiling with the application's own build system and then handing over the object files to the Unikraft build system for linking would be a feasible approach.

Taking `Node.js` as an example, this complex application consists of over 2500 source code files distributed across more than 35 libraries.
However, using this new build method, we were able to complete the porting work in just a few hours.
This significantly speeds up the efficiency of porting applications.

## AArch64 support for binary compatibility

To support binary syscalls for the AArch64 architecture, I introduced an adapter function into the AArch64 interrupt handling module, redirecting system calls to the `syscall-shim` layer.
After testing, this has proven to work effectively.
However, it's worth noting that there are many libraries within Unikraft that do not offer complete support for the AArch64 architecture, which will require further effort.

## PR Contributions

* [[unikraft] AArch64 binary syscall support](https://github.com/unikraft/unikraft/pull/1009)

* [[app-elfloader] AArch64 support](https://github.com/unikraft/app-elfloader/pull/24)

* [[Static-pie-apps] Node.js static build](https://github.com/unikraft/static-pie-apps/pull/85)

* [[dynamic-apps] Busybox dynamic build](https://github.com/unikraft/dynamic-apps/pull/17)

* [[dynamic-apps] .Net 7.0.7 dynamic build](https://github.com/unikraft/dynamic-apps/pull/60)

* [[unikraft] Add dependencies for dotnet runtime](https://github.com/unikraft/unikraft/pull/1004)

* [[app-elfloader] VDSO & vsyscall support](https://github.com/unikraft/app-elfloader/pull/23)

* [[fork-musl] vsyscall support for musl](https://github.com/unikraft/fork-musl/pull/1)

* [[fork-glibc] vsyscall support for glibc](https://github.com/unikraft/fork-glibc/pull/1)

* [[unikraft] A tiny fix for build system](https://github.com/unikraft/unikraft/pull/817)

## Journals

The success of this project wouldn't have been possible without the help of community members.
We've engaged in numerous relevant discussions.
For details, you can check our meeting notes in the [Unikraft meeting notes repo](https://github.com/unikraft/meeting-notes/tree/staging/discussions/app-compat) and on [discord channel](https://discord.com/channels/762976922531528725/1118149304717160508).

## Blog posts

During GSoC'23, I provided 4 blog posts which offer a comprehensive status for the 3 months that I was involved with Unikraft:

* [1st blog post: Project proposal and initial steps](https://unikraft.org/blog/2023-06-23-unikraft-gsoc-app-compat-1/)

* [2nd blog post: Design details and history of vDSO](https://github.com/unikraft/docs/pull/292/files)

* [3rd blog post: Adaptation process for .NET 7.0](https://github.com/unikraft/docs/pull/300/files)

* [4th blog post: AArch64 Support](https://github.com/unikraft/docs/pull/306/files)

## Current status

All planned work has been completed.

## Acknowledgement

I would like to express my sincere gratitude to my mentors and the Unikraft community for their support throughout my participation in the project.
Throughout the course of this project, I've significantly deepened my understanding of unikernels.
Also, Unikraft's modular architecture and streamlined implementation have provided me with a clearer insight into how kernels function, enhancing my proficiency in kernel development.
Throughout the project, the community has always been a tremendous source of support and valuable insights.
I feel fortunate to have had the opportunity to work with all of you.
