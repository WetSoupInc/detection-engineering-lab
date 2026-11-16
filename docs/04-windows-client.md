# WIN-CLIENT: Attack Target

## Overview
[Domain-joined Windows machine where Atomic Red Team tests run]

## Why it's needed
[Plays the role of a user workstation, the most common starting point for real attacks, and produces the endpoint logs the detections are built on]

## Specs
| | |
|---|---|
| OS | [Windows 10/11 version] |
| CPU | [x] vCPU |
| RAM | [x] GB |
| Disk | [x] GB |
| IP | DHCP from [source] |

## Setup
[Domain join, Sysmon with my config, Universal Forwarder, Atomic Red Team install]

![Domain join](images/win-client-domain-join.png)

![Sysmon running](images/win-client-sysmon.png)

## Notes
[Defender exclusions, snapshots taken, problems]
