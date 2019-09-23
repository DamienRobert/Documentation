vim: ft=markdownlight fdm=expr

Documentation
=============

* Books: https://www.quora.com/What-is-the-best-way-resource-to-learn-Linux-networking-concepts-and-practices-like-open-source-training-manuals-and-hand-outs
- Guide to IP Layer Network Administration with Linux: http://linux-ip.net/html/index.html [2007]
- Linux Advanced Routing & Traffic Control HOWTO: http://tldp.org/HOWTO/Adv-Routing-HOWTO/index.html [2002-07-22]
  Newer version: https://www.lartc.org/ [2012-05]
- IP Command Reference (by the author of iproute, Alexey Kuznetsov's): http://linux-ip.net/gl/ip-cref/ip-cref.html [1999]
- http://www.policyrouting.org/iproute2.doc.html
  From Policy Routing With Linux - Online Edition, by Matthew G. Marsh [2001]
  http://www.policyrouting.org/PolicyRoutingBook/ONLINE/TOC.html
  [expand the above ip command reference]
- http://freecomputerbooks.com/Linux-Network-Administrators-Guide-2nd-Edition.html [2000]
  Pdf: http://www.tldp.org/LDP/nag2/nag2.pdf
- https://www.oreilly.com/library/view/understanding-linux-network/0596002556/

* Cheatsheets
- Cheatsheet: https://www.computerhope.com/unix/ip.htm
              https://packetpushers.net/linux-ip-command-ostensive-definition/
- https://www.tecmint.com/linux-networking-commands/

* Network:
- Arp (Address resolution protocol): Layer 3 MAC <-> IP
- FDB (Forward database): Layer 2 MAC <-> corresponding port of the bridge
- Realm: https://en.wikipedia.org/wiki/Realm-Specific_IP
  (Note this is different from 'ip route realm' which simply are a way to
  group certain routes)

* Tools:
- tc show / manipulate traffic control settings (qos)
- ss: replacement of netstat

* TOS
- https://en.wikipedia.org/wiki/Type_of_service
  https://en.wikipedia.org/wiki/Explicit_Congestion_Notification
- https://www.slashroot.in/understanding-differentiated-services-tos-field-internet-protocol-header
=> 8 bits, bits 0-5 = DSCP (differentiated service codepoint)
                6-7 = ECN (explicit congestion notification)
   DSCP = CS (class selector) 3 bits
   CS: CS0 = 000000 [default]
       CS1 = 001000
       ...
       CS7 = 111000
   Note: CS6 and CS7 is reserved for network protocol and control related stuff. This means that CS5 class value is the highest class value.

   CS1 to CS4 can have a drop precedence of low (01), medium (10) and high (11) (high drop precedence will get dropped more than low drop precedence)

   Setting drop precedence means the dscp if of type AF (assured forwarding)
   => AF11 = 001010 = CS1 + low drop precedence
      AF12 = 001100 = CS1 + medium drop precedence
      AF13 = 001110 = CS1 + high drop precedence
      ...
      AF43 = 100110 = CS4 + high drop precedence
   
   CS5 does not have drop precedence. But we can add '11' to make it of
   class EF (expediated forwarding):
   => EF = 101110

ip
==
http://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/
https://tty1.net/blog/2010/ifconfig-ip-comparison_en.html

ip l / ip link
ip a / ip addr
ip r / ip route

Options:
-s, -stats, -statistics
-d, -details
-a, -all
-br, -brief

## ip link
* Change
$ ip link set eth0 up/down
Rename interface: $ ip link set old_name name new_name
Specify a group: $ ip link set eth1 group 1 [we cannot set a group directly
when adding a device]

* Add
$ sudo ip link add v-eth1 type veth peer name v-peer1
$ sudo ip link add foobar type dummy

* Show
$ ip link show wlan0
    3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
        link/ether 80:19:34:0c:f6:48 brd ff:ff:ff:ff:ff:ff
$ ip -br link show eno1 #-br = -brief
eno1             DOWN           08:60:6e:e6:08:c7 <NO-CARRIER,BROADCAST,MULTICAST,UP> 

* Options
        group GROUP
            specify the group the device belongs to.  The available groups
            are listed in file /etc/iproute2/group.

## ip addr

ip address show [dev] eth0: Shows the addresses assigned to network interface eth0
ip addr add 2001:0db8:85a3::0370:7334/64 dev eth1: Adds an IPv6 address to network interface eth1
ip addr add 192.168.1.2/24 dev eth0
 # Nowadays the broadcast is set up correctly from the netmask, no need to specify it directly
 # ip addr add 192.168.1.2/24 broadcast 192.168.1.255 dev eth0
ip a add 192.168.1.200/255.255.255.0 dev eth0
ip addr del 192.168.0.77/24 dev eth0
ip addr flush dev eth4: Removes all addresses from device eth4

* Add alias interface: ip addr add 10.0.0.1/8 dev eth0 label eth0:1
https://www.kernel.org/doc/html/latest/networking/alias.html
    IP-aliases are an obsolete way to manage multiple IP-addresses/masks
    per interface. Newer tools such as iproute2 support multiple
    address/prefixes per interface, but aliases are still supported for
    backwards compatibility.

* Show
$ ip a s wlan0 / ip addr show wlan0
    3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
        link/ether 80:19:34:0c:f6:48 brd ff:ff:ff:ff:ff:ff
        inet 172.17.2.186/24 brd 172.17.2.255 scope global dynamic wlan0
           valid_lft 2705sec preferred_lft 2705sec
        inet6 fe80::8219:34ff:fe0c:f648/64 scope link 
           valid_lft forever preferred_lft forever

[ifconfig: ifconfig eth0 "$1" broadcast "$2" netmask "$3"
           route add default gw "$2"
           echo "nameserver $3" > /etc/resolv.conf]

$ ip -br show wlan0
wlp6s0           UP             192.168.0.11/24 fe80::de85:deff:fe74:f5f5/64 


## ip neigh

ip neigh replace the arp utility

* add
ip neigh add 192.168.0.1 lladdr 00:11:22:33:44:55 nud permanent dev eth0
ip link set dev eth0 arp off #switch off arp

* proxy:
Was supposed to be deprecated, cf
http://lkml.iu.edu/hypermail/linux/kernel/0110.2/0523.html
but still there.
ip neighbor add proxy 192.168.100.1 dev eth0
(note they are not shown in ip neigh show by default, we need to use
`ip neigh show proxy`)

* Show:
$ ip neigh
192.168.0.17 dev wlp6s0 lladdr f0:d5:bf:b7:d1:53 REACHABLE
192.168.0.1 dev wlp6s0 lladdr 24:7f:20:ab:5e:44 DELAY

* Ping: arping

### arp proxy:


* via proxy_arp sysctl
- https://wiki.debian.org/BridgeNetworkConnectionsProxyArp
The bridge host will proxy ARP requests from the inside network to the outside, and respond to ARPs from the outside network on behalf of inside hosts. Linux will only do this for hosts that are known via the routing table, so a /32 host route must be created pointing to the inside host (one for each inside host). The route is also required for IP forwarding to work, i.e. when IP traffic arrives after the ARP process has completed. 
  bridge# echo 1 > /proc/sys/net/ipv4/conf/all/proxy_arp
  bridge# echo 1 > /proc/sys/net/ipv4/ip_forward
  bridge# ip ro add 10.42.0.11/32 dev eth0

- https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt
proxy_arp - BOOLEAN
	Do proxy arp.
	proxy_arp for the interface will be enabled if at least one of
	conf/{all,interface}/proxy_arp is set to TRUE,
	it will be disabled otherwise

=> More details here: https://blog.noc.grnet.gr/2018/09/10/arp-proxy-going-rogue-part-2-tracing-the-kernel/
For every ARP request linux will lookup the routing tables for the destination IP. Most probably a route will be found, even if it’s the default gateway. Then, in case proxy_arp is enabled and if and only if the host would route traffic to that destination IP through a device that is different than one the ARP request originated from, then a ARP reply will be produced.

=> https://serverfault.com/questions/212074/is-it-possible-to-use-proxy-arp-back-to-the-same-interface
But enabling proxy_arp_pvlan allow to proxy even if the destination ip is
on the same interface.
proxy_arp_pvlan - BOOLEAN
    Private VLAN proxy arp.
    Basically allow proxy arp replies back to the same interface
    (from which the ARP request/solicitation was received).

* by hand http://linux-ip.net/html/adv-proxy-arp.html
    - arp -s $IPADDR -i $INTERFACE -D $INTERFACE pub
    (-D use this device ethernet address, -i: associate the arp to this
    interface [if not specified the kernel uses the routing table to
    guess])

    Note that arp's man page says:
    /usr/sbin/arp -i eth0 -Ds 10.0.0.2 eth1 pub
       This will answer ARP requests for 10.0.0.2 on eth0 with the MAC address
       for eth1.
    but in practice this does not seems to work, I get
    10.0.0.2   *   <from_interface>    MP enp0s25
    so it will answer with enp0s25's interface.

    - $ ip neigh add proxy $IPADDR dev $INTERFACE

* parprouted is designed to monitor the ARP table and both proxy ARP requests and install matching /32 host routes. Running parprouted with the inside and outside interfaces handles the ARP and routing completely automatically. Note that the kernel's proxy ARP mechanism (/proc/sys/net/ipv4/conf/all/proxy_arp) is not required. 
  https://linux.die.net/man/8/parprouted

* Other arp settings:
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/networking/ip-sysctl.txt?h=v4.19#n1171
Note that arp_filter by default is 0: The kernel can respond to arp requests with addresses from other interfaces.

## ip route
* Doc
- http://cpham.perso.univ-pau.fr/ENSEIGNEMENT/UERHD/DescriptifCmdIP.pdf
- Plus de détails dans le livre: http://linux-ip.net/html/tools-ip-route.html
- realm: http://linux-ip.net/gl/ip-cref/ip-cref-node172.html#RT-REALMS
- route lookups are stored in fib table: https://vincent.bernat.ch/en/blog/2017-ipv4-route-lookup-linux

* Usage
ip route => ip route show => ip route show table main

* ip route add
ip route add {NETWORK/MASK} via {GATEWAYIP}
ip route add {NETWORK/MASK} dev {DEVICE}
ip route add ... src SRC => when we have several ip, prefer using this SRC when using this route

Exemples:
ip route add default [via] 192.168.1.1
ip route add 192.168.1.0/24 via 192.168.1.254
ip route add 192.168.1.0/24 dev eth0 #route via eth0

Note: ip route add 192... is a shortcut for ip route add unicast 192...
The other route types are unreachable, blackhole, prohibit, local, broadcast, throw, nat[no longer supported], anycast, multicast.
Eg: ip route add prohibit 209.10.26.51 [from 192.168.99.35]

* specify several hops
ip route add default nexthop via 10.10.10.1 weight 1 nexthop dev ens33 weight 10
[sends 10 packets to ens33 for 1 packet to 10.10.10.1]

* Other options:
ip route add TYPE=unicast/unreachable to ...
ip route add to $IPADDR scope global protocol static preference 42 ...
ip route ... encap ...: attach tunnel encapsulation to this route

- TYPE=unicast,unreachable,blackhole,prohibit,local,broadcast,throw,[nat]
- preference number [lower = prefered]
- pref: low/medium/high [ipv6 preference]
- protocol: redirect/kernel/boot [default]/static (=boot but routing daemon preserve these routes)/ra
- encap: mpls, ip, bpf, seg6
- scope: host/link/global
  If this parameter is omitted, ip assumes scope global for all gatewayed
  unicast routes, scope link for direct unicast and broadcast routes and scope
  host for local routes.

  Scope | Description (https://serverfault.com/questions/63014/ip-address-scope-parameter)
  global | valid everywhere
  site | valid only within this site (IPv6)
  link | valid only on this device
  host | valid only inside this host (machine)
- onlink: pretend that the nexthop is directly attached to this link, even if it does not match any interface prefix.
- dsfield TOS
- realm REALMID

* Flush the cache:
$ ip route flush cache

* Simulate a route:
$ ip route get 20.10.3.3

## ip rule

https://openwrt.org/docs/guide-user/network/ip_rules

* ip rule => Multiple routing tables:
https://unix.stackexchange.com/questions/345862/is-it-possible-to-have-multiple-default-gateways-for-outbound-connections

$ ip rule
0:	from all lookup local # handled by the kernel
32766:	from all lookup main
32767:	from all lookup default

$ ip route show table local
...

 # give the name 'special' to the table 7
echo 7 special >> /etc/iproute2/rt_tables

$ sudo ip rule add to 10.10.10.10/32 lookup 123 priority 10
Note: pref is an alias to priority.
If not specified, use the highest lowest priority possible: for instance if the lowest priority is 100, then the new rule has priority 99.

* Exemple: https://www.linuxjournal.com/content/linux-advanced-routing-tutorial
$ echo "121 dmz" >> /etc/iproute2/rt_tables
$ ip route add default via 203.0.113.37 dev vlan-shdsl table dmz
$ ip rule add pri 1000 from 203.0.113.208/28 iif vlan-dmz lookup dmz

* Details
       SELECTOR := [ not ] [ from PREFIX ] [ to PREFIX ] [ tos TOS ] [ fwmark
               FWMARK[/MASK] ] [ iif STRING ] [ oif STRING ] [ pref/priority NUMBER ] [ l3mdev ] [ uidrange NUMBER-NUMBER ] [ ipproto PROTOCOL ] [ sport [ NUMBER | NUMBER-NUMBER ] ] [ dport [ NUMBER | NUMBER- NUMBER ] ] [ tun_id TUN_ID ]
       ACTION := [ table TABLE_ID ] [ protocol PROTO ] [ nat ADDRESS ] [ realms [SRCREALM/]DSTREALM ] [ goto NUMBER ] SUPPRESSOR
       SUPPRESSOR := [ suppress_prefixlength NUMBER ] [ suppress_ifgroup GROUP]

* Types:
       The RPDB may contain rules of the following types:
        -  unicast - the rule prescribes to return the route found in the routing table referenced by the rule.
        -  blackhole - the rule prescribes to silently drop the packet.
        - unreachable - the rule prescribes to generate a 'Network is unreachable' error.
        - prohibit - the rule prescribes to generate 'Communication is administratively prohibited' error.
        [- nat - the rule prescribes to translate the source address of the IP packet into some other value. (deprecated)]

        Exemple: ip rule add unreachable iif eth2 tos 0xc0
                 ip rule add to 10.17.100.1 prohibit

* Action:
goto NUMBER => go to this rule priority number
 Exemple: sudo ip rule add from 10.17.100.2 goto 1
          => Error: Backward goto not supported.

* Options:
              suppress_prefixlength NUMBER
                     reject routing decisions that have a prefix length of
                     NUMBER or less.

              suppress_ifgroup GROUP
                     reject routing decisions that use a device belonging to
                     the interface group GROUP.

* Realm http://linux-ip.net/gl/ip-cref/ip-cref-node172.html#RT-REALMS

 For each packet the kernel calculates a tuple of realms: source realm and destination realm, using the following algorithm:
  1. If the route has a realm, the destination realm of the packet is set to it.
  2. If the rule has a source realm, the source realm of the packet is set to it. If the destination realm was not inherited from the route and the rule has a destination realm, it is also set.
  3. If at least one of the realms is still unknown, the kernel finds the reversed route to the source of the packet.
  4. If the source realm is still unknown, get it from the reversed route.
  5. If one of the realms is still unknown, swap the realms of reversed routes and apply step 2 again.
  After this procedure is completed we know what realm the packet arrived from and the realm where it is going to propagate to. If some of the realms are unknown, they are initialized to zero (or realm unknown). 

  Exemple: ip rule add to 10.17.100.2 realms 2/
  => from all to 10.17.100.2 lookup main realms 2/cosmos
           ip rule add to 10.17.100.2 realms 4/5
  => 32762:	from all to 10.17.100.2 lookup main realms 4/5

  Application as a tc filter: https://www.tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.adv-filter.route.html

  Match any packet that has a route entry
  $ tc filter add dev eth1 parent 1:0 protocol ip prio 100 route classid CLASSID

  Route to realm 10 go to class 1:10
  $ ip route add 192.168.10.0/24 via 192.168.10.1 dev eth1 realm 10
  $ tc filter add dev eth1 parent 1:0 protocol ip prio 100 route to 10 classid 1:10

  Route from realm 2 go to class 1:2
  $ ip route add 192.168.2.0/24 dev eth2 realm 2
  $ tc filter add dev eth1 parent 1:0 protocol ip prio 100 route from 2 classid 1:2

* l3mdev: https://netdevconf.org/1.2/papers/ahern-what-is-l3mdev-paper.pdf
 L3  Master  Device  (l3mdev) is  a  separate  API  that  can  be  leveraged  by other  drivers  that  want  to  influence  FIB  lookups  or  want  to manipulate  packets  at  layer. Used by VRF and ipvlan.

 When a VRF device is created, a rule of type l3mdev and prio 1000 is
 automatically created, so there is no need usually to configure it. A VRF
 associate a virtual device to a specific routing table, and the l3mdev
 rule associate the lookup of each vrf to its corresponding table. Cf VRF
 below for more details.

 IE when doing $ ip link add vrf-blue type vrf table 10
 the l3mdev rule is equivalent to doing
     $ ip ru add oif vrf-blue table 10
     $ ip ru add iif vrf-blue table 10
 for every vrf [with the benefit that it is only one rule, vs 2*N rules for
 N vrf]

## ip tunnel

ip tunnel -> ipip, sit, isatap, vti, gre
  Note: it seems like `ip link` can also set up a ipip,... tunnel.
ip tuntap #ex: ip tuntap add name tap0 mode tap
ip fou, ip gue #for receive port configuration; use ip link for transmit

* Exemples:
sudo ip tunnel add ipiptun mode ipip local 10.3.3.3 remote 10.4.4.4 ttl 64 dev eno1
sudo ip link add ipiptun2 type ipip local 10.6.6.6 remote 10.7.7.7 ttl 64 dev eno1 #curiously the mode is any/ip here
    ipiptun: ip/ip remote 10.4.4.4 local 10.3.3.3 dev eno1 ttl 64
    ipiptun2: any/ip remote 10.7.7.7 local 10.6.6.6 dev eno1 ttl 64

- gre: ip link add name gre1 type gre local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR [seq] key KEY

$  ip link add name tun1 type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225  encap fou encap-sport auto encap-dport 5555

* Fou / Gue
Both sides fou:
  receive  $ ip fou add port 5555 ipproto 4
  transmit $  ip link add name tun1 type ipip \
       remote 192.168.1.1 local 192.168.1.2 ttl 225 \
       encap fou encap-sport auto encap-dport 5555

Both sides gue:
  Server A # ip fou add port 5555 gue
  Server B # ip link add name tun1 type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225 encap gue encap-sport auto encap-dport 5555

## (Dumb) nat
http://linux-ip.net/html/nat-stateless.html

$ ip route add nat 205.254.211.17 via 192.168.100.17
=> the parameter via tells the NAT code to rewrite the packet bound for
205.254.211.17 with the new destination address 192.168.100.17. Note, that
this only handles inbound packets; that is, packets whose destination
address contains 205.254.211.17.
[Note: no longer supported since linux 2.6]

$ ip rule add nat 205.254.211.17 from 192.168.100.17 
=> rewrite any packet from 192.168.100.17 with the specified source address
(205.254.211.17). Any packet originating from 192.168.100.17 which passes
through this router will trigger this rule. In short, this command rewrites
the source address of outbound packets so that they appear to originate
from the NAT IP.
[for a subnet: ip rule add nat 205.254.211.32 from 192.168.100.32/29]

[root@masq-gw]# ip route show table all | grep ^nat                 4
nat 205.254.211.17 via 192.168.100.17  table local  scope host
[root@masq-gw]# ip rule show                                        5
0:      from all lookup local 
32765:  from 192.168.100.17 lookup main map-to 205.254.211.17 
32766:  from all lookup main 
32767:  from all lookup 253

Remark: doing that I get `Warning: route NAT is deprecated`

## Divers:

- realm: http://linux-ip.net/gl/ip-cref/ip-cref-node172.html
  (classify routes together => rtacct)
- tos/dsfield

Route Configuration:
- scope: SCOPE_VAL may be a number or a string from the file /etc/iproute2/rt_scopes.
  [global nowhere host link site]
- protocol RTPROTO: /etc/iproute2/rt_protos
  [redirect kernel boot static gated ra...]
- realm REALMID: /etc/iproute2/rt_realms
  [cosmos]
- route table: /etc/iproute2/rt_tables
  [local main default]
- dsfield TOS: /etc/iproute2/rt_dsfield

Ip link configuration:
- group GROUP: /etc/iproute2/group [default]

iw
==

iw dev wlan0 link #Getting link status.
  Shortcut: iw wlp2s0 link
iw dev wlan0 info #interface info
iw dev wlan0 set power_save on #Enabling power save.

iw dev wlan0 scan #Scanning for available access points.
iw dev wlan0 set type ibss #Setting the operation mode to ad-hoc.
iw dev wlan0 connect your_essid #Connecting to open network.
iw dev wlan0 connect your_essid 2432 #Connecting to open network specifying channel.
iw dev wlan0 connect your_essid key 0:your_key #Connecting to WEP encrypted network

bridge
======

Replace brctl

* fdb (forwarding database entry)
bridge fdb - forwarding database management
       fdb objects contain known Ethernet addresses on a link.
$ bridge fdb (=bridge fdb show)

bridge fdb { add | append | del | replace } LLADDR dev DEV { local |
               static | dynamic } [ self ] [ master ] [ router ] [ use ] [ ex‐
               tern_learn ] [ sticky ] [ dst IPADDR ] [ src_vni SRC_VNI ] [
               vni VNI ] [ port PORT ] [ via DEVICE ]

* link
bridge link => show bridge interfaces created with
 # ip link add name bridge_name type bridge
 # ip link set bridge_name up
 # ip link set eth0 master bridge_name
 
* mdb (multicast group database entry)

* vlan (vlan filter list)

* monitor

wpa_supplicant
==============

wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf
wpa_supplicant -B -i interface -c <(wpa_passphrase essid passphrase)

$ wpa_passphrase essid passphrase
network={
    ssid="essid"
    #psk="passphrase"
    psk=f5d1c49e15e679bebe385c37648d4141bc5c9297796a8a185d7bc5ac62f954e3
}

tc
==

* Articles:
- https://web.archive.org/web/20190427083833/https://www.coverfire.com/articles/queueing-in-the-linux-network-stack/
- codel: https://queue.acm.org/detail.cfm?id=2209336
- cake: https://www.bufferbloat.net/projects/codel/wiki/Cake/
        http://man7.org/linux/man-pages/man8/tc-cake.8.html

* Doc
- https://wiki.archlinux.org/index.php/Advanced_traffic_control
- https://wiki.debian.org/TrafficControl

* Basic usage
- Change a qdisc:
qdisc tbf: token bucket filter
$ tc qdisc add dev eth1 root tbf rate 220kbit latency 50ms burst 1540
$ tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms

qdisc sfq: stochastic fairness queueing
$ tc qdisc add dev eth1 root sfq perturb 10

- Delete a qdisc:
$ tc qdisc del dev eth1 root

- Stats (-s) and details (-d)
$ tc -s -d qdisc show [dev device]

Note: for tc filter, it seems that 'classid' is a synonym for 'flowid'

## Stateless qdisc
- noqueue
- pfifo_fast
- mq https://serverfault.com/questions/474230/linux-traffic-control-qdisc-mq
   classful multiqueue ("mq") dummy scheduler; used by default for multiqueue devices instead of the regular pfifo_fast qdisc (ie with several processors, one can get a mq with a queue by processor, and then each queue is a pfifo_fast)
- fq_codel (cf https://lists.fedoraproject.org/pipermail/devel/2015-March/209508.html)

* https://linux.die.net/man/8/tc-pfifo_fast
3 queues

* https://linux.die.net/man/8/tc-prio
[this one actually has a class by queue]

Use the old TOS meaning to map priority to channel
0   1   2   3   4   5   6   7
+---+---+---+---+---+---+---+---+
|           |               |   |
|PRECEDENCE |      TOS      |MBZ|
|           |               |   |
+---+---+---+---+---+---+---+---+

More infos on tc-prio:
- https://lartc.org/howto/lartc.qdisc.classless.html
[- https://serverfault.com/questions/894054/tc-prio-how-the-packets-are-prioritized]

* http://man7.org/linux/man-pages/man8/tc-codel.8.html

  CoDel (pronounced "coddle") is an adaptive "no-knobs" active queue
  management algorithm (AQM)

       tc qdisc ... codel [ limit PACKETS ] [ target TIME ] [ interval TIME
       ] [ ecn | noecn ] [ ce_threshold TIME ]

* http://man7.org/linux/man-pages/man8/tc-fq_codel.8.html

   FQ_Codel (Fair Queuing Controlled Delay) is queuing discipline that
   combines Fair Queuing with the CoDel AQM scheme. FQ_Codel uses a
   stochastic model to classify incoming packets into different flows and
   is used to provide a fair share of the bandwidth to all the flows using
   the queue. Each such flow is managed by the CoDel queuing discipline.
   Reordering within a flow is avoided since Codel internally uses a FIFO
   queue.

       tc qdisc ... fq_codel [ limit PACKETS ] [ flows NUMBER ] [ target
       TIME ] [ interval TIME ] [ quantum BYTES ] [ ecn | noecn ] [
       ce_threshold TIME ] [ memory_limit BYTES ]

* http://man7.org/linux/man-pages/man8/tc-cake.8.html
       tc qdisc ... cake
    SHAPER PARAMETERS:
       [ bandwidth RATE | unlimited* | autorate-ingress ]
    OVERHEAD COMPENSATION PARAMETERS
       [ overhead N | conservative (= overhead 48 atm) | raw* ]
       [ ptm | atm | noatm* ]
       [ mpu N ]
    ROUND TRIP TIME PARAMETERS
       [ rtt TIME | datacentre | lan | metro | regional | internet* | oceanic | satellite | interplanetary ]
    FLOW ISOLATION PARAMETERS
       [ flowblind | srchost | dsthost | hosts | flows | dual-srchost | dual-dsthost | triple-isolate* ]
       [ nat | nonat* ]
    PRIORITY QUEUE PARAMETERS
       [ besteffort | diffserv8 | diffserv4 | diffserv3* ]
       [ fwmark MASK ]
    OTHER PARAMETERS
       [ wash | nowash* ]
       [ split-gso* | no-split-gso ]
       [ memlimit LIMIT ]

    ???
       [ ack-filter | ack-filter-aggressive | no-ack-filter* ]
       [ ingress | egress* ]

  - Override priority tin: To assign a priority tin, the major number of the priority field needs to match the qdisc handle of the cake instance; if it does, the minor number will be interpreted as the tin index.
  $ tc qdisc replace dev eth0 handle 1: root cake diffserv3
  $ tc filter add dev eth0 parent 1: protocol ip prio 1 u32 match icmp type 0 0 action skbedit priority 1:3 #set priority tin to 3
  Note: the 'fwmark' option can also be used to set a packet priority from
  its fwmark.

  - Flow hash override: To override flow hashing, the classid can be set.
CAKE will interpret  the  major number of the classid as the host hash used in host isolation mode, and the minor number as the flow hash used for flow-based  queueing.
     This example will assign all ICMP packets to the first queue:
     $ tc qdisc replace dev eth0 handle 1: root cake
     $ tc filter add dev eth0 parent 1: protocol ip prio 1 u32 match icmp type 0 0 classid 0:1

  * priority parameters:
       diffserv4
            Provides a general-purpose Diffserv implementation with four tins:
                 Bulk (CS1), 6.25% threshold, generally low priority.
                 Best Effort (general), 100% threshold.
                 Video  (AF4x,  AF3x, CS3, AF2x, CS2, TOS4, TOS1), 50% thresh‐
       old.
                 Voice (CS7, CS6, EF, VA, CS5, CS4), 25% threshold.

       diffserv3 (default)
            Provides a simple, general-purpose  Diffserv  implementation  with
       three tins:
                 Bulk (CS1), 6.25% threshold, generally low priority.
                 Best Effort (general), 100% threshold.
                 Voice  (CS7, CS6, EF, VA, TOS4), 25% threshold, reduced Codel
       interval.

## Hierarchical Token Bucket (HTB) qdisc / Hierarchical fair-service curve (hfsc)

* https://wiki.debian.org/TrafficControl

Creating root 1: and 1:1 using HTB (default 6 means follow 1:6 if no rule matched) 
$ tc qdisc add dev eth1 root handle 1: htb default 6
$ tc class add dev eth1 parent 1: classid 1:1 htb rate 2mbit ceil 2mbit

Create leaf 1:5 and add filter to redirect to it
$ tc class add dev eth1 parent 1:1 classid 1:5 htb rate 1mbit ceil 1.5mbit
$ tc filter add dev eth1 protocol ip parent 1:0 prio 0 u32 match ip src YOUR_MAIL_SERVER_IP/32 flowid 1:5
$ tc filter add dev eth1 protocol ip parent 1:0 prio 0 u32 match ip sport 22 0xffff flowid 1:5

Leaf 1:6
$ tc class add dev eth1 parent 1:1 classid 1:6 htb rate 0.5mbit ceil 1.5mbit

Leaf 1:7
$ tc class add dev eth1 parent 1:1 classid 1:7 htb rate 0.2mbit ceil 1mbit
$ tc filter add dev eth1 protocol ip parent 1:0 prio 5 u32 match ip src VIDEO_STREAM_IP/32 flowid 1:7

Add a qdisc leaf (otherwise the leaf behaves as a pfifo_fast qdisc):
$ tc qdisc add dev eth1 parent 1:5 handle 20: sfq perturb 10

=> Result
$ tc qdisc show dev eth1
qdisc htb 1: root refcnt 2 r2q 10 default 0x6 direct_packets_stat 0 direct_qlen 1000
qdisc sfq 20: parent 1:5 limit 127p quantum 1514b depth 127 divisor 1024 perturb 10sec 
$ tc class show dev eth1
class htb 1:1 root rate 2Mbit ceil 2Mbit burst 1600b cburst 1600b 
class htb 1:5 parent 1:1 prio 0 rate 1Mbit ceil 1500Kbit burst 1600b cburst 1599b 
class htb 1:6 parent 1:1 prio 0 rate 500Kbit ceil 1500Kbit burst 1600b cburst 1599b 
class htb 1:7 parent 1:1 prio 0 rate 200Kbit ceil 1Mbit burst 1600b cburst 1600b
$ tc filter show dev eth1

* HFSC:
- http://man7.org/linux/man-pages/man7/tc-hfsc.7.html
- http://linux-ip.net/articles/hfsc.en/

 For complex traffic shaping scenarios, hierarchical algorithms are necessary. Current versions of Linux support the algorithms HTB and HFSC. While HTB basically rearranges token bucket filter (TBF) into a hierarchical structure, thereby retaining the principal characteristics of TBF, HFSC allows proportional distribution of bandwidth as well as control and allocation of latencies. This allows for better and more efficient use of a connection for situations in which both bandwidth intensive data services and interactive services share a single network link. 

$ tc qdisc add dev $dev root handle $ID: hfsc [default $classID ] 
In the second step, the class hierarchy is constructed with consecutive class additions.
$ tc add class dev $dev parent parentID classid $ID hfsc [ [ rt  SC ] [ ls  SC ] | [ sc  SC ] ]  [ ul  SC ]

## Netem qdisc

* https://netbeez.net/blog/how-to-use-the-linux-traffic-control/
=> netem: network emulator; to add a wan property

- simulate delay
$ tc qdisc add dev eth0 root netem delay 200ms
change in place:
$ tc qdisc change dev eth0 root netem delay 300ms

- simulate packet loss / corruption
$  tc qdisc add dev eth0 root netem loss 10%
$ tc qdisc change dev eth0 root netem corrupt 5%
$ tc qdisc change dev eth0 root netem duplicate 1% 

## https://www.tldp.org/HOWTO/html_single/Traffic-Control-HOWTO/

[root@leander]# tc class add    \ (1)
>                  dev eth0     \ (2)
>                  parent 1:1   \ (3)
>                  classid 1:6  \ (4)
>                  htb          \ (5)
>                  rate 256kbit \ (6)
>                  ceil 512kbit   (7)

(1) Add a class. The verb could also be del. 
(2) Specify the device onto which we are attaching the new class. 
(3) Specify the parent handle to which we are attaching the new class. 
(4) This is a unique handle (major:minor) identifying this class. The minor number must be any non-zero (0) number. 
(5) Both of the classful qdiscs require that any children classes be classes of the same type as the parent. Thus an HTB qdisc will contain HTB classes. 
(6)(7) This is a class specific parameter. Consult Section 7.1 for more detail on these parameters. 


[root@leander]# tc filter add               \ (1)
>                  dev eth0                 \ (2)
>                  parent 1:0               \ (3)
>                  protocol ip              \ (4)
>                  prio 5                   \ (5)
>                  u32                      \ (6)
>                  match ip port 22 0xffff  \ (7)
>                  match ip tos 0x10 0xff   \ (8)
>                  flowid 1:6               \ (9)
>                  police                   \ (10)
>                  rate 32000bps            \ (11)
>                  burst 10240              \ (12)
>                  mpu 0                    \ (13)
>                  action drop/continue       (14)

(1) Add a filter. The verb could also be del. 
(2) Specify the device onto which we are attaching the new filter. 
(3) Specify the parent handle to which we are attaching the new filter. 
(4) This parameter is required. It's use should be obvious, although I don't know more. 
(5) The prio parameter allows a given filter to be preferred above another. The pref is a synonym. [lower number=higher prec]
(6) This is a classifier, and is a required phrase in every tc filter command. 
(7)(8) These are parameters to the classifier. In this case, packets with a type of service flag (indicating interactive usage) and matching port 22 will be selected by this statement. 
(9) The flowid specifies the handle of the target class (or qdisc) to which a matching filter should send its selected packets. 
(10) This is the policer, and is an optional phrase in every tc filter command. 
(11) The policer will perform one action above this rate, and another action below (see action parameter). 
(12) The burst is an exact analog to burst in HTB (burst is a buckets concept). 
(13) The minimum policed unit. To count all traffic, use an mpu of zero (0). 
(14) The action indicates what should be done if the rate based on the attributes of the policer. The first word specifies the action to take if the policer has been exceeded. The second word specifies action to take otherwise. 

## lartc

### Classless qdisc
* https://lartc.org/howto/lartc.qdisc.classless.html

9.2.1. pfifo_fast [3 bands]
Do not confuse this classless simple qdisc with the classful PRIO one! Although they behave similarly, pfifo_fast is classless and you cannot add other qdiscs to it with the tc command.

###  Classful qdisc
* https://lartc.org/howto/lartc.qdisc.classful.html

                     1:   root qdisc
                      |
                     1:1    child class
                   /  |  \
                  /   |   \
                 /    |    \
                 /    |    \
              1:10  1:11  1:12   child classes
               |      |     | 
               |     11:    |    leaf class
               |            | 
               10:         12:   qdisc
              /   \       /   \
           10:1  10:2   12:1  12:2   leaf classes

* Exemple with a prio qdisc:
          1:   root qdisc
         / | \ 
       /   |   \
       /   |   \
     1:1  1:2  1:3    classes
      |    |    |
     10:  20:  30:    qdiscs    qdiscs
     sfq  tbf  sfq
band  0    1    2

$ tc qdisc add dev eth0 root handle 1: prio 
This *instantly* creates classes 1:1, 1:2, 1:3

$ tc qdisc add dev eth0 parent 1:1 handle 10: sfq
$ tc qdisc add dev eth0 parent 1:2 handle 20: tbf rate 20kbit buffer 1600 limit 3000
$ tc qdisc add dev eth0 parent 1:3 handle 30: sfq

* Exemple with an htb class:

$ tc qdisc add dev eth0 root handle 1: htb default 30

$ tc class add dev eth0 parent 1: classid 1:1 htb rate 6mbit burst 15k

$ tc class add dev eth0 parent 1:1 classid 1:10 htb rate 5mbit burst 15k
$ tc class add dev eth0 parent 1:1 classid 1:20 htb rate 3mbit ceil 6mbit burst 15k
$ tc class add dev eth0 parent 1:1 classid 1:30 htb rate 1kbit ceil 6mbit burst 15k

The author then recommends SFQ for beneath these classes:

$ tc qdisc add dev eth0 parent 1:10 handle 10: sfq perturb 10
$ tc qdisc add dev eth0 parent 1:20 handle 20: sfq perturb 10
$ tc qdisc add dev eth0 parent 1:30 handle 30: sfq perturb 10

Add the filters which direct traffic to the right classes:

$ U32="tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32"
$ $U32 match ip dport 80 0xffff flowid 1:10
$ $U32 match ip sport 25 0xffff flowid 1:20

### Filter

https://lartc.org/howto/lartc.qdisc.filters.html
### Imq

* https://lartc.org/howto/lartc.imq.html

  - Only egress shaping is possible (an ingress qdisc exists, but its possibilities are very limited compared to classful qdiscs).
  - A qdisc can only see traffic of one interface, global limitations can't be placed.

IMQ is there to help solve those two limitations. In short, you can put everything you choose in a qdisc. Specially marked packets get intercepted in netfilter NF_IP_PRE_ROUTING and NF_IP_POST_ROUTING hooks and pass through the qdisc attached to an imq device. An iptables target is used for marking the packets.

This enables you to do ingress shaping as you can just mark packets coming in from somewhere and/or treat interfaces as classes to set global limits. You can also do lots of other stuff like just putting your http traffic in a qdisc, put new connection requests in a qdisc, ...

$ iptables -t mangle -A PREROUTING -i eth0 -j IMQ --todev 0
$ ip link set imq0 up

$ tc qdisc add dev imq0 root handle 1: htb default 20
...

Note: now replaced by ifb
cf https://wiki.linuxfoundation.org/networking/ifb
   https://wiki.archlinux.org/index.php/Advanced_traffic_control

Virtual networking
==================

## VRF

* https://lwn.net/Articles/632522/
Original patch, with exemple usages

Exemple:
  - eth1: 1.1.1.1/24: group 1, vrf 11
  - eth5: 1.1.1.1/24 (not a typo, duplicate address in different vrfs) group 1, vrf 13
  $ ip vrf exec 11 ip addr show dev eth1
          inet 1.1.1.1/24 brd 1.1.1.255 scope global eth1

  3. start ssh in group1 namespace
  $ ip netns exec group1 ip vrf exec 11 /usr/sbin/sshd -d
    ssh to 1.1.1.1 via eth1

  $ ip netns exec group1 ip vrf exec 13 /usr/sbin/sshd -d
    ssh to 1.1.1.1 via eth5
    --> same namespace but different VRFs

  4. One ssh instance handles VRFs in group1 namespace
  $ ip netns exec group1 ip vrf exec any /usr/sbin/sshd
    --> ssh to any address in the namespace works

* https://www.netdevconf.org/1.1/proceedings/slides/ahern-vrf-tutorial.pdf
Slides

* https://www.kernel.org/doc/Documentation/networking/vrf.txt

The VRF device combined with ip rules provides the ability to create virtual
routing and forwarding domains (aka VRFs, VRF-lite to be specific) in the Linux
network stack. One use case is the multi-tenancy problem where each tenant has
their own unique routing tables and in the very least need different default
gateways.

An important feature of the VRF device implementation is that it impacts only
Layer 3 and above. In addition, VRF devices allow VRFs to be nested within
namespaces. For example network namespaces provide separation of network
interfaces at the device layer, VLANs on the interfaces within a namespace
provide L2 separation and then VRF devices provide L3 separation.

Exemple
1. VRF device is created with an association to a FIB table.
   e.g, ip link add vrf-blue type vrf table 10
           ip link set dev vrf-blue up

2. An l3mdev FIB rule directs lookups to the table associated with the device.
   => 1000:	from all lookup [l3mdev-table]

   A single l3mdev rule is sufficient for all VRFs. The VRF device adds the
   l3mdev rule for IPv4 and IPv6 when the first device is created with a
   default preference of 1000. Users may delete the rule if desired and add
   with a different priority or install per-VRF rules.

   Prior to the v4.8 kernel iif and oif rules are needed for each VRF device:
       ip ru add oif vrf-blue table 10
       ip ru add iif vrf-blue table 10

3. Set the default route for the table (and hence default route for the VRF).
       ip route add table 10 unreachable default metric 4278198272

   This high metric value ensures that the default unreachable route can
   be overridden by a routing protocol suite.  FRRouting interprets
   kernel metrics as a combined admin distance (upper byte) and priority
   (lower 3 bytes).  Thus the above metric translates to [255/8192].

4. Enslave L3 interfaces to a VRF device.
       ip link set dev eth1 master vrf-blue

   Local and connected routes for enslaved devices are automatically moved to
   the table associated with VRF device. Any additional routes depending on
   the enslaved device are dropped and will need to be reinserted to the VRF
   FIB table following the enslavement.

   The IPv6 sysctl option keep_addr_on_down can be enabled to keep IPv6 global
   addresses as VRF enslavement changes.
       sysctl -w net.ipv6.conf.all.keep_addr_on_down=1

5. Additional VRF routes are added to associated table.
       ip route add table 10 ...

=> one can use 'ip vrf exec VRFNAME' to get l3 isolation like one use 'ip netns exec NETNSNAME' to get l1 isolation:
$ ip link add red type vrf table 11
$ ip link set dev red up
$ ip vrf exec red ssh 10.100.1.254
cf man 'ip-vrf'

## Virtual interfaces

* Documentation
https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/
https://events.static.linuxfound.org/sites/events/files/slides/2016%20-%20Linux%20Networking%20explained_0.pdf

https://news.ycombinator.com/item?id=8930702 LXC container networking deep dive (flockport.com) => Extending L2 with Ethernet over GRE via ip link add testgre type gretap

* Exemples divers
- Bonded / Team: unify two physical interfaces
  # ip link add bond1 type bond miimon 100 mode active-backup
  # ip link set eth0 master bond1
  # ip link set eth1 master bond1

- VLan: VLAN, aka virtual LAN, separates broadcast domains by adding tags to network packets => needs router support to handle the vlan tags
  # ip link add link eth0 name eth0.2 type vlan id 2
  # ip link add link eth0 name eth0.3 type vlan id 3

- MACsec
MACsec (Media Access Control Security) is an IEEE standard for security in wired Ethernet LANs

### vxlan
VXLAN (Virtual eXtensible Local Area Network) is a tunneling protocol
designed to solve the problem of limited VLAN IDs (4,096).

  # ip link add vx0 type vxlan id 100 local 1.1.1.1 remote 2.2.2.2 dev eth0 dstport 4789

* Notes: vlan vs vxlan vs qinq
https://community.fs.com/blog/qinq-vs-vlan-vs-vxlan.html
https://www.quora.com/What-is-the-difference-between-VLAN-and-VXLAN

* More infos:
https://www.kernel.org/doc/Documentation/networking/vxlan.txt
https://ilearnedhowto.wordpress.com/2017/02/16/how-to-create-overlay-networks-using-linux-bridges-and-vxlans/
https://blogs.vmware.com/vsphere/2013/04/vxlan-series-different-components-part-1.html How vxlan use multicast to simulate broadcasting; and how it creates forwarding entries (https://blogs.vmware.com/vsphere/2013/05/vxlan-series-how-vtep-learns-and-creates-forwarding-table-part-5.html)

* Mode of operations: https://vincent.bernat.ch/en/blog/2017-vxlan-linux
1) Use multicast:
ip -6 link add vxlan100 type vxlan id 100 dstport 4789 local 2001:db8:1::1 \
                group ff05::100 dev eth0 ttl 5
(note: port 8472 is linux's default but the rfc specify 4789)

2) Unicast with static flooding
 ip -6 link add vxlan100 type vxlan id 100 dstport 4789 local 2001:db8:1::1
 bridge fdb append 00:00:00:00:00:00 dev vxlan100 dst 2001:db8:2::1
 bridge fdb append 00:00:00:00:00:00 dev vxlan100 dst 2001:db8:3::1
where 2001:db8:2/3::1 are the ip addresses of the vtep (VXLAN Tunnel Endpoint)

Raccourci: https://blog.wescale.fr/2018/02/15/les-reseaux-doverlay-principes-et-fonctionnement/
  # ip link add vxlan10 type vxlan id 10 local 10.0.0.2 remote 10.0.0.3
  => ajoute automatiquement dans la fdb: $bridge fdb show
  00:00:00:00:00:00 dev vxlan10 dst 10.0.0.3 self permanent
  Et l'autodécouverte donne, par exemple avec un
  # ping 192.168.0.3
  ajoute l'adresse mac de 192.168.0.3 sur la fdb
  96:3d:d7:db:24:ee dev vxlan10 dst 10.0.0.3 self
  et notre adresse mac sur la fdb de 10.0.0.3:
  2e:f0:64:67:76:9f dev vxlan10 dst 10.0.0.2 self

3) Unicast with static L2 entries
 ip -6 link add vxlan100 type vxlan id 100 dstport 4789 local 2001:db8:1::1 nolearning

 Thanks to the nolearning flag, source-address learning is disabled. Therefore, if a MAC is missing, the frame will always be sent using the all-zero entries.

 bridge fdb append 00:00:00:00:00:00 dev vxlan100 dst 2001:db8:2::1
 bridge fdb append 00:00:00:00:00:00 dev vxlan100 dst 2001:db8:3::1
 bridge fdb append 50:54:33:00:00:09 dev vxlan100 dst 2001:db8:2::1
 bridge fdb append 50:54:33:00:00:0a dev vxlan100 dst 2001:db8:2::1
 bridge fdb append 50:54:33:00:00:0b dev vxlan100 dst 2001:db8:3::1

3) Unicast with static L3 entries
  In the previous example, we had to keep the all-zero entries for ARP and IPv6 neighbor discovery to work correctly. However, Linux can answer to neighbor requests on behalf of the remote nodes. When this feature is enabled, the default entries are not needed anymore (but you could keep them):

 ip -6 link add vxlan100 type vxlan id 100 dstport 4789 local 2001:db8:1::1 nolearning proxy
 ip -6 neigh add 2001:db8:ff::11 lladdr 50:54:33:00:00:09 dev vxlan100
 ip -6 neigh add 2001:db8:ff::12 lladdr 50:54:33:00:00:0a dev vxlan100
 ip -6 neigh add 2001:db8:ff::13 lladdr 50:54:33:00:00:0b dev vxlan100
 bridge fdb append 50:54:33:00:00:09 dev vxlan100 dst 2001:db8:2::1
 bridge fdb append 50:54:33:00:00:0a dev vxlan100 dst 2001:db8:2::1
 bridge fdb append 50:54:33:00:00:0b dev vxlan100 dst 2001:db8:3::1

4) Unicast with dynamic L3 entries
via l2miss and l3miss options

