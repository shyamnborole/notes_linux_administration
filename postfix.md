==#📧 **POSTFIX MAIL SERVER - ROCKY LINUX**==

---

==**INSTALLATION**==
> Package name: postfix, mutt  
> Service name: postfix  
> Default port: 25 (SMTP)

```bash
# Install postfix and mutt
sudo dnf install postfix mutt -y

# Stop and disable sendmail if running
sudo systemctl stop sendmail
sudo systemctl disable sendmail

# Start and enable postfix
sudo systemctl start postfix
sudo systemctl enable postfix
sudo systemctl status postfix
```

**Open Firewall:**
```bash
# Open SMTP port in firewall
sudo firewall-cmd --permanent --add-service=smtp
sudo firewall-cmd --reload

# Verify firewall rules
sudo firewall-cmd --list-all
```

---

==**IMPORTANT FILES AND DIRECTORIES**==

| Purpose | Path |
|---------|------|
| Main config file | `/etc/postfix/main.cf` |
| Master config file | `/etc/postfix/master.cf` |
| Mail spool directory | `/var/spool/mail/` or `/var/mail/` |
| Postfix queue | `/var/spool/postfix/` |
| Mail logs | `/var/log/maillog` |
| Mutt config (global) | `/etc/Muttrc` |
| Mutt config (user) | `~/.muttrc` |
| Skeleton directory | `/etc/skel/` |

---

==**STEP 1: CONFIGURE HOSTNAME**==

```bash
# Set hostname
sudo hostnamectl set-hostname mail.demo.lab

# Verify hostname
hostname
hostname -f

# Update /etc/hosts
sudo nano /etc/hosts
```

**Add to `/etc/hosts`:**
```
127.0.0.1   localhost localhost.localdomain
192.168.1.100   mail.demo.lab mail
```

---

==**STEP 2: CONFIGURE POSTFIX**==

**Backup original config:**
```bash
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
```

**Edit main configuration:**
```bash
sudo nano /etc/postfix/main.cf
```

**Add/modify these settings:**
```conf
# Server hostname
myhostname = mail.demo.lab

# Domain name
mydomain = demo.lab

# Origin of outgoing mail
myorigin = $mydomain

# Network interfaces to listen on (all interfaces)
inet_interfaces = all

# Trusted networks (allow local network)
mynetworks = 127.0.0.0/8, 192.168.1.0/24

# Destination domains
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Home mailbox location
home_mailbox = Maildir/

# Mail directory
mail_spool_directory = /var/mail
```

**Or use commands to modify:**
```bash
sudo postconf -e "myhostname = mail.demo.lab"
sudo postconf -e "mydomain = demo.lab"
sudo postconf -e "myorigin = \$mydomain"
sudo postconf -e "inet_interfaces = all"
sudo postconf -e "mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain"
sudo postconf -e "mynetworks = 127.0.0.0/8, 192.168.1.0/24"
sudo postconf -e "home_mailbox = Maildir/"
```

**Restart postfix:**
```bash
sudo systemctl restart postfix
sudo systemctl status postfix
```

---

==**STEP 3: CREATE MUTTRC FILE IN /etc/skel**==

```bash
# Create .muttrc in /etc/skel (for new users)
sudo tee /etc/skel/.muttrc > /dev/null << 'EOF'
# Mutt configuration file

# Set default email domain
set hostname = "demo.lab"
set from = "$USER@demo.lab"
set realname = "$USER"

# Use local sendmail (postfix)
set sendmail = "/usr/sbin/sendmail -oem -oi"

# Mailbox location
set mbox_type = Maildir
set folder = "~/Maildir"
set spoolfile = "+/"
set mbox = "+/"
set record = "+/Sent"
set postponed = "+/Drafts"

# Sort by threads
set sort = threads
set sort_aux = reverse-last-date-received

# Show headers
ignore *
unignore from: to: cc: subject: date:

# Colors (optional)
color normal white default
color indicator brightwhite blue
color status brightgreen blue
color tree brightmagenta default

# Editor
set editor = "vi"
EOF

# Verify file created
sudo cat /etc/skel/.muttrc
```

---

==**STEP 4: CREATE USERS SAM AND JACK**==

```bash
# Create user sam
sudo useradd -m -s /bin/bash sam
sudo passwd sam

# Create user jack
sudo useradd -m -s /bin/bash jack
sudo passwd jack

# Verify users created
id sam
id jack

# Check if .muttrc is in their home directories
ls -la /home/sam/.muttrc
ls -la /home/jack/.muttrc

# Create Maildir for both users
sudo mkdir -p /home/sam/Maildir/{new,cur,tmp}
sudo mkdir -p /home/jack/Maildir/{new,cur,tmp}
sudo chown -R sam:sam /home/sam/Maildir
sudo chown -R jack:jack /home/jack/Maildir
sudo chmod -R 700 /home/sam/Maildir
sudo chmod -R 700 /home/jack/Maildir
```

