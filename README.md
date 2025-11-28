## High-Level Homelab Diagram

```mermaid
flowchart LR
    internet(("Internet"))
    vpn["Network Switch & VPN Router (MAC filtering + Firewall rules + Port Forwarding + DDNS)"]
    bridge["Proxmox Linux bridge (Wired connection)"]
    internet --- vpn ---bridge
    
    subgraph standalone["Proxmox Servers (3 nodes)"]
    node1["Node 1 - 48 x Intel Xeon E5-2680 (NUMA) + Quadro M2000 + 2.5G NIC"]
    node2["Node 2 - 4 x Intel Core i5-6500T CPU @ 2.50GHz + 8 TB NFS: LVM 5 TB LV + 3 TB local"]
    node3["Node 3 - 4 x Intel Core i5-6500T CPU @ 2.50GHz + 8 TB NFS: Exfat + Virtual file system + Cloud Drive"]
end
    subgraph vm["Cluster"]
    subgraph winvm["Windows VM: Game Emulation + Trading Algos + Rufus + Live Tv + Adobe"]


end
end
    subgraph new["Off-Cluster"]
    subgraph gf["Tri-Drive"]
end
end

bridge ---> node1
bridge ---> node2
bridge ---> node3

node1 ---> vm
node2 ---> vm
node3 ---> new
