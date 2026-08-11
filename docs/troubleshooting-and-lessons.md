# Troubleshooting and Lessons Learned

The most useful part of the original NexusLeonis project was not the list of applications. It was the troubleshooting required to keep a growing multi-system environment working.

This document summarizes issues explicitly captured in the historical project notes. Sensitive addresses, credentials, and machine-specific access commands have been removed.

## 1\. Docker API Version Mismatch

### Symptom

A Homepage integration for Watchtower would not return the expected data.

### Investigation / Fix

The documented fix required specifying a compatible Docker API version and correctly configuring the Watchtower API token.

### Lesson

A container can be healthy while an integration that depends on its API is broken. The failure may be version negotiation or authentication rather than the service itself.

\---

## 2\. Homepage Host Validation

### Symptom

Homepage rejected access because the requested host was not allowed.

### Investigation / Fix

The environment required explicit allowed-host configuration.

### Lesson

Application-level host validation can look like a networking problem even when the container and port mappings are correct.

\---

## 3\. Dashboard Widget Configuration

### Symptom

Some Homepage widgets displayed API errors or no data even though the underlying service was reachable.

### Investigation / Fix

Troubleshooting included checking:

* Service-specific API requirements
* Authentication settings
* Correct endpoint configuration
* Container logs
* API compatibility

### Lesson

A working web UI does not prove an API integration is configured correctly. Testing the endpoint and reading logs is faster than repeatedly redeploying the container.

\---

## 4\. Port Conflicts

### Symptom

Multiple containers expected the same default host port.

### Investigation / Fix

External ports were remapped while leaving the service's internal port unchanged where appropriate.

### Lesson

A larger Docker host needs deliberate port tracking. Default examples copied from individual application documentation cannot always coexist on one machine.

\---

## 5\. Docker-in-LXC Permissions

### Symptom

Some Docker containers had permission problems when using expected UID/GID mappings inside the Proxmox LXC.

### Investigation / Workaround

The notes document several cases where running a container as root inside the LXC avoided the immediate mapping problem, with exceptions for applications that worked normally with standard IDs.

### Lesson

Nested isolation layers complicate ownership. A UID that makes sense on the Docker host may not map cleanly through the LXC layer to storage outside the container.

This was a practical workaround in a private learning lab, not a general security recommendation.

\---

## 6\. Unreliable Direct SSH to the Docker LXC

### Symptom

Direct SSH into the Docker LXC was unreliable.

### Workaround

The reliable management path was to connect to the Proxmox host first and enter the LXC from the hypervisor.

### Lesson

Keeping an out-of-band or host-level management path is valuable when guest networking or remote access is misbehaving.

\---

## 7\. NAS Mount Failures

### Symptom

Linux systems occasionally lost access to NAS-backed shares after restart or configuration changes.

### Investigation

The troubleshooting process included:

* Verifying network connectivity to the NAS
* Confirming the NFS/CIFS client packages and mounts
* Checking whether the export/share was enabled
* Reviewing filesystem permissions
* Remounting and validating the filesystem before troubleshooting the application

### Lesson

When an application loses files, start at the storage layer. A container error may simply be the downstream symptom of a missing or incorrectly mounted filesystem.

\---

## 8\. NAS / LXC / Docker Ownership Complexity

### Symptom

Permissions could behave differently across the NAS, Linux host, LXC, and Docker container layers.

### Lesson

Storage architecture matters. Each additional namespace or translation layer creates another place where UID/GID ownership and mount behavior can diverge.

\---

## 9\. GPU Passthrough

### Goal

Provide a full Proxmox VM with direct access to the host's NVIDIA GPU for local AI and GPU-accelerated workloads.

### Work Performed

The historical reference documents:

* IOMMU enabled on the host
* VFIO modules loaded
* NVIDIA driver isolated from the host for the passed device
* PCIe device assignment to the VM
* IOMMU group verification
* NVIDIA/CUDA validation inside the guest

### Result

The later system reference shows the GPU successfully attached to the VM with CUDA available.

### Lesson

GPU passthrough crosses firmware, kernel, PCIe, driver, hypervisor, and guest-OS boundaries. Problems at any of those layers can produce the same visible symptom: the guest cannot use the GPU.

\---

## 10\. LVM-Thin Metadata Corruption and the Decision to Retire the Build

### Symptom

The Proxmox storage pool began reporting only a small fraction of the NVMe drive's actual capacity. The host had roughly 1 TB of physical NVMe storage, but the LVM-thin pool exposed only about 68 GB as usable. The GPU-enabled VM was already consuming most of that visible space and could not be expanded normally.

### Diagnosis

The project notes identify corrupted LVM-thin metadata as the cause. Repair would have required taking the affected storage offline and attempting a `thin\_repair` operation.

