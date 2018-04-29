# DeepDNS ?

DeepDNS is part of cryptostorm's internal DNS infrastructure.    
It's a combination of several different DNS related programs that provide our users with direct access to most darknet resources (.onion, .i2p, .bit, .p2p, .eth etc.) in a transparent way, so no additional software is required.   
It also provides more secure alternatives to traditional DNS.    
All of our DeepDNS servers also act as public DNS servers.   
This is mostly so our users can protect their DNS when they're connecting to cryptostorm, but it's also for anyone else who wants to use DNS servers that don't log their activities.   

# techie explaination

In the rest of this README, two abbreviations are used for the sake of brevity:   

  - CS = <a href="https://cryptostorm.is" target="_blank">cryptostorm</a>
  - ddns = <a href="https://cryptostorm.org/viewforum.php?f=46" target="_blank">deepDNS</a>

For cryptostorm clients, DNS over ddns normally happens as such:
 * client connects to CS, OpenVPN pushes the exit node's ddns IP to the client, client then uses that for all DNS requests.
 * client tries to resolve whatever.
 * DNS request hits the internet facing <a href="https://doc.powerdns.com/md/recursor?" target="_blank">powerdns-recursor</a> process.
 * * Our pdns-recursor uses this in it's config:  
 * * *    forward-zones=bit.=127.0.0.1:5333,dns.=127.0.0.1:5333,eth.=127.0.0.1:5333,p2p.=127.0.0.1:5333,onion.=127.0.0.1:5335,i2p.=127.0.0.1:5399
 * * * * That translates to:
 * * * * * send DNS requests for whatever.bit to the DNS server at 127.0.0.1:5333
 * * * * * whatever.dns to the DNS server at 127.0.0.1:5333
 * * * * * whatever.eth to the DNS server at 127.0.0.1:5333
 * * * * * whatever.p2p to the DNS server at 127.0.0.1:5333
 * * * * * whatever.onion to the DNS server at 127.0.0.1:5335
 * * * * * whatever.i2p to the DNS server at 127.0.0.1:5399
 
* 127.0.0.1:5333 is DNSChain, which forwards to namecoind to provide blockchain-based DNS for those TLDs (.bit, .dns, .eth, .p2p).
*  127.0.0.1:5335 is tor, or more specifically, a tor instance with "DNSPort 127.0.0.1:5335" in it's torrc.
*  127.0.0.1:5399 is for .i2p, but this one is a little tricky...
*  the daemon actually listening on port 5399 is unbound,
  (only because I don't know of a way to add a single, static wildcard A record for a TLD to pdns-recursor, so unbound is used).
* *  unbound does:
* * * local-zone: "i2p" redirect
* * * local-data: "i2p A 10.98.0.1"
* * so all *.i2p domains resolve to 10.98.0.1. 
* privoxy is then setup to listen on ddns-ip:8118 and to forward all .i2p requests to i2prouter's http proxy server on port 4444.
* * (the lovely config directive "forward / fuckoff" is used in privoxy to prevent people from using privoxy to go anywhere else, which is redundant since this is a VPN, but it does prevent 10.* access, and iptables stops anything external, plus it's funny =P).
* Then iptables comes into play:
* * iptables -t nat -A OUTPUT     -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:8118
* * iptables -t nat -A PREROUTING -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:8118
* That means DNS requests for .i2p (planet.i2p is my goto test site) goes to unbound, which goes to privoxy, which goes to iptables,
  which goes to the http proxy the actual i2p daemon uses, which was executed with `~/i2p --httpproxyport=4444`.
  
# And for the paranoid:
* privoxy, i2p, tor, basically everything mentioned above that binds to an internet accessible IP is firewalled with iptables to prevent the internet from touching it.
Also, every single instance runs under an extremely restrictive account.

# Another fun fact:     
The powerdns-recursor servers mentioned way above are internet accessible. 
That's intentional.

  Anyone who wants to force their system to use our DNS servers before they even connect (to prevent any degree of ISP level DNS hijacks), can do so. So if you worry your ISP provided DNS servers are caching, poisoning, whatever. Use CS's ddns. All are welcome :)

They are public DNS servers, and we maintain a full list of all the <a href="https://github.com/cryptostorm/cstorm_deepDNS/blob/master/dnscrypt-resolvers.csv" target="_blank">deepDNS public resolver IPs</a>, which can also be enumerated via a DNS lookup of "public.deepdns.net".

It's possible that there will be a future split between the public DeepDNS resolvers and those used for the VPN, but we haven't yet seen a good reason to implement such a split so have not yet done so.