---

==**STEP 5: LOGIN AS SAM AND SEND EMAIL**==

```bash
# Switch to sam user
su - sam

# Send email using mutt
mutt jack@demo.lab

# In mutt interface:
# 1. Enter subject when prompted: "Hello from Sam"
# 2. Press Enter
# 3. Mutt will open editor (vi)
# 4. Type your message:
#    Hi Jack,
#    This is a test email from sam.
#    Regards,
#    Sam
# 5. Save and exit: Press Esc, then :wq
# 6. Press 'y' to send
# 7. Email sent!

# Alternative: Send email directly
echo "This is a test email from sam" | mutt -s "Test Email" jack@demo.lab

# Exit sam user
exit
```

**Complete mutt send command:**
```bash
# As sam user
echo "Hi Jack,

This is a test email from sam user.

Best regards,
Sam" | mutt -s "Hello from Sam" jack@demo.lab
```

---

==**STEP 6: LOGIN AS JACK AND CHECK EMAIL**==

```bash
# Switch to jack user
su - jack

# Check email using mutt
mutt

# In mutt:
# - You should see the email from sam
# - Press Enter to read the email
# - Press 'r' to reply
# - Type your reply message in the editor
# - Save and exit: Esc, then :wq
# - Press 'y' to send reply

# Exit mutt
# Press 'q' to quit

# Exit jack user
exit
```

**Alternative check:**
```bash
# As jack user
# Check mail spool directly
cat /var/mail/jack

# OR check Maildir
ls -l ~/Maildir/new/
cat ~/Maildir/new/*
```

---

==**STEP 7: REPLY TO EMAIL (AS JACK)**==

**Method 1: Using mutt interactively**
```bash
# As jack user
mutt

# Navigate to email from sam (use arrow keys)
# Press Enter to open
# Press 'r' to reply
# Type reply message in editor
# Save and exit (Esc :wq)
# Press 'y' to send
```

**Method 2: Send new email to sam**
```bash
# As jack user
echo "Hi Sam,

Thanks for your email. I received it successfully.

Best regards,
Jack" | mutt -s "Re: Hello from Sam" sam@demo.lab
```

---

==**COMPLETE COMMAND SEQUENCE**==

**On Mail Server (as root):**
```bash
# 1. Install packages
sudo dnf install postfix mutt -y

# 2. Configure hostname
sudo hostnamectl set-hostname mail.demo.lab
echo "192.168.1.100   mail.demo.lab mail" | sudo tee -a /etc/hosts

# 3. Configure postfix
sudo postconf -e "myhostname = mail.demo.lab"
sudo postconf -e "mydomain = demo.lab"
sudo postconf -e "myorigin = \$mydomain"
sudo postconf -e "inet_interfaces = all"
sudo postconf -e "mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain"
sudo postconf -e "mynetworks = 127.0.0.0/8, 192.168.1.0/24"
sudo postconf -e "home_mailbox = Maildir/"

# 4. Start postfix
sudo systemctl restart postfix
sudo systemctl enable postfix

# 5. Configure firewall
sudo firewall-cmd --permanent --add-service=smtp
sudo firewall-cmd --reload

# 6. Create .muttrc in /etc/skel
sudo tee /etc/skel/.muttrc > /dev/null << 'EOF'
set hostname = "demo.lab"
set from = "$USER@demo.lab"
set realname = "$USER"
set sendmail = "/usr/sbin/sendmail -oem -oi"
set mbox_type = Maildir
set folder = "~/Maildir"
set spoolfile = "+/"
set editor = "vi"
EOF

# 7. Create users
sudo useradd -m -s /bin/bash sam
sudo useradd -m -s /bin/bash jack
sudo passwd sam
sudo passwd jack

# 8. Create Maildir
sudo mkdir -p /home/sam/Maildir/{new,cur,tmp}
sudo mkdir -p /home/jack/Maildir/{new,cur,tmp}
sudo chown -R sam:sam /home/sam/Maildir
sudo chown -R jack:jack /home/jack/Maildir
sudo chmod -R 700 /home/sam/Maildir
sudo chmod -R 700 /home/jack/Maildir
```

**As sam user:**
```bash
# Login as sam
su - sam

# Send email to jack
echo "Hi Jack,

This is a test email from sam.

Best regards,
Sam" | mutt -s "Hello from Sam" jack@demo.lab

# Exit
exit
```

**As jack user:**
```bash
# Login as jack
su - jack

# Check email
mutt
# Read email, press 'r' to reply, compose reply, send

# OR send direct reply
echo "Hi Sam,

Thanks for your email!

Best regards,
Jack" | mutt -s "Re: Hello from Sam" sam@demo.lab

# Exit
exit
```

---

==**VERIFY EMAIL DELIVERY**==

