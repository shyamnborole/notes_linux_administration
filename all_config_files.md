>kea-dhcp4 ---------service name for dhcp
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
