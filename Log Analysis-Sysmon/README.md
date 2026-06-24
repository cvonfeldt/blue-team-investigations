# Overview: 
**We are provided with Sysmon logs from a compromised endpoint, and we need to analyze the logs to answer the prompt questions to ultimately find out the steps and techniques used by the attacker.**

<br> 

![Inv](screenshots/Sys0.png)

<br>

**This is a unique challenge because we are tasked with manually sifting through the sysmon log file without any sort of SIEM or logging aggregate tool, so I will instead use powershell to analyze the logs.**

<br>

## Investigation:

### 1. What is the file that gave access to the attacker?

So for this one I'm going to first look for command line execution (Event ID 1) and look for a suspicious parent process or spawning file since I know that's a super common initial access vector. 

First I tried this powershell command to convert the log file into individual events (handy powershell tool), then output important info about the parent and child image: 

```powershell
Get-Content "sysmon-events.json" | ConvertFrom-Json | Where-Object { $_.Event.System.EventID -eq 1 } | Select-Object -ExpandProperty Event | Select-Object -ExpandProperty EventData | Select-Object Image, ParentImage, CommandLine | Format-List
```

That gave me errors since the JSON objects weren't wrapped into proper arrays (needs to have a comma after each event and for all of it to be wrapped in brackets), so I ran this to wrap them into a proper array first, then give us the important info we need: 

```powershell
Get-Content "sysmon-events.json" -Raw | ForEach-Object { $_ -replace '}\s*{', '},{' } | ForEach-Object { "[$_]" } | ConvertFrom-Json | Where-Object { $_.Event.System.EventID -eq 1 } | ForEach-Object { $_.Event.EventData } | Select-Object Image, ParentImage, CommandLine | Format-List
```

So I know the events are in chronological order from the top down so we start at the top, and we don't have to scroll far down the list to see suspicious activity: a powershell instance spawned from mshta.exe with an obviously encoded command used in the commandline. 

![Q1](screenshots/Sys1.png)

We can see right above it that a file downloaded from chrome called "updater.hta" was run by mshta.exe in the commandline - we can see given what followed that this is the file that initially compromised the machine. 

**Answer: updater.hta**

---


### 2. What is the powershell cmdlet used to download the malware file and what is the port? 

Given that a cmdlet was used, we know this will also have Event ID 1 so we can just keep scrolling down. Looks like we see it here:

![Q2](screenshots/Sys4.png)

We see the cmdlet is: "INvoke-WebRequest" - which allows interaction with a web page like file downloads, parsing, API interaction, etc. - and the port is 6969. The malware file downloaded appears to be supply.exe. 

**Answer: INvoke-WebRequest, 6969**

---

### 3. What is the name of the environment variable set by the attacker?

So for this we know that a registry change (env variable set) would have Event ID 13, but we might be able to find it with Event ID 1 if the variable was set by the malware as an argument of a cmd.exe creation. 

Scrolling down we find this exactly:

![Q3](screenshots/Sys5.png)

Where we see that the environment variable set is comspec=C:\windows\temp\supply.exe, which makes that folder the default location of command-line execution. We also see that the parent process actually isn't supply.exe, so we can assume that it's likely from the encoded Gzip compression powershell payload that spawned from updater.hta. We will keep that in mind going forward.

**Answer: comspec=C:\windows\temp\supply.exe**

---

### 4. What is the process used as a LOLBIN to execute malicious commands? 

So we know that the living off the land binary is a process used to execute malicious commands, so this should still have Event ID 1. A couple of logs later we see ftp.exe which is a legit process used to transfer files:

![Q4](screenshots/Sys6.png)

We can see that it's launched from cmd.exe, then launches a process in the location of the new comspec location of C:\windows\temp\supply.exe. This chain precisely fits the LOLBIN process we are looking for, so we know it's ftp.exe.

**Answer: ftp.exe**

---

### 5. Malware executed multiple same commands at a time, what is the first command executed?

So for this one, this still applies: we might be able to find it with Event ID 1 if the variable was set by the malware as an argument of a cmd.exe creation. 

That is indeed the case, as scrolling down we see the same commands of "ipconfig," and "whoami." However knowing that they're simultaneous, that means they'll ahve the same timestamp, so the chronological order of our Sysmon events from top-down won't necessarily apply here. Instead we will need to refine our powershell command to output EventRecordID as well to show which commands really came first:

```powershell
Get-Content "sysmon-events.json" -Raw | ForEach-Object { $_ -replace '}\s*{', '},{' } | ForEach-Object { "[$_]" } | ConvertFrom-Json | Where-Object { $_.Event.System.EventID -eq 1 -and $_.Event.EventData.ParentImage -eq "C:\windows\temp\supply.exe" } | ForEach-Object { [PSCustomObject]@{ RecordID = $_.Event.System.EventRecordID; CommandLine = $_.Event.EventData.CommandLine } } | Sort-Object RecordID | Format-List
```

This should allow us to do what our previous PowerShell command did, but this time return only events with a parent image of C:\windows\temp\supply.exe, as well as take the EventRecordIDs and sort them. However, it didn't output correctly and treated the file all as one object instead of individual objects. 

I tried this: 

```powershell
$raw = Get-Content "sysmon-events.json" -Raw
$json = "[" + ($raw -replace '}\s*{', '},{') + "]"
$events = $json | ConvertFrom-Json
$events | Where-Object { $_.Event.System.EventID -eq 1 -and $_.Event.EventData.ParentImage -eq "C:\windows\temp\supply.exe" } | ForEach-Object { [PSCustomObject]@{ RecordID = $_.Event.System.EventRecordID; CommandLine = $_.Event.EventData.CommandLine } } | Sort-Object RecordID | Format-List
```

To store each step in a variable instead of piping it all in one go, and it worked:

![Q5](screenshots/Sys7.png)

We can see the first command of the simultaneous commands is "ipconfig"

**Answer: ipconfig**

---

### 6. Looking at the dependency events around the malware, can you figure out the language the malware is written in? 

---

### 7. Malware then downloads a new file, find out the full url of the file download 

---

### 8. What is the port the attacker attempts to get reverse shell? 
