# Proxmox Homelab: Resource Utilization > Raw Power

This repo documents my current homelab built around Proxmox VE 8.4. Feel free to keep the diagram up in the background to get a visual: https://tyonnchie-berry-1996.github.io/Proxmox-Home-Lab-Setup/
The core idea behind this setup is simple:

<strong>
<h3><code>Resource utilization matters more than raw resource numbers</code></h3>
</strong>

What is Proxmox VE?
>It’s a Debian-based Linux distribution, but it’s a specialized distro for virtualization, not a generic one


<p>I’ve always wondered: if you match the right architecture with the right Linux distro, can you allocate and utilize resources in a way that truly maximizes the efficiency of your machines? I also wanted to experiment with a type-1 hypervisor, because there were so many times I wanted to spin up a new project but felt restricted by a single host OS that hogged every resource for no good reason. Having a type-1 hypervisor changes that. It’s like being able to install VirtualBox or VMware as your <strong>actual</strong> operating system, and then build everything else on top of it.

Proxmox VE takes that idea and turns your hardware architecture into a virtual environment that’s accessible through a web browser. It transforms a basic homelab into a homelab/cloud hybrid: a real test ground for learning systems, scripting, system administration, networking, Linux bridges, PCIe passthrough, and more. It’s the kind of environment that feels very much like a **“for motivated wizards only”** playground.</p>

Instead of just throwing hardware at problems, I’m aiming to:
- Control the lifecycle of my machines
- Stretch their usable lifespan
- Learn how to allocate resources intentionally
- Reduce wasted compute while still getting high-end performance out of the same equipment.

---

## Specs at a Glance

**OS:** Proxmox VE 8.4  
- **2-node Proxmox cluster**
- **1 standalone Proxmox node** (outside the cluster)

**CPU & RAM (combined):**
- **56 logical CPUs / 2 sockets total**
- **Hyperthreading enabled**
- **48 of them are Intel Xeon E5-2680 @ 2.50 GHz** (NUMA-aware)
- **~220 GB RAM**

**Storage:**
- **Two NFS-backed storage pools** (16 TB total)
  - 8 TB NFS/LVM external drive inside the cluster
  - 8 TB “tri-directional” external drive outside the cluster

**Other hardware:**
- **GPU:** NVIDIA Quadro M2000 (PCIe passthrough)
- **Network:** 2.5 Gbps-capable NIC PCIe
- **Security / Router:** RT-AX86U Pro: VPN Profile + Firewall Rules + MAC filtering + DDNS + Port Forwarding 

---

## Why Proxmox and Type-1 Hypervisors?

The first big shift for me was understanding **type-1 hypervisors**:

> If you imagine VMware or VirtualBox *being* your OS, that’s the idea behind a type-1 hypervisor.

Running Proxmox directly on bare metal:
- Frees the “host OS” from desktop overhead
- Turns the machine into a **virtual playground**
- Lets me treat hardware like a pool of resources I can slice up however I want

On paper, “48 CPUs” sounds like overkill. But:
- Without hyperthreading, my Windows host would only see 24 cores
- On bare-metal Windows, I often felt limited by how the system handled resources and background workloads, and enabling hyperthreading never gave me the same       sense of control I get with Proxmox. With a type-1 hypervisor, I can decide exactly how many vCPUs each VM gets and how threads are allocated, instead of          relying on a single host OS to make those decisions for everything.
- With Proxmox, I can expose more logical CPUs to VMs and shape exactly how they’re used

This setup has taught me a lot about how my hardware actually behaves, including:

- When “more cores” actually makes a difference  
- How NUMA impacts VM placement and performance  
- How to balance **vCPUs per VM** against overall system responsiveness  

---

## Storage Architecture

I use **two main 8 TB external hard drives**, each with a specific role.

### 1. Cluster NFS/LVM External Drive (8 TB, internal to the cluster)

This drive is attached to the cluster and formatted as **LVM**:

- **5 TB LV:**  
  - Dedicated to **VM disks** and **ISO images**  
  - Exported via **NFS** to all cluster nodes  
  - Acts as the main shared storage for the Proxmox cluster  

- **3 TB (remaining LVM space):**  
  - Used as a **local fallback** storage pool  
  - If there’s an issue with NFS, I still have space that can be mounted and used locally on the node  

This keeps all the “VM stuff” (disks + ISOs) centralized, while still giving me a safety net.

