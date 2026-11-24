# Segmented-Network-Firewall-Lab
A virtualized Linux firewall with inside/outside segmentation, built and documented on an M1 Mac using UTM and Ubuntu Server. All screenshots captured from macOS Terminal interface.
* NOTE FROM THE AUTHOR: some screenshots taken after to show verification of completed steps.

## Project Goal
Build a small segmented network behind a Linux-based firewall, and document each step.

## Lab Environment
- Host: M1 MacBook
- Hypervisor: UTM
- Guest OS: Ubuntu Server 26.04.1 LTS (ARM64)
- Firewall VM: 2GB RAM, 2 CPU cores, 20GB disk
- Networking: one Shared Network adapter (outside) and one Host Only adapter (inside)

## Lab Progression
- [x] Step 1: Build the VM, verify connectivity, apply updates, add a second network adapter
- [x] Step 2: Assign a static IP address to the inside interface
- [ ] Step 3: Enable IP forwarding
- [ ] Step 4: NAT and firewall rules with nftables
- [ ] Step 5: Add a client VM and test rules
