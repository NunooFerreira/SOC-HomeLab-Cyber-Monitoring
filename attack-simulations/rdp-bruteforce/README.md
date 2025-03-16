## Simulated Attack: RDP Brute-Force  

### Target  
- **Windows 10 Workstation**: `192.168.10.20` (RDP port `3389`).  

### Attack Command (Kali Linux)  
```bash
hydra -L /usr/share/wordlists/common_users.txt -P /usr/share/wordlists/rockyou.txt rdp://192.168.10.20
```
### 1. Sysmon
We can see that Sysmon logged the network connection from the Kali’s IP to RDP port 3389 (Event ID 3):

![Captura de ecrã 2025-03-16 043741](https://github.com/user-attachments/assets/1f83df8c-13a5-46c4-9a28-638b76a7ad82)



### 2. CrowdSec Alert & Decision
Then CrowdSec detected the brute-force attack and banned the Kali’s IP (192.168.10.30) after multiple attemps:
![Captura de ecrã 2025-03-16 041244](https://github.com/user-attachments/assets/34501442-2c73-4697-a19d-5d397f631ab2)


