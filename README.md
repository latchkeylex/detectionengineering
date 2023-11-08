# Network Monitoring, Security Testing, and SIEM Implementation
![Detection Engineering / SOC](https://docs.google.com/drawings/d/e/2PACX-1vQlf6JkCcMwXWq2LWWQAZniB1o0Q8zOZ8YyfQX6tfYgshRBHAhe4jLI9v2ZfufrcdsfJMqkF-Latgpe/pub?w=960&h=720)

## Introduction

Designed and implemented a network monitoring solution using Zeek on an Ubuntu VM for real-time traffic analysis. Conducted ethical hacking and security testing on Windows 11 systems, specializing in keylogging, nmap scans, and OWASP ZAP assessments. Configured Windows 11 machines with Sysmon to strengthen log analysis capabilities. Established an Elastic Stack-based SIEM system for centralized log aggregation, enhancing network security and threat detection. Crafted advanced queries, searches, and alerts, including threshold alerts, to facilitate detailed traffic monitoring. Meticulously applied MITRE rules to detections for precise categorization.
- VirtualBox
- Virtual Machines (Windows 11, Ubuntu, ParrotOS)
- Zeek
- Sysmon
- Elastic (SIEM)
- Kusto Query Language (KQL)
- Nmap, ZAP, Nikto
- Powershell
- Command Line Interface (CLI)

## Begin Lab Setup
To set up the lab, I began by installing the mentioned virtual machines using VirtualBox. I created a default snapshot (in VBox) to save their initial, unaltered state. Next, I disabled Windows Defender to enhance the lab environment capabilties. Following that, I installed Zeek on our Ubuntu machine to monitor network traffic.
Afterward, I registered for a trial version of Elastic and integrated Agents into my Windows and Ubuntu machines through Zeek. To validate Elastic Agent Logging on Windows, I conducted tests using an EICAR file and PowerShell.
Additionally, I installed and configured Sysmon to improve logging capabilities. It's important to have quality data in both Elastic Agent and Sysmon, as they complement each other. You can utilize both rather than relying solely on one. Ensure Windows integration via Elastic to install the Agent on the Windows machine.
Finally, I performed tests with the EICAR file and enabled module logging, PowerShell script block logging, allowed all scripts for script execution, and activated PowerShell transcription. 
These steps served as prerequisites for launching the lab. Once everything was installed and set up, we were ready to initiate our first attack scenario.


## Attack Scenario 1 Diagram
![Architecture Diagram](https://docs.google.com/drawings/d/e/2PACX-1vS7tQuuYhOwpr3IlI2Uq00ef5vEIgyZZU954Z1rJR920bUkwW0pp12TQdETXnQE2NQUM5dIYmJM-9Tj/pub?w=960&h=720)

## Attack Scenario 1 Step by Step
1. Deploy Zeek on our Ubuntu machine
2. Initiate Python HTTP server on Windows machine
3. In a new PowerShell tab, create a file named "testfile.txt"
4. Log on to ParrotOS machine and access web server via browser. Locate the testfile.txt file in the servers directory
5. Look at Python Web Server traffic in Windows PowerShell terminal to see GET requests
6. Open Elastic and inspect 'event.dataset: zeek.http' query in search bar. You will see the same results as you did from viewing the Web Server activity in the PowerShell terminal
7. Return to ParrotOS machine, begin an Nmap scan of the Windows Python HTTP Server machine. The Nmap scan will validate that our Web Sever Windows Machine is running 'SimpleHTTPServer 0.6 (Python 3.11.4)'
8. Now, run a Nikto scan on the same Windows target. You will see it returns a similar output
9. Finally, execute a OWASP ZAP attack on the Windows machine hosting the Python Web Server. Go back to the Windows machine to see all of requests made by ZAP
10. Our attack is completed. Lets analyze our logs in Elastic. Use the 'event.dataset: zeek.http' query in search bar. You will see an entry that includes Nikto
11. Analyze the log and filter our columns around relevant fields
12. After analyzing our results thoroughly, this is the query we are adapting to a custom query rule in our Elastic Environment.
13. This rule/query filters through Zeek http (zeek.http) traffic and specifically looks for user_agent activity strings that include "Nmap" or "Nikto". This the step where I create the rule for our environment to perpertually recognize this behavior
14. We must surpress alerts by 'destination.ip' because if we don't, our logs will include way too many entries. We need to narrow down our results to more easily filter desired fields.
15. We must name the rule, provide a description and severity level
16. Add the MITRE tactics and techniques that pertain to this specific alert/rule. In this case, the tactic would be 'Discovery' and the technique would be 'Network Service Discovery'
17. Our rule is created :)
18. Now, we must crate a threshold rule. This allows us to minimize false positive alerts. This ensures that our team is focusing on genuine threats, saving time and resources. To do this, we are grouping by 'source.ip' and 'destination.ip' and setting our threshold to 1000.
19. We will then name and describe our rule (Triggers when more than 1000 queries to a web server have been observed in a 5 minute window)
20. I am changing the severity and default risk score
21. Then we set our MITRE techniques and edit our investigation guide.
22. Lastly, schedule rule to occur every 5 minutes
23. Now it's time to confirm our alerts!
24. As you can see, an alert will populate recognizing our nmap scan 'Web Scanner Activity'. Click to view details
25. Here are our alert insights showing the reason for the alert
26. Naviate back to our Rules page and allow time to pass before seeing another alert appear
27. Now our alerts for nmap and nikto will populate. If we open the alert and view the JSON file. We can see this was our Nmap detection
28. Our threshold alert has also surfaced, we can tell what type of rule it was here
29. Lastly, change all alerts from open to closed. We have completed attack scenario one!


## Attack Scenario 2
![Architecture Diagram](https://docs.google.com/drawings/d/e/2PACX-1vSsTfyoQPPKpBbPp0TCB10qKyUGELqnJj4YBoqS9aWliFM2CD92BVm2tUMCnF6ML8Cdu-pp1VMruHY8/pub?w=960&h=720)

