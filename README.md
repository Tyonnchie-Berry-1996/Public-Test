## High-Level Homelab Diagram

```mermaid
flowchart LR
    internet(("Internet"))
    vpn["VPN router\n(MAC filtering + VPN)"]
    bridge["Proxmox Linux bridge (vmbr)"]

    internet --> vpn --> bridge

    subgraph standalone["Proxmox cluster (2 nodes)"]

    end

    subgraph standalone["Standalone Proxmox node"]
        

        subgraph vms["Key VMs"]

        end
    end

    bridge --> cluster_nodes
    bridge --> standalone

    subgraph storage["Storage layer (~16 TB)"]

    end

    node1 --- nfs_cluster
    node2 --- nfs_cluster
    node3 --- nfs_cluster
    node3 --- nfs_external

    nfs_external --> filecloud
    nfs_external --> winvm
    nfs_external --> jellyfin

    filecloud --> clients["Clients\n(PCs, VMs, browsers via URL)"]
