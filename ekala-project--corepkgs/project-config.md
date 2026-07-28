---
trigger: always_on
description: This document provides a quick reference for AI agents working with the cross-platform service management system in core-pkgs.
---

# Agent Guide for Cross-Service Interface

This document provides a quick reference for AI agents working with the cross-platform service management system in core-pkgs.

## Overview

The `services/` directory provides a **unified service interface** that works across multiple service managers:
- **systemd** (Linux - user & system services)
- **launchd** (macOS - user agents & system daemons)
- **runit** (Linux/containers - simple supervision)
- **BSD rc.d** (FreeBSD/OpenBSD/NetBSD/DragonFly)

Services are defined once using common options, then automatically translated to platform-specific formats.

## Service Definition Structure

```nix
{
  services.my-service = {
    # Common options (work everywhere)
    enable = true;
    description = "My Service";
    command = "${pkgs.python3}/bin/python3";
    args = [ "-m" "http.server" "8080" ];
    user = "myuser";
    workingDirectory = "/var/lib/myservice";
    environment = { PORT = "8080"; };
    restartPolicy = "always";  # or "on-failure", "never"
    preStart = "echo 'Starting...'";
    postStop = "echo 'Stopped'";

    # Platform-specific extensions (optional)
    systemd = {
      wantedBy = [ "multi-user.target" ];
      after = [ "network.target" ];
      serviceConfig = {
        PrivateTmp = true;
        NoNewPrivileges = true;
      };
    };

    launchd = {
      label = "com.example.my-service";
      keepAlive = true;
      runAtLoad = true;
    };

    runit = {
      logScript = ''
        #!/bin/sh
        exec svlogd -tt /var/log/my-service
      '';
    };

    rcd = {
      variant = "freebsd";  # or "openbsd", "netbsd", "dragonfly"
      rcRequire = [ "NETWORKING" ];
    };
  };
}
```

## Integration with ekaos

ekaos modules use the same service interface. Services are defined at `services.*` and automatically translated to systemd units.

**Example ekaos module:**
```nix
{ config, pkgs, ... }:

{
  services.my-app = {
    enable = true;
    command = "${pkgs.myapp}/bin/myapp";
    restartPolicy = "always";

    systemd = {
      wantedBy = [ "multi-user.target" ];
    };
  };
}
```

**Location:** Service modules are in `ekaos/modules/services/`

## ekaos Service Module Conventions

When creating ekaos service modules:

1. **Standard service options** - Always include:
   - `enable` (bool)
   - `description` (str, default provided)
   - `command` (str, internal/automatic)
   - `args` (list of str, internal/automatic)
   - `user` (str, defaults to appropriate user)
   - `restartPolicy` (str, usually "always")
   - `systemd` (attrset for systemd-specific options)

2. **Application-specific settings** - Use `settings` submodule:
   ```nix
   services.openssh.settings = {
     ports = 22;
     permitRootLogin = "prohibit-password";
     passwordAuthentication = true;
   };
   ```

3. **Service definition** - Set in config section:
   ```nix
   config = mkIf cfg.enable {
     services.openssh = {
       command = "${pkgs.openssh}/bin/sshd";
       args = [ "-D" "-f" "${sshdConfig}" ];
       user = "root";
       restartPolicy = "always";

       systemd = {
         after = [ "network.target" ];
         wantedBy = [ "multi-user.target" ];
       };
     };
   };
   ```

## Porting Services from nixpkgs

When porting service modules from nixpkgs to ekaos, refactor them to use the reusable services interface.

### Refactoring Pattern

**ekaos reusable style:**
```nix
{ config, lib, pkgs, ... }:

let
  cfg = config.services.myservice;
in

{
  options.services.myservice = {
    enable = mkOption {
      type = types.bool;
      default = false;
      description = "Whether to enable My Service.";
    };

    description = mkOption {
      type = types.str;
      default = "My Service";
      description = "Service description";
    };

    command = mkOption {
      type = types.str;
      internal = true;
      description = "Command to run (set automatically)";
    };

    args = mkOption {
      type = types.listOf types.str;
      internal = true;
      default = [];
      description = "Command arguments (set automatically)";
    };

    user = mkOption {
      type = types.str;
      default = "myservice";
      description = "User to run service as";
    };

    restartPolicy = mkOption {
      type = types.str;
      default = "always";
      description = "Restart policy";
    };

    systemd = mkOption {
      type = types.attrsOf types.anything;
      default = {};
      description = "Systemd-specific options";
    };

    settings = mkOption {
      type = types.submodule {
        options = {
          port = mkOption {
            type = types.port;
            default = 8080;
            description = "Port to listen on";
          };
          # Other application-specific options
        };
      };
      default = {};
      description = "Application-specific configuration";
    };
  };

  config = mkIf cfg.enable {
    # Define using cross-platform interface
    services.myservice = {
      command = "${pkgs.myservice}/bin/myservice";
      args = [ "--port" (toString cfg.settings.port) ];
      user = cfg.user;
      restartPolicy = "always";

      systemd = {
        after = [ "network.target" ];
        wantedBy = [ "multi-user.target" ];

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekala-project/corepkgs](https://github.com/ekala-project/corepkgs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