```bash
# Check mail logs
sudo tail -f /var/log/maillog

# Check postfix queue
mailq
sudo postqueue -p

# Check user's mailbox (as root)
sudo ls -l /var/mail/
sudo cat /var/mail/jack
sudo cat /var/mail/sam

# Check Maildir
sudo ls -l /home/jack/Maildir/new/
sudo ls -l /home/sam/Maildir/new/
```

---

==**MUTT COMMANDS REFERENCE**==

| Key | Action |
|-----|--------|
| `mutt` | Open mutt mail client |
| `mutt user@domain` | Compose new email to recipient |
| `Enter` | Open/read email |
| `r` | Reply to email |
| `g` | Group reply (reply all) |
| `f` | Forward email |
| `d` | Delete email |
| `u` | Undelete email |
| `s` | Save email to folder |
| `c` | Change to different mailbox |
| `m` | Compose new message |
| `q` | Quit mutt |
| `?` | Show help |
| `/pattern` | Search for pattern |

**Compose commands:**
```bash
# Send email with subject
echo "Message body" | mutt -s "Subject" recipient@domain

# Send email with attachment
mutt -s "Subject" -a /path/to/file recipient@domain

# Send to multiple recipients
mutt -s "Subject" user1@domain user2@domain
```

---

==**SELINUX CONFIGURATION (IF ENABLED)**==

```bash
# Check SELinux status
getenforce

# Set SELinux booleans for postfix
sudo setsebool -P allow_postfix_local_write_mail_spool on

# Restore contexts
sudo restorecon -Rv /var/mail/
sudo restorecon -Rv /var/spool/postfix/
sudo restorecon -Rv /home/*/Maildir/
```

---

==**TROUBLESHOOTING**==

```bash
# Check postfix status
sudo systemctl status postfix

# Check if postfix is listening
sudo netstat -tlnp | grep :25
sudo ss -tlnp | grep :25

# View mail logs
sudo tail -50 /var/log/maillog
sudo journalctl -u postfix -f

# Test postfix configuration
sudo postfix check

# Flush mail queue
sudo postfix flush

# Check mail queue
mailq

# Test sending mail locally
echo "Test" | mail -s "Test Subject" root

# Check postfix configuration
sudo postconf -n

# Verify hostname
hostname
hostname -f

# Check DNS/hosts resolution
ping mail.demo.lab
getent hosts mail.demo.lab

# Check user mailbox
ls -la /home/sam/Maildir/
ls -la /home/jack/Maildir/
```

**Common Issues:**

| Issue | Solution |
|-------|----------|
| Mail not delivered | Check `/var/log/maillog` for errors |
| Connection refused | Verify postfix is running: `systemctl status postfix` |
| Permission denied | Check Maildir ownership and permissions |
| Relay denied | Verify `mynetworks` in `main.cf` |
| Name resolution failed | Check `/etc/hosts` and hostname |

---

==**POSTFIX CONFIGURATION EXPLAINED**==

| Parameter | Description |
|-----------|-------------|
| `myhostname` | Full hostname of mail server |
| `mydomain` | Domain name |
| `myorigin` | Domain name that appears in sender address |
| `inet_interfaces` | Network interfaces postfix listens on |
| `mydestination` | Domains for which postfix accepts mail |
| `mynetworks` | Trusted networks that can relay mail |
| `home_mailbox` | Mailbox location relative to home directory |

---

==**MAILDIR VS MBOX**==

| Format | Location | Description |
|--------|----------|-------------|
| **mbox** | `/var/mail/username` | Single file containing all emails |
| **Maildir** | `~/Maildir/` | Directory structure with separate files |

**Maildir structure:**
```
~/Maildir/
├── cur/     # Current emails (read)
├── new/     # New unread emails
└── tmp/     # Temporary files
```

---

==**USEFUL POSTFIX COMMANDS**==

```bash
# Reload configuration
sudo postfix reload

# Stop postfix
sudo postfix stop

# Start postfix
sudo postfix start

# Check configuration
sudo postfix check

# Show configuration
sudo postconf

# Show only non-default settings
sudo postconf -n

# Flush mail queue
sudo postfix flush

# View queue
mailq
postqueue -p

# Delete all mail in queue
sudo postsuper -d ALL

# Delete deferred mail
sudo postsuper -d ALL deferred
```

---

==**IMPORTANT NOTES**==

- This is a **local mail server** configuration (not for internet mail)
- For production, configure proper DNS records (MX, A, PTR)
- For internet mail, configure authentication (SASL) and encryption (TLS)
- Default configuration accepts mail only for local domains
- Users must exist on the system to receive mail
- `.muttrc` in `/etc/skel/` is copied to new user home directories
- Maildir is preferred over mbox for concurrent access
- Always check `/var/log/maillog` for troubleshooting
- Postfix is more secure and feature-rich than sendmail