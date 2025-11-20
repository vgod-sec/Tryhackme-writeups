<!-- 
SEO: TryHackMe SMTP Writeup, SMTP Penetration Testing, Network Services TryHackMe, SMTP Enumeration, VRFY SMTP Attack, Ethical Hacking Walkthrough, SMTP Banner Grabbing, smtp-user-enum, Cyber Security CTF Writeup
-->

# TryHackMe – Network Services (SMTP Lab) – My Write-Up

This is my personal walkthrough of how I solved the SMTP challenge from the TryHackMe Network Services learning path.

---

## Step 1: Port Scan to Identify SMTP
I started with an Nmap scan to look for SMTP services.

nmap -sV -p 25,110,143 <TARGET-IP>

Port 25 was open → SMTP confirmed.

---

## Step 2: SMTP Banner Grabbing
I connected using Netcat:

nc <TARGET-IP> 25

I received the server banner and confirmed the SMTP service.

HELO test  
250 OK

---

## Step 3: Enumerating Users with VRFY
I used the VRFY command to discover valid system users.

VRFY root  
VRFY admin  
VRFY martin  

Valid users returned “250 User exists”.

---

## Step 4: Trying EXPN
I also tried EXPN:

EXPN staff  
EXPN admin-list

If enabled, this reveals mailing lists or usernames.  
In this room, VRFY was the main working command.

---

## Step 5: Automated Enumeration
I used smtp-user-enum for reliability:

smtp-user-enum -M VRFY -U /usr/share/wordlists/nmap.lst -t <TARGET-IP>

It confirmed valid usernames found earlier.

---

## Step 6: Capturing Flag
The lab questions required the discovered valid user.  
Once identified, I submitted it as the flag.

---

## Summary
1. Scanned SMTP ports  
2. Grabbed banner  
3. Enumerated users with VRFY  
4. Checked EXPN  
5. Automated with smtp-user-enum  
6. Submitted valid username as flag  

---

## Author
Shivam Singh  
Penetration Tester & Bug Hunter  
GitHub: https://github.com/vgod-sec
