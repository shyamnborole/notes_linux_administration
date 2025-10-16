==#📁 **VSFTPD FTP SERVER - ROCKY LINUX**==

---

==**INSTALLATION**==
> Package name: vsftpd  
> Service name: vsftpd  
> Default port: 21 (control), 20 (data)

```bash
# Install vsftpd
sudo dnf install vsftpd -y

# Start and enable service
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
sudo systemctl status vsftpd
```

**Open Firewall:**
```bash
# Open FTP service in firewall
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --reload

# Verify firewall rules
sudo firewall-cmd --list-all
```

---

==**IMPORTANT FILES AND DIRECTORIES**==

| Purpose | Path |
|---------|------|
| Main config file | `/etc/vsftpd/vsftpd.conf` |
| User list (blocked users) | `/etc/vsftpd/user_list` |
| FTP root directory | `/var/ftp` |
| User home directories | `/home/username` |
| Log file | `/var/log/xferlog` or `/var/log/vsftpd.log` |

---

==**STEP 1: CREATE FTP USERS**==

```bash
# Create three FTP users
sudo useradd -m -s /bin/bash ftp1
sudo useradd -m -s /bin/bash ftp2
sudo useradd -m -s /bin/bash ftp3

# Set passwords for users
sudo passwd ftp1
sudo passwd ftp2
sudo passwd ftp3

# Verify users created
id ftp1
id ftp2
id ftp3
```

---

==**STEP 2: CREATE FILES FOR FTP2 USER**==

```bash
# Switch to ftp2 user or create files directly
sudo su - ftp2

# Create some test files
echo "This is file1 from ftp2" > file1.txt
echo "This is file2 from ftp2" > file2.txt
echo "This is file3 from ftp2" > file3.txt

# Exit back to your user
exit

# OR create files directly as root
sudo bash -c 'echo "This is file1 from ftp2" > /home/ftp2/file1.txt'
sudo bash -c 'echo "This is file2 from ftp2" > /home/ftp2/file2.txt'
sudo bash -c 'echo "This is file3 from ftp2" > /home/ftp2/file3.txt'

# Set ownership
sudo chown ftp2:ftp2 /home/ftp2/file*.txt

# Verify files
ls -l /home/ftp2/
```

---

==**STEP 3: CONFIGURE VSFTPD**==

**Backup original config:**
```bash
sudo cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
```

**Edit configuration:**
```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

**Key configuration settings:**
```conf
# Allow local users to login
local_enable=YES

# Enable write operations
write_enable=YES

# Local users will be chrooted to their home directory
chroot_local_user=YES

# Allow chroot writable
allow_writeable_chroot=YES

# Enable logging
xferlog_enable=YES
xferlog_file=/var/log/xferlog

# Use local time
use_localtime=YES

# Enable passive mode (optional but recommended)
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
```

**Restart vsftpd:**
```bash
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

---

==**STEP 4: CLIENT - LOGIN AS FTP2 AND DOWNLOAD FILES**==

**From client machine:**
```bash
# Connect to FTP server
ftp server-ip-address

# Login credentials
Name: ftp2
Password: [ftp2 password]

# List files
ftp> ls
ftp> dir

# Download files
ftp> get file1.txt
ftp> get file2.txt
ftp> get file3.txt

# OR download all files
ftp> mget *.txt

# Exit FTP
ftp> bye

# Verify files downloaded
ls -l
cat file1.txt
cat file2.txt
cat file3.txt
```

---

==**STEP 5: CLIENT - LOGIN AS FTP1 AND UPLOAD FILES**==

```bash
# Connect to FTP server
ftp server-ip-address

# Login credentials
Name: ftp1
Password: [ftp1 password]

# Upload the files downloaded by ftp2
ftp> put file1.txt
ftp> put file2.txt
ftp> put file3.txt

# OR upload all files
ftp> mput *.txt

# List files to verify upload
ftp> ls
ftp> dir

# Exit FTP
ftp> bye
```

**Verify on server:**
```bash
# Check ftp1 home directory
sudo ls -l /home/ftp1/
sudo cat /home/ftp1/file1.txt
```

---

==**STEP 6: CLIENT - LOGIN AS FTP3 USER**==

```bash
# Connect to FTP server
ftp server-ip-address

# Login credentials
Name: ftp3
Password: [ftp3 password]

# List files
ftp> ls

# Exit FTP
ftp> bye
```

---

==**STEP 7: BLOCK FTP3 USER FROM FTP ACCESS**==

**Method 1: Using user_list file**
```bash
# Edit user_list file
sudo nano /etc/vsftpd/user_list

# Add ftp3 to the list
ftp3

# Save and exit
```

**Ensure vsftpd.conf has these settings:**
```bash
sudo nano /etc/vsftpd/vsftpd.conf
```

**Add/verify these lines:**
```conf
# Enable user_list
userlist_enable=YES

# Deny users in user_list (not allow)
userlist_deny=YES

# Path to user_list file
userlist_file=/etc/vsftpd/user_list
```

**Restart vsftpd:**
```bash
sudo systemctl restart vsftpd
```

**Method 2: Using ftpusers file**
```bash
# Edit ftpusers file
sudo nano /etc/vsftpd/ftpusers

# Add ftp3 to the list
ftp3

# Save and exit
# Restart vsftpd
sudo systemctl restart vsftpd
```

---

==**STEP 8: TEST BLOCKED USER**==

**From client machine:**
```bash
# Try to connect as ftp3
ftp server-ip-address

# Login credentials
Name: ftp3
Password: [ftp3 password]

# Should get error message:
# 530 Login incorrect
# Login failed
```

