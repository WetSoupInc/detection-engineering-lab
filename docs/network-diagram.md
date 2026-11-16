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


```mermaid
flowchart LR
  subgraph Lab["Isolated lab network 10.0.10.0/24"]
    DC["DC01<br/>10.0.10.10<br/>AD, DNS, Sysmon, UF"]
    CL["WIN-CLIENT<br/>DHCP<br/>Sysmon, UF, Atomic Red Team"]
    SP["SPLUNK<br/>10.0.10.20<br/>Ubuntu"]
    CL <-->|DNS, Kerberos, LDAP| DC
    DC -->|9997| SP
    CL -->|9997| SP
  end
  GW["pfSense"] -.->|updates only| NET((Internet))
  GW --- Lab
```
