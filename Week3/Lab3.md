# Lab Report  

## Course Information  
- **Course:** CSYS 4483-01 / CSYS 6683-01 – Network Defense  
- **Instructor:** Dr. Tirthankar Ghosh  
- **Semester:** Fall 2025  
- **Lab Title:** Network Packet Analysis with Wireshark  
- **Submission Deadline:** September 16, 2025  

---

## 1. Objective  
The objective of this lab is to analyze captured network packets using Wireshark to detect malicious activities and map them to MITRE ATT&CK techniques.  

---

## 2. Systems Needed  
- Any system with Wireshark installed  

---

## 3. Implementation Steps  

### Part 1: Analyzing `pcap.pcapng`  
1. Download `pcap.pcapng` and `exploit.pcapng` from Canvas.  
2. Open `pcap.pcapng` in Wireshark.  

**Unwanted activities**  
- ICMP Echo Requests were used as ping sweeps to test if the target was alive.  
  - *Figure 1: ICMP Echo Request evidence*  
  - *Figure 2: ICMP Echo Reply evidence*  

- UDP scanning was performed across ~1000 destination ports, confirmed by ICMP “Port Unreachable” replies.  
  - *Figure 3: Wireshark Endpoints Statistics – UDP traffic*  

- TCP SYN scanning was conducted with ~1000 SYN probes sent to check for open TCP ports.  
  - *Figure 4: Wireshark Endpoints Statistics – TCP traffic*  
  - *Figure 5: Snippet showing SYN probes and SYN/ACK replies*  

- A smaller set of TCP connect attempts (15 ports) completed the three-way handshake before being reset.  
  - *Figure 6: TCP handshake completed evidence*  

These activities are consistent with an Nmap-style port scan.  

- **Attacker IP address:** 10.1.1.34  
- **Target IP address:** 10.1.1.36  

**MITRE ATT&CK Mapping**  
- **Tactics:** Reconnaissance, Discovery  
- **Techniques:**  
  - T1595 – Active Scanning  
  - T1046 – Network Service Scanning  

---

### Part 2: Analyzing `exploit.pcapng`  
1. Open `exploit.pcapng` in Wireshark.  

**Unwanted activities**  
The attacker at 10.1.1.34 interacted with the victim at 10.1.1.36 using TFTP.  

- Step 1: The victim (10.1.1.36) issued a **Read Request (RRQ)** to pull `post_exploit.ps1` from the attacker.  
  - The script contained malicious functions:  
    - Create a new admin account  
    - Establish persistence via registry keys  
    - Exfiltrate files using TFTP  
    - Clean up traces after execution  
  - *Figure 7: Wireshark capture showing TFTP RRQ for post_exploit.ps1*  
  - *Figure 8: Reconstructed script content from Wireshark “Follow UDP Stream”*  

- Step 2: After executing the script, the victim sent a **Write Request (WRQ)** to upload `exfiltrate.txt` back to the attacker at 10.1.1.34.  
  - *Figure 9: TFTP WRQ showing exfiltrate.txt*  

- The protocol used was Trivial File Transfer Protocol (TFTP, UDP/69).  
  - *Figure 10: TFTP protocol evidence*  

**MITRE ATT&CK Mapping**  

#### Tactics  
- **Execution (TA0002):** Attacker downloads and executes malicious script (`post_exploit.ps1`).  
- **Persistence (TA0003):** Script (`post_exploit.ps1`) adds registry run key for autorun.  
- **Privilege Escalation (TA0004):** Script (`post_exploit.ps1`) creates new admin account (`User_Admin`).  
- **Exfiltration (TA0010):** Upload of `exfiltrate.txt` to attacker-controlled machine.  
- **Command and Control (TA0011):** Use of TFTP as a lightweight C2 channel.  

#### Techniques  
- **T1105 – Ingress Tool Transfer:** Downloading `post_exploit.ps1` onto victim.  
- **T1547.001 – Registry Run Keys / Startup Folder:** Persistence mechanism implemented inside `post_exploit.ps1`.  
- **T1136 – Create Account:** Creation of new admin user via `post_exploit.ps1`.  
- **T1048.003 – Exfiltration Over Unencrypted/Obsolete Protocol:** Exfiltration via TFTP (`exfiltrate.txt`).  
- **T1071.004 – Application Layer Protocol: TFTP:** Abusing TFTP for communication.  

---

## 4. Observations  
The analysis revealed two phases of attacker activity:  

- **Phase 1 (pcap.pcapng):** The attacker (10.1.1.34) performed reconnaissance and service discovery against the victim (10.1.1.36) using ICMP, UDP, and TCP scanning.  
- **Phase 2 (exploit.pcapng):** The attacker (10.1.1.34) delivered a malicious script (`post_exploit.ps1`) and received exfiltrated data (`exfiltrate.txt`) from the victim (10.1.1.36) via TFTP.  

This progression represents a full attack chain: **Reconnaissance → Execution → Persistence → Privilege Escalation → Exfiltration**.  

---

## 5. Conclusion  
Wireshark analysis revealed both the reconnaissance and exploitation stages of an attack. In the first capture, the attacker scanned the victim with ICMP, UDP, and TCP to map services. In the second capture, the attacker delivered and executed a malicious PowerShell script (`post_exploit.ps1`) that created persistence, escalated privileges, and staged exfiltration. Data was exfiltrated using TFTP, an unencrypted protocol.  

This lab demonstrates how packet-level analysis supports detection of attacker behavior mapped to MITRE ATT&CK and provides practical insights for network defense and incident response.  

---
