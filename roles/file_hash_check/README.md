# file_hash_check

This role validates MD5 hashes of large files in parallel on remote hosts.

## Features

- Checks that files exist before hashing  
- Computes MD5 hashes remotely (no transfer)  
- Runs hashing jobs in parallel by distributing accross the host group  
- Validates each file's hash  
- Fails if a file is missing or hash mismatches  

## Variables
defaults/main.yml
file_hash_check_files - Dictionary - Structured dictionary containing grouped list objects of file targets and their expected hashes.
file_hash_check_do_hashes - Boolean - toggle to control whether MD5 execution and validation routines run (true/false).

### Required

```yaml
# Global toggle to control whether MD5 execution or just check present 
file_hash_check_do_hashes: true

# Structured dictionary containing grouped list objects of file targets and their expected hashes.
file_hash_check_files:
  some_grouping1:
    - name: "/data/volume1/large_archive.tar"
      hash: "fe6befdcc2190619430ccd4ecb54e5b2"
    - name: "/data/volume1/config_metadata.json"
      hash: "54d374c4dbdc82d180282a6d1e3fbb6d"
  another_grouping:
    - name: "/data/volume2/database_dump.sql"
      hash: "a8b39cdb12190619430ccbc4ecb54e8e"

```

Example Playbook
```yaml
- hosts: file_hash_check
  gather_facts: yes
  tags: file_hash_check
  tasks:
    - name: Include the hash validation role
      include_role:
        name: file_hash_check