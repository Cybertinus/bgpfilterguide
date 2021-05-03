---
layout: page
title: Bogon Prefixes
permalink: /guides/bogon_prefixes/
---

* TOC
{:toc}

# Filter Bogon prefixes

## Purpose

These prefixes are not globally unique prefixes. IETF didn't intend for
these to be routed on the public Internet.

# Configuration Examples IPv4

## BIRD
```
define BOGON_PREFIXES = [
  0.0.0.0/8+,         # RFC 1122 'this' network
  10.0.0.0/8+,        # RFC 1918 private space
  100.64.0.0/10+,     # RFC 6598 Carrier grade nat space
  127.0.0.0/8+,       # RFC 1122 localhost
  169.254.0.0/16+,    # RFC 3927 link local
  172.16.0.0/12+,     # RFC 1918 private space
  192.0.2.0/24+,      # RFC 5737 TEST-NET-1
  192.88.99.0/24+,    # RFC 7526 6to4 anycast relay
  192.168.0.0/16+,    # RFC 1918 private space
  198.18.0.0/15+,     # RFC 2544 benchmarking
  198.51.100.0/24+,   # RFC 5737 TEST-NET-2
  203.0.113.0/24+,    # RFC 5737 TEST-NET-3
  224.0.0.0/4+,       # multicast
  240.0.0.0/4+ ];     # reserved

function reject_bogon_prefixes()
prefix set bogon_prefixes;
{
  bogon_prefixes = BOGON_PREFIXES;

  if (net ~ bogon_prefixes) then {
    print "Reject: Bogon prefix: ", net, " ", bgp_path;
    reject;
  }
}

...

filter transit_in {
  reject_invalids();
  reject_bogon_asns();
  reject_bogon_prefixes();
  reject_long_aspaths();
  reject_small_prefixes();
  reject_default_route();

...

  honor_graceful_shutdown();
  accept;
}

filter ixp_in {
  reject_invalids();
  reject_bogon_asns();
  reject_bogon_prefixes();
  reject_long_aspaths();
  reject_transit_paths();
  reject_small_prefixes();
  reject_default_route();

...

  honor_graceful_shutdown();
  accept;
}

```
## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L97-L109)

```
deny from any prefix 0.0.0.0/8 prefixlen >= 8           # 'this' network [RFC1122]
deny from any prefix 10.0.0.0/8 prefixlen >= 8          # private space [RFC1918]
deny from any prefix 100.64.0.0/10 prefixlen >= 10      # CGN Shared [RFC6598]
deny from any prefix 127.0.0.0/8 prefixlen >= 8         # localhost [RFC1122]
deny from any prefix 169.254.0.0/16 prefixlen >= 16     # link local [RFC3927]
deny from any prefix 172.16.0.0/12 prefixlen >= 12      # private space [RFC1918]
deny from any prefix 192.0.2.0/24 prefixlen >= 24       # TEST-NET-1 [RFC5737]
deny from any prefix 192.88.99.0/24 prefixlen >= 24     # 6to4 anycast relay [RFC7526]
deny from any prefix 192.168.0.0/16 prefixlen >= 16     # private space [RFC1918]
deny from any prefix 198.18.0.0/15 prefixlen >= 15      # benchmarking [RFC2544]
deny from any prefix 198.51.100.0/24 prefixlen >= 24    # TEST-NET-2 [RFC5737]
deny from any prefix 203.0.113.0/24 prefixlen >= 24     # TEST-NET-3 [RFC5737]
deny from any prefix 224.0.0.0/4 prefixlen >= 4         # multicast
deny from any prefix 240.0.0.0/4 prefixlen >= 4         # reserved for future use
```

