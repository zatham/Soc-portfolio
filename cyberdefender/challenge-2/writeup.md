# Oski Lab - Malware Analysis
**Platform:** CyberDefender | **Difficulty:** Easy
**Category:** Threat Intel | **Date:** May 2026
**Score:** 7/7

## What Was This About?
A company accountant got a phishing email called "Urgent New 
Order". When he opened the attached file the SIEM flagged it 
as suspicious. My job was to dig into the malware and figure 
out what it was doing.

## Tools I Used
- VirusTotal (to check the file and find network connections)
- Any.run (to watch the malware run live in a sandbox)

## How I Investigated

**Step 1 - Checking the file on VirusTotal**
I started by searching the file hash on VirusTotal. It came 
back with 62 out of 71 vendors flagging it as malicious. I 
then went to the Details tab and found the History section 
which showed the malware was created on 2022-09-28 at 17:40.

**Step 2 - Finding the C2 server**
I clicked on the Relations tab in VirusTotal and scrolled to 
Contacted URLs. I found two suspicious URLs both pointing to 
the same IP address 171.22.28.221. The main C2 server was:
http://171.22.28.221/5c06c05b7b34e8e6.php

**Step 3 - Analyzing behavior on Any.run**
I opened the Any.run sandbox report and watched the malware 
run. In the Files tab under Behavior I could see the first 
library it requested was sqlite3.dll which is used for 
database access — meaning it was going after stored data.

**Step 4 - Finding the RC4 key**
Still in Any.run I dug into the malware configuration and 
found the RC4 key it uses to decrypt its base64 strings:
5329514621441247975720749009

**Step 5 - Mapping to MITRE ATT&CK**
In the process details panel on Any.run I could see the 
MITRE techniques listed. The main technique for stealing 
passwords was T1555 (Credentials from Password Stores) with 
the sub-technique T1555.003 targeting web browsers.

**Step 6 - Finding where DLLs get deleted**
I looked at the child processes in Any.run and saw cmd.exe 
running a command that deleted files from C:\ProgramData — 
this is how the malware covers its tracks.

**Step 7 - Self-delete timer**
The cmd.exe process used timeout.exe with the command 
"timeout /t 5" meaning the malware waits 5 seconds before 
deleting itself after stealing the data.

## What I Found
- **Malware creation time:** 2022-09-28 17:40
- **C2 server:** http://171.22.28.221/5c06c05b7b34e8e6.php
- **First library loaded:** sqlite3.dll
- **RC4 key:** 5329514621441247975720749009
- **MITRE technique:** T1555 - Credentials from Password Stores
- **DLL deletion directory:** C:\ProgramData
- **Self-delete timer:** 5 seconds

## What I Learned
This was my first time using Any.run to analyze live malware 
behavior. Watching the malware actually run and seeing the 
network connections in real time was really eye opening. I 
also learned how to map what I see to MITRE ATT&CK which is 
something SOC analysts do every day to categorize threats.
