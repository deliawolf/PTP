# PTP

PTP is defined in IEEE 1588 as Precision Clock Synchronization for Networked Measurements and Control Systems. PTP was developed to synchronize clocks in packet-based networks that include distributed device clocks of varying precision and stability. PTP is designed specifically for industrial, networked measurement and control systems, and is optimal for use in distributed systems because it requires minimal bandwidth and little processing overhead.

Network Time Protocol (NTP) and Precision Time Protocol (PTP) are both protocols used for clock synchronization between computer systems over packet-switched networks. While they both serve the purpose of synchronizing time, they differ in their precision and use cases.

NTP (Network Time Protocol):

NTP is typically used when you need to synchronize time across devices on the internet or a local network.
It is capable of synchronizing time over variable paths on the internet with a precision of a few milliseconds over the public internet and sub-millisecond on LANs.
NTP is suitable for applications that can tolerate small inaccuracies in time synchronization.
PTP (Precision Time Protocol):

PTP, defined in the IEEE 1588 standard, is used when very high precision time synchronization is required.
It can achieve clock accuracy in the sub-microsecond range under ideal conditions, making it suitable for applications that require highly precise timing, such as telecommunications, data center networks, and some industrial systems.
PTP typically requires hardware support to reach its highest accuracy levels.
In summary, if you require very high precision time synchronization, PTP is the preferred choice. If you just need to maintain relatively accurate time among a group of devices, NTP would be sufficient and easier to implement.

## PTP Clocks
A PTP network is made up of PTP-enabled devices and devices that are not using PTP.

The PTP-enabled devices typically consist of the following clock types:

1. The Grandmaster Clock is the primary source of time for clock synchronization using PTP.

2. An Ordinary Clock is a PTP clock with a single PTP port.

3. A Boundary Clock in a PTP network operates in place of a standard network switch or router.

4. A Transparent Clock in a PTP network updates the time-interval field that is part of the PTP event message. This update compensates for switch delay and has an accuracy of within one picosecond.

There are two types of transparent clocks:

End-to-end (E2E) transparent clocks measure the PTP event message transit time for SYNC and DELAY_REQUEST messages. The secondary uses this information when determining the offset between the secondary’s and the primary’s time. End-to-end transparent clocks do not provide correction for the propagation delay of the link itself.

Peer-to-peer (P2P) transparent clocks measure PTP event message transit time in the same way end-to-end transparent clocks do. In addition, peer-to-peer transparent clocks measure the upstream link delay. The upstream link delay is the estimated packet propagation delay between the upstream neighbor peer-to-peer transparent clock and the peer-to-peer transparent clock under consideration. These two times (message transit time and upstream link delay time) are both added to the correction field of the PTP event message, and the correction field of the message received by the secondary contains the sum of all link delays. In theory, this is the total end-to-end delay (from primary to secondary) of the SYNC packet.

## Configure and Verify PTP
Specify the synchronization clock mode:
```
ptp mode {boundary {delay-req | pdelay-req } | e2etransparent | p2ptransparent}
```
The boundary mode enables the switch to participate in choosing the best primary clock. If no better clocks are detected, the switch becomes the grandmaster clock on the network and the parent clock to all connected devices. If the best primary is determined to be a clock connected to the switch, the switch synchronizes to that clock as a child to the clock, then acts as a parent clock to devices connected to other ports. After initial synchronization, the switch and the connected devices exchange timing messages to correct time skew caused by clock offsets and network delays. Use this mode when overload or heavy load conditions produce significant delay jitter.

The e2etransparent mode enables the switch to synchronize all switch ports with the grandmaster clock connected to the switch. This mode is the default clock mode. The switch corrects for the delay incurred by every packet passing through it. This mode causes less jitter and error accumulation than boundary mode.

The p2ptransparent mode disables the switch to synchronize its clock with the primary clock. A switch in this mode does not participate in primary clock selection and uses the default PTP clock mode on all ports.

Let's say you have Router1 as the PTP Grandmaster, Router2 as a PTP Boundary Clock (acts as a slave to Router1 and as a master to Switch1), and Switch1 as a PTP client. Here is a basic configuration for such a setup:

Router1 (PTP Grandmaster):
```
Router1(config)#ptp domain 0
Router1(config)#ptp mode grandmaster
Router1(config)#interface GigabitEthernet0/0
Router1(config-if)#ptp enable
```
Router2 (PTP Boundary Clock):
```
Router2(config)#ptp domain 0
Router2(config)#ptp mode boundary
Router2(config)#interface GigabitEthernet0/0
Router2(config-if)#ptp enable
Router2(config-if)#ptp port state slave
Router2(config)#interface GigabitEthernet0/1
Router2(config-if)#ptp enable
Router2(config-if)#ptp port state master
```
Switch1 (PTP Client):
```
Switch1(config)#ptp domain 0
Switch1(config)#interface GigabitEthernet0/0
Switch1(config-if)#ptp enable
Switch1(config-if)#ptp port state slave
```

The Ordinary Clock is usually a device that operates as a source (master) or destination (slave) but not both. Here's how you could incorporate an Ordinary Clock (Router3) into the previous setup:
```
Router3(config)#ptp domain 0
Router3(config)#interface GigabitEthernet0/0
Router3(config-if)#ptp enable
Router3(config-if)#ptp port state slave
```

Configure the PTP on Layer 2 interface:
```
Device(config-if)# ptp vlan 5
```

Configure the IPv4 as the PTP transport mode on SVI or Layer 3 interface:
```
Device(config-if)# ptp transport ipv4 udp
```

Configure the source IP on PTP:
```
Device(config)# ptp source source address
Device(config)# ptp source loopback
Device(config)# ptp source vlan
```
The ptp source address command enables the PTP messages in all the interfaces to carry the source IP address.

The ptp source loopback command enables the PTP messages in all the interfaces to carry the IP address configured on the loopback interface.

The ptp source vlan command enables the PTP messages to carry the IP address configured on the SVI interface corresponding to the port.

These are Cisco IOS XE verification commands for PTP feature:

## Verify the PTP port state:
```
show ptp port interface interface-name
```
Verify the PTP port states on all interfaces:
```
show ptp brief
```
Verify the PTP clock identity details and configured values of Priority1 and Priority2:
```
show ptp clock
```
Identify which Grandmaster Clock identity the device is synced to in boundary mode:
```
show ptp parent
```





