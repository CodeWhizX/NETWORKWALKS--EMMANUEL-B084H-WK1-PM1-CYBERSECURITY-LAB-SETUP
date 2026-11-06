# NETWORKWALKS--EMMANUEL-B084H-WK1-PM1-CYBERSECURITY-LAB-SETUP
Isolated virtual lab built with VirtualBox and Kali Linux for cybersecurity testing and penetration testing practice.
# Cybersecurity Virtual Lab Setup (Networkwalks WK1-PM1)

An isolated, controlled virtual lab environment built with **Oracle VM VirtualBox**, **Kali Linux** (attacker and analysis node), and **Windows 10** (target endpoint) running inside a private virtual NAT Network (`10.0.0.0/24`).

---

## 💻 Hardware & System Requirements

### Host Machine Specifications
* **Operating System:** Linux (Linux Mint / Ubuntu / Debian) or Windows 10/11 64-bit
* **Processor:** 64-bit multi-core CPU with Hardware Virtualization (`Intel VT-x` or `AMD-V`) enabled in BIOS/UEFI
* **RAM:** 16 GB recommended (Minimum: 8 GB)
* **Storage:** 100+ GB free SSD storage for virtual disk images (`.vdi`)

### Virtual Machine Resource Allocations

| Virtual Machine | Guest OS | Base Memory (RAM) | Processors (vCPU) | Virtual Disk Size | Network Attachment |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Attacker Node** | Kali Linux (64-bit) | 2048 MB – 4096 MB | 2 vCPUs | 30 GB (Dynamically allocated) | NAT Network (`NatNetwork`) |
| **Target Node** | Windows 10 (64-bit) | 4096 MB | 2 vCPUs[cite: 6] | 65 GB (Dynamically allocated)[cite: 6] | NAT Network (`NatNetwork`)[cite: 6] |

---

## 📥 Official Download Sources

