# Flowspec (RFC5575)

GoBGP supports [RFC5575](https://tools.ietf.org/html/rfc5575),
[draft-ietf-idr-flowspec-redirect-rt-bis-05](http://tools.ietf.org/html/draft-ietf-idr-flowspec-redirect-rt-bis-05)
and [draft-ietf-idr-flow-spec-v6-06](https://tools.ietf.org/html/draft-ietf-idr-flow-spec-v6-06).

## Prerequisites

Assume you finished [Getting Started](https://github.com/osrg/gobgp/blob/master/docs/sources/getting-started.md).

## Contents
- [Configuration](#section0)
- [Add Flowspec routes through CLI](#section1)

## <a name="section0"> Configuration

To advertise flowspec routes, enumerate `ipv4-flowspec` to neighbor's
afi-safis like below.

```toml
[global.config]
  as = 64512
  router-id = "192.168.255.1"

[[neighbors]]
[neighbors.config]
  neighbor-address = "10.0.255.1"
  peer-as = 64512
[[neighbors.afi-safis]]
  afi-safi-name = "ipv4-flowspec"
```

## <a name="section1"> Add Flowspec routes through CLI

CLI syntax to add flowspec is

```shell
% global rib add match <MATCH_EXPR> then <THEN_EXPR> -a [ipv4-flowspec|ipv6-flowspec]
    <MATCH_EXPR> : { destination <PREFIX> | source <PREFIX> |
                     protocol <PROTO>... | fragment <FRAGMENT_TYPE> | tcp-flags <TCPFLAG>... |
                     { port | destination-port | source-port | icmp-type | icmp-code | packet-length | dscp } <ITEM>... }...
        <PROTO> : ipip, sctp, unknown, igmp, tcp, egp, rsvp, pim, icmp, igp, udp, gre, ospf
        <FRAGMENT_TYPE> : not-a-fragment, is-a-fragment, first-fragment, last-fragment
        <TCPFLAG> : push, ack, urgent, fin, syn, rst
        <ITEM> : &?{<|>|=}<value>
    <THEN_EXPR> : { accept | discard | rate-limit <value> | redirect <RT> | mark <value> | action { sample | terminal | sample-terminal } | rt <RT>... }...
        <RT> : xxx:yyy, xx.xx.xx.xx:yyy, xxx.xxx:yyy
```

### Examples

```shell
# add a flowspec rule which redirect flows with dst 10.0.0.0/24 and src 20.0.0.0/24 to VRF with RT 10:10
% gobgp global rib -a ipv4-flowspec add match destination 10.0.0.0/24 source 20.0.0.0/24 then redirect 10:10

# show flowspec table
% gobgp global rib -a ipv4-flowspec
   Network                                       Next Hop             AS_PATH              Age        Attrs
*> [destination:10.0.0.0/24][source:20.0.0.0/24] 0.0.0.0                                   00:00:04   [{Origin: i} {Extcomms: [redirect: 10:10]}]

# add another flowspec rule which discard flows whose ip protocol is tcp and destination port is 80 or greater than or equal to 8080 and lesser than or equal to 8888
% gobgp global rib -a ipv4-flowspec add match protocol tcp destination-port '=80' '>=8080&<=8888' then discard

% gobgp global rib -a ipv4-flowspec
   Network                                              Next Hop             AS_PATH              Age        Attrs
*> [destination:10.0.0.0/24][source:20.0.0.0/24]        0.0.0.0                                   00:03:19   [{Origin: i} {Extcomms: [redirect: 10:10]}]
*> [protocol: tcp][destination-port: =80 >=8080&<=8888] 0.0.0.0                                   00:00:03   [{Origin: i} {Extcomms: [discard]}]

# delete a flowspec rule
% gobgp global rib -a ipv4-flowspec del match destination 10.0.0.0/24 source 20.0.0.0/24 then redirect 10:10

% gobgp global rib -a ipv4-flowspec
   Network                                              Next Hop             AS_PATH              Age        Attrs
*> [protocol: tcp][destination-port: =80 >=8080&<=8888] 0.0.0.0                                   00:00:03   [{Origin: i} {Extcomms: [discard]}]
```
