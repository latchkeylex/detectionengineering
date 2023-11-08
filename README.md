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

## Attack Scenario 2 Step by Step
Summary. The droppers purpose is to see if we have windows defender enabled or not on our machine. There is nothing malicious about the file. If the defender is enabled, the dropper isnt going to go and download the return shell. We want to be sneaky, so were trying to say download the dropper, execute it, but only download the malicious return shell if defender is disabled. 
Then we execute that downloaded return shell, then check out logs in elastic, create some detections and rerun the test. 
Overlaying the detections were going to create, were gonna have a detection on the inital dropper thats downloaded, on the return shell thats downloaded, and the shell execution. 

1. Check Windows machine to see if Defender, RealTimeProtection, and other attributes are enabled
2. If we run this command, we will filter to see if 'RealTimeProtectionEnabled' is 'False' (disabled) within our Get-MPComputerStaus output. If this is true, it will produce the text 'property is false'. Now that we see it is false and RealTimeProtection is turned off, we can proceed to the next step
3. Now it's time to expound on our previous command. We're using Notepad++ to create our batch file test. Start bat files with "@ECHO off" because the bat file will not just print the command on the screen, we dont need to see the commands repeated back to us in the console, then insert your code in the powershell line. (If you type out this script and save it as 'testing.bat', you will see the text change color). Make sure it says '-like 'False' and not 'True' in the script
4. Double click the testing.bat file to run it. You will see output in command line that reads...
5. Go back into Notepad++ and edit the 'testing.bat' file. Delete the 'pause' line. This turns off the 'press any key to continue' in the command line
6. Time to start out attacks. Open ParrotOS machine and create a file called test.txt
7. Start an HTTP server in our ParrotOS machine by typing...
8. Now that our server is initalized, open our Windows machine and run this PowerShell command... What 'Invoke-WebRequest' does is retrieve information from web pages, in this case, our web server. We enter the IP of our ParrotOS machine, the port number our server is hosted on, and the file we want to retrieve from the server. We then say where we want the file we're retrieving to be saved on our Windows machine (c:\...)
9. If we navigate to the path we described in the previous photo, you will see the file was successfully retrieved from the web server
10. Now, we will simply replace our previous shell command with our 'Invoke-WebRequest' to automate the retrieval of the test.txt file from the web server. Delete the pause from the end of the script. Double click the 'testing.bat' file to run it again
11. The file was successfully retrieved once again
12. Now, navigate to your ParrotOS machine. We are going to write a reverse shell using msfvenom, directed at this l(listening)host (Parrot ip) and l(listening)port (8443), and save it as 'shell.bat'. You will see the 'shell.bat' file saved in our directory
13. If you 'cat' the file, you will see the payload text that was written to the 'shell.bat' file. This is the code for the reverse shell
14. Navigate back to your Windows machine and replace 'testing.bat' with 'shell.bat'. This will retrieve the 'shell.bat' file and save it to the listed Windows path
15. Confirm the 'shell.bat' file retrieval in the 'Temp' directory
16. Now it's time to launch our reverse shell attack on our Windows machine. Open your ParrotOS machine and type...
17. We are telling Metasploit we want to use a reverse shell payload
18. Type 'options' to view payload options. We are essentially choosing the first option, since we are specifying what port we would like to use, instead of using 'port 4444'
19. We are setting the 'lhost' and 'lport' to what we listed in the previous msfvenom configuration photo
20. Finally, run it!
21. Navigate to your Windows machine and make sure you insert 'shell.bat' accordingly in the 'testing.bat' file. (If you notice the semi colon after the first Windows path, this is how you execute multiple tasks in a single PowerShell script)
22. You will see the following output. This is the code from the payload of the reverse shell we saw earlier!
23. Navigate back to ParrotOS machine and, voila... You should have successfully executed the reverse shell! Validate your entry by running 'whoami'
24. Now, exit the command shell by typing 'exit'
25. Traditionally, we would like to be more stealthy about executing '.bat' files. With that being said, let's mask our bat files as text files
26. Re-run the multi handler Metasplot command option
27. It will remember our lhost and lport, just type 'run' to rerun it
28. Now, run 'mv shell.bat shell.txt'. This turns our bat file into a txt file
29. Go back to your Windows machine and change the InvokeRequest to retrieve 'shell.txt' but keep the saved file name as 'shell.bat'. This is how we mask the bat file. Double click 'testing.bat' to run the script
30. Ta-Da... Great success!
31. Lastly, navigate to Elastic and filter our documents for any trace of our attack scenarios
32. We can filter for sysmon results by clicking the '+' on the event.dataset field. Expand the 'sysmon' field
33. We changed our filters to look for relevant fields (process.command_line, process.parent.command_line, and process.parent.name). We will see 'shell.bat', our msfvenom payload text, powershell.exe, and more!
34. Here you can see the 'shell.bat'. This is the Metasploit reverse shell payload expanded




