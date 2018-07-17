# Infrastructure setup


## Set up DHCP server

```
sudo apt-get install isc-dhcp-server
```

```
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

host media {
  hardware ethernet 00:0c:29:3c:ce:53;
  fixed-address 10.98.95.231;
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
media.technoff.eu.                      IN      A      10.98.95.231
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
231             IN      PTR     media.technoff.eu.              ; 10.98.95.231
```

Start bind9 and enable it at startup.

```
systemctl enable bind9
systemctl start bind9
```

## Set up NFS server

View storage devices.

```
> lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0 86.6M  1 loop /snap/core/4486
sda      8:0    0    3G  0 disk 
├─sda1   8:1    0    1M  0 part 
└─sda2   8:2    0    3G  0 part /
sdb      8:16   0   40G  0 disk 
sr0     11:0    1 1024M  0 rom  
```

Confirm next partition number to use.

```
> sudo gdisk -l /dev/sda
GPT fdisk (gdisk) version 1.0.3

Problem opening /dev/sdab for reading! Error is 2.
The specified file does not exist!
defo@media:~$ sudo gdisk -l /dev/sda 
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk /dev/sda: 6291456 sectors, 3.0 GiB
Model: Virtual disk    
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 304C050B-5B24-4A9E-AAEA-E53C45DDCFF0
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 6291422
Partitions will be aligned on 2048-sector boundaries
Total free space is 4029 sectors (2.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  
   2            4096         6289407   3.0 GiB     8300  
```

Create partition on a disk dedicated for NFS sharing.

```
> sudo gdisk /dev/sdb
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-83886046, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-83886046, default = 83886046) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8300
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```

Create filesystem on a partition.

```
> sudo mkfs.ext4 -L nfs /dev/sdb1

mke2fs 1.44.1 (24-Mar-2018)
Discarding device blocks: failed - Remote I/O error
Creating filesystem with 10485499 4k blocks and 2621440 inodes
Filesystem UUID: 0968159a-720b-4a5c-9d76-1559e2acf2ad
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done
```

Mount partition on startup.

```
> sudo mkdir /mnt/nfs
> blkid
/dev/sda2: UUID="41de22aa-88eb-11e8-8ae2-000c293cce53" TYPE="ext4" PARTUUID="18878d4a-2df5-4d3c-9efc-c44bee6aee5d"
/dev/sdb1: LABEL="nfs" UUID="0968159a-720b-4a5c-9d76-1559e2acf2ad" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="4631e537-9db0-4079-8a44-382ac6fce7ef"

--> sudo vim /etc/fstab
UUID=41de22aa-88eb-11e8-8ae2-000c293cce53 / ext4 defaults 0 0
UUID=0968159a-720b-4a5c-9d76-1559e2acf2ad / ext4 defaults 0 1

```

Install and configure NFS.

```
sudo apt-get install -y nfs-kernel-server
```

Create a share for the first Persistent Volume.

```
> sudo mkdir /mnt/nfs/pv0001

--> /etc/exports
/mnt/nfs/pv0001        10.98.95.0/24(rw,root_squash,no_wdelay,no_subtree_check)
```

```
sudo systemctl start nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```