## FRR (vtysh)
```
ip prefix-list BOGONS_v4 deny 0.0.0.0/8 le 32
ip prefix-list BOGONS_v4 deny 10.0.0.0/8 le 32
ip prefix-list BOGONS_v4 deny 100.64.0.0/10 le 32
ip prefix-list BOGONS_v4 deny 127.0.0.0/8 le 32
ip prefix-list BOGONS_v4 deny 169.254.0.0/16 le 32
ip prefix-list BOGONS_v4 deny 172.16.0.0/12 le 32
ip prefix-list BOGONS_v4 deny 192.0.2.0/24 le 32
ip prefix-list BOGONS_v4 deny 192.88.99.0/24 le 32
ip prefix-list BOGONS_v4 deny 192.168.0.0/16 le 32
ip prefix-list BOGONS_v4 deny 198.18.0.0/15 le 32
ip prefix-list BOGONS_v4 deny 198.51.100.0/24 le 32
ip prefix-list BOGONS_v4 deny 203.0.113.0/24 le 32
ip prefix-list BOGONS_v4 deny 224.0.0.0/4 le 32
ip prefix-list BOGONS_v4 deny 240.0.0.0/4 le 32
```

## Mikrotik
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=0.0.0.0/8 prefix-length=8-32 protocol=bgp action=discard comment="RFC 1122 'this' network"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=10.0.0.0/8 prefix-length=8-32 protocol=bgp action=discard comment="RFC 1918 private space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=100.64.0.0/10 prefix-length=10-32 protocol=bgp action=discard comment="RFC 6598 Carrier grade nat space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=127.0.0.0/8 prefix-length=8-32 protocol=bgp action=discard comment="RFC 1122 localhost"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=169.254.0.0/16 prefix-length=16-32 protocol=bgp action=discard comment="RFC 3927 link local"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=172.16.0.0/12 prefix-length=12-32 protocol=bgp action=discard comment="RFC 1918 private space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=192.0.2.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 5737 TEST-NET-1"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=192.88.99.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 7526 6to4 anycast relay"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=192.168.0.0/16 prefix-length=16-32 protocol=bgp action=discard comment="RFC 1918 private space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=198.18.0.0/15 prefix-length=15-32 protocol=bgp action=discard comment="RFC 2544 benchmarking"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=198.51.100.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 5737 TEST-NET-2"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=203.0.113.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 5737 TEST-NET-3"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=224.0.0.0/4 prefix-length=4-32 protocol=bgp action=discard comment="multicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=240.0.0.0/4 prefix-length=4-32 protocol=bgp action=discard comment="multicast"
```

## Junos
```
policy-statement reject-bogon-prefixes {
    term reject-bogon-prefixes-v4 {
        from {
            route-filter 0.0.0.0/8 orlonger;
            route-filter 10.0.0.0/8 orlonger;
            route-filter 100.64.0.0/10 orlonger;
            route-filter 127.0.0.0/8 orlonger;
            route-filter 169.254.0.0/16 orlonger;
            route-filter 172.16.0.0/12 orlonger;
            route-filter 192.0.2.0/24 orlonger;
            route-filter 192.88.99.0/24 orlonger;
            route-filter 192.168.0.0/16 orlonger;
            route-filter 198.18.0.0/15 orlonger;
            route-filter 198.51.100.0/24 orlonger;
            route-filter 203.0.113.0/24 orlonger;
            route-filter 224.0.0.0/4 orlonger;
            route-filter 240.0.0.0/4 orlonger;
        }
        then reject;
    }
    term reject-bogon-prefixes-v6 {
        from {
            route-filter ::/8 orlonger;
            route-filter 100::/64 orlonger;
            route-filter 2001:2::/48 orlonger;
            route-filter 2001:10::/28 orlonger;
            route-filter 2001:db8::/32 orlonger;
            route-filter 2002::/16 orlonger;
            route-filter 3ffe::/16 orlonger;
            route-filter fc00::/7 orlonger;
            route-filter fe80::/10 orlonger;
            route-filter fec0::/10 orlonger;
            route-filter ff00::/8 orlonger;
        }
        then reject;
    }
}
```
## IOS-XR
```
prefix-set BOGONS_V4
  0.0.0.0/8 le 32,
  10.0.0.0/8 le 32,
  100.64.0.0/10 le 32,
  127.0.0.0/8 le 32,
  169.254.0.0/16 le 32,
  172.16.0.0/12 le 32,
  192.0.2.0/24 le 32,
  192.88.99.0/24 le 32,
  192.168.0.0/16 le 32,
  198.18.0.0/15 le 32,
  198.51.100.0/24 le 32,
  203.0.113.0/24 le 32,
  224.0.0.0/4 le 32,
  240.0.0.0/4 le 32
end-set
!
route-policy BGP_FILTER_IN
  if destination in BOGONS_V4 then
    drop
  endif
