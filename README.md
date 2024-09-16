# DeepDNS

DeepDNS is part of cryptostorm's internal DNS infrastructure.    
It's a combination of several different DNS related programs that provide our users with direct access to some darknet resources (.onion and .i2p) in a transparent way, so no additional software is required.   
It also provides more secure alternatives to traditional DNS via DNSCrypt.    
These were once public DNS servers, but we're phasing out public DNS because there's too many vulnerabilities inherent in the DNS protocol that make us more susceptible to distributed/reflected denial of service attacks, and other forms of DNS related abuse.    
If anyone needs to protect their DNS before they connect to the VPN, use DNSCrypt. Those resolvers/relays are still public.

If you're on the VPN and you want to double check that you're using one of our DNS IPs, they're still listed on https://cryptostorm.is/dns.txt    
We also have a DNS leak checking page at https://cryptostorm.is/leaktest that will tell you (as well as IP and WebRTC leaks),    
and a DNS leak test at https://cryptostorm.is/dnsleaktest that doesn't require Javascript

# Technical details

In the rest of this README, two abbreviations are used for the sake of brevity:   

  - CS = <a href="https://cryptostorm.is" target="_blank">cryptostorm</a>
  - ddns = deepDNS

For CS clients, DNS over ddns normally happens as such:
 1. client connects to CS, OpenVPN pushes 10.31.33.8 to the client (or in WireGuard's case, DNS = 10.31.33.8 would be in the client's config), client then uses that for all DNS requests.
 2. client tries to resolve whatever.
 3. DNS request hits the local <a href="https://doc.powerdns.com/md/recursor?" target="_blank">powerdns-recursor</a> process. That pdns-recursor config contains:
 ```config
 forward-zones-recurse=onion=127.0.0.1:5335
 auth-zones=i2p=/var/zones/i2p
 lua-dns-script=/etc/powerdns/onion.lua
 ````
 That translates to:
 1. send DNS requests for whatever.onion to the DNS server at 127.0.0.1:5335
 2. use the zone file /var/zones/i2p for DNS requests for whatever.i2p
    
The onion.lua is a simple LUA script that PowerDNS uses to prevent pointless AAAA/MX record lookups on .onion's, since most operating systems will try to do an A (IPv4), AAAA (IPv6), MX (mail record) lookup whenever resolving anything:
```lua
function preresolve(dq)
  local domain = dq.qname:toString()
  local qtype = dq.qtype
  if domain:match("%.onion%.?$") then
    if qtype == pdns.A then
      return false
    else
      return true
    end
  end
  return false
end
```
That script basically means "If the client is resolving a .onion, only respond to A record lookups". Without it, you would get two NXDOMAIN responses whenever the OS tries to do the usual IPv6 (AAAA) and mail (MX) record lookups:
```console
[df@abox ~]$ host test.onion 10.31.33.8
Using domain server:
Name: 10.31.33.8
Address: 10.31.33.8#53
Aliases:

test.onion has address 10.99.192.234
Host test.onion not found: 3(NXDOMAIN)
Host test.onion not found: 3(NXDOMAIN)
```
 
The 127.0.0.1:5335 in forward-zones-recurse=onion=127.0.0.1:5335 is tor, or more specifically, a tor instance with "DNSPort 127.0.0.1:5335" in it's torrc.
the /var/zones/i2p zone file contains the single line: ```*.i2p. IN A 10.98.0.1```
so any request for an .i2p domain will resolve to 10.98.0.1, which is handled by iptables:
```bash
iptables -t nat -A OUTPUT     -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:4444
iptables -t nat -A PREROUTING -d 10.98.0.1 -p tcp --dport 80 -j DNAT --to-destination ddns-ip:4444
```
The server on port 4444 is an http proxy that i2pd daemon has built-in, restricted to the VPN/local IPs specific to that particular server.

Obviously, you would need to be connected to <a href="https://cryptostorm.is/" target="_blank">cryptostorm</a> in order to use the transparent .i2p/.onion feature described above.

As previously stated, the powerdns-recursor servers mentioned above are not longer internet accessible. For anyone who wants more security than regular DNS offers (which is basically none), our DNSCrypt resolvers/relays are still publicly accessible.    
Server-side, we run <a href="https://github.com/jedisct1/encrypted-dns-server" target="_blank">https://github.com/jedisct1/encrypted-dns-server</a> on TCP port 443 of every DeepDNS IP, and we also supported Anonymized DNSCrypt relays. See <a href="https://cryptostorm.is/blog/anondns" target="_blank">https://cryptostorm.is/blog/anondns</a> for more info on that.
