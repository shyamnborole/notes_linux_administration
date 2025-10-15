>==**kea-dhcp4**== ---------service name for dhcp
	/etc/kea/kea-dhcp4.conf
```
{
  "Dhcp4": {
    "interfaces-config": {
      // specify network interfaces to listen on
      "interfaces": ["ens160"]
    },
    // settings for expired-leases (defaults shown)
    "expired-leases-processing": {
      "reclaim-timer-wait-time": 10,
      "flush-reclaimed-timer-wait-time": 25,
      "hold-reclaimed-time": 3600,
      "max-reclaim-leases": 100,
      "max-reclaim-time": 250,
      "unwarned-reclaim-cycles": 5
    },
    // T1 timer: client begins the renewal process (sec)
    "renew-timer": 900,
    // T2 timer: client begins the rebind process (sec)
    "rebind-timer": 1800,
    // lease time (seconds) for IP addresses
    "valid-lifetime": 3600,
    "option-data": [
      {
        // specify your DNS server
        // to specify multiple entries, separate with commas
        "name": "domain-name-servers",
        "data": "192.168.100.1"
      }
      //{
      //  "name": "domain-name",
      //  "data": "demo.lab"
      //}
      // specify your domain-search base
    ],
    "subnet4": [
      {
        "id": 1,
        // subnet using DHCP
        "subnet": "192.168.100.1/24",
        // range of IPs to lease
        "pools": [
          { "pool": "192.168.100.50 - 192.168.100.150" }
        ],
        "option-data": [
          {
            // specify your gateway
            "name": "routers",
            "data": "192.168.100.254"
          }
        ]
      }
    ],
    // logging settings
    "loggers": [
      {
        "name": "kea-dhcp4",
        "output-options": [
          {
            "output": "/var/log/kea/kea-dhcp4.log"
          }
        ],
        "severity": "INFO",
        "debuglevel": 0
      }
    ]
  }
}

```
 >==**DNS**==
 >	package name : bind-utils
 >	 service name : named
   

| File/Directory                                  | Purpose                                                                          |
| ----------------------------------------------- | -------------------------------------------------------------------------------- |
| `/etc/named.conf` or `/etc/named.rfc1912.zones` | Main BIND DNS server configuration file defining server options and zones.       |
| `/etc/resolv.conf`                              | Resolver configuration file for client systems specifying DNS servers to query.  |
| `/var/named/`                                   | Directory containing DNS zone files, holding DNS records for configured domains. |
| DNS Zone Files                                  | Files that define DNS resource records for each zone (e.g., `db.example.com`).   |
| /etc/hosts                                      | Local hostname to IP address mappings (static overrides).                        |
| /etc/hostname                                   | Host's own hostname configuration.                                               |

>`/var/named/db.example.zone`
```
>$TTL 86400
@   IN  SOA ns1 admin.example.com. (
        2025101501 ; serial (YYYYMMDDxx)
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        86400      ; minimum TTL
    )
; Nameservers
    IN  NS  ns1

; Other hosts
@    IN  A   192.168.1.100
www  IN  A   192.168.1.101
ftp  IN  CNAME www.example.com.

```
>`/var/named/db.1.168.192.zone`
```
>$TTL 86400
@   IN  SOA ns1 admin.example.com. (
        2025101501 ; serial
        3600
        1800
        604800
        86400
    )
; Nameservers
    IN  NS  ns1

; PTR records - IP to hostname mappings
10  IN  PTR ns1.example.com.   //here dot is imp at the end of the domain to tell                                 it that this is the fully qualified domain name
11  IN  PTR ns2.example.com.
100 IN  PTR example.com.
101 IN  PTR www.example.com.

```
>named.conf Zone Entries Pointing to `/var/named`
```
zone "example.com" IN {
    type master;
    file "/var/named/db.example.zone`";
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/db.1.168.192.zone";
};

```
>simple thing is to just make an copy of /var/named/named.localhost for the zone file




>==**USER MANAGEMENT**==
	`sudo usermod -G xyz abc ` --------makes abc user the member of xyz group, whatever file                                                               has owner group as xyz, the abc user will get access to                                                                that file. a user can be a member of multiple group, this                                                              is called as secondary group.                                              To add user `abc` to supplementary group(s) without removing existing groups, use:  
	`sudo usermod -aG xyz abc`
	`sudo usermod -g xyz abc`--------adds the abc user in the xyz group, whatever he creates                                                              by default the xyz group will have access to the file and                                                              the file group owner will be xyz group. A user can have                                                              only one primary group.

>==**Apache Web Server**==
>