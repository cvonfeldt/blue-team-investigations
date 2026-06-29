# Overview: 
**We are provided with a malicious sample related to the "Follina" vulnerability, and we need to analyze it to answer the prompt questions and understand how this RCE exploit works in the wild.**
![Inv](screenshots/Inv.png)

<br>

![warning](screenshots/warning.png)

**This challenge contains real malware so I won't unzip the file on my host machine - instead I'll use this opportunity to showcase the strength and ability of OSint - VirusTotal, Any.Run, etc. - as an alternative in analyzing malware.**

<br>

## Investigation:

### 1. What is the SHA1 hash value of the sample?
For this I'm just going to have claude ai unzip the file and calculate the sha1 hash value:
![Q1](screenshots/1.png)

If I were to do it myself, I would simply run sha1sum sample.doc in wsl and get the same value of 
**Answer: 06727ffda60359236a8029e0b3e8a0fd11c23313**

---


### 2. According to VirusTotal, what is the full filetype of the provided sample? 
Now that we have the sha1 hash, we can enter that into virustotal and get more info regarding the attack:
![Q2](screenshots/2.png)

We see it's an Office Open XML Document. 
**Answer: Office Open XML Document**

---

### 3. Extract the URL that is used within the sample and submit it 
In VirusTotal we can see the network communications which we know would need to be used here:
![Q3](screenshots/3.png)

We can see a successful HTTP GET: GET https://www.xmlformats.com:443/office/word/2022/wordprocessingDrawing/RDF842l.html 200, and we can assume that this is where/how the payload is downloaded.
**Answer: https://www.xmlformats.com:443/office/word/2022/wordprocessingDrawing/RDF842l.html**

---

### 4. What is the name of the XML file that is storing the extracted URL? 
Of the bundled xml files:
![Q4](screenshots/4.png)

We can see there are only 2 with antivirus engine warnings, and we know that .rels (relations) files contain URLs, so we can assume this is the XML that stores the malicious URL.
**Answer: word/_rels/document.xml.rels**

---

### 5. The extracted URL accesses a HTML file that triggers the vulnerability to execute a malicious payload. According to the HTML processing functions, any files with fewer than <Number> bytes would not invoke the payload. Submit the <Number>

For this one I first attempted to use any.run, but couldn't get past the CAPTCHA:
![Q5](screenshots/5.png)

So I knew I needed to use other OSint. I went to Huntress.com - a trusted and well-known MSP - and saw they had documentation regarding the Follina attack:
![Q5](screenshots/6.png)

In their documentation they noted that they found the byte threshold was 4096 - meaning the attack staged on the html at that time needed to be at least 4096 bytes to invoke the payload. 
**Answer: 4096**

---

### 6. After execution, the sample will try to kill a process if it is already running. What is the name of this process? 


**Answer:**

---

### 7. You were asked to write a process-based detection rule using Windows Event ID 4688. What would be the ProcessName and ParentProcessname used in this detection rule? [Hint: OSINT time!] 


**Answer:**

---

### 8.  Submit the CVE associated with the vulnerability that is being exploited


**Answer:**

---

**Completed:**
