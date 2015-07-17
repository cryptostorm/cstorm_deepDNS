# deepDNS
This is cryptostorm's <a href="http://dnschain.org">DNSchain</a>'d, <a href="https://dnscurve.org">DNScurve</a>'d, <a href="https://cryptostorm.org/viewtopic.php?f=47&t=8553">native .onion + .bit + .i2p</a> DNS resolution & integrity validation framework is a groundbreaking, integrated, systems-level solution to the vast security and reliability problems of the conventionally-provided domain name system architecture of the public internet today. 

Here we have collected the totality of public-facing source materials in one location - available directly via <a href="http://deepdns.dk">deepdns.dk</a>. We will continue to expand on the resources published here, as deepDNS continues to evolve from a purely internal cryptostorm function to a publicly-available resource for broad community usage, extension, and enhancement.

Additional commentary and explanatory discussion threads may be found in the <a href="http://deepdns.net">deepdns.net</a> subforum of our <a href="https://cryptostorm.org">cryptostorm.org member forum</a>, collectively. Bug/error reports on specific items in this repository are best filed via issue tracking here in the repository, however broader crtitique of discussion is generally best served via threads in the forum. 


# public deepDNS resolve access
we've mirrored over the current listing of all deepDNS resolvers available for public use. THis publicly-available function has grown from an incidental by-product of using deepDNS-enabled nameserver queries in cryptostorm session spin-up, into a resource increasingly used by the community outside of cryptostorm-specific scenarios.

At some point, we were faced with the decision either to lock the resolver framework down to on-cstorm access exclusively, or to continue supporting broader community availablity of deepDNS. As a team, the decision to choose the latter pathway was unanimous.

These are collected together under the HAF-style balancers via:

<b>public.deepdns.net</b> | <b>public.deepdns.dk</b>

We used to call these <i>cryptostorm-shared.deepdns.net/dk</i> but that's so cumbersome since everyone always referred to them as "the public deepDNS resolvers" anyway; to avoid dual maintenance overhead in the future, we've rm'd all the cryptostorm-shared A Records entirely, so folks who used those will need to switch to public.deepdns.net/dk once and from then on it's all smooth sailing :-)

Also, we never actually got around to announcing the public availability of these deepDNS resolvers... which is sort of sad, because they're quite handy. We should really do that...

# ...don't be shy :-)
This repository - and the deepDNS project itself - is very much of a work in progress. If the spirit moves you, please do share your ideas, suggestions, code, bug reports, or whatever else may help deepDNS continue to improve and broaden its successes and capabilities. If you'd like to participate in full edit capability, drop a note & we'll be happy to add you to the repository's admin team.
