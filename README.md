# deepDNS - wtf?

df_cryptostorm has been threatened, harrassed, abused (okay, maybe just asked nicely) for technical details on the internal structure of deepDNS that cryptostorm uses network-wide. So let's see if I can articumulate...

  - CS = cryptostorm
  - ddns = deepDNS
  - For the sake of brevity

DNS over ddns normally happens as such:
 * client gets on CS, OpenVPN pushes the exit node's ddns IP to the client, client then uses that for all DNS requests.
 * client tries to resolve whatever..
 * DNS request hits the internet facing CurveDNS (http://curvedns.on2it.net/) process.
 * *Note: we haven't actually implemented DNSCurve at this point, but once we figure out a half-decent way to do so for Windows ppl, we can implement it with this existing structure.*
 * *Another note: The reason we prefer CurveDNS as the public facing daemon, is simply because the code appears to be solid, and we'll be using it's other features very soon.*
 * So since the request is regular(ish) DNS and not CurveDNS, it gets forwarded to the authoritive DNS server running on 127.0.0.1:53 (on the exit node).
 * 127.0.0.1:53 is the powerdns-recursor process (https://doc.powerdns.com/md/recursor/).
 * Our pdns-recursor uses this in it's config:  
*     forward-zones=bit.=127.0.0.1:5333,dns.=127.0.0.1:5333,eth.=127.0.0.1:5333,p2p.=127.0.0.1:5333,onion.=127.0.0.1:5335,i2p.=127.0.0.1:5399
 * That translates to:
 * * send DNS requests for whatever.tld to the DNS server at 127.0.0.1:5333
* *  * whatever.dns to the DNS server at 127.0.0.1:5333
* *  * whatever.eth to the DNS server at 127.0.0.1:5333
* *  * whatever.p2p to the DNS server at 127.0.0.1:5333
* *  * whatever.onion to the DNS server at 127.0.0.1:5335
* *  * whatever.i2p to the DNS server at 127.0.0.1:5399
 
* 127.0.0.1:5333 is DNSChain, which forwards to namecoind for true blockchain-based DNS for those TLDs (.bit, .dns, .eth, .p2p).
*  127.0.0.1:5335 is tor, more specifically, a tor instance with "DNSPort 127.0.0.1:5335" in it's torrc.
*  127.0.0.1:5399 is for .i2p, but this one is a little tricky...
*  * the daemon actually listening on this port is unbound,
  (only because I don't know a way to add a single, static wildcard A record for a TLD to pdns-recursor, so unbound is used).
* *  unbound does:
* * * local-zone: "i2p" redirect
* * * local-data: "i2p A 10.98.0.1"
* * so all *.i2p domains resolve to 10.98.0.1. 
* privoxy is then setup to listen on ddns-ip:8118 and to forward all .i2p requests to the socks server on port 4444.
* * (the lovely config directive "forward / fuckoff" is used in privoxy to prevent ppl from using privoxy to go anywhere else, which is redundant since this is a VPN, but it does prevent 10.* access, and iptables stops anything external, plus it's funny =P).
* Then iptables comes into play:
* * iptables -t nat -A OUTPUT     -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:8118
* * iptables -t nat -A PREROUTING -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:8118
* That means DNS requests for .i2p (planet.i2p is my goto test site btw) gets to unbound, which gets to privoxy, which goes to iptables,
  which goes to the http proxy the actual i2p daemon uses, which was executed with `~/i2p --httpproxyport=4444`.
  
# And for the paranoid:
* privoxy, i2p, tor, basically everything mentioned above that binds to an internet accessible IP is firewalled with iptables to prevent the internet from touching it.
Also every single instance that uses a port < 1024 runs under an extremely restricitve account.

# Another fun fact:     
The CurveDNS daemons mentioned way above are internet accessible. 
 That's intentional.  
 It's so that once we implement DNSCurve client-side, CS members won't be susceptible to DNS hijacking (or any of the other horrible problems with the DNS protocol).
 
  And anyone who wants to force their system to use our DNS servers before they even connect (to prevent any degree of ISP level DNS hijacks), can do so.  
 So in the mean-time, if you worry your ISP provided DNS servers are caching, poisoning, whatever. Use CS's ddns.    
All are welcome :) . They are public DNS servers.

# deepDNS - I'm not on CS though
All of our deepdns servers are accessible outside of CS. That's intentional.
