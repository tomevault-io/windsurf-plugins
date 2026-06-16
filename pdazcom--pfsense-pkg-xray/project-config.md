---
trigger: always_on
description: A native pfSense package that integrates **Xray-core** (proxy engine) with **tun2socks** to provide
---

# pfSense Xray Plugin — Project Context

## What We're Building

A native pfSense package that integrates **Xray-core** (proxy engine) with **tun2socks** to provide
GUI-managed VPN tunnels with support for VLESS+Reality and other protocols.

The plugin creates TUN interfaces that pfSense recognizes as standard interfaces — allowing users to
route traffic through Xray using pfSense's native tools: Aliases, Firewall Rules, Policy-Based Routing.

## Origin

Ported from the OPNsense plugin at `/Users/kostya/PhpstormProjects/os-xray`.
Key logic (config generation, VLESS link parser, process management, watchdog) is adapted from there.
The framework layer (config storage, GUI, service hooks) is rewritten for pfSense.

## Target Platform

- pfSense CE 2.7.x / 2.8.1
- FreeBSD 14.x
- PHP 8.2

## Architecture

```
pfSense-pkg-xray/
├── pkg/
│   └── xray.xml                      # Package manifest (menus, hooks, install/deinstall)
├── files/
│   ├── usr/local/www/xray/
│   │   ├── xray_instances.php        # Instance list with per-instance status
│   │   ├── xray_edit.php             # Create/edit instance (VLESS link import + manual)
│   │   ├── xray_settings.php         # Global settings (enable, watchdog)
│   │   ├── xray_diagnostics.php      # TUN stats, logs, connection test
│   │   └── xray_ajax.php             # AJAX: start/stop/status/import/validate/logs
│   ├── usr/local/pkg/
│   │   ├── xray.inc                  # Core: config read/write, TUN registration, hooks
│   │   ├── xray_validate.inc         # Input validation before save
│   │   └── xray/includes/
│   │       ├── xray.inc              # Symlink target for package manifest include path
│   │       └── xray_foot.inc         # Footer include (empty)
│   ├── usr/local/scripts/xray/
│   │   ├── xray-service-control.php  # Process management: start/stop/restart/status/validate
│   │   ├── xray-watchdog.php         # Crash recovery daemon (cron every minute)
│   │   ├── xray-ifstats.php          # TUN interface statistics + ping
│   │   └── xray-testconnect.php      # SOCKS5 connectivity test
│   └── usr/local/etc/rc.d/
│       └── xray.sh                   # rc script: start/stop/status all instances
└── install.sh                        # Downloads xray-core + tun2socks binaries
```

## Config Storage

pfSense stores package config in `$config['installedpackages']`.

```php
$config['installedpackages']['xray']['config'][0] = [
    'enabled'          => 'on' | '',
    'watchdog_enabled' => 'on' | '',
];

$config['installedpackages']['xrayinstances']['config'][] = [
    'uuid'                => 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
    'name'                => 'My VPN',
    'config_mode'         => 'wizard' | 'custom',
    'custom_config'       => '',            // raw JSON if config_mode=custom
    'server_address'      => '1.2.3.4',
    'server_port'         => '443',
    'vless_uuid'          => 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
    'flow'                => 'xtls-rprx-vision' | 'none',
    'reality_sni'         => 'www.cloudflare.com',
    'reality_pubkey'      => '',
    'reality_shortid'     => '',
    'reality_fingerprint' => 'chrome' | 'firefox' | 'safari' | 'edge' | 'random',
    'socks5_listen'       => '127.0.0.1',
    'socks5_port'         => '10808',
    'tun_interface'       => 'proxytun0',
    'mtu'                 => '1500',
    'loglevel'            => 'debug' | 'info' | 'warning' | 'error' | 'none',
    'bypass_networks'     => '10.0.0.0/8,172.16.0.0/12,192.168.0.0/16',
];
```

## Key Design Decisions

### Multi-instance
Each instance has a UUID. All runtime files are named by UUID:
- `/var/run/xray_core_{uuid}.pid`
- `/var/run/tun2socks_{uuid}.pid`
- `/usr/local/etc/xray-core/config-{uuid}.json`
- `/usr/local/tun2socks/config-{uuid}.yaml`
- `/var/run/xray_stopped_{uuid}.flag`   ← intentional stop marker (watchdog skip)
- `/var/run/xray_start_{uuid}.lock`     ← per-instance start lock (flock)

### Binaries
- `/usr/local/bin/xray-core`
- `/usr/local/tun2socks/tun2socks`

### Constants (defined in `xray-service-control.php`)
```php
define('XRAY_BIN',          '/usr/local/bin/xray-core');
define('XRAY_CONF_DIR',     '/usr/local/etc/xray-core');
define('T2S_BIN',           '/usr/local/tun2socks/tun2socks');
define('T2S_CONF_DIR',      '/usr/local/tun2socks');
define('XRAY_DAEMON_LOG',   '/var/log/xray-core.log');
define('XRAY_VERSION_FILE', '/usr/local/etc/xray-core/version.txt');
define('XRAY_CTRL',         '/usr/local/scripts/xray/xray-service-control.php');
define('WATCHDOG_LOG',      '/var/log/xray-watchdog.log');
```

### TUN IP Assignment
`xray_tun_ip_for_uuid()` derives a deterministic `/30` subnet from the UUID using CRC32.
This ensures the TUN interface always gets the same IP across restarts without extra config.

### pfSense Routing Integration
TUN interface is registered in `$config['interfaces']` so it appears in pfSense UI.
Users route traffic via: Firewall → Rules → set Gateway to the Xray interface.
Aliases work natively — no changes needed to xray-core routing config.

### Protocol extensibility
`config_mode = 'wizard'` uses the built-in VLESS+Reality form fields.
`config_mode = 'custom'` accepts raw xray-core JSON — supports any protocol/transport.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdazcom/pfSense-pkg-xray](https://github.com/pdazcom/pfSense-pkg-xray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
