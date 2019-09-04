# Introducing runC: a lightweight universal container runtime
# On Infrastructure Plumbing
- To build a platform like Docker you need a lot of infrastructure plumbing; in fact over the past two years even though our code base has grown to tens of thousands of lines of code; roughly 50% of it is plumbing! 
- Infrastructure plumbing is made of small software tools which perform basic fundamental tasks in the most reliable and simple way possible. 
- It is invisible and under-appreciated especially given that plumbing is what holds the world’s Internet infrastructure together.

- To build Docker we have re-used large quantities of plumbing: `Linux`, `Go`, `lxc`, `aufs`, `lvm`, `iptables`, `virtualbox`, `vxlan`, `mesos`, `etcd`, `consul`, `systemd`… the list goes on. 
- Docker wouldn’t be possible without the thousands of people who contributed to create this plumbing.
- When plumbing was not available or not sufficient, with the help of the Docker community, we built some of our own too. 
- And as the volume and scope of contributions grew, so did the quality and quantity of the underlying plumbing. 
- Of the tens of thousands of lines of code that constitute the Docker platform, roughly 50% is plumbing! 
- Docker has plumbing for interacting with both Linux and Windows native capabilities; it has plumbing for networking; service discovery; master election; security; and more.

# Infrastructure Plumbing Manifesto
- How we create and reuse infrastructure plumbing is fundamental to the Docker project. 
- Our approach boils down to 3 fundamental principles which we call “the Infrastructure Plumbing Manifesto”:
1. Whenever possible, re-use existing plumbing and contribute improvements back.
2. When you need to create new plumbing, make it easy to re-use and contribute improvements back. This grows the common pool of available components, and everyone benefits.
3. Follow the unix principles: several simple components are better than a single, complicated one.
4. Define standard interfaces which can be used to combine many simple components into a more sophisticated system.

There has been lots of demand for separating this plumbing from the Docker platform, so that it can be re-used by other infrastructure plumbers in accordance with infrastructure plumbing best practices. Today we are excited to announce that we are doing just that!

# Spinning Out ALL Docker Infrastructure Plumbing
Starting today we will begin spinning out ALL INFRASTRUCTURE PLUMBING from the Docker platform, This is a big deal, and the most important architectural change for the Docker project since its introduction.This has many benefits for Docker:
- If you are deploying Docker in production: this makes Docker more ops-friendly. Because its underlying plumbing will be more cleanly separated, the platform becomes more modular; which in turn makes it easier to scale, easier to troubleshoot, easier to secure and easier to customize.
- If you want to integrate Docker with your favorite tool: because all plumbing exposes standard interfaces, each component becomes a potential integration point.

On June 22 2015, we are introducing our most famous piece of plumbing: our OS container runtime.
# Introducing runC: The universal container runtime
- Docker is a platform to build, ship and rundistributed applications – meaning that it runs applications in a distributed fashion across many machines, often with a variety of hardware and OS configurations. 
- For this to be possible, it needs a sandboxing environment capable of abstracting the specifics of the underlying host (for portability), without requiring a complete rewrite of the application (for ubiquity), and without introducing excessive performance overhead (for scale).
- Over the last 5 years Linux has gradually gained a collection of features which make this kind of abstraction possible. 
- Windows, with its upcoming version 10, is adding similar features as well. 
- Those individual features have esoteric names like “control groups”, “namespaces”, “seccomp”, “capabilities”, “apparmor” and so on. 
- But collectively, they are known as `OS containers` or sometimes `lightweight virtualization`.
- Docker makes heavy use of these features and has become famous for it. 
- Because “containers” are actually an array of complicated, sometimes arcane system features, we have integrated them into a unified low-level component which we simply call `runC`. 
- And today we are spinning out `runC` as a standalone tool, to be used as plumbing by infrastructure plumbers everywhere.

`runC` is a lightweight, portable container runtime. It includes all of the plumbing code used by Docker to interact with system features related to containers. It is designed with the following principles in mind:
- Designed for security.
- Usable at large scale, in production, today.
- No dependency on the rest of the Docker platform: just the container runtime and nothing else.

Popular runC features include:
- Full support for Linux namespaces, including user namespaces
- Native support for all security features available in Linux: Selinux, Apparmor, seccomp, control groups, capability drop, pivot_root, uid/gid dropping etc. If Linux can do it, runC can do it.
- Native support for live migration, with the help of the CRIU team at Parallels
- Native support of Windows 10 containers is being contributed directly by Microsoft engineers
- Planned native support for Arm, Power, Sparc with direct participation and support from Arm, Intel, Qualcomm, IBM, and the entire hardware manufacturers ecosystem.
- Planned native support for bleeding edge hardware features – DPDK, sr-iov, tpm, secure enclave, etc.
- Portable performance profiles, contributed by Google engineers based on their experience deploying containers in production.
- A formally specified configuration format, governed by the Open Container Project  under the auspices of the Linux Foundation. In other words: it’s a real standard.

> The goal of runC is to make standard containers available everywhere

In fact, we have decided to donate the code of runC itself to the `OCP` foundation. Because OCP is designed to work just like the Linux Foundation, we expect that the maintainers – a blend of employees from various container-focused companies and hobbyists – will be largely left alone, and will continue to write the most awesome software possible.

runC is available today and is already under active development. Because it is based on the battle-tested plumbing used by Docker, you can use it in production today, either as part of a Docker deployment or in your own custom platform. 




























