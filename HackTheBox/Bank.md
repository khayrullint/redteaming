10.10.10.29

We have 53/TCP open, which is unusual.
That usually indicates zone transfer might be enabled.

root@kali > nslookup
    # Set the DNS server
    > SERVER 10.10.10.29
        Default server: 10.10.10.29
        Address: 10.10.10.29#53
    # Check if the server expose some data about itself - nothing
    > 127.0.0.1
        1.0.0.127.in-addr.arpa  name = localhost.
    # Check if the server expose some data about itself - nothing
    > 10.10.10.29
        ** server can't find 29.10.10.10.in-addr.arpa: NXDOMAIN
    # Assume the server's name is bank.htb - we are right
    > bank.htb
        Server:         10.10.10.29
        Address:        10.10.10.29#53

        Name:   bank.htb
        Address: 10.10.10.29

# Let's try to find any local records with dnsrecon - failed
root@kali > dnsrecon -r 127.0.0.1/24 -n 10.10.10.29                
    dns.resolver.NoNameservers: All nameservers failed to answer the query F40142531285a7b906DA.com. IN A: Server 10.10.10.29 TCP port 53 answered REFUSED

# Let's try to dns zone transfer data - nothing
root@kali > dig axfr @10.10.10.29
    ; <<>> DiG 9.11.5-P4-5-Debian <<>> axfr @10.10.10.29
    ; (1 server found)
    ;; global options: +cmd
    ;; Query time: 86 msec
    ;; SERVER: 10.10.10.29#53(10.10.10.29)
    ;; WHEN: Tue Jun 18 08:27:28 EDT 2019
    ;; MSG SIZE  rcvd: 28

# Let's try to dns zone transfer data with known name - found extra names  
root@kali > dig axfr bank.htb @10.10.10.29
    ; <<>> DiG 9.11.5-P4-5-Debian <<>> axfr bank.htb @10.10.10.29
    ;; global options: +cmd
    bank.htb.               604800  IN      SOA     bank.htb. chris.bank.htb. 2 604800 86400 2419200 604800
    bank.htb.               604800  IN      NS      ns.bank.htb.
    bank.htb.               604800  IN      A       10.10.10.29
    ns.bank.htb.            604800  IN      A       10.10.10.29
    www.bank.htb.           604800  IN      CNAME   bank.htb.
    bank.htb.               604800  IN      SOA     bank.htb. chris.bank.htb. 2 604800 86400 2419200 604800
    ;; Query time: 21 msec
    ;; SERVER: 10.10.10.29#53(10.10.10.29)
    ;; WHEN: Thu Jun 20 15:43:48 EDT 2019
    ;; XFR size: 6 records (messages 1, bytes 171)

# We can either add the names to /etc/hosts or just add the DNS server
root@kali > vi /etc/resolv.conf
    nameserver 10.10.10.29
