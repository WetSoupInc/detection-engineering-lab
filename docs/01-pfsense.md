# pfSense Firewall

## Overview
pfSense is the gateway between the isolated lab network and the outside world. Every lab VM uses it as its default gateway, so all traffic leaving the lab passes through it.

## Why it's needed
It enforces the segmentation. The lab VMs have no direct connection to the internet, so pfSense is the single place where I control what the lab can reach. Right now egress is open while I build. The plan is to replace that with a rule I turn on only for updates and block outbound traffic the rest of the time.

## Specs
| | |
|---|---|
| OS | pfSense CE [version, from the dashboard] |
| CPU | 1 vCPU |
| RAM | 1 GB |
| Disk | [16-20] GB, UFS filesystem |
| Adapters | WAN: NAT (`VMnet8`), LAN: host-only (`VMnet2`) |

## Setup
I installed pfSense CE with the Netgate Installer, which downloads the packages during the install, so the WAN adapter needed internet access. I matched each adapter to WAN or LAN by its MAC address, then set the LAN to `172.16.10.1/24` with DHCP turned off (the domain controller will hand out addresses instead). I manage the firewall from my desktop at `https://172.16.10.1` through the VMware host adapter on `VMnet2` (`172.16.10.2`).

![WAN and LAN interfaces](images/pfsense-interfaces.png)

[Firewall rules and why you chose them. Currently the default LAN allow rule. WAN "Block private networks" unticked because VMware's NAT subnet is private: confirm you did this.]

![Firewall rules](images/pfsense-firewall-rules.png)

## Notes

### Troubleshooting: the web GUI timed out but ping worked

**Symptom:** I could ping the firewall's LAN address, but the web GUI timed out in every browser (`ERR_CONNECTION_TIMED_OUT`).

**How I narrowed it down:**
- Since every browser failed the same way, it wasn't a browser problem.
- `Test-NetConnection 10.0.10.1 -Port 443` in PowerShell returned `TcpTestSucceeded : False`, so TCP wasn't getting through even though ping worked.
- The first ping returned "Destination host unreachable" from my own VMnet2 address, and `arp -a` had no entry for the firewall under the VMnet2 interface. That meant my desktop couldn't find the firewall on that network at layer 2.
- A packet capture on pfSense saw no port 443 traffic arriving, and the web server was listening, so the requests weren't reaching it.
- `tracert` to the firewall's address went out through campus routers instead of staying on the virtual network, which made me suspect an address overlap with `10.x` ranges on the school network.

**Root cause:** The LAN adapter in the VM settings had been left disconnected (Connect at power on was unticked), so nothing from my desktop could reach pfSense. The pings that seemed to work weren't proof the path was good.

**Fix:** I reconnected the adapter. At the same time I moved the lab from `10.0.10.0/24` to `172.16.10.0/24` to avoid any overlap with campus and VPN networks. I made both changes at once, so I can't say for certain which one fixed it, but the disconnected adapter explains the symptoms by itself. I never fully explained why traffic for the lab address left through Wi-Fi, though the campus overlap is the likely reason.

**What I'd do differently:** Check the VM's adapter settings first, before looking at routes, firewalls, or the network. Checking the simplest layer first would have saved time.

### Other notes
- [Anything else you want to add: what you learned about WAN vs LAN, NAT vs host-only, etc.]
