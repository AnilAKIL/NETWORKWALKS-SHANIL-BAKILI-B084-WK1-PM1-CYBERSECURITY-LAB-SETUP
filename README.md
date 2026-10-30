# NETWORKWALKS-SHANIL-BAKILI-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP
Cybersecurity lab built with VirtualBox and Kali Linux for ethical hacking practice. Networkwalks Week 1 project.
**Building an isolated virtual lab for ethical hacking and penetration testing practice**

## 📌 Project Overview

This project sets up a **virtual cybersecurity testing lab** on my own laptop using VirtualBox and Kali Linux.

The lab is a controlled, isolated environment where security tools, network scanning and ethical hacking techniques can be practised safely and repeatedly, without touching any real systems. It runs on a private virtual network, so more machines can be added later as targets for authorised testing.

> ⚖️ **Legal note:** Everything in this lab is performed on my own virtual machines in my own lab environment. Testing systems you do not own or have written permission to test is illegal.

**Author:** Shanil Bakili
**Program:** Networkwalks Cybersecurity Program
**Task:** WK1-PM1, Lab Setup (VirtualBox + Kali Linux)

---

## 🎯 Objectives

- [x] Use VirtualBox as the base platform
- [x] Set up Kali Linux as the attacking machine
- [x] Create a NAT Network on subnet `10.0.0.0/24`
- [x] Enable shared clipboard and drag-and-drop
- [x] Share the host Downloads folder with the VM
- [x] Configure Kali with the static IP `10.0.0.2/24`
- [x] Confirm full internet access from Kali
- [x] Take a snapshot of the working VM

---

## 🏗️ Lab Architecture

| Component | Details |
|---|---|
| Host OS | Windows |
| Hypervisor | Oracle VirtualBox (version: 7.x) |
| Attacker VM | Kali Linux 2026.2 (64-bit, official VirtualBox image) |
| Network type | NAT Network (`NatNetwork`) |
| Subnet | `10.0.0.0/24` |
| DHCP | Enabled on the NAT Network |
| Kali IP | `10.0.0.2/24` (static) |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |

```
          Windows host (VirtualBox)
                     |
        NatNetwork  10.0.0.0/24
                     |
     Kali Linux  10.0.0.2/24  ---->  Internet
     (other VMs can be added later)
```

---

## 🛠️ Setup Steps

### Step 1: Install 7-Zip
Downloaded and installed 7-Zip from [7-zip.org](https://7-zip.org/download.html), as needed to extract the Kali image.

### Step 2: Install VirtualBox
Downloaded the latest VirtualBox from [virtualbox.org](https://virtualbox.org/wiki/Downloads) and installed it with the default options.

### Step 3: Create the NAT Network
In VirtualBox: **File → Tools → Network → NAT Networks → Create**

- Name: `NatNetwork`
- IPv4 Prefix: `10.0.0.0/24`
- DHCP: enabled

![NAT Network](screenshots/Screenshot_nat_network)

### Step 4: Download and import Kali Linux
1. Downloaded the **64-bit VirtualBox image** from [kali.org/get-kali](https://kali.org/get-kali)
2. Extracted it with 7-Zip
3. Double-clicked the `.vbox` file to import the VM into VirtualBox

![Kali imported](screenshots/Screenshot_Kali_VM.png)

### Step 5: Configure the VM settings (VM powered off)

**Network:** Adapter 1 attached to **NAT Network** (`NatNetwork`), Promiscuous Mode set to **Allow All**.

![Network settings](screenshots/Screenshot_kali_network_settings)

**Clipboard and drag-and-drop:** both set to **Bidirectional** (Settings → General → Features).

![Clipboard and drag-drop](screenshots/Screenshot_clipboard_dragdrop)

**Shared folder:** the host `Downloads` folder shared with Auto-mount and Make Global enabled.

![Shared folder](screenshots/shared-folder)

### Step 6: Set the static IP on Kali

```bash
# Set a static IP, gateway and DNS on the wired connection
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 10.0.0.2/24 \
  ipv4.gateway 10.0.0.1 \
  ipv4.dns 8.8.8.8

# Stop NetworkManager waiting on duplicate address detection
# (fixes a common no-internet issue on VirtualBox 7 + newer Kali)
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0

# Restart the connection so the changes apply
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

### Step 7: Verify the lab

```bash
cat /etc/os-release     # Kali version
ip a                    # confirm 10.0.0.2/24 on eth0
ping -c 4 10.0.0.1      # gateway reachable
ping -c 4 8.8.8.8       # internet by IP
ping -c 4 google.com    # internet + DNS
ls /media               # shared folder mounted (sf_Downloads)
```

![Kali version](screenshots/Screenshots_kali_version)
![Static IP on eth0 & Ping tests](screenshots/Screenshots_ip_&_ping)
![Shared folder mounted](screenshots/Screenshots_shared_folder_mounted)

All pings returned with **0% packet loss**, confirming the static IP, gateway and full internet access.

### Step 8: Snapshot
Took a snapshot of the working VM so I can restore a clean, known-good state at any time.

![Snapshot]

---

## ✅ Results

| Requirement | Status |
|---|---|
| VirtualBox as base | ✅ |
| Kali Linux attacker VM | ✅ Kali 2026.2 |
| NAT Network `10.0.0.0/24` | ✅ |
| Clipboard and drag-and-drop | ✅ Tested, working |
| Shared Downloads folder | ✅ `sf_Downloads` mounted |
| Kali IP `10.0.0.2/24` | ✅ |
| Full internet access | ✅ |
| Snapshot taken | ✅ |

---

## 🧩 Troubleshooting Experience

**1. Forgot the login password on my existing Kali VM**
- *Problem:* I had an older Kali VM from earlier practice and could not log in.
- *Solution:* The old VM was built by hand and was likely out of date, so I imported the **official Kali 2026.2 VirtualBox image** instead. Its default login worked straight away. A password can also be reset from the GRUB menu.

**2. Shared the wrong folder**
- *Problem:* The path I first picked pointed at the Kali VM's own folder (inside Downloads), not the Downloads folder itself.
- *Solution:* Shared the top-level `Downloads` folder instead, with Auto-mount and Make Global enabled.

**3. Copied a Word document, but it would not open in Kali**
- *Problem:* Drag-and-drop worked, but the `.docx` file would not open.
- *Solution:* Kali has no Word. The file transfer was fine; opening `.docx` needs LibreOffice (`sudo apt install libreoffice-writer`).

**4. Potential no-internet issue (known)**
- *Problem:* VirtualBox 7 with newer Kali versions can lose connectivity after a static IP is set.
- *Solution:* Disable duplicate address detection (`ipv4.dad-timeout 0`) and restart the connection, as shown in Step 6. Switching the DNS between `8.8.8.8` and `10.0.0.1` can also help.

---

## 📚 What I Learned

- How a NAT Network gives VMs their own private subnet while keeping internet access
- Configuring static IPs on Linux with `nmcli`
- How VirtualBox shared folders, clipboard and drag-and-drop work
- Why snapshots matter for safe, repeatable lab work
- Why vulnerable machines must stay on an isolated network and never on a real one

---

## 🔭 Next Steps

- Add vulnerable target machines (e.g. Metasploitable2, SickOS) to the same NAT Network
- Practise network scanning and reconnaissance from Kali
- Build more lab machines in later project weeks

---

## ⚠️ Disclaimer

This repository is for **education and research only**. All activity takes place inside a virtual lab on devices I own. Never test systems without written permission from the owner.

---

*Built as part of the Networkwalks Cybersecurity Program.*
