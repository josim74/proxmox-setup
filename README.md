# proxmox-setup
## Native Proxmox Features for lxc container management
Proxmox provides its own built-in tools for managing groups of LXCs:
* Resource Pools: You can group multiple LXC containers together to manage their permissions and resource limits as a single unit.
* pct (Proxmox Container Toolkit): This is the CLI tool for Proxmox. You can use it in bash scripts to automate the creation of multiple linked containers.
* Helper Scripts: Many Proxmox users use community Helper Scripts to quickly deploy pre-configured application stacks in LXC.

## Create and destroy lxc container automatically
1. Automation via CLI (pct)
You can use the Proxmox Container Toolkit (pct) to script this manually. A typical build script would look like this: 
* Create: `pct clone <template_id> <new_id>` (cloning is faster than a fresh install).
* Start: `pct start <new_id>`
* Build: `pct exec <new_id> -- /bin/bash /path/to/build_script.sh`
* Destroy: `pct stop <new_id> && pct destroy <new_id>`
2. CI/CD Runner Integration
The most professional way to do this is by using a CI/CD Runner that treats Proxmox as a "cloud" provider:
* GitLab Runner (Custom Executor): You can use the Custom Executor to run scripts that call the Proxmox API. Every time a build starts, GitLab triggers a script to spin up an LXC, runs the build, and then triggers a cleanup script to delete it.
* GitHub Actions (Self-Hosted Ephemeral): You can register a self-hosted runner in ephemeral mode. Once the build finishes, the runner automatically de-registers. You then use a simple script on your host to delete the LXC container it was running in. 
3. Proxmox Helper Tools
There are community-developed tools and plugins designed specifically for this "disposable" container workflow:
* Proxmox LaunchPad: A GitHub Action that can create and delete Proxmox containers directly from your workflow file.
* Terraform: You can use Terraform to "apply" a container configuration before a build and "destroy" it immediately after. This is common in DevOps pipelines to ensure a clean environment every time. 
** Why choose this over Docker? ** 
* Speed: LXC clones start nearly instantly.
* Full OS Environment: Unlike Docker, the build environment is a full Linux OS, which is helpful for complex builds requiring system-level dependencies. 
Recommendation: If you want this to be automatic, setting up a GitLab Runner with a Custom Executor or using the Proxmox LaunchPad GitHub Action is the most reliable way to ensure containers are always cleaned up after a build. 
