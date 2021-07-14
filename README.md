# DeepDNS ?

DeepDNS is part of cryptostorm's internal DNS infrastructure.    
It's a combination of several different DNS related programs that provide our users with direct access to some darknet resources (.onion and .i2p) in a transparent way, so no additional software is required.   
It also provides more secure alternatives to traditional DNS via DNSCrypt v2.    
All of our DeepDNS servers also act as regular public DNS servers.   
This is mostly so our users can protect their DNS when they're connecting to cryptostorm, but it's also for anyone else who wants to use DNS servers that don't log their activities.   

2021 update:   
It looks like DNSChain hasn't been updated in several years, so we're dropping support for that (and .bit, .eth, .dns, etc.). The below text has been updated to reflect the current DeepDNS setup. <b><i>We do still support transparent .onion and .i2p</i></b>, dropping DNSChain support only means no more .bit/.eth/.dns.

If you're looking for the current DeepDNS IPs, you can find them by resolving "public.deepdns.net", or by going to https://cryptostorm.is/dns.txt

# techie explaination

In the rest of this README, two abbreviations are used for the sake of brevity:   

  - CS = <a href="https://cryptostorm.is" target="_blank">cryptostorm</a>
  - ddns = <a href="https://cryptostorm.org/viewforum.php?f=46" target="_blank">deepDNS</a>

For cryptostorm clients, DNS over ddns normally happens as such:
 * client connects to CS, OpenVPN pushes the exit node's ddns IP to the client, client then uses that for all DNS requests.
 * client tries to resolve whatever.
 * DNS request hits the internet facing <a href="https://doc.powerdns.com/md/recursor?" target="_blank">powerdns-recursor</a> process.
 * * Our pdns-recursor uses this in it's config:  
 * * *    forward-zones=onion.=127.0.0.1:5335
 * * *    auth-zones=i2p=/var/zones/i2p
 * * * * That translates to:
 * * * * * send DNS requests for whatever.onion to the DNS server at 127.0.0.1:5335
 * * * * * use the zone file /var/zones/i2p for DNS requests for whatever.i2p

 
*  127.0.0.1:5335 is tor, or more specifically, a tor instance with "DNSPort 127.0.0.1:5335" in it's torrc.
*  the /var/zones/i2p zone file contains the single line: *.i2p. IN A 10.98.0.1
* any request for an .i2p domain will resolve to 10.98.0.1, which is handled by iptables:
* * iptables -t nat -A OUTPUT     -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:4444
* * iptables -t nat -A PREROUTING -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:4444
* The server on port 4444 is an http proxy the actual i2pd daemon has built-in

The powerdns-recursor servers mentioned above are internet accessible. 
That's intentional.
Anyone who wants to their system to use our DNS servers before they connect can do so. 
They are public DNS servers, and we maintain a full list of all the <a href="https://cryptostorm.is/dns.txt" target="_blank">deepDNS public resolver IPs</a>, which can also be enumerated via a DNS lookup of "public.deepdns.net".  
Obviously, you would need to be connected to <a href="https://cryptostorm.is/" target="_blank">cryptostorm</a> in order to use the transparent .i2p/.onion feature described above.

For anyone who wants more security than regular DNS offers (which is basically none), DNSCrypt v2 is also supported.   
Server-side, we run <a href="https://github.com/jedisct1/encrypted-dns-server" target="_blank">https://github.com/jedisct1/encrypted-dns-server</a> on TCP port 443 of every DeepDNS IP, and we also supported Anonymized DNSCrypt relays. See <a href="https://cryptostorm.is/blog/anondns" target="_blank">https://cryptostorm.is/blog/anondns</a> for more info on that.
