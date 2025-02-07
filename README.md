# SNORT

![Snort Log analyze](https://github.com/user-attachments/assets/5b26f83e-f0b6-4ea9-8701-21b8f6acd208)


# Intrusion Detection and Prevention with Snort

## Project Overview
This project demonstrates the deployment of Snort, an open-source intrusion detection and prevention system (IDS/IPS). The goal is to monitor network traffic, create custom detection rules, and analyze alerts generated by Snort in real-time.

## Objectives
- Install and configure Snort on a Linux-based system
- Capture and inspect live network traffic
- Write custom rules to detect specific threats
- Generate and analyze alerts
- Enable IPS mode for active threat prevention

## Setup

### Prerequisites
- Ubuntu 22.04 (or any Debian-based Linux distribution)
- Snort installed (`apt install snort`)
- A test network with simulated attacks
- Access to a log monitoring tool (e.g., Kibana or Splunk) for analysis

### Installing Snort
```bash
sudo apt update && sudo apt install -y snort
```

### Configuring Snort
Modify the Snort configuration file to define network variables and enable logging.

```bash
sudo nano /etc/snort/snort.conf
```
Set the HOME_NET variable to match your network:
```shell
var HOME_NET 192.168.1.0/24
```
Ensure that logging is enabled:
```shell
output alert_fast: /var/log/snort/alert.log
```
Restart Snort to apply changes:
```bash
sudo systemctl restart snort
```

## Implementation

### Capturing Live Network Traffic
To monitor live traffic in IDS mode, use:
```bash
sudo snort -A fast -q -c /etc/snort/snort.conf -i eth0
```
(`eth0` should be replaced with your actual network interface.)

### Creating Custom Detection Rules
To detect SSH brute force attempts, add a custom rule in `/etc/snort/rules/local.rules`:
```shell
alert tcp any any -> any 22 (msg:"Potential SSH Brute Force"; flow:to_server,established; content:"SSH-2.0"; threshold:type both, track by_src, count 5, seconds 60; sid:1000001; rev:1;)
```
Include the rule in Snort’s configuration:
```bash
echo "include /etc/snort/rules/local.rules" | sudo tee -a /etc/snort/snort.conf
sudo systemctl restart snort
```

### Testing the Rule
Simulate an SSH brute force attack using Hydra:
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<target-ip>
```
Check Snort logs for alerts:
```bash
sudo cat /var/log/snort/alert.log
```
Expected output:
```
02/07/2025-15:45:12.123456 [**] [1:1000001:1] Potential SSH Brute Force [**] [Priority: 2] {TCP} 192.168.1.5:56789 -> 192.168.1.1:22
```

## Enabling IPS Mode
To enable IPS mode and actively block threats, Snort must be run in inline mode:
```bash
sudo snort -Q --daq afpacket -c /etc/snort/snort.conf -i eth0:eth1
```
Modify rules to drop malicious traffic instead of just alerting:
```shell
drop tcp any any -> any 22 (msg:"Blocking SSH Brute Force"; flow:to_server,established; content:"SSH-2.0"; threshold:type both, track by_src, count 5, seconds 60; sid:1000002; rev:1;)
```
Restart Snort and monitor the logs:
```bash
sudo systemctl restart snort
sudo cat /var/log/snort/alert.log
```


This project successfully implemented Snort as both an IDS and an IPS, detecting and mitigating SSH brute force attacks in real-time. Future improvements include integrating Snort with a SIEM solution for enhanced visibility and automated responses to threats.
