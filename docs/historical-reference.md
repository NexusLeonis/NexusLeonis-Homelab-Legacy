# NexusLeonis Original Homelab — Sanitized Historical Reference

This document reconstructs the original NexusLeonis homelab from project notes written between early and late stages of the build. It is intentionally sanitized for public use.

The source documents were operational notes rather than a formal version-controlled build record. As a result, they capture different snapshots of the environment and occasionally disagree about whether a service was planned, deployed, or fully configured.

## 1\. Environment Overview

The homelab began with a custom desktop converted into a **Proxmox VE hypervisor**. The initial architecture placed the majority of application workloads inside a Debian LXC container running Docker.

Over time, the environment expanded to include:

* A dedicated Proxmox virtualization host
* A Debian LXC acting as the original Docker host
* A dedicated multi-drive NAS
* A Windows workstation used for administration and some local AI workloads
* A Linux Mint laptop used as a secondary Docker host and sandbox
* A full Linux VM on Proxmox with direct GPU passthrough
* Tailscale-based remote connectivity between selected systems

The original lab was a learning environment, but it was operated continuously enough that monitoring, automated updates, storage organization, remote access, and recovery procedures became necessary.

## 2\. Primary Hardware

### Proxmox Host

The main server was a repurposed/custom desktop with:

* AMD Ryzen 7 3800X, 8 cores / 16 threads
* 32 GB DDR4 RAM
* NVIDIA RTX 2070 Super
* 1 TB NVMe local storage
* Ethernet network connection

The system ran Proxmox VE and initially hosted the Docker LXC. Later, the RTX 2070 Super was passed directly through to a full Linux VM.

### Dedicated NAS

Later documentation shows a dedicated NAS with approximately 21.7 TB of raw/usable JBOD storage across three 8 TB Seagate IronWolf drives.

The NAS was used for combinations of:

* Media storage
* Homelab/application data
* AI model and audiobook storage
* Backups
* Windows-accessible shared files

Historical planning documents also describe an earlier Unraid-oriented storage design. Because the later system reference documents a different NAS platform, this retrospective treats the NAS role and protocol design as authoritative while avoiding claims that every early storage plan was implemented exactly as written.

### Mimir — Secondary Linux System

An ASUS ROG Zephyrus G14 was repurposed as a Linux Mint workstation and secondary Docker host.

Hardware included:

* AMD Ryzen 9 4900HS
* NVIDIA RTX 2060
* 16 GB RAM
* 1 TB local SSD

Mimir became a portable sandbox for Linux, Docker, AI, automation, and monitoring experiments.

### Mimir-Tower — GPU-Enabled VM

A full Linux VM on the Proxmox host was created as a heavier self-hosted AI and Docker workstation.

The historical reference documents:

* Host CPU passthrough
* 16 GB assigned RAM at the later snapshot
* NVIDIA RTX 2070-class GPU passed directly to the VM
* Arch-based Linux with KDE Plasma
* CUDA available inside the guest
* was only partially realized before a corrupt memory cache issue

## 3\. Original Virtualization and Container Architecture

The first major architecture was intentionally simple:

```text
Custom PC
└── Proxmox VE
    └── Debian LXC
        └── Docker
            ├── Infrastructure services
            ├── Monitoring services
            ├── Automation services
            ├── Media services
            └── Utilities
```

Docker CE provided the container runtime. **Portainer CE** was the primary management interface and stacks were used to deploy services.

**Watchtower** performed scheduled container update checks, while **Uptime Kuma** monitored service availability and sent downtime notifications.

This architecture was lightweight and easy to expand, but nesting Docker inside LXC later contributed to permission and device-access complexity.

## 4\. Management and Monitoring

### Homepage

Homepage became the central visual dashboard for the environment. It was customized with grouped service sections, status information, widgets, system information, search, and other dashboard elements.

The dashboard was not just a bookmark page. Several services exposed API-backed widgets that displayed live operational data, including:

* Container counts and state
* Update status
* Service availability
* DNS filtering statistics
* Network performance
* Media library information

The public repository includes one later screenshot of this interface.

### Portainer

Portainer was used to:

* Deploy Docker stacks
* View and restart containers
* Inspect container state
* Manage the growing service catalog

### Uptime Kuma

Uptime Kuma checked service availability on a regular interval. The environment used external notifications for downtime events.

### Watchtower

Watchtower handled scheduled container update checks. Integrating its API with Homepage required additional configuration, including resolving a Docker API version mismatch.

## 5\. Network and Remote Access

The lab operated on a private home LAN. Services were primarily accessed internally rather than exposed directly to the public internet.

**Tailscale** was added to provide encrypted remote connectivity between selected devices and systems. The exact topology changed during the life of the project. At different points, a Windows system acted as a path back into the LAN, while later Linux systems and the NAS also participated directly in the tailnet.

