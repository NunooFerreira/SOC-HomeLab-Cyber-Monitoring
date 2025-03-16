## Simulated Attack: RDP Brute-Force  

### Target  
- **Windows 10 Workstation**: `192.168.10.20` (RDP port `3389`).  

### Attack Command (Kali Linux)  
```bash
hydra -L /usr/share/wordlists/common_users.txt -P /usr/share/wordlists/rockyou.txt rdp://192.168.10.20
```
### 1. Sysmon
We can see that Sysmon successfully logged the network connection from the Kali’s IP to RDP port 3389 (Event ID 3), this confirms that the brute-force attempt was initiated and reached the target:

![Captura de ecrã 2025-03-16 170811](https://github.com/user-attachments/assets/b3433d01-1a6b-40d9-bdc2-a0232ee1146f)



### 2. CrowdSec Alert & Decision
CrowdSec detected the brute-force attack after multiple unsuccessful login attempts, as a result, it automatically added the Kali Linux IP (192.168.20.30) to its blocklist, effectively banning further access attempts:

![Captura de ecrã 2025-03-16 170621](https://github.com/user-attachments/assets/a6a29df1-e8e1-41ff-9393-c54e4d616f73)


### 3. Windows Event logs filter with EventID 4625 (Log on Faill): 
The Windows Security Event Log captured multiple failed login attempts under Event ID 4625 (Account Logon Failure):
![Captura de ecrã 2025-03-16 170907](https://github.com/user-attachments/assets/b6bf6a89-b0b6-4a23-aae6-fb7c26cd6c51)


