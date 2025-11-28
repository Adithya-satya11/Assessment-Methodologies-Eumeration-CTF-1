# Assessment-Methodologies-Eumeration-CTF-1

# 🛡️ Linux Service Enumeration & Exploitation Lab

**Target:** `target.ine.local`
**Objective:** Enumerate running services (Samba, FTP, SSH) to identify misconfigurations and weak credentials to capture 4 hidden flags.

---

## 1. Reconnaissance 🕵️‍♂️
I started by scanning the target to identify open ports.

**Tool:** Nmap

```
nmap -sV -sC -p- target.ine.local
```


IMAGE

Findings:

Port 22: SSH (OpenSSH)

Port 139, 445: Samba (SMB)

Port 5554: FTP Service (Uncommon port)

## 2. Samba Enumeration (Flag 1) 📂
I began by enumerating the Samba service using enum4linux to look for users and shares.

```
enum4linux target.ine.local
```

IMAGES

The output revealed a list of potential share names, but I needed to check which ones allowed Anonymous Access. I wrote a custom Bash script to automate this check against the discovered share list.

Custom Bash Script (shares.sh):
```
#!/bin/bash

TARGET=target.ine.local
WORDLIST=/root/Desktop/wordlists/unix_password.txt

if [ ! -f "$WORDLIST" ]; then
	echo "Wordlists not found "
	exit 1
fi

while read -r SHARES ; do 
	echo "testing share : $SHARES"
smbclient -L //$TARGET/$SHARES -N -c "ls" &>/dev/null

	if [ $? -eq 0 ]; then
		echo " !!!!! Anonymous access allowed for $SHARES"
	else
		echo " ** Anonymous login failed for $SHARES"
	fi
done < "$WORDLIST"

```

IMAGE


Result: The script identified that the pubfiles share allows anonymous access.

I connected to the share and retrieved the first flag.


```
smbclient //target.ine.local/pubfiles -N
```

IMAGE
🚩 Flag 1:

(Insert Flag Here)

## 3. Samba User Exploitation (Flag 2) 🔓

During the enum4linux scan, I identified 4 valid usernames. I saved these to a file named users.txt.

Objective: Brute-force the SMB login for these users. Tool: Metasploit (auxiliary/scanner/smb/smb_login)

RHOSTS: target.ine.local

USER_FILE: users.txt

PASS_FILE: /root/Desktop/wordlists/unix_passwords.txt

IMAGE

Success! Metasploit found valid credentials for one user.

User: josh Password: purple

I logged in to that user's personal share to find the second flag.

IMAGE

🚩 Flag 2:

(Insert Flag Here)

## 4. FTP Enumeration (Flag 3) 📨
The Nmap scan showed an FTP service running on a non-standard port (5554). Based on the hint found in the previous flag, I used the discovered usernames to brute-force the FTP service.

IMAGE

Tool: Hydra

```
hydra -L usr.txt -P /root/Desktop/wordlists/unix_passwords.txt ftp://target.ine.local:5554
```

IMAGE

Hydra successfully cracked the password. I logged in via FTP and retrieved the third flag.

IMAGE

🚩 Flag 3:

(Insert Flag Here)

## 5. SSH Banner Grabbing (Flag 4) 🚩

The final flag was described as a "warning." I attempted to connect to the SSH service on port 22 to see the login banner.

```
ssh target.ine.local
```

IMAGE

The flag was displayed in the unauthorized access warning banner itself.

🚩 Flag 4:

(Insert Flag Here)

✅ Conclusion
This lab demonstrated the importance of:

Checking for anonymous SMB shares using custom scripts.

Enumerating users (enum4linux) to create targeted wordlists.

Checking non-standard ports (FTP on 5554).

Inspecting service banners (SSH) for information leakage.
