# Infrastructure setup


## Set up DHCP server

```
sudo apt-get install isc-dhcp-server

--> /etc/dhcp/dhcpd.conf

# common options
option domain-name "technoff.eu";
option domain-name-servers 10.98.95.230;
default-lease-time 300;
max-lease-time 7200;

authoritative;
infinite-is-reserved on;

# lab subnet
subnet 10.98.95.0 netmask 255.255.255.0 {
  range 10.98.95.50 10.98.95.80;
  option routers 10.98.95.1;
}

# static reservations

host haproxy1 {
  hardware ethernet 00:0c:29:4b:97:87;
  fixed-address 10.98.95.18;
}

host k8s-controller1 {
  hardware ethernet 00:0c:29:e9:9a:17;
  fixed-address 10.98.95.11;
}

host k8s-controller2 {
  hardware ethernet 00:0c:29:4c:6d:64;
  fixed-address 10.98.95.12;
}

host k8s-controller3 {
  hardware ethernet 00:0c:29:c0:ee:3b;
  fixed-address 10.98.95.13;
}

host k8s-worker1 {
  hardware ethernet 00:0c:29:9c:b6:ab;
  fixed-address 10.98.95.21;
}

host k8s-worker2 {
  hardware ethernet 00:0c:29:c8:8c:8a;
  fixed-address 10.98.95.22;
}

host k8s-worker3 {
  hardware ethernet 00:0c:29:e9:9b:9b;
  fixed-address 10.98.95.23;
}
```

Start ISC DHCP server and enable it at startup.

```
systemctl enable isc-dhcp-server
systemctl start isc-dhcp-server
```

## Set up DNS server

Install bind and tools.

```
sudo apt-get install bind9 bind9utils bind9-doc
```

ACL, options and forwarding.

```
--> /etc/bind/named.conf.options

acl clients {
        10.98.0.0/16;
        localhost;
        localnets;
};

options {
        directory "/var/cache/bind";

        dnssec-enable yes;
        dnssec-validation yes;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };

        recursion yes;
        allow-query { clients; };

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
};
```

Local File with forward zone and reverse zone.

```
--> /etc/bind/named.conf.local

zone "technoff.eu" {
    type master;
    file "/etc/bind/zones/db.technoff.eu"; # zone file path
};

zone "95.98.10.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.10.98.95";  # 10.98.95.0/24 subnet
};
```

Forward zone file - define DNS records for forward DNS lookups (DNS name to IP).

```
--> /etc/bind/zones/db.technoff.eu
$TTL    604800
@       IN      SOA     networker.technoff.eu. admin.networker.technoff.eu. (
                  3       ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL
;
; name servers - NS records
     IN      NS      networker.technoff.eu.

; name servers - A records
networker.technoff.eu.          IN      A     10.98.95.230

; 10.98.95.0/24 - A records
k8s-controller1.technoff.eu.            IN      A      10.98.95.11
k8s-controller2.technoff.eu.            IN      A      10.98.95.12
k8s-controller3.technoff.eu.            IN      A      10.98.95.13
k8s-worker1.technoff.eu.                IN      A      10.98.95.21
k8s-worker2.technoff.eu.                IN      A      10.98.95.22
k8s-worker3.technoff.eu.                IN      A      10.98.95.23
k8s-api.technoff.eu.                    IN      A      10.98.95.18
```

Reverse zone file - define DNS PTR records for reverse DNS lookups (IP to DNS name).

```
--> /etc/bind/zones/db.10.98.95
$TTL    604800
@       IN      SOA     networker.technoff.eu. admin.networker.technoff.eu. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; name servers
     IN      NS      networker.technoff.eu.

; PTR Records
230     IN      PTR     networker.technoff.eu.                  ; 10.128.10.230
11              IN      PTR     k8s-controller1.technoff.eu.    ; 10.98.95.11
12              IN      PTR     k8s-controller2.technoff.eu.    ; 10.98.95.12
13              IN      PTR     k8s-controller3.technoff.eu.    ; 10.98.95.13
21              IN      PTR     k8s-worker1.technoff.eu.        ; 10.98.95.21
22              IN      PTR     k8s-worker2.technoff.eu.        ; 10.98.95.22
23              IN      PTR     k8s-worker3.technoff.eu.        ; 10.98.95.23
18              IN      PTR     k8s-api.technoff.eu.            ; 10.98.95.18
```

Start bind9 and enable it at startup.

```
systemctl enable bind9
systemctl start bind9
```
