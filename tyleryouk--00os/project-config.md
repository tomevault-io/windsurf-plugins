---
trigger: always_on
description: * 00OS Permissions System
---

pt
/**
 * 00OS Permissions System
 * 
 * Manages access control for processes, ensuring they only access
 * resources and perform actions they're authorized to do.
 */

// Permission levels
const PermissionLevels = {
  BASIC: 'basic',           // Basic operations, no system access
  FILE_READ: 'file-read',   // Read access to files
  FILE_WRITE: 'file-write', // Write access to files
  SYSTEM_READ: 'system-read', // Read system configuration
  SYSTEM_WRITE: 'system-write', // Modify system configuration
  NETWORK: 'network',       // Network access
  EXECUTION: 'execution',   // Execute other processes
  ADMIN: 'admin'            // Administrative privileges
};

// Default permission sets
const PermissionSets = {
  // Minimal permissions
  MINIMAL: [
    PermissionLevels.BASIC
  ],
  
  // File operations
  FILE_OPS: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.FILE_WRITE
  ],
  
  // System monitoring
  SYSTEM_MONITOR: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.SYSTEM_READ
  ],
  
  // System administration
  SYSTEM_ADMIN: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.FILE_WRITE,
    PermissionLevels.SYSTEM_READ,
    PermissionLevels.SYSTEM_WRITE
  ],
  
  // Full access (00reaper)
  FULL_ACCESS: [
    PermissionLevels.BASIC,
    PermissionLevels.FILE_READ,
    PermissionLevels.FILE_WRITE,
    PermissionLevels.SYSTEM_READ,
    PermissionLevels.SYSTEM_WRITE,
    PermissionLevels.NETWORK,
    PermissionLevels.EXECUTION,
    PermissionLevels.ADMIN
  ]
};

// Process identity to permission mapping
const IdentityPermissions = {
  // 00reaper has full access
  '00reaper': PermissionSets.FULL_ACCESS
};

// Path-based permission rules
const PathPermissionRules = [
  {
    pattern: /^\/00os\//,
    required: [PermissionLevels.ADMIN],
    description: 'Operating system files'
  },
  {
    pattern: /^\/00reaper\//,
    required: [PermissionLevels.ADMIN],
    description: 'System administrator files'
  },
  {
    pattern: /\.(md|js|json|txt|css|html)$/,
    required: [PermissionLevels.FILE_READ],
    description: 'Common readable files'
  },
  {
    pattern: /\.cursor\/rules/,
    required: [PermissionLevels.ADMIN],
    description: 'Cursor rules directory'
  }
];

// Current active identity
let currentIdentity = '00reaper';

/**
 * Check if requested permissions are granted for the current identity
 * @param {Array} requestedPermissions - Permissions requested by process
 * @returns {Object} Permission check result
 */
function checkPermissions(requestedPermissions = [PermissionLevels.BASIC]) {
  const availablePermissions = IdentityPermissions[currentIdentity] || PermissionSets.MINIMAL;
  
  // Check each requested permission
  for (const permission of requestedPermissions) {
    if (!availablePermissions.includes(permission)) {
      return {
        granted: false,
        message: `Permission '${permission}' denied for identity '${currentIdentity}'`
      };
    }
  }
  
  return {
    granted: true,
    identity: currentIdentity,
    permissions: availablePermissions
  };
}

/**
 * Check if path access is permitted
 * @param {string} path - Path to check
 * @param {string} operation - Operation type (read, write)
 * @returns {Object} Permission check result
 */
function checkPathPermission(path, operation = 'read') {
  const identityPermissions = IdentityPermissions[currentIdentity] || PermissionSets.MINIMAL;
  
  // Admin always has access
  if (identityPermissions.includes(PermissionLevels.ADMIN)) {
    return {
      granted: true,
      identity: currentIdentity
    };
  }
  
  // Check against path rules
  for (const rule of PathPermissionRules) {
    if (rule.pattern.test(path)) {
      // Check if identity has all required permissions
      const missingPermissions = rule.required.filter(
        permission => !identityPermissions.includes(permission)
      );
      
      if (missingPermissions.length > 0) {
        return {
          granted: false,
          message: `Access to ${path} requires permissions: ${missingPermissions.join(', ')}`,
          requiredPermissions: rule.required,
          description: rule.description
        };
      }
    }
  }
  
  // Check basic file operation permissions
  if (operation === 'read' && !identityPermissions.includes(PermissionLevels.FILE_READ)) {
    return {
      granted: false,
      message: `File read permission required for ${path}`
    };
  }
  
  if (operation === 'write' && !identityPermissions.includes(PermissionLevels.FILE_WRITE)) {
    return {
      granted: false,
      message: `File write permission required for ${path}`
    };
  }
  
  // If no rules match and basic permissions are met, access is granted
  return {
    granted: true,
    identity: currentIdentity
  };
}

/**
 * Set the active identity
 * @param {string} identity - Identity to set
 * @returns {Object} Result of identity change
 */
function setIdentity(identity) {
  // Check if identity exists
  if (!IdentityPermissions[identity]) {
    return {
      success: false,
      message: `Unknown identity: ${identity}`
    };
  }
  
  // Set new identity
  currentIdentity = identity;
  
  return {
    success: true,
    identity: currentIdentity,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tyleryouk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
