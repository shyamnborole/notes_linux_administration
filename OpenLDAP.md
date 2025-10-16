**An active directory  sort of similar implementation for Linux**
>Still in development, is glitchy

/etc/resolv.conf --------------where dns address is stored
-frst set hostname
-hostname should have domain name as this is the domain controller
-ldapsrv1.demo.lab
`sudo yum install epel-release -y` -------adds a repo(epel in our yum repo directory for                                                                                    additional repos(extra packet for enterprise linux)
`sudo yum update`
	-Go to Putty
	-sudo vi /etc/hosts -----------------local dns file
		192.168.65.131                    ldapsrv.demo,lab                            ldapsrv
		==ip of the server==                  ==fqdn==                                                 ==hostname's first==
	-sudo yum install openldap-servers openldap-client -y
	- `systemctl start slapd` -------------service name for openldap
	-` systemctl status slapd `
	- `sudo firewall-cmd --add-service={ldap,ldaps} --permanent`
	- In windows admin is by default the manager of the domain, but here we have to specify the password for the manager `slappasswd` then give password, hash key will be generated copy that key, and paste it in some file, we will need it later.
	- ALL LDAP RELATED FILE EXTENSION IS .ldif
	- in ~ `sudo vi changerootpasswd.ldif` and then change/override the by default password
	File content, whatever password hash key we copied earlier, we are supposed to paste it here
	```
	```dn: olcDatabase={0}config,cn=config
	changetype: modify
	add: oldRootPW
	oldRootPW: (paste here)
	```
	```
	-`ldap -Y EXTERNAL -H ldapi:/// -f changerootpass.ldif` ------------set pass for ldap                                                                                                                            user
	-`ls /etc/openldap/schema`
	- `vi setdomain.ldif` in ~
    Cn (common name) = username
    Dc(domain name)=demo
    Dc=lab
    Dn (distinguished name)
    - `sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f setdomain.ldif
    - `sudo ldapsearch -H ldap:// -x -s base -b "" -LLL"nameingContexts" ` -----------to                                                                                                                                 verify dn and                                                                                                                                      cn 
    - ==creating ou (organizational unit)==  
     In ~/ `vi addou.ldif`
     `sudo ldapadd -x -D cn=Manager,dc=demo,dc=lab -W -f addou.ldif`
     `ldapsearch -x -b "dc=demo,dc=lab" ou` ------searching ou in our domain
     -==add user==
     `~/sudo vi adduser.ldif` ------start uid and gid from 2000, so it should not overlap with                                                          client  uid or gid
     `ldapadd -x -D[^2] cn=Manager, dc=demo,dc=lab -W -f[^1] adduser.ldif`
     `ldapsearch -x -b "ou=People,dc=demo,dc=lab"`

[^1]: File name

[^2]: For login at the domain controller, our username for domain controller is manager


# **==For Client==**

- `sudo yum install openldap-client oddjob-mkhomedir(creates home directory for the created users)`
- `sudo yum install epel-release -y`
- `sudo yum install nss-pam-ldap -y`
- `sudo vi /etc/nslcd.conf`
 Base dc=demo,dc=lab 
 - `sudo vi /etc/nsswitch.conf` ------ whenever linux get username and password this file tells                                                               where to look for the authentication,
 > passwd        ldap
Group            ldap

- `sudo systemctl start nslcd`
- `sudo systemctl status nslcd`
- `getent passwd` ------will show users on the ldap server
- `id ldap1` --------verifies the uid of the user
- 