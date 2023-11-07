# Network Monitoring, Security Testing, and SIEM Implementation
![Detection Engineering / SOC](https://docs.google.com/drawings/d/e/2PACX-1vQlf6JkCcMwXWq2LWWQAZniB1o0Q8zOZ8YyfQX6tfYgshRBHAhe4jLI9v2ZfufrcdsfJMqkF-Latgpe/pub?w=960&h=720)

## Introduction

Designed and implemented a network monitoring solution using Zeek on an Ubuntu VM for real-time traffic analysis. Conducted ethical hacking and security testing on Windows 11 systems, specializing in keylogging, nmap scans, and OWASP ZAP assessments. Configured Windows 11 machines with Sysmon to strengthen log analysis capabilities. Established an Elastic Stack-based SIEM system for centralized log aggregation, enhancing network security and threat detection. Crafted advanced queries, searches, and alerts, including threshold alerts, to facilitate detailed traffic monitoring. Meticulously applied MITRE rules to detections for precise categorization. Additionally, executed atomic tests with the Atomic Red Team framework and contributed to the development of custom tests, demonstrating a commitment to comprehensive security testing and threat detection.

- VirtualBox
- Virtual Machines (Windows 11, Ubuntu, ParrotOS)
- Zeek
- Sysmon
- Elastic
- Nmap, ZAP, Nikto

## Attack Scenario 1
![Architecture Diagram](https://docs.google.com/drawings/d/e/2PACX-1vS7tQuuYhOwpr3IlI2Uq00ef5vEIgyZZU954Z1rJR920bUkwW0pp12TQdETXnQE2NQUM5dIYmJM-9Tj/pub?w=960&h=720)

## Attack Scenario 2
![Architecture Diagram](https://docs.google.com/drawings/d/e/2PACX-1vSsTfyoQPPKpBbPp0TCB10qKyUGELqnJj4YBoqS9aWliFM2CD92BVm2tUMCnF6ML8Cdu-pp1VMruHY8/pub?w=960&h=720)

The architecture of the mini honeynet in Azure consists of the following components:

- VirtualBox
- Virtual Machines (Windows 11, Ubuntu, ParrotOS)
- Log Analytics Workspace
- Azure Key Vault
- Azure Storage Account
- Microsoft Sentinel

For the "BEFORE" metrics, all resources were originally deployed, exposed to the internet. The Virtual Machines had both their Network Security Groups and built-in firewalls wide open, and all other resources are deployed with public endpoints visible to the Internet; aka, no use for Private Endpoints.

For the "AFTER" metrics, Network Security Groups were hardened by blocking ALL traffic with the exception of my admin workstation, and all other resources were protected by their built-in firewalls as well as Private Endpoint

## Attack Maps Before Hardening / Security Controls
![NSG Allowed Inbound Malicious Flows](https://i.imgur.com/1qvswSX.png)<br>
![Linux Syslog Auth Failures](https://i.imgur.com/G1YgZt6.png)<br>
![Windows RDP/SMB Auth Failures](https://i.imgur.com/ESr9Dlv.png)<br>

## Metrics Before Hardening / Security Controls

The following table shows the metrics we measured in our insecure environment for 24 hours:
Start Time 2023-03-15 17:04:29
Stop Time 2023-03-16 17:04:29

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 19470
| Syslog                   | 3028
| SecurityAlert            | 10
| SecurityIncident         | 348
| AzureNetworkAnalytics_CL | 843

## Attack Maps Before Hardening / Security Controls

```All map queries actually returned no results due to no instances of malicious activity for the 24 hour period after hardening.```

## Metrics After Hardening / Security Controls

The following table shows the metrics we measured in our environment for another 24 hours, but after we have applied security controls:
Start Time 2023-03-18 15:37
Stop Time	2023-03-19 15:37

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 8778
| Syslog                   | 25
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0

## Conclusion

In this project, a mini honeynet was constructed in Microsoft Azure and log sources were integrated into a Log Analytics workspace. Microsoft Sentinel was employed to trigger alerts and create incidents based on the ingested logs. Additionally, metrics were measured in the insecure environment before security controls were applied, and then again after implementing security measures. It is noteworthy that the number of security events and incidents were drastically reduced after the security controls were applied, demonstrating their effectiveness.

It is worth noting that if the resources within the network were heavily utilized by regular users, it is likely that more security events and alerts may have been generated within the 24-hour period following the implementation of the security controls.
