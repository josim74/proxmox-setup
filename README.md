# proxmox-setup

A practical guide and examples for using Proxmox LXC containers as ephemeral build/test runners and automated environments. This repository documents native Proxmox features, example commands and scripts, and recommended CI/CD integrations to create, run and destroy LXCs automatically.

Table of contents
- [What this is](#what-this-is)
- [Key Proxmox features used](#key-proxmox-features-used)
- [Quick start — pct commands](#quick-start---pct-commands)
- [Example build lifecycle script](#example-build-lifecycle-script)
- [CI/CD integration options](#cicd-integration-options)
- [Tools & community projects](#tools--community-projects)
- [Why choose LXC over Docker](#why-choose-lxc-over-docker)
- [Recommendations & best practices](#recommendations--best-practices)
- [Security notes](#security-notes)
- [Contributing](#contributing)
- [License](#license)

What this is
------------
Short, opinionated documentation showing how to use Proxmox LXCs as disposable build/test environments (ephemeral runners). It focuses on the built-in Proxmox tooling and practical CI/CD approaches so you can spin up clean, full-OS containers for deterministic builds.

Key Proxmox features used
-------------------------
- Resource Pools – group containers and manage permissions + resource limits together.
- pct (Proxmox Container Toolkit) – CLI used to clone, start, exec and destroy LXCs.
- Templates – create a golden LXC template and clone from it to get fast, consistent environments.

Quick start — pct commands
--------------------------
Typical manual workflow using pct (replace <...> placeholders):

- Clone a template (fast):  
```bash
pct clone <TEMPLATE_ID> <NEW_ID> --hostname <HOSTNAME> --storage <STORAGE>
```

- Start the container:
```bash
pct start <NEW_ID>
```

- Run a build script inside:
```bash
pct exec <NEW_ID> -- /bin/bash /path/to/build_script.sh
```

- Stop and destroy:
```bash
pct stop <NEW_ID> && pct destroy <NEW_ID>
```

Example build lifecycle script
------------------------------
A minimal example to create, run, and clean up an LXC (illustrative — adapt to your environment):

```bash
#!/usr/bin/env bash
set -euo pipefail

TEMPLATE_ID=9000
NEW_ID=9100
HOSTNAME="ephemeral-runner"
STORAGE="local-lvm"
BUILD_SCRIPT="/root/build.sh"

# Clone, start, run build, cleanup
pct clone "$TEMPLATE_ID" "$NEW_ID" --hostname "$HOSTNAME" --storage "$STORAGE"
pct start "$NEW_ID"

# Copy or create a build script into the container (optional)
pct push "$NEW_ID" "$BUILD_SCRIPT" /root/build.sh

# Execute the build
pct exec "$NEW_ID" -- bash /root/build.sh

# Stop and destroy the container
pct stop "$NEW_ID"
pct destroy "$NEW_ID"
```

CI/CD integration options
------------------------
1. GitLab Runner (Custom Executor)
   - Use a Custom Executor that invokes pct or the Proxmox API to provision ephemeral LXCs for each job.
   - Advantages: tight control, can expose job metadata to your provisioning scripts.

2. GitHub Actions (self-hosted ephemeral)
   - Register a self-hosted runner machine that orchestrates ephemeral LXCs (the runner itself can be a thin controller).
   - Use a workflow step to call the controller script that creates, uses, and destroys the LXC.

3. Terraform
   - Use the Proxmox Terraform provider to "apply" ephemeral container resources at the start of a job and "destroy" afterwards.
   - Good for declarative pipelines and versioned config.

4. Proxmox LaunchPad / Community Actions
   - Use existing GitHub Actions or community plugins that call Proxmox APIs to create/destroy containers from workflow files.

Tools & community projects
--------------------------
- Proxmox LaunchPad (community GitHub Action) — creates and removes Proxmox containers directly from GitHub Actions.
- Terraform Proxmox provider — declarative resource provisioning and lifecycle management.
- Helper scripts — many community repos provide helper scripts to quickly deploy app stacks inside LXCs.

Why choose LXC over Docker?
---------------------------
- Speed: LXC clones from templates start nearly instantly.
- Full OS environment: You get a complete Linux userland, which simplifies builds that need systemd, kernel modules, or system-level packages.
- Isolation: Stronger isolation than a simple process container for certain workloads.
- Recommendation: Use LXC if your CI jobs require full OS features or closely mirror production OS-level behavior.

Recommendations & best practices
--------------------------------
- Maintain a minimal, hardened template image to clone from (reduce attack surface and startup steps).
- Label containers clearly (job id, commit sha) so automatic cleanup is straightforward.
- Use resource limits (memory, CPU, disk) to avoid noisy neighbors in multi-tenant hosts.
- Use a dedicated Proxmox user with scoped permissions or API tokens for automation.
- Implement robust cleanup (timeouts, GC job) to ensure ephemeral containers don’t accumulate.

Security notes
--------------
- Never run untrusted build scripts as root without sandboxing and strict resource/permission constraints.
- Use Proxmox’s firewall, network isolation (VLANs) and storage permissions to limit lateral movement.
- Rotate API tokens regularly and grant least privilege.

Examples & snippets
-------------------
- pct clone + start example (see Quick start).
- GitLab Runner: call a wrapper script that provisions the LXC, runs the job, then tears it down.
- Terraform: write a module that creates a proxmox_lxc resource and outputs the IP/credentials for the job, then destroys it after the run.

Contributing
------------
Contributions, suggestions and corrections welcome. If you have proven helper scripts or CI examples that work well, please open an issue or PR.

License
-------
This repository is provided "as-is". Add your preferred license here.

Notes
-----
This README provides patterns and examples — adapt them to your Proxmox cluster, storage backend, and CI tooling. If you want, I can:
- add a ready-to-run GitLab Runner custom executor example,
- produce a Github Actions workflow that uses a controller script,
- or create a Terraform module/template for ephemeral LXC runners.