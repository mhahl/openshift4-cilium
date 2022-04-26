# Setup

1. Configure your vars

```yaml
---
assisted_service_api: "https://assisted-ui.apps.k8s.hahl.id.au"
pull_secret: ""
ssh_public_key: ""
cilium:
  git: https://github.com/cilium/cilium-olm
  branch: "master"
  version: "v1.10.4"
cluster:
  name: "infra"
  additional_ntp_sources: "ntp1.anu.edu.au"
  base_dns_domain: "hahl.id.au"
  email_domain: "hahl.id.au"
  ocp_release_image: "quay.io/openshift-release-dev/ocp-release:4.8.14-x86_64"
  openshift_version: "4.8.14"
  cluster_host_network: "172.16.17.96/27"
  cluster_network_cidr: "10.128.0.0/14"
  cluster_network_host_prefix: 23
  cluster_machine_network_cidr: "172.16.17.96/27"
  service_network_cidr: "172.30.0.0/16"
```

2. Run the playbook.

```
ansible-playbook cluster.yaml
```

## Optional
Cilium Config example without using sha256sum.

* Foreach deployment, add section for image to config.
* Add `useDigest: false` to ensure it will use a tag instead.
* Read https://github.com/cilium/cilium/blob/master/install/kubernetes/cilium/values.yaml for a list of values.

```yaml
---
apiVersion: cilium.io/v1alpha1
kind: CiliumConfig
metadata:
  name: cilium
  namespace: cilium
spec:
  ipam:
    mode: "cluster-pool"
    operator:
      clusterPoolIPv4PodCIDR: "{{cluster.cluster_network_cidr}}"
      clusterPoolIPv4MaskSize: "{{cluster.cluster_network_host_prefix}}"
  nativeRoutingCIDR: "{{cluster.cluster_network_cidr}}"
  cni:
    binPath: "/var/lib/cni/bin"
    confPath: "/var/run/multus/cni/net.d"
  prometheus:
    serviceMonitor: {enabled: false}
  hubble:
    tls: {enabled: false}
   operator:
    image:
      pullPolicy: IfNotPresent
      repository: harbor.apps.infra.hahl.id.au/cilium/operator
      tag: v1.10.4
      useDigest: false # Imortant
   clustermesh:
    apiserver:
      etcd:
        image:
          pullPolicy: IfNotPresent
          repository: harbor.apps.infra.hahl.id.au/coreos/etcd
          tag: 3.4.13
      image:
        pullPolicy: IfNotPresent
        repository: harbor.apps.infra.hahl.id.au/coreos/etcd
        tag: v1.10.4
        useDigest: false # Imortant!

```