end-policy
```
## Nokia SR OS
```
#
# Classic CLI
#
#--------------------------------------------------
echo "Policy Configuration"
#--------------------------------------------------
        policy-options
            begin
            prefix-list "BOGONS_V4"
                prefix 0.0.0.0/8 exact
                prefix 10.0.0.0/8 exact
                prefix 100.64.0.0/10 exact
                prefix 127.0.0.0/8 exact
                prefix 169.254.0.0/16 exact
                prefix 172.16.0.0/12 exact
                prefix 192.0.2.0/24 exact
                prefix 192.88.99.0/24 exact
                prefix 192.168.0.0/16 exact
                prefix 198.18.0.0/15 exact
                prefix 198.51.100.0/24 exact
                prefix 203.0.113.0/24 exact
                prefix 224.0.0.0/4 exact
                prefix 240.0.0.0/4 exact
            exit
            policy-statement "BGP_FILTER_IN"
                entry 10
                    from
                        prefix-list "BOGONS_V4"
                    exit
                    action drop
                    exit
                exit
            exit
            commit
        exit
----------------------------------------------

#
# Paste-friendly Classic CLI blob
#
/configure router policy-options begin
/configure router policy-options prefix-list "BOGONS_V4" prefix 0.0.0.0/8 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 10.0.0.0/8 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 100.64.0.0/10 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 127.0.0.0/8 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 169.254.0.0/16 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 172.16.0.0/12 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 192.0.2.0/24 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 192.88.99.0/24 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 192.168.0.0/16 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 198.18.0.0/15 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 198.51.100.0/24 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 203.0.113.0/24 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 224.0.0.0/4 exact
/configure router policy-options prefix-list "BOGONS_V4" prefix 240.0.0.0/4 exact
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 10 from prefix-list "BOGONS_V4"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 10 action drop
/configure router policy-options commit

#
# Model-Driven CLI (MD-CLI)
#
[gl:configure policy-options]
    prefix-list "BOGONS_V4" {
        prefix 0.0.0.0/8 type exact {
        }
        prefix 10.0.0.0/8 type exact {
        }
        prefix 100.64.0.0/10 type exact {
        }
        prefix 127.0.0.0/8 type exact {
        }
        prefix 169.254.0.0/16 type exact {
        }
        prefix 172.16.0.0/12 type exact {
        }
        prefix 192.0.2.0/24 type exact {
        }
        prefix 192.88.99.0/24 type exact {
        }
        prefix 192.168.0.0/16 type exact {
        }
        prefix 198.18.0.0/15 type exact {
        }
        prefix 198.51.100.0/24 type exact {
        }
        prefix 203.0.113.0/24 type exact {
        }
        prefix 224.0.0.0/4 type exact {
        }
        prefix 240.0.0.0/4 type exact {
        }
    }
    policy-statement "BGP_FILTER_IN" {
    entry 10 {
        from {
            prefix-list ["BOGONS_V4"]
        }
        action {
            action-type reject
        }
    }

#
# Paste-friendly MD-CLI blob
#
/configure policy-options prefix-list "BOGONS_V4" { }
/configure policy-options prefix-list "BOGONS_V4" { prefix 0.0.0.0/8 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 10.0.0.0/8 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 100.64.0.0/10 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 127.0.0.0/8 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 169.254.0.0/16 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 172.16.0.0/12 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 192.0.2.0/24 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 192.88.99.0/24 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 192.168.0.0/16 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 198.18.0.0/15 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 198.51.100.0/24 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 203.0.113.0/24 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 224.0.0.0/4 type exact }
/configure policy-options prefix-list "BOGONS_V4" { prefix 240.0.0.0/4 type exact }
/configure policy-options policy-statement "BGP_FILTER_IN" { }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 10 }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 10 from prefix-list ["BOGONS_V4"]
/configure policy-options policy-statement "BGP_FILTER_IN" entry 10 action action-type reject
```
## Arista
```
ip prefix-list BOGONS_v4
   seq 1 permit 0.0.0.0/8 le 32
   seq 2 permit 10.0.0.0/8 le 32
   seq 3 permit 100.64.0.0/10 le 32
   seq 4 permit 127.0.0.0/8 le 32
   seq 5 permit 169.254.0.0/16 le 32
   seq 6 permit 172.16.0.0/12 le 32
   seq 7 permit 192.0.2.0/24 le 32
   seq 8 permit 192.88.99.0/24 le 32
   seq 9 permit 192.168.0.0/16 le 32
   seq 10 permit 198.18.0.0/15 le 32
   seq 11 permit 198.51.100.0/24 le 32
   seq 12 permit 203.0.113.0/24 le 32
   seq 13 permit 224.0.0.0/4 le 32
   seq 14 permit 240.0.0.0/4 le 32