---

==**COMPLETE COMMAND SEQUENCE**==

**On FTP Server:**
```bash
# 1. Install and configure
sudo dnf install vsftpd -y
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --reload

# 2. Create users
sudo useradd -m -s /bin/bash ftp1
sudo useradd -m -s /bin/bash ftp2
sudo useradd -m -s /bin/bash ftp3
sudo passwd ftp1
sudo passwd ftp2
sudo passwd ftp3

# 3. Create files for ftp2
sudo bash -c 'echo "This is file1 from ftp2" > /home/ftp2/file1.txt'
sudo bash -c 'echo "This is file2 from ftp2" > /home/ftp2/file2.txt'
sudo bash -c 'echo "This is file3 from ftp2" > /home/ftp2/file3.txt'
sudo chown ftp2:ftp2 /home/ftp2/file*.txt

# 4. Configure vsftpd
sudo cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.backup
sudo sed -i 's/^#local_enable=YES/local_enable=YES/' /etc/vsftpd/vsftpd.conf
sudo sed -i 's/^#write_enable=YES/write_enable=YES/' /etc/vsftpd/vsftpd.conf
echo "chroot_local_user=YES" | sudo tee -a /etc/vsftpd/vsftpd.conf
echo "allow_writeable_chroot=YES" | sudo tee -a /etc/vsftpd/vsftpd.conf
echo "userlist_enable=YES" | sudo tee -a /etc/vsftpd/vsftpd.conf
echo "userlist_deny=YES" | sudo tee -a /etc/vsftpd/vsftpd.conf
sudo systemctl restart vsftpd

# 5. Block ftp3 user
echo "ftp3" | sudo tee -a /etc/vsftpd/user_list
sudo systemctl restart vsftpd
```

**On FTP Client:**
```bash
# Test as ftp2 - download files
ftp server-ip
# Login as ftp2
# ftp> mget *.txt
# ftp> bye

# Test as ftp1 - upload files
ftp server-ip
# Login as ftp1
# ftp> mput *.txt
# ftp> ls
# ftp> bye

# Test as ftp3 - should fail
ftp server-ip
# Login as ftp3 - should fail with "530 Login incorrect"
```

---

==**SELINUX CONFIGURATION (IF ENABLED)**==

```bash
# Check SELinux status
getenforce

# If enforcing, set SELinux booleans
sudo setsebool -P ftpd_full_access on
sudo setsebool -P allow_ftpd_full_access on

# Set proper context for home directories
sudo restorecon -Rv /home/ftp1
sudo restorecon -Rv /home/ftp2
sudo restorecon -Rv /home/ftp3
```

---

==**FTP CLIENT COMMANDS REFERENCE**==

| Command | Description |
|---------|-------------|
| `ftp server-ip` | Connect to FTP server |
| `ls` or `dir` | List files |
| `cd directory` | Change directory |
| `lcd directory` | Change local directory |
| `pwd` | Print working directory on server |
| `lpwd` | Print local working directory |
| `get filename` | Download single file |
| `mget *.txt` | Download multiple files |
| `put filename` | Upload single file |
| `mput *.txt` | Upload multiple files |
| `delete filename` | Delete file on server |
| `mkdir dirname` | Create directory |
| `rmdir dirname` | Remove directory |
| `binary` | Switch to binary mode |
| `ascii` | Switch to ASCII mode |
| `bye` or `quit` | Exit FTP |
| `help` | Show available commands |

---

==**PASSIVE MODE FIREWALL (OPTIONAL)**==

**If using passive mode, open passive port range:**
```bash
sudo firewall-cmd --permanent --add-port=40000-40100/tcp
sudo firewall-cmd --reload
```

---

==**TROUBLESHOOTING**==

```bash
# Check vsftpd status
sudo systemctl status vsftpd

# View vsftpd logs
sudo tail -f /var/log/xferlog
sudo journalctl -u vsftpd -f

# Check if vsftpd is listening
sudo netstat -tlnp | grep vsftpd
sudo ss -tlnp | grep vsftpd

# Test FTP from server itself
ftp localhost

# Check firewall rules
sudo firewall-cmd --list-all

# Check SELinux denials
sudo ausearch -m avc -ts recent | grep ftp

# Verify user_list file
cat /etc/vsftpd/user_list

# Verify ftpusers file
cat /etc/vsftpd/ftpusers

# Check user home directory permissions
ls -ld /home/ftp1 /home/ftp2 /home/ftp3
```

**Common Issues:**

| Issue | Solution |
|-------|----------|
| 530 Login incorrect | Check username/password, verify user in `/etc/passwd` |
| 500 OOPS: vsftpd: refusing to run with writable root | Add `allow_writeable_chroot=YES` |
| Cannot create directory | Check `write_enable=YES` in config |
| User blocked | Check `/etc/vsftpd/user_list` and `/etc/vsftpd/ftpusers` |
| Connection timeout | Check firewall and SELinux settings |

---

==**IMPORTANT NOTES**==

- Default FTP is **not encrypted** - use FTPS or SFTP for production
- Users are chrooted to their home directories for security
- `user_list` with `userlist_deny=YES` blocks listed users
- `user_list` with `userlist_deny=NO` allows only listed users
- `/etc/vsftpd/ftpusers` always denies listed users (system accounts)
- Passive mode requires additional firewall rules for port range
- For encrypted FTP, configure FTPS with SSL/TLS certificates
- Regular users need valid shell in `/etc/passwd` to login via FTP