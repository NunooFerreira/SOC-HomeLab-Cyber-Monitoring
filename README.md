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


## Tools and Usage

1. **pfSense**: Acts as a network firewall to control and monitor traffic between virtual machines.
2. **Active Directory (AD)**: Manages users and devices in a centralized domain (`soc.lab`).
3. **Windows Workstation**: Simulates personal user machine to detect malicious activities.
4. **Sysmon**: Provides advanced event logging to monitor system-level activity on Windows.
5. **CrowdSec**: Detects and mitigates threats.
6. **[BadBlood](https://github.com/davidprowe/BadBlood)**: Populates the Active Directory with intentional Users and Groups with misconfigurations (e.g., weak passwords, excessive privileges) to simulate real-world vulnerabilities.  



## Configurations

### 1. **pfSense Configuration**
- **Install and set up pfSense** as the core firewall for the lab network.

### 2. **Active Directory (AD) Setup**
- **Windows Server 2022** configured as an Active Directory Domain Controller.

### 3. **Windows Workstation Setup**
- **Windows 10** as a target machine for **attack simulations**.

### 4. **Sysmon Configuration**
- **System Monitor (Sysmon)** is installed for detailed Windows event logging.

#### Install Sysmon:
1. Download Sysmon from [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon), and execute the installer.

### 6. **Integrate CrowdSec with pfSense**
1. Install **CrowdSec Bouncer** on pfSense.





