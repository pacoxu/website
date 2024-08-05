---
content_type: "reference"
title: OS Kernel Version Requirements
weight: 10
---

Many features rely on specific kernel functionalities and have minimum kernel version requirements.

## Pod [sysctls](/docs/tasks/administer-cluster/sysctl-cluster/#safe-and-unsafe-sysctls)

`sysctl` is a Linux-specific command-line tool used to configure various kernel parameters
and it is not available on non-Linux operating systems.

The following sysctls are supported in the safe set, which has a minimal kernel version requirement:

- `net.ipv4.ip_local_reserved_ports` (since Kubernetes 1.27, needs kernel 3.16+);
- `net.ipv4.tcp_keepalive_time` (since Kubernetes 1.29, needs kernel 4.5+);
- `net.ipv4.tcp_fin_timeout` (since Kubernetes 1.29, needs kernel 4.6+);
- `net.ipv4.tcp_keepalive_intvl` (since Kubernetes 1.29, needs kernel 4.5+);
- `net.ipv4.tcp_keepalive_probes` (since Kubernetes 1.29, needs kernel 4.5+);
- `net.ipv4.tcp_syncookies` (namespaced since kernel 4.6+).
- `net.ipv4.vs.conn_reuse_mode` (used in `ipvs` proxy mode, needs kernel 4.1+);

### kube proxy `nftables` proxy mode

[`nftables` proxy mode](/docs/reference/networking/virtual-ips/#proxy-mode-nftables) is introduced in v1.29.
The (alpha) nftables mode of kube-proxy now requires version 1.0.1 or later
of the nft command-line, and kernel 5.13 or later. (For testing/development
purposes, you can use older kernels, as far back as 5.4, if you set the
`nftables.skipKernelVersionCheck` option in the kube-proxy config, but this is not
recommended in production since it may cause problems with other nftables
users on the system.

## [Cgroup v2](/docs/concepts/architecture/cgroups/#requirements) requirements

Cgroup v2 requires that Linux Kernel version is 5.8 or later.
In [Linux 5.8](https://github.com/torvalds/linux/commit/4a7e89c5ec0238017a757131eb9ab8dc111f961c), the system-level `cpu.stat` file was added to the root cgroup for convenience.

In [runc document](https://github.com/containerd/cgroups/blob/0c03de4a3d82a5f02f455ccc8174cb0dc9c2a532/cgroup2/manager.go#L411-L430), Kernel older than 5.2 is not recommended due to lack of freezer.

## Others kernel requirements

Some features may depend on new kernel functionalities and have specific kernel requirements:

1. [Recursive read only mount](/docs/concepts/storage/volumes/#recursive-read-only-mounts): This is implemented by applying the `MOUNT_ATTR_RDONLY` attribute with the `AT_RECURSIVE` flag using `mount_setattr`(2) added in Linux kernel v5.12.
2. Pod user namespace support requires minimal kernel version 6.5+, according to [KEP-127](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/127-user-namespaces/README.md).
3. For [node system swap](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/2400-node-swap/README.md), tmpfs noswap is not supported until kernel 6.3.

## Longterm release kernel versions

Active kernel releases can be found in [kernel.org](https://www.kernel.org/category/releases.html).

There are usually several "longterm maintenance" kernel releases provided for the purposes of backporting bugfixes for older kernel trees. Only important bugfixes are applied to such kernels and they don't usually see very frequent releases, especially for older trees. Longterm release kernels like 4.19, 5.15.

## {{% heading "whatsnext" %}}

- See [sysctls](/docs/tasks/administer-cluster/sysctl-cluster/) for more details.
- Allow running kube-proxy with in [nftables mode](/docs/reference/networking/virtual-ips/#proxy-mode-nftables).
- Read more information in [cgroups v2](/docs/concepts/architecture/cgroups/).
