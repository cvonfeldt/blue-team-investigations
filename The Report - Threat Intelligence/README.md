# Overview: 
## We are provided with Red Canary's 2022 threat report and tasked with analyzing it to answer questions that help build actionable intel for a newly established SOC.

![Inv](screenshots/Inv)

<br>

## This challenge is unique because rather than hands-on log analysis or forensics, it requires careful reading and interpretation of a real-world threat report - extracting specific techniques, CVEs, adversary groups, and detection insights to inform SOC operations.

<br>

## Investigation:

### 1. Name the supply chain attack related to Java logging library in the end of 2021.
![Inv](screenshots/Inv.png)

We can see in the table of contents that page 14 is where the supply chain compromise is located, so we will go there:

![Q1](screenshots/1.png)

We see the supply chain attack at the end of 2021 (December) was called Log4j which was a java logging library for third party apps that was compromised to allow remote code execution.

**Answer: Log4j**

---


### 2. Mention the MITRE Technique ID which effected more than 50% of the customers. 
In the table of contents we see techniques is on page 71, so we will scroll to that:

![Q2](screenshots/2.png)

We see the technique that affected more than 50% of customers (53.1%) was T1059: Command Scripting Interpreter, and it was split into powershell (35%) and command line (28.1%). This gives us a good idea of how dangerous command line scripting can be in compromising systems. 

**Answer: T1059**

---

### 3. Submit the names of 2 vulnerabilities belonging to Exchange Servers.
We see vulnerabilities on page 17 on the TOC:
![Q3](screenshots/3.png)

and going to that we see there is 1 vuln called ProxyLogon that belonged to exhange servers, which was actually 4 vulns chained together that ultimately led to remote access and execution on an exchange server. Adversaries dropped web shells to exfiltrate data from thousands of exchange servers.
![Q3](screenshots/4.png)

The next is ProxyShell, which very similarly allowed attackers to execute remote code without auth leading to system and network compromise


**Answer: ProxyLogon, ProxyShell**

---

### 4. Submit the CVE of the zero day vulnerability of a driver which led to RCE and gain SYSTEM privileges.
Scrolling down we see PrintNightmare: 
![Q4](screenshots/5.png)

Which was a zero day that allowed remote users to gain elevated access using the printspooler (users loading print driver DLLs).

**Answer: CVE-2021-34527**

---

### 5. Mention the 2 adversary groups that leverage SEO to gain initial access.

**Answer:**

---

### 6. In the detection rule, what should be mentioned as parent process if we are looking for execution of malicious js files.


**Answer:**

---

### 7.  Ransomware gangs started using affiliate model to gain initial access. Name the precursors used by affiliates of Conti ransomware group.

**Answer:**

---

### 8. The main target of coin miners was outdated software. Mention the 2 outdated software mentioned in the report.

**Answer:**


---

### 9. Name the ransomware group which threatened to conduct DDoS if they didn't pay ransom

**Answer:**

---

### 9.  What is the security measure we need to enable for RDP connections in order to safeguard from ransomware attacks? 

**Answer:**

---

**Completed:**