* Control pane
The dynamic mode allow to have a control pane.

Cf the book https://www.cisco.com/c/dam/en/us/td/docs/switches/datacenter/nexus9000/sw/vxlan_evpn/VXLAN_EVPN.pdf sur VXLAN + BGP EVPN
MP-BGP => comme BGP mais les infos de routage contiennent layer 2 mac +
layer 3 ip (ce qu'on appelle du EVPN).

### Tun/Tap

Tun/tap interfaces are a feature offered by Linux (and probably by other
UNIX-like operating systems) that can do userspace networking, that is,
allow userspace programs to see raw network traffic (at the ethernet or IP
level) and do whatever they like with it.
cf: https://backreference.org/2010/03/26/tuntap-interface-tutorial/
    https://www.fir3net.com/Networking/Terms-and-Concepts/virtual-networking-devices-tun-tap-and-veth-pairs-explained.html
Note: tap = layer 2 (eth), tun = layer 3 (ip)

### VETH
The VETH (virtual Ethernet) device is a local Ethernet tunnel. Devices are created in pairs, as shown in the diagram below.

Use a VETH configuration when namespaces need to communicate to the main host namespace or between each other.

Exemples:
  sudo ip link add v-eth1 type veth peer name v-peer1

  # ip netns add net1
  # ip netns add net2
  # ip link add veth1 netns net1 type veth peer name veth2 netns net2

### Bridge = virtual switch
Exemple: connect eth0 with two taps and one veth device
  # ip link add br0 type bridge
  # ip link set eth0 master br0
  # ip link set tap1 master br0
  # ip link set tap2 master br0
  # ip link set veth1 master br0

### Macvlan
With VLAN, you can create multiple interfaces on top of a single one and filter packages based on a VLAN tag. With MACVLAN, you can create multiple interfaces with different Layer 2 (that is, Ethernet MAC) addresses on top of a single one.

Before MACVLAN, if you wanted to connect to physical network from a VM or
namespace, you would have needed to create TAP/VETH devices and attach one
side to a bridge and attach a physical interface to the bridge on the host
at the same time, as shown below.

Now, with MACVLAN, you can bind a physical interface that is associated with a MACVLAN directly to namespaces, without the need for a bridge.

There are five MACVLAN types:
1. Private: doesn’t allow communication between MACVLAN instances on the same physical interface, even if the external switch supports hairpin mode.
2. VEPA: data from one MACVLAN instance to the other on the same physical interface is transmitted over the physical interface. Either the attached switch needs to support hairpin mode or there must be a TCP/IP router forwarding the packets in order to allow communication.
3. Bridge: all endpoints are directly connected to each other with a simple bridge via the physical interface.
4. Passthru: allows a single VM to be connected directly to the physical interface.
5. Source: the source mode is used to filter traffic based on a list of allowed source MAC addresses to create MAC-based VLAN associations

* Exemples:
  # ip link add macvlan1 link eth0 type macvlan mode bridge
  # ip link add macvlan2 link eth0 type macvlan mode bridge
  # ip netns add net1
  # ip netns add net2
  # ip link set macvlan1 netns net1
  # ip link set macvlan2 netns net2

- Add the host to a macvlan so it can communicate with the slaves directly:
https://unix.stackexchange.com/questions/400190/how-to-make-reachable-macvlan-aliases-in-a-different-namespaces

* Notes:
http://hicu.be/bridge-vs-macvlan

### IPVLAN

IPVLAN is similar to MACVLAN with the difference being that the endpoints have the same MAC address.
- IPVLAN L2 mode acts like a MACVLAN in bridge mode. The parent interface looks like a bridge or switch.
- In IPVLAN L3 mode, the parent interface acts like a router and packets are routed between endpoints, which gives better scalability.

=> In mode l3, needs to setup 'ip neigh' explicitely or set up routing
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-ipvlan_configuring-and-managing-networking

* More on ipvlan:
https://www.kernel.org/doc/Documentation/networking/ipvlan.txt
https://hicu.be/macvlan-vs-ipvlan
https://gist.github.com/nerdalert/c0363c15d20986633fda Macvlan, Ipvlan and 802.1q Trunk Driver Notes (Seting up macvlan/ipvlan in docker)
https://gist.github.com/nerdalert/f493d475d9ad36e194d6 Quick Paste Ipvlan L3 Instructions

* Exemples:
  # ip netns add ns0
  # ip link add name ipvl1 link eth0 type ipvlan mode l2
  # ip link set dev ipvl0 netns ns0

Cf https://superuser.com/questions/1113812/how-to-configure-macvlan-interface-for-getting-the-ip
  Macvlans are not built to work on wireless interfaces.
  To circumvent this problem, you should use an ipvlan instead, which uses the mac address of the physical interface
  $ ip link add link wlan0 ipvl0 type ipvlan mode l2
  $ ip link set dev ipvl0 up
  $ ip addr add 192.168.73.201/24 dev ipvl0
  (my LAN is 192.168.73.0/24, change as needed to adapt it to your case) and you can also change the default gateway to be accessible on the new virtual interface, instead of the old, physical one:
  $ ip route del default
  $ ip route add default via 192.168.73.1 dev ipvl0 src 192.168.73.201 metric 1
  The only noteworthy comment is that I used mode L2: ipvlans have three
  modes of operation, L2,L3,L3S (never mind that it is generally said that
  they have two modes, there is also the L3S which is similar to L3 but
  allows operation of iptables/conntrack). The difference between L2 and L3
  is that L2 allows the virtual interface to be bridged with the physical
  interface, which means it can have an address in the same subnet as the
  physical interface and L2 traffic is correctly relayed. L3 mode instead
  does not relay L2 traffic, and requires configuration as an IPv4 router:
  different subnets, need to setup routes, and so on. More hassle than
  worth, most of the times.

=> Full example with a namespace:
sudo ip link add link wlp2s0 ipvl0 type ipvlan mode l2
sudo ip netns add nsip
sudo ip link set ipvl0 netns nsip
sudo ip -n nsip addr add 192.168.0.51/24 dev ipvl0
sudo ip -n nsip link set ipvl0 up
sudo ip -n nsip route add default via 192.168.0.1

### MACVTAP/IPVTAP
MACVTAP/IPVTAP is a new device driver meant to simplify virtualized bridged networking. When a MACVTAP/IPVTAP instance is created on top of a physical interface, the kernel also creates a character device/dev/tapX to be used just like a TUN/TAP device, which can be directly used by KVM/QEMU.

With MACVTAP/IPVTAP, you can replace the combination of TUN/TAP and bridge drivers with a single module:

  # ip link add link eth0 name macvtap0 type macvtap

## Tunnels
[cf to_exterieur for more tunneling and vpns]

* Documentation
https://developers.redhat.com/blog/2019/05/17/an-introduction-to-linux-virtual-interfaces-tunnels/
    IPIP Tunnel [ip over ip]
    SIT Tunnel [IPv6/ipv4 over IPv4 tunneling mode]
    ip6tnl Tunnel [like sit but over ipv6]
    VTI and VTI6 [for ipsec]
    GRE and GRETAP [GRE could encapsulate any Layer 3 protocol with a valid Ethernet type (so multicast, icmp), unlike IPIP, which can only encapsulate IP. GRE: OSI Layer 3, GRETAP: OSI Layer 2]
    GRE6 and GRE6TAP [GRE for ipv6]
    FOU [FOU (foo over UDP) is UDP-level tunneling]
    GUE [Generic Udp encapsulation]
    GENEVE [Generic Network Virtualization Encapsulation (GENEVE) supports all of the capabilities of VXLAN, NVGRE, and STT and was designed to overcome their perceived limitations. => Encapsule ethernet over udp]
    ERSPAN and IP6ERSPAN

=> ip tunnel add mode ipip/sit/isatap/vti/gre/ip6ip6/...

* ipip
Cf https://www.netdevconf.org/0.1/docs/herbert-UDP-Encapsulation-Linux.pdf for using gre.

* udp encapsulation:
https://lwn.net/Articles/614348/ => foo over udp
  receive  $ ip fou add port 5555 ipproto 4
  transmit $  ip link add name tun1 type ipip \
       remote 192.168.1.1 local 192.168.1.2 ttl 225 \
       encap fou encap-sport auto encap-dport 5555

* fou vs gue:
http://man7.org/linux/man-pages/man8/ip-gue.8.html
The receiver infers the protocol of a packet received on a FOU UDP port to be
the protocol configured for the port.
Generic UDP Encapsulation (GUE) encapsulates packets of an IP
protocol within UDP and an encapsulation header. The encapsulation header
contains the IP protocol number for the encapsulated packet.
   Configure a FOU receive port for GRE bound to 7777
       # ip fou add port 7777 ipproto 47
   Configure a FOU receive port for IPIP bound to 8888
       # ip fou add port 8888 ipproto 4
   Configure a GUE receive port bound to 9999
       # ip fou add port 9999 gue

### Exemples
* ipip

  On Server A:
  # ip link add name ipip0 type ipip local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR
  # ip link set ipip0 up
  # ip addr add INTERNAL_IPV4_ADDR/24 dev ipip0
  Add a remote internal subnet route if the endpoints don't belong to the same subnet
  # ip route add REMOTE_INTERNAL_SUBNET/24 dev ipip0

  On Server B:
  # ip link add name ipip0 type ipip local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR
  # ip link set ipip0 up
  # ip addr add INTERNAL_IPV4_ADDR/24 dev ipip0
  # ip route add REMOTE_INTERNAL_SUBNET/24 dev ipip0

* sit

  # ip link add name sit1 type sit local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR mode any
  # ip link set sit1 up
  # ip addr add INTERNAL_IPV4_ADDR/24 dev sit1

* ip6tnl
  # ip link add name ipip6 type ip6tnl local LOCAL_IPv6_ADDR remote REMOTE_IPv6_ADDR mode any

* gre/gretap

  # ip link add name gre1 type gre local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR [seq] key KEY
  # ip link add name gretap1 type gretap local LOCAL_IPv4_ADDR remote REMOTE_IPv4_ADDR

* fou/gue
  Server A # ip fou add port 5555 ipproto 4
  Server B # ip link add name tun1 type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225 encap fou encap-sport auto encap-dport 5555

  Server A # ip fou add port 5555 gue
  Server B # ip link add name tun1 type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225 encap gue encap-sport auto encap-dport 5555

* Geneve
  # ip link add name geneve0 type geneve id VNI remote REMOTE_IPv4_ADDR

User space networking
=====================

* Fullnetwork stack
- https://github.com/rootless-containers/slirp4netns [used by rootless containers]
- https://www.linuxjournal.com/content/userspace-networking-dpdk
  DPDK is a fully open-source project that operates in (root) userspace. Used by Open vSwitch (OvS)

* Tun/Tap

* Switch VDE
  http://manpages.ubuntu.com/manpages/trusty/man1/vde_switch.1.html
  vde_switch - Virtual Distributed Ethernet switch

  Exemple: https://renatonel.wonderhowto.com/news/networking-virtual-machines-using-vde-0128489/
  $ sudo ip tuntap add mode tap tap0
  $ sudo vde_switch -s /tmp/vde.ctl -tap tap0
  Connect kvm to the vde switch:
  # vdeq kvm -hda local_hard_drive -net nic -net vde,sock=/tmp/vde.ctl -m 256

  More details: http://wiki.virtualsquare.org/wiki/index.php/VDE_Basic_Networking
  $ vdeq qemu -hda qemu-image -m 128 

  + Slirp: a virtual NAT router as a process
    The simplest way to connect your virtual network to the Internet is slirp. Slirp is a process that appear as a router on the virtual network. The slirp process forwards all the communications coming from the virtual network to the Internet. 
  => $ slirpvde -s /tmp/switch1 --dhcp

  Or use a tap to connect to the host network:
  # vde_switch -s /tmp/switch1 -tap tap0 -m 666

  + connect two vde switches
  $ dpipe vde_plug /tmp/switch1 = vde_plug /tmp/switch2

* Divers:
- https://github.com/snabbco/snabb
  Snabb: Simple and fast packet networking
- https://blog.scottlowe.org/2013/11/26/lxc-open-vswitch-and-gre-tunnels/

Routing and forwarding
======================

- rp_filter: cf [sysctl] pour rp_filter.
https://linuxfr.org/users/glandos/journaux/routage-avance-avec-marquage-de-paquet-et-rp_filter
https://serverfault.com/questions/932205/advanced-routing-with-firewall-marks-and-rp-filter
=> rp_filter ne regarde pas la valeur des marks pour les décisions de routing
(contrairement au vrai routing), sauf si `src_valid_mark=1`. Par contre il
regarde bien la valeur du tos.

- https://wiki.zionetrix.net/informatique:reseau:routage-internet-multi-liens

Routing daemon
--------------

https://en.wikipedia.org/wiki/Routing_Information_Protocol
https://en.wikipedia.org/wiki/Border_Gateway_Protocol

https://www.networkcomputing.com/data-centers/comparing-dynamic-routing-protocols
RIP, EIGRP and OSPF are all interior gateway protocols (IGP) while BGP is
an exterior gateway protocol (EGP). Basically, interior protocols are meant
to dynamically route data across a network that you fully control and
maintain. Exterior routing protocols are used to exchange routes between
distinctly separate networks that you have no administrative control over.

- quagga (fork of gnu zebra)
- bird routing daemon: https://bird.network.cz/?index
- scripting bgp: exabgp

Exemples
========

Modules
-------
sudo modprobe -r rt73usb rt73
sudo modprobe rt73
Forwarding
----------
see [sysctl] for other sysctls
see https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt

sysctl net.ipv4.ip_forward=1
Sysctl rules:
  conf/default/*: Change the interface-specific default settings.
  conf/all/*: Change all the interface-specific settings.
  net.ipv4.ip_forward = 1 #resets all configuration parameters to their default state
  net.ipv4.conf.all.forwarding = 1 #set all specific interfaces (including default); this seems to be equivalent to net.ipv4.ip_forward
  net.ipv4.conf.default.forwarding = 1
  net.ipv4.conf.eth0.forwarding = 1 #enable ip forwarding on this interface
  net.ipv4.conf.eth0.mc_forwarding = 0 #enable multicast routing
  net.ipv6.conf.all.forwarding = 0
  net.ipv6.conf.eth0.forwarding = 0

More details:https://utcc.utoronto.ca/~cks/space/blog/linux/IpForwardingSettings
- all/forwarding: setting this is the same as setting the global sysctl.
- default/forwarding: controls the default state of forwarding; this state gets used by interfaces that have not set a specific value. Setting the global sysctl counts as giving all existing interfaces a specific value.

Masquerading:
iptables -t nat -A POSTROUTING -o output_interface -j MASQUERADE

If we have a firewall open it for the NAT:
- Accepts all already established connection
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
- Accept input packets from our NAT:
iptables -A FORWARD -i input_interface -o output_interface -j ACCEPT

Spoof mac address
-----------------
sudo ip link set dev wlan0 down
sudo ip link set dev wlan0 address 00:08:a1:b0:a6:a2
sudo ip link set dev wlan0 up
sudo iw dev wlan0 set type managed
sudo iwpriv wlan0 set AuthMode=WPAPSK
sudo iwpriv wlan0 set EncrypType=AES
sudo iwpriv wlan0 set SSID="SSID"
sudo iwpriv wlan0 set WPAPSK="PASSWD"
sudo dhclient/dhcpcd wlan0

http://unix.stackexchange.com/questions/21841/make-some-virtual-mac-address
ip link add link eth0 name eth0.1 address 00:11:11:11:11:11 type macvlan
ip link set eth0.1 up
dhclient -v eth0.1
ip link set dev eth0.1 address 00:08:a1:b0:a6:a2
sudo ip link delete eth0.1

wowlan
------

sudo iw phy phy0 wowlan enable magic-packet
iw help wowlan

Network namespaces
------------------

https://blogs.igalia.com/dpino/2016/04/10/network-namespaces/

 # Create namespace
sudo ip netns add ns1

 # Create veth link.
sudo ip link add v-eth1 type veth peer name v-peer1

 # Add peer-1 to NS.
sudo ip link set v-peer1 netns ns1

 # Setup IP address of v-eth1.
sudo ip addr add 10.200.1.1/24 dev v-eth1
sudo ip link set v-eth1 up

 # Setup IP address of v-peer1.
sudo ip netns exec ns1 ip addr add 10.200.1.2/24 dev v-peer1
sudo ip netns exec ns1 ip link set v-peer1 up
sudo ip netns exec ns1 ip link set lo up

 # Routing
sudo ip netns exec ns1 ip route add default via 10.200.1.1

 ## Share internet access between host and NS.

 # Enable IP-forwarding.
 # echo 1 > /proc/sys/net/ipv4/ip_forward
sudo sysctl net.ipv4.ip_forward=1

 # Flush forward rules, policy DROP by default.
sudo iptables -P FORWARD DROP
sudo iptables -F FORWARD

 # Flush nat rules.
sudo iptables -t nat -F

 # Enable masquerading of 10.200.1.0.
 dev=wlp2s0 #the device connected to the internet
sudo iptables -t nat -A POSTROUTING -s 10.200.1.0/255.255.255.0 -o $dev -j MASQUERADE

 # Allow forwarding between $dev and v-eth1.
sudo iptables -A FORWARD -i $dev -o v-eth1 -j ACCEPT
sudo iptables -A FORWARD -o $dev -i v-eth1 -j ACCEPT

sudo ip netns exec ns1 bash
sudo ip netns exec ns1 sudo -u dams zsh
sudo ip netns exec ns1 su dams

 # https://www.cyberciti.biz/faq/how-to-find-my-public-ip-address-from-command-line-on-a-linux/
curl ifconfig.me

Réseaux virtuels
----------------
* https://vincent.bernat.ch/en/blog

- https://vincent.bernat.ch/fr/blog/2011-lab-reseau-uml
  user mode linux connectés par tap ou vde_switch

  Exemple tap: on crée deux taps tap-R1, tap-R2 reliés en bridge
  $ linux init=/bin/sh rootfstype=hostfs eth0=tuntap,tap-R1
  $ linux init=/bin/sh rootfstype=hostfs eth0=tuntap,tap-R2

  Avec vde_switch:
  $ vde-switch
  $ linux init=/bin/sh rootfstype=hostfs eth0=vde

- https://vincent.bernat.ch/en/blog/2011-lab-site-to-site-vpn
  BGP via bird

- https://vincent.bernat.ch/fr/blog/2018-vpn-wireguard-route
  VPN routé avec Linux et WireGuard (+BGP via bird)

- https://vincent.bernat.ch/fr/blog/2017-vpn-ipsec-route
  VPN IPsec routé avec Linux et strongSwan (+bird)