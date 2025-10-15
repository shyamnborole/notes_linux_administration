
**Installing GUI in minimal installation**
	mount the cdrom, and add it to the yum.repos.d folder and do the all basic mounting stuff.
	`sudo yum groupinstall "Server with GUI"`
	
	
	
**ACL for proxy squid**
acl mynet src 192.168.50.0/24
acl srv1 src 192.168.50.10
acl srv2 src 192.168.50.15
acl microsoft dstdomain .microsoft.com
http_access deny microsoft srv1



http_access allow mynet -----should be at last to allow all otherwise