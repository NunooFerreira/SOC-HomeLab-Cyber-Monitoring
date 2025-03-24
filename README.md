# SOC-Lab-Home-Cybersecurity-Monitoring
This project sets up a Security Operations Center (SOC) lab to simulate, detect, and mitigate cyberattacks using open-source security tools. Developed experience with threat detection, incident response, and system monitoring.

## Project Structure

| Component             | Purpose                                 |
|-----------------------|-----------------------------------------|
| **pfSense**           | Network firewall and traffic management |
| **Active Directory**  | Centralized user and domain management  |
| **Windows Workstation**| Simulates user endpoint for testing    |
| **Sysmon**            | Advanced logging for Windows events     |
| **CrowdSec**          | Detects and mitigates malicious activity|

### Network Configuration

| Device                 | IP Address       | Gateway        | Subnet Mask     | Zone    |
|------------------------|------------------|----------------|-----------------|---------|
| pfSense (Firewall)     | 192.168.10.1     | -              | 255.255.255.0   |         |
| Active Directory (SOC-AD1) | 192.168.10.10    | 192.168.10.1  | 255.255.255.0   | LAN  |
| Windows Workstation    | 192.168.10.20    | 192.168.10.1   | 255.255.255.0   | LAN     |
| Kali Linux             | 192.168.20.30    | 192.168.20.1   | 255.255.255.0   | DMZ     |

## Network Architecture

![teste](https://github.com/user-attachments/assets/3a6ae53d-8579-4038-8958-1352c3359474)



## Tools and Usage

1. **pfSense**: Acts as a network firewall to control and monitor traffic between virtual machines.
2. **Active Directory (AD)**: Manages users and devices in a centralized domain (soc.lab).
3. **Windows Workstation**: Simulates personal user machine to detect malicious activities.
4. **Sysmon**: Provides advanced event logging to monitor system-level activity on Windows.
5. **CrowdSec**: Detects and mitigates threats.
6. **[BadBlood](https://github.com/davidprowe/BadBlood)**: Populates the Active Directory with intentional Users and Groups with misconfigurations (e.g., weak passwords, excessive privileges) to simulate real-world vulnerabilities.  

## Attack Simulation Sequence

Initially, [Hydra](https://hackviser.com/tactics/tools/hydra) was used to launch a brute-force attack against the RDP service, aiming to guess the password of a random user on a Windows 10 workstation after a leaked users list was discovered online. Next, while operating from that user's machine within the SOC.LAB domain that was found from brute-force, an attempt was made to extract Kerberos Ticket Granting Service (TGS) hashes from the Active Directory Domain Controller. The extracted hashes were then cracked offline using [Hashcat](https://github.com/hashcat/hashcat) with the [rockyou.txt](https://www.kaggle.com/datasets/wjburns/common-password-list-rockyoutxt) wordlist along with additional tools, ultimately resulting in a list of valid usernames and passwords. With these valid credentials, remote access via RDP was gained, and one user was selected to access the Active Directory. Once inside, data was compressed and exfiltrated via [Netcat](https://nmap.org/ncat/) . Finally, an AD user was induced to install malware that allowed the establishment of a reverse shell, enabling remote command execution and potentially starting a botnet with a command and control server, for future work.




                      ┌──────────────────────┐                
                      │         START        │  
                      └──────────┬───────────┘
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 1: Brute-force RDP (Hydra)    │  
                      │ Target: Windows WS (192.168.10.20)  │  
                      └──────────┬──────────────────────────┘  
                                 │  
                                 ▼  
                      ┌──────────────────────────────────────┐  
                      │ PHASE 2: Kerberos TGS Hash Extraction│  
                      │ From: AD Controller (192.168.10.10)  │  
                      └──────────┬───────────────────────────┘  
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 3: Hashcat Cracking           │  
                      │ rockyou.txt → Valid Credentials List│  
                      └──────────┬──────────────────────────┘  
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 4: RDP Access to Domain       │  
                      │ (Using Stolen Credentials)          │  
                      └──────────┬──────────────────────────┘  
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 5: AD Data Collection         │  
                      │ (Compress Sensitive Files)          │  
                      └──────────┬──────────────────────────┘  
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 6: Exfiltration via Netcat    │  
                      │ (Data → Attacker's Server)          │  
                      └──────────┬──────────────────────────┘  
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 7: Malware Deployment         │  
                      │ (Reverse Shell Payload)             │  
                      └──────────┬──────────────────────────┘  
                                 │  
                                 ▼  
                      ┌─────────────────────────────────────┐  
                      │ PHASE 8: Persistent Access          │  
                      │ [Potential Botnet C2 Setup]         │  
                      └─────────────────────────────────────┘  
