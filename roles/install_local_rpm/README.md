
# install_local_rpm Role

This role installs local RPM packages downloaded from HTTP/HTTPS file servers. It supports optional checksum validation, automatic hash algorithm detection, and cleans up downloaded files after installation.

## Role Purpose

The role downloads an RPM, optionally verifies its checksum, installs it via the system package manager, and removes temporary installation files. It is fully idempotent and skips all tasks if the package is already installed.

## Variables

`local_rpm_to_install` must be defined as a dictionary in inventory or group_vars:

```yaml
local_rpm_to_install:
  bcl-convert:   # key used to detect package
    download_url: "http://server/software/bcl-convert-4.4.6-2.el8.x86_64.rpm"
    download_path: "/tmp"
    rpm_file: "bcl-convert-4.4.6-2.el8.x86_64.rpm"      # optional
    checksum_file_url: "http://server/software/software.sha256"  # optional
  petasuite-proxy:
    download_url: "http://server/software/other-proxy-1.4.7-1.x86_64.rpm"
    download_path: "/tmp"

```

### Variable Fields

- **download_url**: URL of the RPM file.
- **download_path**: Local directory where RPM will be saved.
- **rpm_file**: RPM filename. If not provided, the role uses the basename of `download_url`.
- **checksum_file_url**: URL of a checksum file. If omitted, checksum validation is skipped.

## Package Detection

The role determines whether the package is already installed using `package_facts` and searches for any installed package whose name begins with the key (e.g., `bcl-convert`). If detected, all installation steps are skipped.

## Checksum Verification

If `checksum_file_url` is provided, the role:

1. Downloads the checksum file.
2. Extracts the checksum matching the RPM filename.
3. Automatically detects the hash algorithm based on checksum length.
4. Verifies the RPM before installation.

Checksum file example:

```
743cf1f241a2ed4daad3796a20992f81  ./other-proxy-1.4.7-1.x86_64.rpm
```

## Cleanup

After successful installation, the role removes:

- The downloaded RPM file.
- The checksum file (if used).



