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


![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127091727.png?raw=true)


Findings:

Port 22: SSH (OpenSSH)

Port 139, 445: Samba (SMB)

Port 5554: FTP Service (Uncommon port)

## 2. Samba Enumeration (Flag 1) 📂
I began by enumerating the Samba service using enum4linux to look for users and shares.

```
enum4linux target.ine.local
```
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127231508.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127232056.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127231713.png?raw=true)

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

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127225306.png?raw=true)


Result: The script identified that the pubfiles share allows anonymous access.

I connected to the share and retrieved the first flag.


```
smbclient //target.ine.local/pubfiles -N
```
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127232825.png?raw=true)

🚩 Flag 1:

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127232934.png?raw=true)

## 3. Samba User Exploitation (Flag 2) 🔓

During the enum4linux scan, I identified 4 valid usernames. I saved these to a file named users.txt.

Objective: Brute-force the SMB login for these users. Tool: Metasploit (auxiliary/scanner/smb/smb_login)

RHOSTS: target.ine.local

USER_FILE: users.txt

PASS_FILE: /root/Desktop/wordlists/unix_passwords.txt

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127113527.png?raw=true)

Success! Metasploit found valid credentials for one user.

User: josh Password: purple

I logged in to that user's personal share to find the second flag.

```
smbclient //target.ine.local/josh -U josh
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127233448.png?raw=true)

🚩 Flag 2:

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127233755.png?raw=true)

## 4. FTP Enumeration (Flag 3) 📨
The Nmap scan showed an FTP service running on a non-standard port (5554). Based on the hint found in the previous flag, I used the discovered usernames to brute-force the FTP service.

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127230137.png?raw=true)

I have 3 users save it to usr.txt file and with the usr file and passwd file do brute-force

Tool: Hydra

```
hydra -L usr.txt -P /root/Desktop/wordlists/unix_passwords.txt ftp://target.ine.local:5554
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127231023.png?raw=true)

Hydra successfully cracked the password. I logged in via FTP and retrieved the third flag.

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127234205.png?raw=true)
🚩 Flag 3:

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127234236.png?raw=true)

## 5. SSH Banner Grabbing (Flag 4) 🚩

The final flag was described as a "warning." I attempted to connect to the SSH service on port 22 to see the login banner.

```
ssh target.ine.local
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251127234519.png?raw=true)

The flag was displayed in the unauthorized access warning banner itself.

🚩 Flag 4:

```
FLAG4{db7b867087594ac5a829b431a6845fc3}
```

✅ Conclusion
This lab demonstrated the importance of:

Checking for anonymous SMB shares using custom scripts.

Enumerating users (enum4linux) to create targeted wordlists.

Checking non-standard ports (FTP on 5554).

Inspecting service banners (SSH) for information leakage.
