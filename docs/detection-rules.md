# Detection Rules

## Overview
This document describes the detection logic used in the Packetbeat SIEM lab.

## Detection Cases
| Rule Name | Suspicious Behavior | Data Source | Example Query | Analyst Action |
|---|---|---|---|---|
| SSH brute force | Multiple failed SSH logins from one source | filebeat + packetbeat | ... | Check source IP, count failures, block if needed |
| Nmap scan | Many connection attempts across ports in a short time | packetbeat | ... | Confirm scan, isolate source |
| Telnet access | Telnet traffic to port 23 | packetbeat | ... | Validate if Telnet is approved |
| FTP access | Legacy FTP traffic to port 21 | packetbeat | ... | Check if file transfer is expected |
| ICMP sweep | Repeated ICMP echo requests to multiple hosts | packetbeat | ... | Verify recon activity |

## Rule Details
...
# SSH brute force
event.dataset: "system.auth" and message: ("Failed password" or "authentication failure")

# Packetbeat SSH traffic
network.transport: "tcp" and destination.port: 22

# FTP
destination.port: 21

# Telnet
destination.port: 23

# ICMP
network.protocol: "icmp"
