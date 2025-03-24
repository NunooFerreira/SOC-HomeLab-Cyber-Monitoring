# SOC-Lab-Home-Cybersecurity-Monitoring

This project sets up a Security Operations Center (SOC) lab to simulate, detect, and mitigate cyberattacks using open-source security tools. It provides hands-on experience with threat detection, incident response, and system monitoring.

---

## Project Structure

| Component             | Purpose                                   |
|-----------------------|-------------------------------------------|
| **pfSense**           | Network firewall and traffic management   |
| **Active Directory**  | Centralized user and domain management    |
| **Windows Workstation**| Simulates a user endpoint for testing    |
| **Sysmon**            | Advanced logging for Windows events       |
| **CrowdSec**          | Detects and mitigates malicious activity  |

---

## Network Configuration

| Device                 | IP Address       | Gateway        | Subnet Mask     | Zone    |
|------------------------|------------------|----------------|-----------------|---------|
| pfSense (Firewall)     | 192.168.10.1     | -              | 255.255.255.0   | -       |
| Active Directory (SOC-AD1) | 192.168.10.10    | 192.168.10.1  | 255.255.255.0   | LAN     |
| Windows Workstation    | 192.168.10.20    | 192.168.10.1   | 255.255.255.0   | LAN     |
| Kali Linux             | 192.168.20.30    | 192.168.20.1   | 255.255.255.0   | DMZ     |

---

## Network Architecture

![Network Architecture](https://github.com/user-attachments/assets/3a6ae53d-8579-4038-8958-1352c3359474)

---

## Tools and Usage

1. **pfSense**: Acts as a network firewall to control and monitor traffic between virtual machines.  
2. **Active Directory (AD)**: Manages users and devices in a centralized domain (`soc.lab`).  
3. **Windows Workstation**: Simulates a standard user endpoint to observe and detect malicious activities.  
4. **Sysmon**: Provides advanced logging for system-level activities, aiding in threat detection and forensic analysis.  
5. **CrowdSec**: Monitors logs, detects suspicious patterns, and mitigates attacks via collaborative threat intelligence.  
6. **[BadBlood](https://github.com/davidprowe/BadBlood)**: Populates the Active Directory with users and groups containing misconfigurations (e.g., weak passwords, excessive privileges) to simulate real-world vulnerabilities.  

---

## Attack Simulation Sequence

The attack simulation followed a multi-step process mimicking real-world intrusion tactics:

1. **Brute-Force RDP Attack:**  
   - Used [Hydra](https://hackviser.com/tactics/tools/hydra) to launch a brute-force attack on the Windows 10 Workstation RDP service.  
   - This attack aimed to guess user credentials from a leaked user list.  

2. **Kerberoasting Attack:**  
   - With access to the compromised workstation, [Impacket](https://github.com/fortra/impacket) was used to extract **Kerberos Ticket Granting Service (TGS) hashes** from the AD controller.  
   - These hashes were cracked offline using [Hashcat](https://github.com/hashcat/hashcat) and the **rockyou.txt** wordlist, revealing valid usernames and passwords.  

3. **Remote Access and Data Exfiltration:**  
   - Using the obtained credentials, remote access via RDP was established.  
   - Data was compressed and exfiltrated using [Netcat](https://nmap.org/ncat/), simulating a real-world data breach.  

4. **Reverse Shell Backdoor:**  
   - A PowerShell-based reverse shell was deployed to maintain persistent access.  
   - This backdoor allows remote command execution and could serve as a basis for a botnet in future simulations.  

---

## Future Work

1. **Botnet Simulation:** Expand the reverse shell into a **Command & Control (C2)** infrastructure to simulate a botnet.  
2. **Log Analysis Automation:** Implement automated scripts to parse **Sysmon** and **CrowdSec** logs for quicker threat detection.  
3. **Implement a SIEM**

---

## How to Recreate This Lab

1. Set up virtual machines for **pfSense**, **Active Directory**, and a **Windows Workstation**.  
2. Configure network segmentation (LAN and DMZ).  
3. Install **Sysmon** and **CrowdSec** on the Windows Workstation and Active Directory Server.  
4. Simulate attacks using **Hydra**, **Impacket**, and **Netcat**.  
5. Monitor and analyze logs using **CrowdSec** dashboards and Windows Event Viewer with the help of Sysmon.  

---

## References

- [LetsDefend](https://app.letsdefend.io/training/lessons/building-a-soc-lab-at-home)
- [Hydra – Brute Force Tool](https://hackviser.com/tactics/tools/hydra)  
- [BadBlood – AD Misconfiguration Tool](https://github.com/davidprowe/BadBlood)  
- [Hashcat – Password Cracking](https://github.com/hashcat/hashcat)  
- [CrowdSec – Collaborative Security](https://www.crowdsec.net/)