* **Oracle VM VirtualBox & Extension Pack:**
  * Hypervisor & Extension Pack: [VirtualBox Official Downloads](https://www.virtualbox.org/wiki/Downloads)
* **Kali Linux:**
  * Pre-built VirtualBox Image / ISO: [Kali Linux Get-Kali Portal](https://www.kali.org/get-kali/)
* **Microsoft Windows 10:**
  * Windows 10 Disk Image (ISO): [Microsoft Windows 10 Download Portal](https://www.microsoft.com/software-download/windows10)
  * Windows 10 Enterprise (90-Day Lab Trial): [Microsoft Evaluation Center](https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise)

---

## 🎯 Lab Architecture & Network Plan

```text
                        ┌────────────────────────┐
                        │   Physical Host / OS   │
                        │     (Linux Mint)       │
                        └───────────┬────────────┘
                                    │
                    ┌───────────────▼──────────────┐
                    │ VirtualBox NAT Engine        │
                    │ Gateway: 10.0.0.1            │
                    │ Subnet:  10.0.0.0/24         │
                    └───────┬──────────────┬───────┘
                            │              │
             ┌──────────────┴───┐      ┌───┴──────────────┐
             │    Kali Linux    │      │    Windows 10    │
             │   10.0.0.2/24    │◄────►│   10.0.0.4/24    │
             │ (Security/Attacker)    │ (Target System)  │
             └──────────────────┘      └──────────────────┘

Step-by-Step ImplementationStep 1: VirtualBox Installation & Hypervisor SetupDownload Oracle VM VirtualBox and the matching version VirtualBox Extension Pack. Install the package on your host system and install the Extension Pack to support USB 2.0/3.0 controllers and host device passthrough.

  Step 2: Configure Dedicated NAT NetworkIn Oracle VM VirtualBox Manager, open Tools -> Network -> NAT Networks. Click Create to initialize a new network named NatNetwork.   Set the IPv4 Prefix to 10.0.0.0/24 and ensure Enable DHCP is checked.

 Step 3: Target VM (Windows 10) ConfigurationProvision a new Windows 10 64-bit virtual machine allocated with 4096 MB RAM, 2 processors, and a 65 GB virtual disk[cite: 6].Under VM Settings -> Network, set Attached to: NAT Network and select NatNetwork with the adapter type set to Intel PRO/1000 MT Desktop[cite: 6].Boot Windows 10, open Network Connections (ncpa.cpl), and manually assign static parameters to Internet Protocol Version 4 (TCP/IPv4):IP Address: 10.0.0.4   Subnet Mask: 255.0.0.0   Default Gateway: 10.0.0.1   Preferred DNS Server: 8.8.8.8 .

  Verify internal interface assignment and external gateway reachability using Command Prompt:
ipconfig
ping 8.8.8.8

Step 4: Attacker VM (Kali Linux) Setup & Interface Tuning
In the Kali Linux VM network settings, bind Adapter 1 to NatNetwork.
Boot Kali Linux. If network initialization delays or fails due to IPv4 Duplicate Address Detection (DAD), disable the timeout in NetworkManager and reload the connection:
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"

Step 5: Connectivity & Inter-VM Verification
Verify Kali IP Assignment & Internet Access:
Confirm interface eth0 holds 10.0.0.2/24 and test ICMP resolution to Google DNS and web targets:
ip addr show
ping -c 4 8.8.8.8
ping -c 4 google.com

Verify Cross-Subnet Ping to Windows 10:
Ping the Windows 10 VM (10.0.0.4) from the Kali terminal to confirm bidirectional connectivity within NatNetwork
ping -c 4 10.0.0.4

Troubleshooting Notes
Windows Firewall ICMP Blocking: By default, Windows Defender Firewall drops incoming ICMP Echo Requests (ping). If Kali cannot ping Windows 10, enable the predefined rule in Windows:
DOS
netsh advfirewall firewall add rule name="Allow ICMPv4-In" protocol=icmpv4:8,any dir=in action=allow

NAT vs. NAT Network: A standard NAT adapter puts each VM on its own isolated routing instance, preventing VM-to-VM traffic. NAT Network pools all participating virtual machines onto the same virtual switch while providing outbound WAN access.
NetworkManager DAD Timeout: Setting ipv4.dad-timeout 0 avoids infinite DHCP negotiation stalls on virtual network adapters in Debian/Kali environments.

Problems Encountered & Solutions
Documenting problems is an important part of the project.

Problem 1. Internet Connectivity After Static IP Configuration
After manually configuring the IPv4 settings, Internet connectivity may fail depending on the Kali/NetworkManager configuration.

One workaround used during this lab was:

sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
The network connection was then restarted/rebooted and connectivity was tested again.

Important: Network interface and connection names may differ between systems. Students should first identify their actual connection name before running an nmcli command.

Problem 2. VirtualBox VT-x / Virtualization Error
The VM initially failed to start because hardware virtualization was disabled in the system firmware/BIOS.

The issue was resolved by:

Restarting the computer.
Entering BIOS/UEFI settings.
Enabling Intel VT-x / hardware virtualization.
Saving the configuration.
Restarting the computer.
Starting the Kali VM again.
After enabling virtualization, the VM started successfully.


What I Learned
Through this project, I learned how to create and configure a virtual environment for cybersecurity practice.

The most important concepts I learned include:

1. NAT vs NAT Network
A standard NAT configuration and a NAT Network serve different purposes.

A NAT Network allows multiple VMs connected to the same virtual network to communicate with one another while providing network address translation for external connectivity.

This makes it useful for building a multi-machine cybersecurity laboratory.

2. Virtual Machine Networking
I learned how VirtualBox virtual network adapters connect virtual machines to different types of networks and how network configuration affects communication between machines.

3. Static IP Configuration
I learned how to configure and verify IPv4 addressing, subnet masks, gateways, and DNS settings in Kali Linux.

4. VM Snapshots
I learned that a clean snapshot should be created before performing risky or experimental activities.

This provides a known-good recovery point for future cybersecurity exercises.

5. Documentation
I learned that documenting commands, configuration, screenshots, problems, and solutions is an important part of a professional cybersecurity project.

 Security & Ethical Use
This laboratory is intended strictly for education purposes only.



Project Information
Program Name: Cybersecurity at Networkwalks | Week: 01 | Project: Cybersecurity & Pentesting Lab Setup | Repository: GitHub

