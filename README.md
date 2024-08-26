# Install and configure DNS Server bind9
How to install and configure bind9 on Ubuntu22.04

In my case, I will use the IP 10.0.26.71 on the DNS server and apply the domain ogomez.local. You should change the IP address to yours and use the domain name you want.

## Install

Update the repository and install bind9

```
sudo apt update
sudo apt install bind9 bind9utils bind9-doc
```

## Configure

Edit the local BIND configuration file:

```
sudo nano /etc/bind/named.conf.local
```

Add the following zones for your ogomez.local domain in /etc/bind/named.conf.local:

```
zone "ogomez.local" {
    type master;
    file "/etc/bind/db.ogomez.local";
};

zone "26.0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.0.26";
};

```
Create the forward zone file for ogomez.local:

```
sudo cp /etc/bind/db.local /etc/bind/db.ogomez.local
sudo nano /etc/bind/db.ogomez.local
```
Edit the file to look like this:

```
;
; BIND data file for ogomez.local
;
$TTL    604800
@       IN      SOA     ns1.ogomez.local. root.ogomez.local. (
                             2         ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.ogomez.local.
ns1     IN      A       10.0.26.71
@       IN      A       10.0.26.71
www     IN      A       10.0.26.71
```

Create the reverse zone file:

```
sudo cp /etc/bind/db.127 /etc/bind/db.10.0.26
sudo nano /etc/bind/db.10.0.26
```

Edit the file to look like this:

```
;
; BIND reverse data file for 10.0.26.0/24
;
$TTL    604800
@       IN      SOA     ns1.ogomez.local. root.ogomez.local. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.ogomez.local.
71      IN      PTR     ns1.ogomez.local.
```

Before restarting BIND, check the configuration:

```
sudo named-checkconf
sudo named-checkzone ogomez.local /etc/bind/db.ogomez.local
sudo named-checkzone 26.0.10.in-addr.arpa /etc/bind/db.10.0.26
```

If everything is correct, restart BIND9:

```
sudo systemctl restart bind9
```

## Configure DNS on your server

Finally, configure your server to use the local DNS. Edit the /etc/resolv.conf file:

```
nameserver 127.0.0.1
options edns0 trust-ad
search ogomez.local
```

## Verify the configuration

```dig ogomez.local``` or ```nslookup ogomez.local``` 

You should get a response showing the IP address 10.0.26.71 for ogomez.local.
