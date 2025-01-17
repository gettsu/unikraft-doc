---
title: "Unikraft vs. UKL: What's the Difference?"
date: "2022-10-19T09:00:00+01:00"
author: "Felipe Huici"
tags: ["comparison"]
---

You may have seen recent news about a set of [patches](https://lore.kernel.org/lkml/20221003222133.20948-1-aliraza@bu.edu/) hitting the Linux kernel mailing list for Unikernel Linux (UKL), "... a research project aimed at integrating application-specific optimizations to the Linux kernel."
In fact, [the original UKL paper was published back in 2019](https://dl.acm.org/doi/10.1145/3317550.3321445), and follows a somewhat longish line of projects aimed at turning the Linux kernel into a "unikernel".

But what exactly is a unikernel?
Perhaps the easiest definition is that it is a virtual machine specialized to the needs of a particular target application, the key word being specialized.
And because of specialization the resulting unikernel can provide efficiency much higher than a general-purpose operating system such as Linux or FreeBSD could.
Here I am using efficiency as an umbrella term to mean higher I/O throughput, smaller images, lower memory consumption, lower latency and quick boot times, among others. And, because the entire software stack is specialized, that typically results in a much smaller Trusted Computing Base (TCB) and potentially fewer vulnerabilities. 

In addition to specialization, many unikernels have a single-memory address space: since many of them target cloud or virtual deployments where a hypervisor is running underneath and providing isolation already, they get rid of the user-space/kernel divide since it's a source of overhead.

Having a single memory address space is what projects like Unikernel Linux do; unfortunately, they obviate one of the main features of "true" unikernels: specialization.
In UKL, the Linux kernel is still there as it would be with a standard Ubuntu or Debian virtual machine, just running in the same address space as the application. 

Why would one bother turning a general-purpose OS like Linux into a unikernel in the first place, given that it was clearly not intended for unikernels/specialization?
The answer is great application compatibility: ultimately it's still Linux underneath, so all your favorite apps still run unmodified – or at least that's the theory.
In practice, UKL requires recompiling all applications/runtimes against a custom glibc and the kernel. 

In short, there are two ways to go about application compatibility:

1. Take an OS that already provides compatibility and try to specialize it (UKL)
2. Take a modular OS and build a compatibility layer (Unikraft)

While the UKL approach leverages Linux's inherent application support, Linux is essentially a monolithic operating system.
While it might be possible to pick and choose certain components, it was never designed to enable easy customization for each particular application one might want to run.
Trying to fully specialize Linux, while not impossible, is hard and time-consuming; and doing it for each separate application even more so.
Another project in this space was [RUMP](https://github.com/rumpkernel), which used OpenBSD to build single-address space “unikernels”, with lackluster performance (see graph below).
[Lupine Linux](https://github.com/hckuo/Lupine-Linux) uses [KML](https://github.com/sonicyang/KML) to also run user-space applications in the same address space as the kernel but also results in less-than-ideal performance and requires compiling against a custom version of the [musl libc library](https://www.musl-libc.org/).
This type of libc-level compatibility, also offered by UKL, is less than ideal: it prevents all kernel invocations that do not go through the libc to transparently benefit from unikernel performance acceleration.
This is problematic for certain languages (e.g. Go), for which most such invocations are made directly by the runtime and not through the libc.
In contrast, Unikraft offers system-call level compatibility, seamlessly boosting the performance of all kernel invocations.

{{< img
  class="max-w-lg mx-auto"
  src="/assets/imgs/figs/nginx-perf.svg"
  caption="Performance of NGINX on Unikraft vs. other runtime implementations."
  position="center"
>}}


We designed and built Unikraft with specialization in mind from the beginning, around a highly modular architecture where every component is a library that can be added, removed or configured, from low-level architecture-specific code, to schedulers, memory allocators and network stacks.
The result is Unikraft images that contain only the code that an application needs to run, nothing more, nothing less, always with optimization and performance in mind.
Beyond speed, we have shown in [past research](https://arxiv.org/pdf/2104.12721.pdf) that unikernels based on Linux (e.g. Lupine) cannot achieve the same degree of specialization as projects like Unkraft, built from scratch with modularity and lightweightness in mind: for example, even after careful optimization, the memory footprint and disk image size of a Linux unikernel are still one order of magnitude larger than that of Unikraft.

In a future post we'll do a head-to-head performance comparison against UKL, so please stay tuned!
