---
trigger: always_on
description: There are three ways of passing information to plugins using the Container Network Interface (CNI), none of which require the [spec](SPEC.md) to be updated. These are
---

# Extension conventions                                                        
                                                                                 
There are three ways of passing information to plugins using the Container Network Interface (CNI), none of which require the [spec](SPEC.md) to be updated. These are 
- plugin specific fields in the JSON config
- `args` field in the JSON config
- `CNI_ARGS` environment variable 

This document aims to provide guidance on which method should be used and to provide a convention for how common information should be passed.
Establishing these conventions allows plugins to work across multiple runtimes. This helps both plugins and the runtimes.

## Plugins
* Plugin authors should aim to support these conventions where it makes sense for their plugin. This means they are more likely to "just work" with a wider range of runtimes.
* Plugins should accept arguments according to these conventions if they implement the same basic functionality as other plugins. If plugins have shared functionality that isn't coverered by these conventions then a PR should be opened against this document.

## Runtimes
* Runtime authors should follow these conventions if they want to pass additional information to plugins. This will allow the extra information to be consumed by the widest range of plugins.
* These conventions serve as an abstraction for the runtime. For example, port forwarding is highly implementation specific, but users should be able to select the plugin of their choice without changing the runtime.

# Current conventions
Additional conventions can be created by creating PRs which modify this document.

## Dynamic Plugin specific fields (Capabilities / Runtime Configuration)
[Plugin specific fields](https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration) formed part of the original CNI spec and have been present since the initial release.
> Plugins may define additional fields that they accept and may generate an error if called with unknown fields. The exception to this is the args field may be used to pass arbitrary data which may be ignored by plugins.

A plugin can define any additional fields it needs to work properly. It should return an error if it can't act on fields that were expected or where the field values were malformed.

This method of passing information to a plugin is recommended when the following conditions hold:
* The configuration has specific meaning to the plugin (i.e. it's not just general meta data)
* the plugin is expected to act on the configuration or return an error if it can't

Dynamic information (i.e. data that a runtime fills out) should be placed in a `runtimeConfig` section. Plugins can request
that the runtime insert this dynamic configuration by explicitly listing their `capabilities` in the network configuration.

For example, the configuration for a port mapping plugin might look like this to an operator (it should be included as part of a [network configuration list](https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration-lists).
```json
{
  "name" : "ExamplePlugin",
  "type" : "port-mapper",
  "capabilities": {"portMappings": true}
}
```

But the runtime would fill in the mappings so the plugin itself would receive something like this.
```json
{
  "name" : "ExamplePlugin",
  "type" : "port-mapper",
  "runtimeConfig": {
    "portMappings": [
      {"hostPort": 8080, "containerPort": 80, "protocol": "tcp"}
    ]
  }
}
```

### Well-known Capabilities
| Area  | Purpose | Capability | Spec and Example | Runtime implementations | Plugin Implementations |
| ----- | ------- | -----------| ---------------- | ----------------------- | ---------------------  |
| port mappings | Pass mapping from ports on the host to ports in the container network namespace. | `portMappings` | A list of portmapping entries.<br/>  <pre>[<br/>  { "hostPort": 8080, "containerPort": 80, "protocol": "tcp" },<br />  { "hostPort": 8000, "containerPort": 8001, "protocol": "udp" }<br />  ]<br /></pre> | kubernetes | CNI `portmap` plugin |
| ip ranges | Dynamically configure the IP range(s) for address allocation. Runtimes that manage IP pools, but not individual IP addresses, can pass these to plugins. | `ipRanges` | The same as the `ranges` key for `host-local` - a list of lists of subnets. The outer list is the number of IPs to allocate, and the inner list is a pool of subnets for each allocation. <br/><pre>[<br/> [<br/>  { "subnet": "10.1.2.0/24", "rangeStart": "10.1.2.3", "rangeEnd": 10.1.2.99", "gateway": "10.1.2.254" } <br/>  ]<br/>]</pre> | none | CNI `host-local` plugin |
| bandwidth limits | Dynamically configure interface bandwidth limits | `bandwidth` | Desired bandwidth limits. Rates are in bits per second, burst values are in bits. <pre> { "ingressRate": 2048, "ingressBurst": 1600, "egressRate": 4096, "egressBurst": 1600 } </pre> | none | CNI `bandwidth` plugin |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudnativer/kube-ipam](https://github.com/cloudnativer/kube-ipam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