---

### 2. Tri-Directional External Drive (8 TB, outside the cluster)

This one is more experimental and has **three roles**:

- **Formatted on my Mac mini**  
  - Uses an Apple-friendly GUID partition scheme  
  - Plugs into my **Mac mini** as a “normal” external drive  
  - I can browse/manage files directly from macOS

- **Mounted on a Proxmox node with a virtual ext4 layer**  
  - When attached to the standalone node, part of the drive is used as an **ext4 filesystem**  
  - That ext4 layer is a virutal file system:
    - Exported via **NFS** to the cluster as another storage backend
    - Mounted into VMs as a shared filesystem for file exchange

- **Cloud drive backing storage (FileCloud VM)**  
  - Inside an Ubuntu VM, I run **FileCloud**
  - A portion of the **ext4 virtual layer** is mounted into this VM (1TB)
  - FileCloud exposes it as a 1TB **cloud drive** that can be reached from:
    - My machines on the cluster
    - My personal computers
    - Any other device that has access to the VPN rotuer

In short, this 8 TB disk acts as:
- A local external drive
- A cluster-accessible NFS filesystem
- Backing storage for a self-hosted “cloud drive” across the network

---

## GPU Passthrough & Workloads

I use **GPU passthrough** on the standalone Proxmox node for multiple workloads via the NVIDIA Quadro M2000:

- **Windows VM**
  - Game emulation (e.g., PCSX2, PPSSPP)
  - Trading Algo's (MT4, MT5, oanda)
  - Streaming Apps
  - General GPU-accelerated desktop usage

- **Ubuntu Jellyfin VM**
  - IPTV
  - Movies and Shows
  - Works as a local netflix but instead of using netflix's servers we use our own. Local installed media (shows & moives) and I make a custom m3u playlist for        IPTV:https://github.com/Tyonnchie-Berry-1996/Build-Your-Own-IPTV. On the backend I use NextPVR for the m3u playlist
  - Hardware-accelerated transcoding for media playback and IPTV

- **Kali Linux VM**
  - Light AI testing
  - Experiments with private GPT-style models / tools

This lets a single mid-range (unicorn) GPU serve:
- Gaming
- Transcoding
- Light AI / tooling
- IPTV hardware acceleration

---

## Networking & Security

Networking is built to balance **performance, control, and security**:

- **2.5 Gbps NIC**
  - Used heavily for:
    - Live TV streaming
    - MetaTrader 4 (MT4) / trading workloads
    - MetaTrader 5 (MT5) / trading workloads
    - Oanda trading platform
    - IPTV
    - Other high-bandwidth VM use cases
  - The higher throughput made it easier to comfortably assign **~5 vCPUs per VM** without the system feeling sluggish
  - With the windows machine I need more than 5 vCPUs but that's the only machine that I tune or max out resources on     occasionally
  - Escpecially with PS2 & PSP emulation you have to pump up the resources
- **VPN Router**
  - **MAC filtering** is enabled so:
    - Only devices I approve can reach the VMs and servers
    - This includes media devices (e.g., a Firestick) and personal machines

- **(future) port forwarding**
  - Next step: clean **port-forwarding** rules on the VPN router to selectively expose:
    - Jellyfin / live TV
    - FileCloud
    - Other services I decide to publish
---

## NUMA and Resource Allocation

Because the CPUs are **NUMA-aware**, Proxmox lets me:

- Bind VMs to specific NUMA nodes
- Keep vCPU allocation sane (e.g., ~5 vCPUs per VM)
- Maintain good performance without overcommitting everything

The guiding principle:

> Don’t just max everything out because you can.  
> **Give each VM exactly what it needs to feel fast, and no more.**

This approach:
- Keeps the host responsive
- Reduces heat and power usage
- Extends the usable lifespan of the hardware
- Makes it easier to understand what’s actually needed for my workloads

---
<h3>This lab is less about flexing specs and more about learning discipline:</h3>

- Use a type-1 hypervisor (Proxmox) to turn hardware into a flexible resource pool
- Centralize and manage VM storage with NFS, but keep smart local fallbacks
- Reuse the same disks in multiple roles (local, NFS, cloud drive)
- Share a single GPU across gaming, media, and light AI
- Wrap it all in sane networking, MAC filtering, and DDNS access
- Focus on **resource utilization > raw performance**
