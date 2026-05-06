---
trigger: always_on
description: Guidelines for handling DEB, RPM, Arch, and Alpine packages
---


# Package Processing Rules

## General Package Handling

### Architecture Support
- Support **x86_64** (amd64) and **ARM64** (aarch64) architectures
- Use correct architecture identifiers per format:
  - DEB: `amd64`, `arm64`
  - RPM: `x86_64`, `aarch64`
  - Arch: `x86_64`, `aarch64`
  - Alpine: `x86_64`, `aarch64`
- Create separate repository paths per architecture
- Test packages on both architectures when possible

### Package Validation
- Verify package format before processing
- Check package signatures when present
- Validate package metadata
- Reject corrupted or invalid packages
- Check for required fields in package metadata
- Validate version numbers and naming conventions

### Error Handling
- Provide clear error messages for validation failures
- Log all package processing errors
- Clean up partial uploads on failure
- Return appropriate HTTP status codes
- Don't leave repository in inconsistent state

## DEB (Debian/Ubuntu) Packages

### Package Structure
- Understand control file format
- Parse control fields correctly
- Handle multi-line control fields
- Support all required control fields:
  - Package name
  - Version
  - Architecture
  - Maintainer
  - Description
  - Dependencies

### Repository Structure
```
deb/
├── dists/
│   └── stable/
│       └── main/
│           ├── binary-amd64/
│           │   └── Packages(.gz)
│           ├── binary-arm64/
│           │   └── Packages(.gz)
│           ├── Release
│           └── Release.gpg
└── pool/
    └── main/
        └── <first-letter>/
            └── <package-name>/
                └── <package>.deb
```

### Metadata Files
- **Packages**: List of all packages with metadata
- **Packages.gz**: Compressed version
- **Release**: Repository metadata with checksums
- **Release.gpg**: GPG signature of Release file
- **InRelease**: Combined signed Release file (optional)

### APT Repository Generation
- Create Packages file with proper format
- Include all required fields in Packages file
- Generate correct checksums (MD5, SHA1, SHA256)
- Compress Packages file with gzip
- Sign Release file with GPG
- Update Release file with correct checksums
- Support component structure (main, contrib, non-free)

### Best Practices
- Follow Debian package naming: `<name>_<version>_<arch>.deb`
- Store packages in pool directory by first letter
- Maintain consistent directory structure
- Support multiple distributions if needed
- Handle package dependencies correctly

## RPM (RHEL/Fedora/Rocky) Packages

### Repository Structure
```
rpm/
├── x86_64/
│   ├── Packages/
│   │   └── *.rpm
│   └── repodata/
│       ├── repomd.xml
│       ├── repomd.xml.asc
│       ├── primary.xml.gz
│       ├── filelists.xml.gz
│       └── other.xml.gz
└── aarch64/
    └── (same structure)
```

### Metadata Files
- **repomd.xml**: Repository metadata index
- **repomd.xml.asc**: GPG signature
- **primary.xml.gz**: Package metadata
- **filelists.xml.gz**: File listings
- **other.xml.gz**: Additional package info

### YUM/DNF Repository Generation
- Use `createrepo_c` for repository creation
- Generate repomd.xml with checksums
- Create primary, filelists, and other metadata
- Sign repomd.xml with GPG
- Update repository atomically
- Support repository groups (optional)

### Best Practices
- Follow RPM naming: `<name>-<version>-<release>.<arch>.rpm`
- Parse RPM headers correctly
- Handle epoch versions properly
- Support weak dependencies (Recommends, Suggests)
- Test with both YUM and DNF clients

### RPM-Specific Considerations
- Handle %pre, %post, %preun, %postun scripts
- Check for file conflicts
- Validate RPM signature if present
- Handle package obsoletes correctly

## Arch Linux Packages

### Repository Structure
```
arch/
├── x86_64/
│   ├── *.pkg.tar.zst
│   ├── custom.db
│   ├── custom.db.tar.gz
│   └── custom.files
└── aarch64/
    └── (same structure)
```

### Metadata Files
- **custom.db**: Package database (symlink to custom.db.tar.gz)
- **custom.db.tar.gz**: Compressed package database
- **custom.files**: File listing database
- **.PKGINFO**: Package metadata inside package

### Pacman Repository Generation
- Extract .PKGINFO from package
- Create database entries for each package
- Update *.db and *.files databases
- Create symlinks for database versioning
- Maintain database consistency

### Best Practices
- Follow Arch naming: `<name>-<version>-<release>-<arch>.pkg.tar.zst`
- Handle package compression formats (.zst, .xz, .gz)
- Parse PKGINFO correctly
- Support split packages
- Handle package groups
- Validate package signatures

### Arch-Specific Considerations
- Support makedepends and checkdepends
- Handle optdepends correctly
- Support provides/conflicts/replaces
- Maintain proper database format

## Alpine Linux Packages

### Repository Structure
```
alpine/
├── v3.19/
│   └── main/
│       ├── x86_64/
│       │   ├── *.apk
│       │   ├── APKINDEX.tar.gz
│       │   └── APKINDEX.tar.gz.asc
│       └── aarch64/
│           └── (same structure)
```

### Metadata Files
- **APKINDEX.tar.gz**: Package index
- **APKINDEX.tar.gz.asc**: GPG signature (detached)
- **.PKGINFO**: Package metadata inside .apk

### APK Repository Generation
- Extract package metadata from .apk files
- Create APKINDEX with all packages
- Compress APKINDEX with gzip

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/package-repository-server](https://github.com/quinnjr/package-repository-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
