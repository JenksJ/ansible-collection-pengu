# repo_template (Ansible Role)

Simple Ansible role to **clone a git repository on the controller**, deploy it to remote hosts, and **template selected files in-place** while keeping the directory structure.

Designed for configuration repos, pipelines, and HPC-style deployments where files must end up owned by a specific runtime user.

---

## What this role does

1. Clones or updates a git repo on the Ansible control host
2. Copies repo contents to a remote destination directory
3. Templates only selected file types (e.g. `.conf`, `.yml`, `.env`)
4. Optionally excludes files or directories from templating
5. Optionally deploys files as another user (`become_user`)
6. Optionally writes git branch + commit metadata to the remote host

`.git` is excluded from deployment by default.

---

## Requirements

* If using `repo_template_become_user`, the SSH user must be able to sudo:
rocky ALL=(<repo_template_become_user>) NOPASSWD: ALL

## Role Variables

```yaml
repo_template_repo_url: ""          # REQUIRED
repo_template_controller_dest: "{{ lookup('env','HOME') }}/ansible_build/repos"

repo_template_git_key_file: ""
repo_template_git_accept_hostkey: true
repo_template_git_version: ""       # branch/tag/commit (empty = repo default)
repo_template_git_force: yes
repo_template_git_update: yes

# Deployment
remote_dest: "/opt/location_2"

# Optional: deploy as another user
repo_template_become_user: ""

# File extensions to template
template_extensions: # regex on find file basename - use single quote or double \\ escape 
  - '.*\.j2$'
  - '.*\.yaml$'
  - '^qc\.prod\.config$' # whole file basename 
# regex patterns to remove for templating list 
exclude_template_list: # regex - use single quote or double \\ escape 
  - '.*/software_configs/.*'
# rsync exclude glob pattern 
exclude_patterns: ["*pengu_logo.png", "*bin/*"]

template_extensions_to_strip:
  - ".j2"


repo_template_template_mode: "0644"
repo_template_exclude_git: true
repo_template_fail_on_undefined: true

# Version Info
# If enabled, a JSON file containing branch + commit info will be written to the remote host.
repo_template_write_version_info: false
repo_template_version_info_dir: "/var/lib/pipeline_versions"
repo_template_version_info_file: "pipeine_name_version.json"
```

