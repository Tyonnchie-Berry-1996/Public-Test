## High-Level Homelab Diagram

```mermaid
flowchart LR
    internet(("Internet"))
    vpn["VPN router\n(MAC filtering + VPN)"]
    bridge["Proxmox Linux bridge (vmbr)"]

    internet --> vpn --> bridge

    subgraph standalone["Proxmox cluster (3 nodes)"]
        subgraph vms["Key VMs"]
