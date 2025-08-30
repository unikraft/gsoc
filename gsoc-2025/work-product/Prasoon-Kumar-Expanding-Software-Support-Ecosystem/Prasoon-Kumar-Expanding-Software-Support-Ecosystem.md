# Unikraft GSoC'25: Expanding the Unikraft Software Support Ecosystem

<img src="https://unikraft.org/images/gsoc25.jpeg"/>

## Summary

Unikraft makes it easy to run existing applications in two ways:

1. **Musl libc support**:
    Applications can be compiled against Musl libc and linked directly with Unikraft to produce small, efficient unikernels.

2. [**Binary-compatibility mode**](https://unikraft.org/docs/concepts/compatibility):
    Unikraft can run unmodified Linux ELF binaries by using an ELF loader app and a system-call shim layer.
    When a binary-compatible unikernel starts, the `app-elfloader` parses and maps the ELF segments into memory, then transfers control to the application’s entry point.
    Each time the application makes a Linux system call, the shim intercepts the call and routes it to the matching Unikraft handler.
    If a call is not implemented, it returns `ENOSYS`.
    Many applications can still handle or bypass this because Unikraft follows the Linux x86_64 ABI closely.
    This allows existing binaries to run on top of Unikraft without recompilation.

These approaches make it possible to run applications directly on hypervisors such as KVM, QEMU, or Firecracker, while benefiting from the performance and security advantages of unikernels.

My GSoC project focused on **expanding the set of applications that work in binary compatibility mode**.
I tested multiple applications, identified the system calls and features they rely on, and added support for several of them in the Unikraft ecosystem.
This included adding applications as libraries in [the Unikraft catalog](https://github.com/unikraft/catalog), creating workflows for continuous integration, and introducing workarounds where full implementations are not yet available.
The outcome of this work is not just a larger [catalog](https://github.com/unikraft/catalog) of supported applications, but also a clearer understanding of what needs to be implemented next to grow binary compatibility further.

## GSoC Contributor

Name: Prasoon Kumar

Email: prasoonkumar054@gmail.com

Github profile: [prasoon054](https://github.com/prasoon054)

## Mentors

* [Răzvan Vîrtan](https://github.com/razvanvirtan)
* [Răzvan Deaconescu](https://github.com/razvand)

## Contributions

### Adding applications

During the GSoC program, I tested 16 applications in binary compatibility mode to evaluate their support in Unikraft.
From these, I was able to add full support for 3 applications and also contributed their Github Actions workflow YAML files, making it easier to build and test them automatically.

- **SQLite**:
    - A small, serverless SQL database engine that stores all data in a single file, perfect for edge and cloud applications needing fast, local storage without a separate server.
    - [PR: Introduce SQLite as library](https://github.com/unikraft/catalog/pull/163)

- **Mosquitto**:
    - A lightweight MQTT message broker used in microservice systems, enabling efficient, real-time publish / subscribe communication.
    - [PR: Introduce Mosquitto as library](https://github.com/unikraft/catalog/pull/207)

- **Traefik**:
    - A modern HTTP reverse proxy and load balancer with automatic service discovery.
    - [PR: Introduce Traefik as library](https://github.com/unikraft/catalog/pull/210)

In addition to these, I also worked on adding **Prometheus**.
The application itself could be integrated, but it required `mmap` with `MAP_SHARED` support on file descriptors.
I implemented a workaround for this, which is explained in the next subsection.

- **Prometheus**:
    - A monitoring and alerting system for cloud-native environments, collecting and querying metrics to help operators observe and maintain distributed services.
    - [PR: Introduce Prometheus as library](https://github.com/unikraft/catalog/pull/227)

### Adding workaround for `mmap` with `MAP_SHARED`

Unikraft already had partial support for `mmap` with `MAP_SHARED` on file descriptors, but only for read-only mappings.
Many of the applications I tested needed writable mappings, which was not yet supported.
To make progress, I added a workaround where writable `MAP_SHARED` mappings are downgraded to `MAP_PRIVATE`.
This workaround is useful only when applications do not require persistence or synchronization of writes back to the file.
In such cases, the application behaves correctly without any changes.
Prometheus is one such workload, and the workaround made it possible to run it in binary compatibility mode.

- [PR: Add workaround for writable MAP_SHARED](https://github.com/unikraft/unikraft/pull/1679)

### Blockers for applications which could not be added

Besides adding support for the above applications, I also tracked the missing system calls and features that are blocking support for others.
This helps to clearly identify what needs to be implemented next in the syscall layer.

- **Applications blocked by writable `MAP_SHARED`**
    - `deno`, `prometheus`, `netdata`, `rabbitMQ`, `tomcat`
    - Out of these, `deno`, `prometheus`, and `netdata` do not rely on writeback or synchronization, so they can be supported with the current workaround. Workloads like `rabbitMQ` and `tomcat`, however, need proper writeback support, which is not yet available.

- **Applications blocked by NETLINK sockets**
    - `syncthing`, `etcd`, `minio`, `vault`, `consul`, `loki`, `rabbitMQ`, `tomcat`, `influxDB`, `netdata`, `wireguard`
    - All of these rely on NETLINK socket, which is still a work in progress in Unikraft.

- **Applications blocked by clone() without CLONE_VM**
    - `gitea`
    - This requires creating a child process with a separate virtual memory space, which is also a work in progress.

- **Partial Support**
    - Although I added support for `traefik`, the `watch: true` configuration is not yet supported because it depends on `inotify_init1` and `inotify_add_watch`, which are currently missing.

## Blog Posts

 - [Blog post #1: Expanding the Unikraft Software Support Ecosystem Part-I](https://github.com/unikraft/docs/pull/493)
 - [Blog post #2: Expanding the Unikraft Software Support Ecosystem Part-II](https://github.com/unikraft/docs/pull/507)
 - [Blog post #3: Expanding the Unikraft Software Support Ecosystem Part-III](https://github.com/unikraft/docs/pull/514)
 - [Blog post #4: Expanding the Unikraft Software Support Ecosystem Part-IV](https://github.com/unikraft/docs/pull/518)

## Future Work

The outcomes of this project provide a clear picture of what is needed next to extend Unikraft’s binary compatibility support.
By testing and documenting application requirements, it is now easier to prioritize the system calls and features that should be implemented first.

From my observations, **the biggest blocker is NETLINK socket support**, as many widely used applications (`syncthing`, `etcd`, `minio`, `vault`, `consul`, `loki`, `rabbitMQ`, `tomcat`, `influxDB`, `netdata`, `wireguard`) depend on it.
Enabling NETLINK sockets would immediately unlock support for a large number of applications.

Another important direction is to add **proper writable `MAP_SHARED` support**.
The workaround I introduced allows applications like `prometheus`, `deno` and `netdata` to run without persistence or synchronization, but for workloads such as `rabbitMQ` and `tomcat`, full support is required.

There is also a need to support cases where the **`clone()` system call creates a child process with a separate virtual memory space**.
This is currently not supported but is required by applications like `gitea`.

In addition, support for **inotify system calls** would make features like Traefik’s `watch: true` configuration possible.

Overall, the next steps involve focusing on these missing features in the syscall layer.
Addressing them will not only expand the set of applications supported by Unikraft, but also make it more practical for real-world cloud-native deployments.

## Acknowledgements

This has been a very fun and challenging journey, and I am grateful to everyone in the Unikraft community who made this GSoC project possible.
I would like to thank my mentors, Răzvan Vîrtan and Răzvan Deaconescu for their constant guidance and support.
The help and encouragement I received from other members from community made a big difference and made me enjoy working on the project even more.
I truly appreciate the welcoming and helpful spirit of the community, which made the whole experience even better.
