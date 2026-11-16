# DC01: Domain Controller

## Overview
[Windows Server, Active Directory and DNS for the lab domain]

## Why it's needed
[Gives the lab a realistic domain environment: authentication, users, and the Kerberos/LDAP traffic that attacks often target and detections watch]

## Specs
| | |
|---|---|
| OS | [Windows Server version] |
| CPU | [x] vCPU |
| RAM | [x] GB |
| Disk | [x] GB |
| IP | 10.0.10.10 (static) |

## Setup
[Domain name, promotion to DC, test users or OUs you created and why]

![Domain promotion](images/dc01-domain-promotion.png)

[Sysmon and Universal Forwarder install]

![Forwarder sending to Splunk](images/dc01-forwarder.png)

## Notes
[Problems, lessons]
