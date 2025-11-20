# TryHackMe – Network Services (SMB Lab) – My Write-Up
This is my personal walkthrough of how I solved the SMB room in the TryHackMe Network Services learning path.

## Step 1: Initial Scan
First, I started with an Nmap scan to confirm whether SMB was running on the target.

nmap -sV -p 139,445 <TARGET-IP>

The results showed:
- 139/tcp open netbios-ssn
- 445/tcp open microsoft-ds

So SMB was definitely running.

---

## Step 2: Enumerating Shares (Anonymous)
My next step was to check if the server allowed anonymous access. I listed the shares using:

smbclient -L //<TARGET-IP>/ -N

I found the following shares:
- anonymous
- secure
- IPC$

I decided to explore **anonymous** first.

---

## Step 3: Accessing the Anonymous Share
I connected directly to the anonymous share:

smbclient //<TARGET-IP>/anonymous -N

Inside the share, I ran:

ls

I saw two files:
- staff.txt
- note.txt

I downloaded them:

get staff.txt
get note.txt

After reading both files, I got one username and a hint about internal access. This looked useful for the secure share.

---

## Step 4: Enum4linux Scan
To confirm the usernames and gather more info, I ran:

enum4linux -a <TARGET-IP>

This confirmed the username I found earlier and gave me more information about the system.

---

## Step 5: Accessing the Secure Share
The note from the anonymous share revealed credentials:  
**username:** martin  
**password:** securepassword

I used these to access the secure share:

smbclient //<TARGET-IP>/secure -U martin

After entering the password, I listed the files:

ls

I found **flag.txt**, so I downloaded it:

get flag.txt

Then I read it:

cat flag.txt

This gave me the second flag.

---

## Step 6: Searching for the Final Hidden Flag
I suspected there might be a hidden folder or a deeper directory structure, so I went back to the main share listing and enabled recursive listing:

smbclient //<TARGET-IP>/ -N
smb: \> recurse ON
smb: \> ls

With recursion enabled, I found an extra hidden folder inside one of the shares containing the **final flag**. I navigated into that directory and downloaded the last file:

get final.txt
cat final.txt

This gave me the final flag of the lab.

---

## Final Summary (How I Solved It)
1. Scanned SMB ports using Nmap  
2. Enumerated shares using smbclient (anonymous)  
3. Collected usernames + hints from anonymous share  
4. Confirmed details using enum4linux  
5. Logged into secure share using discovered credentials  
6. Downloaded secure flag  
7. Used recursive listing to find hidden directory  
8. Retrieved final flag

The SMB room was fully solved.

---

## Author
Shivam Singh  
Penetration Tester & Bug Hunter  
GitHub: https://github.com/vgod-sec
