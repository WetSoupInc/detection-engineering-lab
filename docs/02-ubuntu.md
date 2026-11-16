# SPLUNK: Log Collection and Search

## Overview
[Ubuntu server running Splunk [Enterprise/Free], the central place where all logs land]

## Why it's needed
[Collects Sysmon and Windows event logs from both machines so I can write and test detections against real attack data]

## Specs
| | |
|---|---|
| OS | Ubuntu [version] |
| CPU | [x] vCPU |
| RAM | [x] GB |
| Disk | [x] GB |
| IP | 10.0.10.20 (static) |

## Setup
[Install, receiving port 9997, index you created, anything you configured on purpose]

![Receiving port enabled](images/splunk-receiver-9997.png)

![Logs arriving](images/splunk-sysmon-ingest.png)

## Notes
[Problems, lessons]