**AdGuard Home** was deployed for DNS-based filtering and later documentation identifies it as the network-wide DNS/ad-blocking service.

**Nginx Proxy Manager** was also deployed as a reverse-proxy management service during the expanded phase of the lab.

## 6\. Storage Architecture

Storage evolved from local Proxmox/LXC volumes into a centralized NAS model.

The design used different file-sharing protocols based on client type:

* **NFS** for Linux systems and container workloads
* **SMB/CIFS** for Windows access

The NAS provided separate logical areas for application data, media, AI-related files, backups, and shared user files.

A representative design looked like this:

```text
                         Dedicated NAS
                         /           \\
                        /             \\
                     NFS               SMB
                      |                 |
                Linux / Docker      Windows
                  systems           workstation
```

Container configuration and media data were gradually moved away from relying only on the Proxmox host's local disk.

### Storage Management Challenges

The move to centralized storage created several recurring issues:

* NFS/CIFS mounts not always returning cleanly after restart
* UID/GID and ownership mismatches
* Permission behavior across NAS → Linux → LXC → Docker layers
* Need to verify mounts before diagnosing application failures

These problems made storage troubleshooting a significant part of the project.

## 7\. Secondary Linux/Docker Host

Mimir ran Linux Mint and hosted a separate set of Docker workloads. Historical documentation lists services such as:

* Portainer
* A secondary dashboard
* Local AI interfaces
* Private metasearch
* Jellyfin
* Storyteller
* Glances
* Uptime Kuma
* n8n
* ArchiveBox
* MeTube

The laptop also used the NAS for shared media storage.

### Basic Administration Automation

Mimir had a small set of scripts and scheduled tasks for:

* Daily security auditing
* Weekly package updates
* Login-time system status
* On-demand diagnostics

Logs were written to dedicated directories for later review.

These were simple scripts, but they were useful for learning routine Linux administration and for reducing repetitive checks.

## 8\. GPU Passthrough VM

Mimir-Tower was created as a full VM rather than an LXC workload and received direct access to the Proxmox host's NVIDIA GPU.

The passthrough configuration required:

* Enabling IOMMU
* Loading VFIO modules
* Binding the GPU to VFIO rather than the host NVIDIA driver
* Passing the PCIe device into the VM
* Verifying IOMMU grouping
* Installing and validating NVIDIA/CUDA support inside the guest

The historical system reference shows the GPU passthrough working and CUDA available inside the VM.

The VM was intended to become the primary AI and Docker host, with the Linux laptop returning to a sandbox role. That migration was still incomplete in the final project notes.

## 9\. Operational Practices

As the lab grew, it developed recurring operational patterns:

* Deploy services as Docker stacks
* Add services to the dashboard after deployment
* Add availability monitoring
* Check logs before changing configuration
* Use web management interfaces where practical
* Track port assignments to avoid collisions
* Keep services organized by purpose
* Run scheduled updates and periodic resource reviews
* Maintain quick-reference documentation for common tasks

The documentation itself evolved into a practical handoff mechanism so the environment could be understood after time away from the project.

## 10\. End-State Snapshot

The most complete January documentation describes service categories for:

* Infrastructure
* Monitoring
* AI tools
* Utilities
* Automation
* Media servers
* Media automation
* Download clients

That document refers to approximately 30 service entries. A small number were placeholders or not fully configured, so the figure should be read as the size of the documented service catalog rather than a claim that 30 independent applications were simultaneously healthy at every point in time.

By the later May snapshot, the environment had become a small cluster of purpose-built systems rather than a single Docker host:

```text
                       Private LAN / Tailscale
                                |
        +-----------------------+-----------------------+
        |                       |                       |
  Windows Workstation      Proxmox Host           Dedicated NAS
                                |
                     +----------+----------+
                     |                     |
                Docker LXC          GPU-enabled VM
                (original)          (Mimir-Tower)

             Secondary Linux/Docker Host
                      (Mimir)
```

The original Docker LXC was still running the older homelab stack while newer workloads were being tested or migrated elsewhere.

## 11\. Known Unfinished Work

The historical notes show several items still incomplete at the end of the documented project period:

* Full migration away from the original Docker LXC
* Repair of the Proxmox LVM-thin metadata pool
* Storage expansion of the GPU-enabled VM after repair
* Completion of some Tailscale configuration on the new VM
* Some service migrations and clean-up

This repository intentionally leaves those as unfinished work rather than rewriting the history to make the environment appear cleaner than it was.

## 12\. Security Redactions

The public version removes:

* All private LAN and Tailscale IP addresses
* Usernames
* SSH commands tied to the original systems
* SSH key locations
* Passwords and credential references
* API tokens and keys
* Exact internal URLs
* Personal share names and paths where they are not needed to explain the architecture

Hardware specifications, service names, general architecture, and technical lessons remain because they are the useful part of the project record.

