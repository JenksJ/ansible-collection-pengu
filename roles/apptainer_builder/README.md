
# Apptainer Builder Role

This Ansible role discovers Apptainer/Singularity definition files (`*.def`), determines whether their corresponding `.sif` images require rebuilding, and builds them in parallel batches using asynchronous execution.

The role ensures:

- Only changed containers are rebuilt  
- Parallel batch execution for speed  
- Clean comparison of definition files (whitespace-normalised)  
- Clear logs and debug output  

---

## Features

### Automatic discovery of `.def` files
All Apptainer definition files are located using `ansible.builtin.find` and converted into internal container metadata.

### Smart rebuild logic
A container rebuild is triggered **only if**:

- The `.sif` file does not exist  
- `apptainer_force_overwrite` is `true`  
- The source `.def` file differs from the definition embedded in the `.sif` image  
  (line endings and trailing whitespace are removed to avoid false positives)

### Parallel batch builds
Containers are executed in batches (size controlled by `apptainer_max_parallel`) using asynchronous execution.

---

## Variables

### Required

| Variable | Description | Example |
|---------|-------------|---------|
| `apptainer_source_dir` | Directory containing `.def` files | `/path/to/defs` |
| `apptainer_output_dir` | Directory where `.sif` images are stored | `/mnt/datastore/apptainer_images` |
| `apptainer_log_dir` | Log output folder for each apptainer build | `~/apptainer_builds_log` |

---

### Optional

| Variable | Description | Default |
|----------|-------------|---------|
| `apptainer_pattern` | Pattern to find `.def` files | "*.def" |
| `apptainer_force_overwrite` | Always rebuild SIFs | `false` |
| `apptainer_max_parallel` | Containers per batch | `3` |
| `apptainer_build_timeout` | Timeout for each Apptainer build | `1800` |
| `apptainer_poll_retries` | Async polling retries | `180` |
| `apptainer_poll_delay` | Delay between polls (in seconds) | `10` |
| `apptainer_build_opts` | Extra CLI options for `apptainer build` | `""` |
| `apptainer_builder_run_dir` | Directory to run builds from | `"~/"` |
| `apptainer_builder_do_build` | If `false`, discovery runs but containers aren’t built | `true` |

---

## How It Works

### 1. Discover definition files  
All `.def` files are found and turned into metadata objects containing:

- `def_path`  
- `name`  
- `sif_path`  
- `log_path`  

### 2. Check whether SIF images exist  
The role runs a `stat` check for each SIF image.

### 3. Extract existing embedded definitions  
If a SIF exists, the following is executed:

```
apptainer inspect -d <image.sif>
```

### 4. Read source `.def` files  
Definition files are read using `slurp` and trimmed of whitespace:

```
| b64decode | trim
```

### 5. Compare definitions  
The role decides a rebuild is necessary when:

```
force_overwrite = true
OR SIF does not exist
OR trimmed(source_def) != trimmed(existing_def)
```

### 6. Build list of containers needing rebuild  
Only containers with `needs_rebuild: true` are added to the build list.

### 7. Batch processing  
Containers are grouped based on `apptainer_max_parallel`:

```yaml
build_batches = containers_to_build | batch(apptainer_max_parallel)
```

### 8. Async batch execution  
Each batch is:

- Executed asynchronously  
- Logged to its own log file  
- Monitored via `async_status` until completion  



---

## Example Playbook

```yaml
- hosts: apptainer_builder
  become: yes
  roles:
    - role: apptainer_builder
      vars:
        apptainer_source_dir: "/home/jobrunner/repos/PenGU_NXF/qc_dsl2/apptainer"
        apptainer_output_dir: "/mnt/datastore/apptainer_images/qc"
        apptainer_log_dir: "~/apptainer_builds_log"
        apptainer_max_parallel: 4
```

---


### Watch logs on failure
Logs are stored under:

```
{{ apptainer_log_dir }}/<container>.log
```

---

## Notes & Known Issues

- `async_status` **must** run with `become: yes` because async job files are owned by root.
- Ansible 2.13 cannot template `loop_control.label` inside `include_tasks`.  
  Batch titles are therefore pre-rendered in the parent task.
- `slurp` preserves whitespace; this role uses `trim` to avoid rebuilds from trailing blank lines.


## Maintainer
Public Health Wales — Bioinformatics  
Primary Developer: **Jonathan Jenkins**
