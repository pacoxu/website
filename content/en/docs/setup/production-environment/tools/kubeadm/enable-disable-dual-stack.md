---
reviewers:
- sig-cluster-lifecycle
- neolit123
- sftim
title: Enable Or Disable Dual-Stack via kubeadm
feature:
  title: Enable Or Disable Dual-Stack via kubeadm
content_type: task
weight: 110
---

<!-- overview -->

{{< feature-state for_k8s_version="v1.21" state="beta" >}}

IPv4/IPv6 dual-stack enables the allocation of both IPv4 and IPv6 addresses to {{< glossary_tooltip text="Pods" term_id="pod" >}} and {{< glossary_tooltip text="Services" term_id="service" >}}.

<!-- body -->

## Enable dual-stack via kubeadm

Install kubeadm following the steps from the [Installing Kubeadm](/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) documentation.

Make sure that nodes allow IPv6 forwarding, if not, run `sudo sysctl -w net.ipv6.conf.all.forwarding=1` on every node in the cluster.

{{< note >}}
`kubeadm upgrade` will change `IPv6DualStack` to true by default if feature gate is not set in old cluster. However, cluster-cidr and service-cidr modification are not supported.
{{< /note >}}

### Create a dual-stack cluster

A simple command with `podCidr` and `serviceCidr` flags to create a dual-stack cluster via `kubeadm init`, it like below:

```shell
kubeadm init --feature-gates IPv6DualStack=true --pod-network-cidr=172.30.0.0/16,fefe:ffff:0::/48 --service-cidr=172.31.0.0/16,fefe:ffff:1::/108
```

To make things more clear, here is an example kubeadm [configuration file](https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2) `kubeadm-config.yml` for the dual-stack control plane node.

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
featureGates:
  IPv6DualStack: true
networking:
  podSubnet: 172.30.0.0/16,fefe:ffff:0::/48
  serviceSubnet: 172.31.0.0/16,fefe:ffff:1::/108
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "10.100.0.1"
  bindPort: 6443
```

`advertiseAddress` in InitConfiguration specifies the IP address the API Server will advertise it's listening on. It equals to `--apiserver-advertise-address` flag of `kubeadm init`.

Run kubeadm to initiate the dual-stack control plane node.

```shell
kubeadm init --config=kubeadm-config.yml
```

Currently, `kube-controller-manager` flags `--node-cidr-mask-size-ipv4|--node-cidr-mask-size-ipv6` are setting with the default value. See [enable IPv4/IPv6 dual stack](/docs/concepts/services-networking/dual-stack#enable-ipv4ipv6-dual-stack).

There is a limitation here, `--apiserver-advertise-address` flag doesn't support dual-stack.

### Join a node to dual-stack cluster

Before joining a node, make sure that the node has IPv6 routable network interface and allows IPv6 forwarding.


Here is an example kubeadm [configuration file](https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2) `kubeadm-config.yml` for joining a worker node to the cluster.

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
discovery:
  bootstrapToken:
    apiServerEndpoint: 10.100.0.1:6443
    token: 0c0z4p.dnafh6vnmouus569
    caCertHashes: ["sha256:fcb3e956a6880c05fc9d09714424b827f57a6fdc8afc44497180905946527adf"]
```

Besides, here is an example kubeadm [configuration file](https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2) `kubeadm-config.yml` for joining another control plane node to the cluster.
```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
controlPlane:
  localAPIEndpoint:
    advertiseAddress: "10.100.0.2"
    bindPort: 6443
discovery:
  bootstrapToken:
    apiServerEndpoint: 10.100.0.1:6443
    token: 0c0z4p.dnafh6vnmouus569
    caCertHashes: ["sha256:fcb3e956a6880c05fc9d09714424b827f57a6fdc8afc44497180905946527adf"]
```

`advertiseAddress` in JoinConfiguration.controlPlane specifies the IP address the API Server will advertise it's listening on. It equals to `--apiserver-advertise-address` flag of `kubeadm join`.

```shell
kubeadm join --config=kubeadm-config.yml ...
```

### Create a single-stack cluster

In 1.21 the `IPv6DualStack` feature is Beta and the feature gate is defaulted to `true`. To disable the feature you must configure the feature gate to `false`. Note that once the feature is GA, the feature gate will be removed.

```shell
kubeadm init --feature-gates IPv6DualStack=false
```

To make things more clear, here is an example kubeadm [configuration file](https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2) `kubeadm-config.yml` for the single-stack control plane node.

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
featureGates:
  IPv6DualStack: false
networking:
  podSubnet: 172.30.0.0/16
  serviceSubnet: 172.31.0.0/16
```

## {{% heading "whatsnext" %}}


* [Validate IPv4/IPv6 dual-stack](/docs/tasks/network/validate-dual-stack) networking