!
route-map Import-Peer deny 20
   match ip address prefix-list BOGONS_v4
!
```
# Configuration Examples IPv6

## BIRD
```
define BOGON_PREFIXES = [ ::/8+,                         # RFC 4291 IPv4-compatible, loopback, et al
                          0100::/64+,                    # RFC 6666 Discard-Only
                          2001:2::/48+,                  # RFC 5180 BMWG
                          2001:10::/28+,                 # RFC 4843 ORCHID
                          2001:db8::/32+,                # RFC 3849 documentation
                          2002::/16+,                    # RFC 7526 6to4 anycast relay
                          3ffe::/16+,                    # RFC 3701 old 6bone
                          fc00::/7+,                     # RFC 4193 unique local unicast
                          fe80::/10+,                    # RFC 4291 link local unicast
                          fec0::/10+,                    # RFC 3879 old site local unicast
                          ff00::/8+                      # RFC 4291 multicast
 ];

function reject_bogon_prefixes()
prefix set bogon_prefixes;
{
    bogon_prefixes = BOGON_PREFIXES;
    if (net ~ bogon_prefixes) then {
        print "Reject: Bogon prefix: ", net, " ", bgp_path;
        reject;
    }
}

...

filter transit_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_small_prefixes();
        reject_default_route();

        honor_graceful_shutdown();
        accept;
}

filter ixp_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_transit_paths();
        reject_small_prefixes();
        reject_default_route();

        honor_graceful_shutdown();
        accept;
}

```

## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L111-L121)

```
deny from any prefix ::/8 prefixlen >= 8
deny from any prefix 0100::/64 prefixlen >= 64          # Discard-Only [RFC6666]
deny from any prefix 2001:2::/48 prefixlen >= 48        # BMWG [RFC5180]
deny from any prefix 2001:10::/28 prefixlen >= 28       # ORCHID [RFC4843]
deny from any prefix 2001:db8::/32 prefixlen >= 32      # docu range [RFC3849]
deny from any prefix 2002::/16 prefixlen >= 16          # 6to4 anycast relay [RFC7526]
deny from any prefix 3ffe::/16 prefixlen >= 16          # old 6bone
deny from any prefix fc00::/7 prefixlen >= 7            # unique local unicast
deny from any prefix fe80::/10 prefixlen >= 10          # link local unicast
deny from any prefix fec0::/10 prefixlen >= 10          # old site local unicast
deny from any prefix ff00::/8 prefixlen >= 8            # multicast
```

# FRR (vtysh)
```
ipv6 prefix-list BOGONS_v6 deny ::/8 le 128
ipv6 prefix-list BOGONS_v6 deny 100::/64 le 128
ipv6 prefix-list BOGONS_v6 deny 2001:2::/48 le 128
ipv6 prefix-list BOGONS_v6 deny 2001:10::/28 le 128
ipv6 prefix-list BOGONS_v6 deny 2001:db8::/32 le 128
ipv6 prefix-list BOGONS_v6 deny 2002::/16 le 128
ipv6 prefix-list BOGONS_v6 deny 3ffe::/16 le 128
ipv6 prefix-list BOGONS_v6 deny fc00::/7 le 128
ipv6 prefix-list BOGONS_v6 deny fe80::/10 le 128
ipv6 prefix-list BOGONS_v6 deny fec0::/10 le 128
ipv6 prefix-list BOGONS_v6 deny ff00::/8 le 128
```

## Mikrotik
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=::/8 prefix-length=8-128 protocol=bgp action=discard comment="RFC 4291 IPv4-compatible, loopback, et al"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=0100::/64 prefix-length=64-128 protocol=bgp action=discard comment="RFC 6666 Discard-Only"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2001:2::/48 prefix-length=48-128 protocol=bgp action=discard comment="RFC 5180 BMWG"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2001:10::/28 prefix-length=28-128 protocol=bgp action=discard comment="RFC 4843 ORCHID"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2001:db8::/32 prefix-length=32-128 protocol=bgp action=discard comment="RFC 3849 documentation"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2002::/16 prefix-length=16-128 protocol=bgp action=discard comment="RFC 7526 6to4 anycast relay"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=3ffe::/16 prefix-length=16-128 protocol=bgp action=discard comment="RFC 3701 old 6bone"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=fc00::/7 prefix-length=7-128 protocol=bgp action=discard comment="RFC 4193 unique local unicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=fe80::/10 prefix-length=10-128 protocol=bgp action=discard comment="RFC 4291 link local unicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=fec0::/10 prefix-length=10-128 protocol=bgp action=discard comment="RFC 3879 old site local unicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=ff00::/8 prefix-length=8-128 protocol=bgp action=discard comment="RFC 4291 multicast"
```