This was not a cosmetic capacity-reporting issue. The hypervisor's storage layer could no longer reliably account for or expose the majority of the physical disk, which directly limited the VMs depending on that pool.

### Status

The issue was never resolved. At the end of the original NexusLeonis build, Proxmox still reported only a small fraction of the drive's true capacity.

### Impact on the Project

This became the primary reason I chose to retire the original environment rather than continue extending it. By that point the lab had accumulated enough architectural and configuration debt that repairing the storage pool in place would still have left me with an environment I wanted to redesign.

Rather than keep layering fixes onto the existing installation, I made the decision to preserve the original build as a historical record, wipe the environment, and eventually rebuild it cleanly using the lessons learned from operating it.

### Lesson

Storage has multiple layers, and physical free space does not guarantee usable virtual storage. Filesystem, logical volume, thin-pool, and metadata health all matter. A failure in the metadata layer can effectively strand otherwise available disk capacity.

The larger lesson was also architectural: once a foundational layer becomes unreliable, continuing to build on top of it can create more risk and complexity than starting clean. Recovery planning, backups, storage design, and documentation of major infrastructure changes need to exist before the storage layer becomes a single point of failure.

\---

## 11\. VM Autostart / Bridge Timing

### Symptom

The GPU-enabled VM did not start cleanly after an initial reboot.

### Fix

A startup delay was added to account for network bridge timing.

### Lesson

Boot-time dependency ordering matters. A configuration that works when started manually may still fail during automated startup if required host resources are not ready yet.

\---

## 12\. noVNC Console and Virtual Display

### Symptom

Changing the VM's display configuration caused boot/console problems.

### Fix

The final notes indicate that retaining a standard virtual display allowed noVNC access to remain functional alongside GPU passthrough.

### Lesson

A physical GPU passed to a guest does not automatically replace every reason to keep a virtual console device, especially for recovery and remote administration.

\---

## 13\. Scheduled Administration Scripts

Both the secondary Linux host and the GPU-enabled VM used small scripts and cron jobs for:

* Security audit output
* Package updates
* Login-time system status
* On-demand diagnostic checks

### Lesson

Automation does not need to be complex to be useful. Repetitive health and maintenance checks are good candidates for simple scripts before reaching for a larger orchestration platform.

\---

## 14\. Documentation Was Part of the Lesson

### Problem

The original NexusLeonis environment was built as a learning project, and documentation was not treated as part of the infrastructure from the beginning. I documented pieces of the environment when I needed a handoff, a troubleshooting reference, or a snapshot of the current state, but I did not maintain one organized source of truth as the system changed.

That created several problems later:

* Documentation represented different points in time and sometimes contradicted newer configurations.
* Troubleshooting history was scattered rather than tied to the change that caused or resolved an issue.
* The exact sequence of architectural changes could not always be reconstructed after the fact.
* There was no meaningful Git history showing how the environment evolved or enabling any sort of version control for when I inevitably broke something.
* Working configuration files were not originally structured with public version control in mind, so they could contain environment-specific details that should not be published.

This is why relatively little exists as a clean, chronological record of the original build even though the environment itself became fairly large. This repository is a reconstructed historical account, not the original development history.

### Lesson

Clear, organized documentation needs to start with the project, not after the project becomes complicated enough to need it. Infrastructure documentation should be written in a structure that can be version controlled from the beginning so architecture changes, configuration decisions, troubleshooting, and lessons learned evolve alongside the environment itself.

For a project like this, that means maintaining things such as:

* A concise README describing the environment and its purpose
* An architecture reference that changes when the architecture changes
* A current service inventory
* Network and storage diagrams that avoid publishing sensitive addressing
* A dated build/change log
* Troubleshooting notes tied to actual failures and fixes
* Sanitized configuration examples that keep secrets and environment-specific values outside version control

Version control is useful for more than code. In an infrastructure project, it creates a record of what changed, when it changed, and what documentation or configuration changed with it. It also makes it much easier to distinguish the current state from an old snapshot.

The lack of that structure in NexusLeonis OG became one of the most important lessons from the project, and the reason this repository has to tell the story retrospectively rather than simply showing the original commit history.

\---

## Overall Takeaway

The project gradually moved from "deploy a service" to "operate an environment." The harder problems usually sat between systems: Docker and LXC, Linux and NAS storage, API clients and services, hypervisor and GPU, or VM startup and host networking.

The unresolved LVM-thin corruption ultimately became the stopping point for the original build. It exposed how much the environment had grown without the same level of planning around storage recovery, documentation, and change tracking. Preserving the project as a historical record instead of continuing to patch it became part of the lesson itself.

That was the value of the lab. It created problems that could not be solved by knowing only one application, and it showed me where better infrastructure design and documentation mattered just as much as getting individual services running.

