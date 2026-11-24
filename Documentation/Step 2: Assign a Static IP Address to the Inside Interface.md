**Date:** 22 November 2026

# Segmented Network & Firewall Lab

*A virtualized Linux firewall with inside/outside segmentation, built and documented on an M1 Mac using UTM and Ubuntu Server.*

## Environment & Setup 

Built in UTM on an M1 MacBook.
VM: Ubuntu Server 26.04.1 LTS (ARM64), 2 GB RAM, 2 CPU cores, 20 GB disk, with an initial Shared Network adapter. 

All screenshots captured from the Terminal interface. 

---
## Step 2: Give `ep0s2` a permanent static address so the firewall has a stable IP that can be used as a gateway.   

![Screenshot 03 showing ls /etc/netplan output](../Lab%20Screenshots/Screenshot%2003%20-%20ls%20etc%20netplan.png)

### Command: `ls /etc/netplan`

<small>

**Purpose:** Lists the network configuration files on the system.`ls` lists a folder's contents. `/etc/netplan` is the path: `/` is the root of the filesystem, `/etc` is where system configuration lives, and `netplan` is Ubuntu's network configuration folder.<br>
**Result:** One file, `00-installer-config.yaml`.<br>
**Conclusion:** Only the installer-generated network configuration exists—no custom network configuration has been added yet.

</small>

---

### Command: `sudo cat /etc/netplan/00-installer-config.yaml`

<small> 
  
**Purpose:** Displays the contents of the installer-generated network configuration.`sudo` runs the command with administrator rights (it prompts for the account password). `cat` prints a file’s contents. `/etc/netplan/00-installer-config.yaml` is the full path to the file.<br> 

**Result:** The file defines only `enp0s1`: DHCP enabled for IPv4 and IPv6, matched by MAC address `b2:7c:d8:a7:c0:9f`, named `enp0s1`. It’s marked as written by `subiquity`, the Ubuntu installer. `enp0s2` isn’t defined anywhere in the file.<br>

**Conclusion:** The second adapter only has an automatic default configuration. A separate netplan file is needed to give it a permanent static address.<br>

</small>

---

### Command:** `sudo tee /etc/netplan/60-lab-inside.yaml > /dev/null << 'EOF'`  
`network:`  
  `version: 2`  
  `ethernets:`  
    `enp0s2:`  
      `match:`  
        `macaddress: 0e:67:4c:75:9f:c4`  
      `set-name: enp0s2`  
      `dhcp4: false`  
      `dhcp6: false`  
      `addresses:`  
        `- 192.168.128.2/24`  
`EOF`  

<small>

**Purpose:** Creates a new netplan configuration file that defines `enp0s2` with a static IPv4 address of 192.168.128.2/24, instead of the DHCP-assigned address it currently uses. A separate file is used so the installer’s working configuration for `enp0s1` stays untouched.<br>

**Result:** No output.<br>  
**Conclusion:** The command is designed to run silently, so the lack of output didn’t confirm success. It was verified in the next step with `ls /etc/netplan`, which showed both `00-installer-config.yaml` and `60-lab-inside.yaml`. The file exists, but the configuration has **not been applied yet**, so `enp0s2` is still on its DHCP address.<br>

</small>
   
---

### Command:`sudo cat /etc/netplan/60-lab-inside.yaml`

<small> 
  
**Purpose:** Verifies the new netplan file contains intended configuration before applied. `sudo` runs the command with administrator rights (it prompts for the account password). `cat` prints a file’s contents. `/etc/netplan/60-lab-inside.yaml` is the full path to the file. Looking to confirm enp0s2, `dhcp4: false` (ensures that it is not competing with a static address) and the IP address matches 192.168.128.2/24.<br>  
**Result:** The file contents matched the intended configuration. The MAC address (`0e:67:4c:75:9f:c4`) corresponds to `enp0s2` as shown in the earlier `ip a` output. DHCP is disabled for both IPv4 and IPv6, the static address 192.168.128.2/24 is defined, and the indentation is correct.<br>  
**Conclusion:** The configuration file is correct and ready to be applied. Verifying the file before applying it reduces the risk of a YAML error disrupting the interface.<br>

</small>

---

### Command: `sudo chmod 600 /etc/netplan/60-lab-inside.yaml` 

<small>
  
**Purpose:** Restricts new netplan file so only root access can have read & write permissions. `chmod` changes a file’s permissions. `600` is the permission value with the three digits representing the owner, group, and everyone else. Each digit is the sum total of read (4), write (2), execute (1), and no access (0).<br>   
**Result:** No output.<br>
**Conclusion:** No output will be verified in the next step.<br>

</small>

---

### Command: `ls -l /etc/netplan`

<small>
  
**Purpose:** Verifies the permission change took effect by listing the files in netplan folder along with read, write, and execute permissions for the owner, group, and other.<br>   
**Result:** 60-lab-inside.yaml shows \-rw-------. \- indicates this is a file. rw- indicates that the owner has read and write permissions but does not have the ability to execute. \--- is listed for both the group and for everyone, indicating that read, write, or execute permissions are not allowed.<br>  
**Conclusion:** The file is protected for root users only to read and modify the network. This applies the principle of least privilege.<br>

</small>

---

**Date:** 23 November 2026

![][image5]  
---

### Command: `sudo netplan try`  

<small> 
  
**Purpose:** To apply the new netplan configuration for `enp0s2` temporarily, with an automatic rollback if is not confirmed – protecting against locking out the remote SSH with a network adjustment.<br> 
**Result:** Netplan reported the configuration was accepted and the SSH session remained connected.<br> 
**Conclusion:** The configuration was valid and was permanently applied. However, this only confirms that netplan accepted the changes and further verification is needed with `ip a`.<br>

</small>

---

### Command: `ip a`  

<small>

**Purpose**: Verifies the new configuration took effect by listing every interface and addresses.<br>  
**Result:** `enp0s2` shows `192.168.128.2/24` without `dynamic` flag, with `valid_lft forever` instead of a countdown lease. `enp0s1` remains unchanged.<br>  
**Conclusion:** The inside interface now has a permanent static address and the outside interface is unaffected.<br>

</small>

---

![][image6]  
---

### Command: `ip route` 

<small>

**Purpose:** To verify where the machine sends traffic and confirm the static configuration did not create a competing default route. `route` command prints the routing table – the set of rules that tells the machine which interface to use to reach which network.<br>   
**Result:** One default route via 192.168.64.1 on `enp0s1`. Directly attached routes exist for 192.168.64.0/24 (`enp0s1`) and 192.168.128.0/24 (**enp0s2**), plus a host route to the gateway 192.168.64.1.<br>  
**Conclusion:** Internet-bound traffic still leaves through the outside interface, and each network is reached through its own interface. The inside interface has no gateway of its own, which is the purpose of isolating the routes.<br>

</small>
---

![][image7]  
---

### Command: `ping -c 4 google.com`   

<small> **Purpose:** Confirms the outbound connectivity and DNS still work after the static configuration was applied to `enp0s2`.<br>  
**Result:** Four packets transmitted, four received, 0% packet loss. Round trip times ranged from 11.2ms (min) to 17.4ms (max). The name was resolved to 142.250.68.206.<br>  
**Conclusion:** The VM can resolve domain names and reach the internet, confirming the network configuration works. The change to the inside interface did not disturb the outside connection.<br>

</small>
---
