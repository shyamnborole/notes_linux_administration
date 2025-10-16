Here is a detailed guide to install and configure Squid proxy with access controls as per your requirements:

***

## Install Squid Proxy Server

For Ubuntu/Debian based systems:

```bash
sudo apt update
sudo apt install squid -y
sudo systemctl start squid
sudo systemctl enable squid
```

For RHEL/CentOS based systems (example with CentOS 9):

```bash
sudo dnf install squid -y
sudo systemctl start squid
sudo systemctl enable squid
```

Check the status:

```bash
sudo systemctl status squid
```

***

## Squid Configuration for Access Control

Squid uses ACLs (Access Control Lists) and `http_access` directives for access control.

***

## Step 1: Define Clients

Identify clients by IP addresses. For example:

- Client 1 IP: 192.168.1.10
- Client 2 IP: 192.168.1.20

***

## Step 2: Prepare Domain Lists

Create files for allowed or blocked domains/keywords:

1. Banking sites for client 1 + google:

**/etc/squid/banking_sites.txt**

```
.hdfcbank.com
.onlinesbi.com
.axisbank.com
.icicibank.com
.google.com
```

2. Development sites for client 2 + google:

**/etc/squid/development_sites.txt**

```
.redhat.com
.microsoft.com
.oracle.com
.python.org
.cisco.com
.google.com
```

3. Blocked keywords (print each keyword in a separate line):

**/etc/squid/blocked_keywords.txt**

```
sports
torrent
netflix
facebook
movies
cricket
```

***

## Step 3: Edit /etc/squid/squid.conf

Add the following ACL and http_access rules:

```conf
# Define clients by IP
acl client1 src 192.168.1.10
acl client2 src 192.168.1.20

# Define allowed sites for client1 and client2
acl banking_sites dstdomain "/etc/squid/banking_sites.txt"
acl development_sites dstdomain "/etc/squid/development_sites.txt"

# Define blocked keywords anywhere in URL (case insensitive)
acl blocked_keywords url_regex -i "/etc/squid/blocked_keywords.txt"

# Access control rules

# Block sites containing blocked keywords for all network
http_access deny blocked_keywords

# Client 1 allowed sites only (banking + google)
http_access allow client1 banking_sites

# Client 2 allowed sites only (development + google)
http_access allow client2 development_sites

# Block all other websites for all clients
http_access deny all
```

***

## Step 4: Restart Squid Service

```bash
sudo systemctl restart squid
```

***

## Summary

- Client 1 can access banking sites + Google.
- Client 2 can access development sites + Google.
- All clients are blocked from sites containing the blocked keywords.
- All other websites are blocked network-wide.

This setup uses domain name ACLs, blocked keyword regex ACL, and client IP ACLs to control access precisely.

If you want, user authentication can also be added for extra control.
