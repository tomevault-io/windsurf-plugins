---
trigger: always_on
description: These notes capture the working process for developing, testing, and releasing `pve-microvm`. They are intentionally generic: replace hostnames, node IPs, storage names, VMIDs, and template IDs with values from your own Proxmox VE cluster.
---

# pve-microvm Agent Notes

These notes capture the working process for developing, testing, and releasing `pve-microvm`. They are intentionally generic: replace hostnames, node IPs, storage names, VMIDs, and template IDs with values from your own Proxmox VE cluster.

## Project shape

`pve-microvm` is a Debian package that adds QEMU `microvm` machine type support to Proxmox VE by:

* Installing `PVE::QemuServer::MicroVM` as a delegated command builder.
* Patching PVE's `Machine.pm` and `QemuServer.pm` so `machine: microvm` is accepted and dispatched to the microVM builder.
* Shipping a minimal kernel/initrd pair for direct kernel boot.
* Adding CLI helpers, template creation from OCI images, filesystem sharing, vsock support, and web UI integration.

The patching model is invasive by design, so every change must be reversible, idempotent, and tested against the current PVE `qemu-server` package.

## Git and release workflow

* Work on `main` unless explicitly creating a long-running branch.
* Do not rebase published history.
* Keep commits outcome-focused: one logical fix or feature per commit.
* For release builds, update `debian/changelog`, tag `vX.Y.Z`, push the commit and tag, then let GitHub Actions publish assets.
* Installation instructions should not hardcode package versions. Prefer commands that resolve the latest release asset dynamically.
* After a release asset is published, test on one non-critical node first before pushing to the rest of the cluster.

Typical release sequence:

```bash
# after committing changelog and code changes
git tag -a v0.3.X -m "v0.3.X — short release description"
git push
git push origin v0.3.X

# wait for GitHub release assets, then on a test PVE node:
curl -sLO $(curl -s https://api.github.com/repos/rcarmo/pve-microvm/releases/latest \
  | grep browser_download_url | grep '.deb' | cut -d'"' -f4)
dpkg -i pve-microvm_*.deb
apt-get install -f
```

## Development checks before committing

Run the checks that are meaningful in the current environment:

* Shell syntax for scripts:

```bash
bash -n tools/pve-microvm-template
bash -n tools/pve-microvm-patch
bash -n kernel/build-kernel.sh
```

* Perl syntax for `MicroVM.pm` is best checked on a PVE host, not on a generic development machine, because it imports PVE Perl modules:

```bash
perl -c /usr/share/pve-microvm/MicroVM.pm
perl -e 'use PVE::QemuServer; use PVE::QemuServer::MicroVM; print "OK\n"'
```

* Verify patches are idempotent:

```bash
pve-microvm-patch status
pve-microvm-patch apply
pve-microvm-patch apply
pve-microvm-patch status
```

The second `apply` must not duplicate imports or delegation blocks.

## PVE node testing process

Use a staged rollout:

1. Pick a non-critical PVE node that has local storage and at least one disposable test VM/template.
2. Install the new `.deb`.
3. Restart `pvedaemon` and verify it still starts.
4. Create or restart a microVM.
5. Verify the QEMU command line uses `-M microvm` and virtio-only devices.
6. Verify the guest boots, mounts root as `/dev/vda`, has networking, and the QEMU guest agent responds if enabled.
7. Only then deploy to the remaining nodes.

Useful checks on a PVE node:

```bash
systemctl restart pvedaemon
systemctl is-active pvedaemon

qm config <vmid> | grep '^machine: microvm'
qm start <vmid>
PID=$(pgrep -f "kvm.*-id <vmid>")
tr '\0' '\n' < /proc/$PID/cmdline | grep -A1 '^-machine$'
tr '\0' '\n' < /proc/$PID/cmdline | grep -E 'virtio-blk|virtio-net|balloon|scsi-hd'

qm agent <vmid> ping
qm guest exec <vmid> -- bash -c 'hostname; df -h /; ip -brief addr'
```

Expected microVM signs:

* Machine line contains `microvm,x-option-roms=off,...,pcie=on`.
* Block devices are `virtio-blk-pci-non-transitional`, not `scsi-hd`.
* Network devices are virtio PCI devices.
* Root filesystem is mounted from `/dev/vda` for Linux guests.

## Testing existing microVM workloads

When upgrading a node that already runs microVMs:

* Record the VMIDs and expected services before stopping anything.
* Restart one microVM at a time unless the host itself must reboot.
* After restart, check the guest agent, root filesystem, and service health.
* For web services, test from both inside the guest and from the host/network.

Example pattern:

```bash
qm stop <vmid>; sleep 3; qm start <vmid>
sleep 15
qm agent <vmid> ping
qm guest exec <vmid> -- bash -c 'hostname; df -h /; systemctl --failed --no-pager'
```

For HTTP services:

```bash
IP=$(qm guest exec <vmid> -- bash -c 'hostname -I' \
  | python3 -c 'import json,sys; print(json.load(sys.stdin).get("out-data","").split()[0])')
curl -I http://$IP:<port>/
```

## Auto-start and boot-order hazards

`onboot: 1` VMs expose ordering bugs. If PVE starts VMs before `pve-microvm` patches are active, `machine: microvm` may be rejected or stripped during config parsing/migration, and the VM can start as a standard PC. The disk will then appear as `/dev/sda` instead of `/dev/vda`, which looks like a lost root filesystem.

Required safeguards:

* `pve-microvm-early.service` must be enabled on every node.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcarmo/pve-microvm](https://github.com/rcarmo/pve-microvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
