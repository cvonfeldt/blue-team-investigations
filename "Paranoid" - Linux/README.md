# Overview: 
**We are provided with Linux audit logs from a compromised endpoint, and we need to analyze the logs to answer the prompt questions to ultimately find out the steps and techniques used by the attacker.**

<br> 

![Inv](screenshots/0.png)

**This is another challenge where we are tasked with manually sifting through the log file without any sort of SIEM or logging aggregate tool, so this time we will use bash to analyze the logs.**

<br>

## Investigation:

### 1. What account was compromised?

For this one we know it's an account being compromised and not a device, so my first thought is to check for attempted logins: 
![Q1](screenshots/1.png)

We can see many successive failed attempts separated by seconds/milliseconds almost certainly indicating a brute force attack over SSH, then we finally see a successful attempt with a timestamp of 1633393393.365:467550 (October 4th 2021):
![Q1](screenshots/2.png)

We see here the account compromised is "btlo"


**Answer: btlo**

---


### 2. What attack type was used to gain initial access?
We see from above that the attack is brute force over SSH!

**Answer: Brute Force**

---

### 3. What is the attacker's IP address?
We see in the successful login log that the attacker's IP is 192.168.4.155.
**Answer: 192.168.4.155**

---

### 4. What tool was used to perform system enumeration?
For this I'm going to search for command line execution that inlcudes common enumeration tools (netcat, linpeas, nmap, etc.): grep -i 'linpeas\|nmap\|wget\|curl\|python\|perl\|ruby\|netcat' audit.log:

![Q4](screenshots/3.png)

We can see in Event 468451 that linpeas is downloaded, and we can see in Event 468664 (and events after) linpeas being run and scanning different directories in the system.
**Answer: linpeas**

---

### 5. What is the name of the binary and pid used to gain root?

**Answer:**

---

### 6. What CVE was exploited to gain root access? (Do your research!) 


**Answer:**

---

### 7. What type of vulnerability is this? 


**Answer:**

---

### 8. What file was exfiltrated once root was gained? 


**Answer:**

---

