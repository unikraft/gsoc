# GSoC’25: Updating Newlib 4.5 & pthread-embedded for Unikraft

<img src="https://unikraft.org/images/gsoc25.jpeg"/>

## Summary

Unikraft’s default C library was **Newlib** until 2022, when **musl** became default with **v0.11.0**.
Since then, Newlib support lagged behind.
My GSoC project updated **Newlib 4.5** and **pthread-embedded** to work seamlessly with modern Unikraft, ensuring applications build and run with either **musl** or **Newlib + pthread-embedded**.

**Goals:**
- Update **Newlib** and **pthread-embedded** for latest Unikraft.
- Upgrade to **Newlib 4.5** upstream.
- Validate with **catalog-core** and **catalog** apps.
- *(Optional)* Add CI pipelines.

**Delivered:**
- Ported **Newlib 4.5** and **pthread-embedded** to compile/link cleanly.
- Mapped Newlib to Unikraft’s syscall shim with POSIX names (`open`, `read`, `write`).
- Scripted verbose build logs to **Makefile.uk** entries.
- Fixed header/compatibility issues.
- Benchmarked **Newlib** vs **musl** (Python, Redis, Nginx).

## GSoC Contributor

**Name:** Salar Samani
**Email:** soltanisamani@gmail.com
**GitHub profile:** https://github.com/SalarSamani

## Mentors

- [Stefan Jumarea](https://github.com/StefanJum)
- [Cezar Craciunoiu](https://github.com/craciunoiuc)

## Contributions

### 1) Port & Build Integration

- Integrated **Newlib 4.5** with Unikraft’s `Makefile.uk`.
- Imported generated headers (e.g., `targ-include/newlib.h`), aligned include paths.
- Used POSIX syscall names for Unikraft’s syscall shim.
- Updated **pthread-embedded** (headers, atomics, signatures) for clean builds.
- **Key Pull Requests:**
  - **[Update pthread-embedded for Unikraft](https://github.com/unikraft/lib-pthread-embedded/pull/14)**
  - **[Update Newlib 4.5 for Unikraft](https://github.com/unikraft/lib-newlib/pull/38)**

### 2) Compatibility & Cleanup

- Fixed headers (e.g., `fcntl.h`, `sys/*`, `signal handling`) for missing/legacy declarations.
- Avoided duplicate symbols by selecting used sources from build output.

### 3) Application Support

- Enhanced application compatibility in Unikraft’s catalog:
- **Key Pull Requests:**
  - **[Improve lwip compatibility](https://github.com/unikraft/lib-lwip/pull/67)**
  - **[Update Python3 support](https://github.com/unikraft/lib-python3/pull/24)**
  - **[Update Redis support](https://github.com/unikraft/lib-redis/pull/15)**

### 4) Blog Posts

- [First blog post](https://github.com/SalarSamani/docs/blob/gsoc_blog_3_4/content/blog/2025-06-26-port_newlib_pthreadembedded_step1.mdx)
- [Second blog post](https://github.com/SalarSamani/docs/blob/gsoc_blog_3_4/content/blog/2025-07-16-port_newlib_pthreadembedded_step2.mdx)
- [Third blog post](https://github.com/SalarSamani/docs/blob/gsoc_blog_3_4/content/blog/2025-08-18-port_newlib_pthreadembedded_step3.mdx)

## Future Work

- Add optional Newlib features (locale/iconv/MB) via Kconfig.
- Expand **catalog-core**/**catalog** app coverage, document edge cases.
- Implement CI for Newlib + pthread-embedded.
- Deepen perf/size analysis vs musl, share scripts.

## Acknowledgements

Thanks to the Unikraft community for feedback and support. Special thanks to my mentors (Stefan Jumarea, Cezar Craciunoiu) for guidance.