## Juniper and Cisco

Gert Doering's [ipv6-filters](https://www.space.net/~gert/RIPE/ipv6-filters.html)

## YAML from Coloclue

Coloclue's network management system [kees](https://github.com/coloclue/kees) considers these the IPv6 Bogons: [yaml file](https://github.com/coloclue/kees/blob/master/vars_example/generic.yml#L188-L274)

## Nokia SR OS
```
#
# Classic CLI
#
#--------------------------------------------------
echo "Policy Configuration"
#--------------------------------------------------
        policy-options
            begin
            prefix-list "BOGONS_V6"
                prefix ::/8 longer
                prefix 100::/64 longer
                prefix 2001:2::/48 longer
                prefix 2001:10::/28 longer
                prefix 2001:db8::/32 longer
                prefix 2002::/16 longer
                prefix 3ffe::/16 longer
                prefix fc00::/7 longer
                prefix fe80::/10 longer
                prefix fec0::/10 longer
                prefix ff00::/8 longer
            exit
            policy-statement "BGP_FILTER_IN"
                entry 20
                    from
                        prefix-list "BOGONS_V6"
                    exit
                    action drop
                    exit
                exit
            exit
            commit
        exit

#
# Paste-friendly Classic CLI blob
#
/configure router policy-options begin
/configure router policy-options prefix-list "BOGONS_V6" prefix ::/8 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 100::/64 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2001:2::/48 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2001:10::/28 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2001:db8::/32 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2002::/16 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 3ffe::/16 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix fc00::/7 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix fe80::/10 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix fec0::/10 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix ff00::/8 longer
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 20 from prefix-list "BOGONS_V6"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 20 action drop
/configure router policy-options commit

#
# Model-driven CLI (MD-CLI)
#
[gl:configure policy-options]
    prefix-list "BOGONS_V6" {
        prefix ::/8 type longer {
        }
        prefix 100::/64 type longer {
        }
        prefix 2001:2::/48 type longer {
        }
        prefix 2001:10::/28 type longer {
        }
        prefix 2001:db8::/32 type longer {
        }
        prefix 2002::/16 type longer {
        }
        prefix 3ffe::/16 type longer {
        }
        prefix fc00::/7 type longer {
        }
        prefix fe80::/10 type longer {
        }
        prefix fec0::/10 type longer {
        }
        prefix ff00::/8 type longer {
        }
    }
    policy-statement "BGP_FILTER_IN" {
        entry 20 {
            from {
                prefix-list ["BOGONS_V6"]
            }
            action {
                action-type reject
            }
        }
    }

#
# Paste-friendly MD-CLI blob
#
/configure policy-options prefix-list "BOGONS_V6" { }
/configure policy-options prefix-list "BOGONS_V6" { prefix ::/8 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 100::/64 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2001:2::/48 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2001:10::/28 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2001:db8::/32 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2002::/16 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 3ffe::/16 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix fc00::/7 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix fe80::/10 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix fec0::/10 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix ff00::/8 type longer }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 20 }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 20 from prefix-list ["BOGONS_V6"]
/configure policy-options policy-statement "BGP_FILTER_IN" entry 20 action action-type reject
```
