

```mermaid
flowchart TD
    NET([Home Network / Internet]) --- PF[pfSense Firewall]
    PF --- LAN[Lab LAN]
    LAN --- DC[Windows Server<br/>Active Directory]
    LAN --- WC[Windows 10/11 Client<br/>Sysmon + Splunk Forwarder<br/>Atomic Red Team]
    LAN --- SP[Ubuntu<br/>Splunk]
    WC -. logs .-> SP
    DC -. logs .-> SP
```
