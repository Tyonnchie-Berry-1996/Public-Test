## High-Level Homelab Diagram

```mermaid
flowchart LR
    internet(("Internet"))
    vpn["VPN router\n(MAC filtering + Firewall rules + Port Forwarding+ DDNS)"]
    node1["Node 1 - Proxmox 8.4\n 48 x Intel Xeon E5-2680 (NUMA) + Quadro M2000 + 2.5G NIC"]
    node2["Node 2 - Proxmox 8.4\n 4 x Intel Core i5-6500T CPU @ 2.50GHz + 8 TB NFS\nLVM: 5 TB VM/ISO + 3 TB local"]
    bridge["Proxmox Linux bridge (Wired connection)"]

    internet --> vpn --> bridge

    end

    subgraph standalone["Standalone Proxmox node"]
        node3["Node 3 - Proxmox 8.4\nQuadro M2000 + 2.5G NIC"]

        subgraph vms["Key VMs"]
            winvm["Windows VM\nEmulation + MT4"]
            jellyfin["Ubuntu Jellyfin VM\nLive TV + transcode"]
            kali["Kali Linux VM\nPrivate GPT / AI tests"]
            filecloud["Ubuntu FileCloud VM\nCloud drive backend"]
        end
    end

    bridge --> cluster_nodes
    bridge --> standalone

    subgraph storage["Storage layer (~16 TB)"]
        nfs_cluster["8 TB NFS (cluster)\nLVM: 5 TB VM/ISO + 3 TB local"]
        nfs_external["8 TB tri-directional drive\nGUID + ext4 + NFS + cloud"]
    end

    node1 --- nfs_cluster
    node2 --- nfs_cluster
    node3 --- nfs_cluster
    node3 --- nfs_external

    nfs_external --> filecloud
    nfs_external --> winvm
    nfs_external --> jellyfin

    filecloud --> clients["Clients\n(PCs, VMs, browsers via URL)"]
