# VM Setup Checklist

Build order: **VMware network → pfSense → DC01 → WIN-CLIENT → SPLUNK → tools**. A VM is only done when it passes its test. Snapshot after each pass, then flip its row in the README.

## Summary

| VM | OS (download) | vCPU / RAM / Disk | Firmware | VMware adapter(s) | IP |
|---|---|---|---|---|---|
| pfSense | pfSense CE, amd64 ISO (Netgate; may arrive as `.iso.gz`, extract with 7-Zip) | 1 / 1 GB / 16-20 GB | BIOS | NIC 1: NAT (`VMnet8`) = WAN; NIC 2: `VMnet2` = LAN | LAN `10.0.10.1/24` |
| DC01 | Windows Server 2022 Standard (Desktop Experience), 180-day evaluation ISO (Microsoft Evaluation Center) | 2 / 4 GB / 60 GB | BIOS or UEFI | `VMnet2` only | `10.0.10.10/24` static |
| WIN-CLIENT | Windows 11 Enterprise, evaluation ISO (Microsoft Evaluation Center) | 2 / 4 GB / 60 GB | UEFI + vTPM | `VMnet2` only | DHCP from DC (`10.0.10.100-200`) |
| SPLUNK | Ubuntu Server 24.04 LTS (x86-64 ISO), plus Splunk Enterprise for Linux (`.deb` or `.tgz`, free splunk.com account) | 4 / 8 GB / 80+ GB | BIOS | `VMnet2` only | `10.0.10.20/24` static |

Notes on the OS choices:
- **Windows 11 Enterprise eval** is the client because it is the current version and the eval ISO is reliably available. Windows 10 Enterprise evals have been inconsistent to find. If you'd rather avoid the TPM step, Windows 10 Enterprise LTSC 2021 evaluation is reportedly still available, but verify on the Evaluation Center first.
- **Ubuntu Server 24.04 LTS** is listed as a supported Splunk Enterprise platform. Don't use a newer Ubuntu release until Splunk's supported-OS page lists it.
- Evaluation copies expire (Server about 180 days, Windows client about 90). That's fine for a lab, but note the install dates in the build log.

Record your decisions here:
- AD domain name: ______________
- Lab admin accounts: stored in a password manager (never in the repo)
- ISO download dates: ______________

## 0. VMware network (once)

- [ ] Virtual Network Editor: add `VMnet2`, **host-only**, subnet `10.0.10.0/24`, **DHCP off**
- [ ] Decide how the host reaches the lab. VMware's host adapter may grab `.1` (pfSense's address). Either set the host adapter to a static `10.0.10.2`, or untick "Connect a host virtual adapter" and manage things from inside the VMs
- [ ] For every VM: remove shared folders, disable clipboard and drag-and-drop, install VMware Tools (or `open-vm-tools` on Ubuntu)

## 1. pfSense (gateway and firewall)

- [ ] New VM, guest OS type FreeBSD 64-bit, 1 vCPU, 1 GB RAM, 16-20 GB disk
- [ ] Add a second network adapter. NIC 1 = NAT (WAN), NIC 2 = `VMnet2` (LAN)
- [ ] Install pfSense. When assigning interfaces, check the MAC addresses so WAN and LAN aren't swapped
- [ ] LAN: `10.0.10.1/24`. **Turn LAN DHCP off** (the DC does DHCP)
- [ ] WAN: untick "Block private networks" (VMware's NAT subnet is private)
- [ ] Firewall: allow LAN-out at first. Later, replace it with a rule you toggle for updates only, and block outbound the rest of the time
- [ ] **Test:** from a LAN VM, ping `10.0.10.1` and load the pfSense web GUI
- [ ] Snapshot: `pfsense-baseline`

## 2. DC01 (Windows Server 2022)

- [ ] New VM, 2 vCPU, 4 GB RAM, 60 GB disk, adapter `VMnet2`
- [ ] Install Server 2022 Standard with Desktop Experience. Rename to `DC01`
- [ ] Static IP `10.0.10.10/24`, gateway `10.0.10.1`, DNS `10.0.10.10` (itself)
- [ ] Install AD DS + DNS, promote to a new forest, create OUs, groups, and test users (script this with PowerShell and a CSV)
- [ ] Install DHCP: scope `10.0.10.100-200`, router option `10.0.10.1`, DNS option `10.0.10.10`, then authorize it
- [ ] DNS forwarder: `10.0.10.1` (pfSense), so external names resolve
- [ ] **Test:** ping `10.0.10.1`; `nslookup` your domain; resolve `microsoft.com` (only while egress is allowed)
- [ ] Snapshot: `dc01-ad-built`

## 3. WIN-CLIENT (Windows 11 Enterprise)

- [ ] New VM, 2 vCPU, 4 GB RAM, 60 GB disk, UEFI firmware, adapter `VMnet2`
- [ ] Add a Trusted Platform Module device (Workstation requires encrypting the VM to enable this; store the encryption password in your password manager, not in the repo)
- [ ] Install Windows 11 Enterprise. Rename to `WIN-CLIENT`
- [ ] **Test before joining:** it received a `10.0.10.1xx` address, gateway `.1`, DNS `.10`. If not, fix DHCP on the DC first
- [ ] Join the domain, log in as a domain user
- [ ] If the join fails: check DNS first, then clock skew
- [ ] Snapshot: `client-domain-joined`

## 4. SPLUNK (Ubuntu Server 24.04)

- [ ] New VM, 4 vCPU, 8 GB RAM, 80+ GB disk, adapter `VMnet2`
- [ ] Install Ubuntu Server (minimal, with OpenSSH). Hostname `splunk`
- [ ] Static IP `10.0.10.20/24` via netplan, gateway `10.0.10.1`, DNS `10.0.10.10`
- [ ] Install `open-vm-tools`
- [ ] Install Splunk Enterprise, enable boot-start, create the admin account
- [ ] Enable receiving on port 9997 and create your index (for example `wineventlog` or a lab index of your choice)
- [ ] If you enable `ufw`, allow 9997 (forwarders), 8000 (web UI), and 22 (SSH)
- [ ] **Test:** `ss -tlnp` shows 9997 and 8000 listening, and the web UI loads from the host or client
- [ ] Snapshot: `splunk-installed`

## 5. Tools and verification

- [ ] Install Sysmon on DC01 and WIN-CLIENT with your chosen config (record the source in `configs/` and the README)
- [ ] Install the Universal Forwarder on both. `outputs.conf` points to `10.0.10.20:9997`, and `inputs.conf` collects Security, Sysmon, and PowerShell logs
- [ ] **Verify:** `index=* | stats count by host, sourcetype` shows events from both hosts
- [ ] Snapshot everything as a clean baseline
- [ ] Install Atomic Red Team on the client last, after logs are flowing, and add the Defender exclusion for its test folder only (document it)

## Connectivity ladder

When something doesn't work, climb this in order. The first rung that fails is the problem.

1. Ping the gateway (`10.0.10.1`)
2. Ping another VM
3. Resolve a DNS name (`nslookup`)
4. Reach the internet (only while egress is allowed)

## Troubleshooting notes

Add anything that breaks here, then copy the good entries into the README's troubleshooting section.

| Date | VM | Symptom | Root cause | Fix |
|---|---|---|---|---|
| | | | | |
