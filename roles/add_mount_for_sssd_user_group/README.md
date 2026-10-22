# add_mount_for_sssd_user_group

This role dynamically creates per-user mounts for members of specified SSSD/Active Directory groups.

The role:

1. Queries an SSSD group using `getent group`.
2. Extracts all users from the group membership.
3. Validates each user exists on the host and has a valid UID.
4. Optionally creates user directories on the remote share.
5. Generates a mount definition for each user.
6. Uses the `custom_mounts` role to configure the mounts.

This is particularly useful for environments where AD group membership controls access to shared storage and each user requires their own mount path.

## Requirements

### External Role

This role depends on:

- `custom_mounts`

The generated mount definitions are passed directly to the `custom_mounts` role.

### System Requirements

- SSSD configured and working.
- AD/LDAP groups resolvable via `getent`.
- Users resolvable via `getent passwd`.
- CIFS/NFS mount utilities installed as required.

---

## Variables

### `sssd_groups_and_mounts`

List of SSSD groups to process.

Default:

```yaml
sssd_groups_and_mounts: {}
```

Example:

```yaml
sssd_groups_and_mounts:
  - sssd_source_group: informatics_AD_Group
    linux_group: informatics
    custom_mounts_options:
      method: autofs
      type: cifs
      fs_path: "//fileserver/user_shares/informatics"
      master_mount_point: /mnt/autofs_cifs/informatics
      credentials: 
        username: user
        password: vault_string
        domain: ORG
        file_path: /etc/.cifs_credentials
      mount_opts: >-
        mfsymlinks,
        _netdev,
        nofail

  - sssd_source_group: Another_AD_Group
    linux_group: anothergroup
    custom_mounts_options: {}
```

---

## Variable Reference

### Group Mapping

| Variable | Description |
|-----------|-------------|
| `sssd_source_group` | Source SSSD/AD group used to discover users. |
| `linux_group` | Linux group ownership assigned to mounted directories. |

### Mount Options

These options are passed through to the `custom_mounts` role.

| Variable | Description |
|-----------|-------------|
| `method` | Mount method (e.g. `autofs`, `fstab`). |
| `type` | Filesystem type (e.g. `cifs`, `nfs`). |
| `fs_path` | Base remote share path. |
| `master_mount_point` | Parent mount location. |
| `credentials` | Credentials used for the mount. |
| `mount_opts` | Additional mount options. |
| `state` | Desired mount state. Defaults to `mounted`. |
| `temp_mount_opts` | Optional mount options used during temporary validation mount. |

---

## How User Mounts Are Generated

For a group member called:

```text
jbloggs
```

And the following configuration:

```yaml
fs_path: "//fileserver/user_shares/bioinformatics"
master_mount_point: /mnt/autofs_cifs/bioinformatics
linux_group: bioinformatics
```

The role generates:

```yaml
jbloggs:
  method: autofs
  type: cifs
  fs_path: "//fileserver/user_shares/bioinformatics/jbloggs"
  master_mount_point: /mnt/autofs_cifs/bioinformatics
  mount_point: jbloggs
  credentials: "{{ pengu_cifs_creds }}"
  mount_opts: >-
    uid=jbloggs,
    gid=bioinformatics,
    mfsymlinks,
    vers=3.02,
    file_mode=0750,
    dir_mode=0750,
    _netdev,
    nofail
```

Resulting mount path:

```text
/mnt/autofs_cifs/bioinformatics/jbloggs
```

---

## Remote Directory Creation

Before creating the final mounts the role:

1. Temporarily mounts the remote share at:

```text
/mnt/temp_autofs_check
```

2. Ensures a directory exists for every valid user:

```text
/mnt/temp_autofs_check/<username>
```

3. Removes the temporary mount.

This allows user directories to be automatically created before autofs/final mounts are configured.

Directory creation is skipped when:

```yaml
state: absent
```

---

## Removing Mount
```yaml
state: absent
```