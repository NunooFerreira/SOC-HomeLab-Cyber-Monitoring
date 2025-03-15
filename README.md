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

| Device                 | IP Address       | Gateway        | Subnet Mask     |
|------------------------|------------------|----------------|-----------------|
| pfSense (Firewall)     | 192.168.10.1     | -              | 255.255.255.0   |
| Active Directory (SOC-AD1) | 192.168.10.10    | 192.168.10.1  | 255.255.255.0   |
| Windows Workstation    | 192.168.10.20    | 192.168.10.1   | 255.255.255.0   |


## Tools and Usage

1. **pfSense**: Acts as a network firewall to control and monitor traffic between virtual machines.
2. **Active Directory (AD)**: Manages users and devices in a centralized domain (e.g., `soc.lab`).
3. **Windows Workstation**: Simulates a user machine to execute and detect malicious activities.
4. **Sysmon**: Provides advanced event logging to monitor system-level activity on Windows.
5. **CrowdSec**: Detects and mitigates threats like brute-force attacks and unauthorized access.
6. ** BadBlood(https://github.com/davidprowe/BadBlood)**: Populates the Active Directory with intentional Users and Groups with misconfigurations (e.g., weak passwords, excessive privileges) to simulate real-world vulnerabilities.  
## Objectives
- Simulate real-world cyberattacks (e.g., RDP brute force, port scanning).
- Use **Sysmon** and **CrowdSec** to monitor, detect, and mitigate attacks.
- Document attack vectors, detection methods, and mitigation strategies.



## Configurations

### 1. **pfSense Configuration**
- **Install and set up pfSense** as the core firewall for the lab network.
- Ensure **logging** is enabled for packet filtering and traffic monitoring.

#### Enable Firewall Logging:
1. Go to **Status > System Logs > Firewall**.
2. Ensure logging for all relevant traffic is active.

### 2. **Active Directory (AD) Setup**
- **Windows Server 2022** configured as an Active Directory Domain Controller.

#### Key AD Configurations:
- Domain Name: `lab.local`
- IP Address: `192.168.1.20`
- Ensure Event Logging for **user logins** and **authentication failures**.

### 3. **Windows Workstation Setup**
- **Windows 10** as a target machine for **attack simulations**.
- Enabled **Remote Desktop Protocol (RDP)** for brute-force simulations.

#### Enable RDP:
1. **Control Panel > System and Security > Remote Desktop**
2. Select **Allow Remote Connections**

### 4. **Sysmon Configuration**

- **System Monitor (Sysmon)** is installed for detailed Windows event logging.

#### Install Sysmon:
1. Download Sysmon from [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
2. Execute the installer:
   ```powershell
   sysmon -accepteula -i sysmonconfig.xml
   ```
3. Verify logs in the **Event Viewer** under:
   ```
Application and Services Logs > Microsoft > Windows > Sysmon
   ```

### 5. **CrowdSec Configuration**

- Installed **CrowdSec** on the Windows Workstation.
- Configured to monitor **Windows Event Logs** and **firewall logs**.

#### Verify CrowdSec Status:
```powershell
cd "C:\Program Files\CrowdSec"
.\cscli.exe config show
```

#### Acquisition File (`acquis.yaml`):
```yaml
##RDP
source: wineventlog
event_channel: Security
event_ids:
  - 4625
labels:
  type: eventlog
---
##Firewall
filenames:
  - C:\Windows\System32\LogFiles\Firewall\*.log
labels:
  type: windows-firewall
```

#### Update CrowdSec Collections:
```powershell
cscli.exe hub update
cscli.exe collections install crowdsecurity/windows
cscli.exe collections install crowdsecurity/windows-firewall
```

#### Check Alerts:
```powershell
.\cscli.exe alerts list
```

### 6. **Integrate CrowdSec with pfSense**

1. Install **CrowdSec Bouncer** on pfSense.
2. Verify that pfSense blocks malicious IPs based on CrowdSec detections.

## Next Steps
1. Simulate cyberattacks (e.g., **RDP brute-force**, **port scans**).
2. Capture logs and confirm detection in **CrowdSec** and **Sysmon**.
3. Document how to detect, analyze, and mitigate threats.

## Future Additions
- Implement advanced detection scenarios (e.g., **lateral movement** and **privilege escalation**).
- Automate alerting and IP blocking using **CrowdSec + pfSense**.
- Document and export findings into a professional PDF report.

---

**This README will evolve as new detections and mitigations are added.**



