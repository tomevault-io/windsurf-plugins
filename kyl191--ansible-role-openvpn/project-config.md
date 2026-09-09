---
trigger: always_on
description: Ansible role to install and configure OpenVPN servers. Created 2015, 1M+ downloads on Ansible Galaxy.
---

# kyl191.openvpn — Development Reference

Ansible role to install and configure OpenVPN servers. Created 2015, 1M+ downloads on Ansible Galaxy.

- **Current version:** 3.1.0
- **Minimum Ansible:** 2.13 (ansible-core)
- **Minimum OpenVPN:** 2.5
- **License:** MIT

## Directory Structure

```
tasks/                  # Modular task files
  main.yml              # Orchestrator — includes all other tasks in order
  validate.yml           # Fail-fast checks (CN length; both-address-families-empty) — always runs first
  install.yml           # Package installation (EPEL, openvpn, openssl, LDAP plugin)
  config.yml            # Server config, scripts, logrotate, CCD, systemd service
  server_keys.yml       # CA + server cert generation or import
  server_keys_crl.yml   # CRL setup (systemd timer, revoke script)
  client_keys.yml       # Client cert generation + .ovpn file creation + fetch
  revocation.yml        # Cert revocation logic
  cert_sync_detection.yml  # Diff existing certs vs openvpn_clients list
  uninstall.yml         # Full removal (stops early with meta: end_play)
  selinux.yml           # SELinux port management (semanage preferred)
  firewall/
    firewall.yml        # Dispatcher: resolves __openvpn_firewall_backend once, then includes
                         # the matching backend file; also hosts the IPv4/IPv6 default-route guards
    firewalld.yml
    iptables.yml
    ufw.yml

defaults/main/          # Role defaults (split by concern)
  openvpn.yml           # OpenVPN config: ports, ciphers, TLS, DNS, network
  role.yml              # Role behavior: dirs, fetch, firewall, CI flag, LDAP toggle
  packaging.yml         # Package names (overrideable per distro)
  ldap.yml              # LDAP auth defaults
  logrotate.yml         # Log rotation defaults

filter_plugins/
  netmask_to_cidr.py    # Dotted-decimal netmask -> CIDR prefix length (stdlib ipaddress only,
                         # no netaddr/ansible.utils dependency)

vars/                   # OS-specific overrides (loaded via with_first_found)
  os/                   # Checked first: distribution+version, then distribution, then family
  empty.yml             # Fallback (intentionally empty)
  RedHat.yml            # iptables save command override
  Debian.yml            # iptables save command override
  FreeBSD.yml           # No firewall mgmt, alternate config path
  Solaris.yml           # No logrotate, no LDAP

templates/
  server.conf.j2        # OpenVPN server config
  client.ovpn.j2        # Embedded-cert client config file
  ca.conf.j2            # OpenSSL CA config for CRL management
  ldap.conf.j2          # openvpn-auth-ldap config
  client_ccd.j2         # Per-client server-pushed options
  revoke.sh.j2          # CRL generation + cert revocation script
  crl-cron.sh.j2        # Checks CRL expiry, calls revoke.sh
  openvpn_logrotate.conf.j2
  openvpn-crl-refresh.service.j2
  selinux_module.te.j2  # Legacy SELinux TE module (being removed)

files/
  openssl-ca.ext        # X509 CA extensions
  openssl-server.ext    # X509 server extensions (TLS server EKU)
  openssl-client.ext    # X509 client extensions (TLS client EKU)
  dh.pem                # Pre-generated 2048-bit DH params (not secret)
  openvpn-server-override.conf  # systemd service override
  openvpn-crl-refresh.timer     # systemd timer (daily CRL check)

tests/
  test.yml              # Main test playbook (localhost, openvpn_ci_build: true)
  revocation-test.yml   # CRL revocation test
  ec2.yml               # E2E test against real AWS EC2 instances
  e2e_config.toml       # AWS region/profile + [terraform] scenario var_files for run_e2e.py
  run_e2e.py            # Thin CLI entry point - delegates to e2e/orchestrator.py:main
  e2e/                  # E2E test implementation, split by concern
    models.py            # Status/Phase enums, InstanceInfo dataclass
    config.py            # TOML + CLI settings (RunSettings)
    aws.py                # EC2 instance discovery
    ssh.py                # SSH readiness polling + OS/user detection
    provisioning.py       # ansible-playbook per instance, output to a durable log file
    verification.py       # OpenVPN client connect + IPv4/IPv6 routing check
    terraform.py          # apply/destroy for the shared EC2 test infrastructure
    report.py             # Markdown report generation
    display.py            # Logging setup + Live per-instance status board (Rich)
    orchestrator.py        # Top-level flow: main() and run_scenario()
    ADR.md                 # Architecture decision records for this package's design
  *.Dockerfile          # Per-distro systemd container images
```

## Key Task Flow (`tasks/main.yml`)

1. Validate variables (`validate.yml`) — CN length, and (once the stashed work lands) that at
   least one address family is enabled. Tagged `always`, runs before OS vars are even loaded.
2. Load OS-specific vars: `distribution+version` → `distribution` → `os_family` → `empty.yml`
3. Uninstall if `openvpn_uninstall` is truthy (exits early)
4. Install packages (`install.yml`)
5. Generate or import server keys (`server_keys.yml`)
6. Enable sysctl IP forwarding (skipped when `openvpn_ci_build` or `openvpn_manage_sysctl: false`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyl191/ansible-role-openvpn](https://github.com/kyl191/ansible-role-openvpn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
