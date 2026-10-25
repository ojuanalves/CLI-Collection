# 🔥 FortiOS · troubleshooting cheatsheet

Day-to-day CLI commands. Replace anything in `<angle-brackets>` with the real value. Examples always sit below the block, so the copy button only grabs the command.

| Icon | Meaning |
|:---:|---|
| 🔴 | **Disruptive.** Drops sessions, tunnels or users, restarts something, or exposes a secret. |
| 🟡 | **Debug or config change.** Loads the CPU or changes the config. Stop or revert it when you're done. |

No icon means **read only**, safe to run.

> [!IMPORTANT]
> **Stop any debug** (keep this one handy):
> ```fortios
> diagnose debug disable
> diagnose debug reset
> ```
> Ex.: run it right after a `diagnose debug application ike -1` session so the console stops scrolling.

> [!WARNING]
> **Use debug commands carefully on production devices.** Filter first, and stop the debug as soon as you have what you need.

## Index

- [🎯 Iprope lookup](#-iprope-lookup)
- [🔑 Sudo (cross-VDOM commands)](#-sudo-cross-vdom-commands)
- [📊 System and status](#-system-and-status)
  - [Device info](#device-info) · [GUI settings](#gui-settings) · [Crash and error logs](#crash-and-error-logs) · [CPU and memory](#cpu-and-memory) · [Restarting processes](#restarting-processes)
- [🔌 Interfaces and hardware](#-interfaces-and-hardware)
  - [Interface status](#interface-status) · [NIC and throughput](#nic-and-throughput)
- [🚦 Routing](#-routing)
  - [Route lookup](#route-lookup) · [Static](#static) · [Policy routes](#policy-routes) · [BGP](#bgp) · [OSPF](#ospf)
- [📇 ARP](#-arp)
- [🔗 Sessions](#-sessions)
  - [List and stats](#list-and-stats) · [Filter and clear](#filter-and-clear)
- [📡 Ping, traceroute and telnet](#-ping-traceroute-and-telnet)
- [🔍 Packet sniffer](#-packet-sniffer)
- [🐞 Debug flow](#-debug-flow)
- [🔄 NAT](#-nat)
- [🧱 Firewall policy](#-firewall-policy)
- [🔐 VPN](#-vpn)
  - [IPsec (site-to-site)](#ipsec-site-to-site) · [SSL-VPN](#ssl-vpn)
- [👤 Authentication](#-authentication)
  - [Local and SSL-VPN users](#local-and-ssl-vpn-users) · [SAML](#saml) · [FSSO](#fsso) · [FortiClient EMS](#forticlient-ems)
- [🌐 DNS](#-dns)
- [📬 DHCP](#-dhcp)
- [🚫 Web filter and security profiles](#-web-filter-and-security-profiles)
  - [FortiGuard](#fortiguard) · [Web filter logs](#web-filter-logs) · [URL filter](#url-filter) · [Security profile engines](#security-profile-engines)
- [📝 Logging](#-logging)
- [🔁 HA (high availability)](#-ha-high-availability)
- [📶 SD-WAN and link monitor](#-sd-wan-and-link-monitor)
- [🤖 Automation stitch](#-automation-stitch)
- [🧩 REST API](#-rest-api)

📖 Companion doc: [Route Process](Route%20Process.md) explains the order in which FortiGate picks a route.

---

## 🎯 Iprope lookup

**Find which policy matches a flow without generating traffic**
```fortios
diagnose firewall iprope lookup <src-ip> <src-port> <dst-ip> <dst-port> <protocol> <src-interface>
```

| Source IP | Source port | Destination IP | Destination port | Protocol | Source interface |
|---|---|---|---|---|---|
| `10.210.14.139` | `0` | `57.144.179.32` | `443` | `6` | `port3` |

Ex.: `diagnose firewall iprope lookup 10.210.14.139 0 57.144.179.32 443 6 port3`

> [!NOTE]
> Use **`0`** as the source port when it doesn't matter. The source interface is **where the traffic enters the firewall**: a port, a VLAN or a tunnel.

**Protocol numbers**

| ICMP | TCP | UDP | GRE | ESP |
|:---:|:---:|:---:|:---:|:---:|
| 1 | 6 | 17 | 47 | 50 |

---

## 🔑 Sudo (cross-VDOM commands)

Runs a command in another VDOM **without switching to it**.

**Run a command in another VDOM**
```fortios
sudo <vdom> <command>
```
Ex.: `sudo FILIAIS get router info routing-table details 10.0.0.1`

---

## 📊 System and status

### Device info

**General status** (version, serial, VDOM mode, HA mode)
```fortios
get system status
```
Ex.: `get system status | grep -i version`

**Performance status** (CPU, memory, sessions, uptime)
```fortios
get system performance status
```
Ex.: `get system performance status | grep -i uptime`

**Hardware status** (model, CPU, RAM, disk)
```fortios
get hardware status
```
Ex.: `get hardware status | grep -i model`

**System info subtree** (admin sessions and similar)
```fortios
get system info <subcommand>
```
Ex.: `get system info admin status`

**Global settings**
```fortios
get system global
```
Ex.: `get system global | grep -i admin`

### GUI settings

🟡 **Show security profile groups in the GUI**
```fortios
config system settings
    set gui-security-profile-group enable
end
```
Ex.: to hide them again, run the same block with `set gui-security-profile-group disable`.

### Crash and error logs

**Read the crash log**
```fortios
diagnose debug crashlog read
```
Ex.: `diagnose debug crashlog read | grep -i application`

**Read the config error log** (worth checking after every upgrade)
```fortios
diagnose debug config-error-log read
```
Ex.: `diagnose debug config-error-log read | grep -i "parse error"`

### CPU and memory

**CPU and memory summary**
```fortios
diagnose sys top-summary
```
Ex.: `diagnose sys top-summary | grep -i wad`

**CPU and memory per process** (useful in conserve mode)

Refreshes every `<seconds>`, `<count>` times, optionally showing only the top `<lines>` processes. Inside `top`: `c` sorts by CPU, `m` by memory, `q` quits.
```fortios
diagnose sys top <seconds> <count> [<lines>]
```
Ex.: `diagnose sys top 5 30` or, top 3 only, `diagnose sys top 5 30 3`

**Total memory**
```fortios
get hardware memory
```
Ex.: `get hardware memory | grep -i MemFree`

**Top memory consumers**
```fortios
diagnose sys top-mem <count>
```
Ex.: `diagnose sys top-mem 10`

**Memory per process, detailed**
```fortios
diagnose sys top-mem detail
```
Ex.: `diagnose sys top-mem detail | grep -i ipsengine`

**Conserve mode state and thresholds**
```fortios
diagnose hardware sysinfo conserve
```
Ex.: `diagnose hardware sysinfo conserve | grep -i "conserve mode"`

**Interrupts per CPU** (spot a NIC pinning one core)
```fortios
diagnose hardware sysinfo interrupts
```
Ex.: `diagnose hardware sysinfo interrupts | grep -i port1`

### Restarting processes

**Find a process ID**
```fortios
diagnose sys process pidof <process>
```
Ex.: `diagnose sys process pidof fnbamd`

🔴 **Restart a process by PID** (signal 11 makes the watchdog respawn it)
```fortios
diagnose sys kill 11 <pid>
```
Ex.: `diagnose sys kill 11 1834`

> [!WARNING]
> **Restarts the process.** Whatever it serves (authentication, IPS, web filter, VPN) stops for a moment. Prefer a maintenance window on production.

🔴 **Restart a daemon through its test menu**
```fortios
diagnose test application <daemon> 99
```
Ex.: `diagnose test application ipsmonitor 99` restarts the IPS engine. `dnsproxy` works the same way.

🔴 **Reboot the unit**
```fortios
execute reboot
```
Ex.: `execute reboot`, then answer `y`.

> [!CAUTION]
> **Last resort.** The unit goes offline until it boots again. Do it in a **maintenance window**.

---

## 🔌 Interfaces and hardware

### Interface status

**All interfaces** (IP, status, mode)
```fortios
get system interface
```
Ex.: `get system interface | grep -i port3`

**Interfaces with MTU, index and counters**
```fortios
diagnose netlink interface list <interface>
```
Ex.: `diagnose netlink interface list port3`

**All IPs owned by the firewall**
```fortios
diagnose ip address list
```
Ex.: `diagnose ip address list | grep 10.11.100.`

**Interface from the Linux side**
```fortios
fnsysctl ifconfig <interface>
```
Ex.: `fnsysctl ifconfig port3`

**Zones and their members**
```fortios
show system zone | grep <text>
```
Ex.: `show system zone | grep VLAN`

### NIC and throughput

**NIC details** (speed, duplex, errors)
```fortios
diagnose hardware deviceinfo nic <port>
```
Ex.: `diagnose hardware deviceinfo nic lan1`

**Interface throughput test** (iperf against a server)
```fortios
diagnose traffictest run -c <iperf-server>
```
Ex.: `diagnose traffictest run -c 10.11.100.50`

---

## 🚦 Routing

Full lookup order (policy route, ISDB, SD-WAN, cache, FIB) is in [Route Process](Route%20Process.md).

### Route lookup

**Which route the firewall uses for an IP**
```fortios
get router info routing-table details <ip>
```
Ex.: `get router info routing-table details 192.168.1.99`

**Active routing table (RIB)**
```fortios
get router info routing-table all
```
Ex.: `get router info routing-table all | grep 0.0.0.0/0`

**Routing database, including inactive routes**
```fortios
get router info routing-table database
```
Ex.: `get router info routing-table database | grep -i inactive`

**Kernel forwarding table (FIB)**
```fortios
get router info kernel
```
Ex.: `get router info kernel | grep 10.11.100.`

### Static

**Static routes in the RIB**
```fortios
get router info routing-table static
```
Ex.: `get router info routing-table static | grep port1`

**Static route config**
```fortios
show router static [<id>]
```
Ex.: `show router static 5`

### Policy routes

**Policy routes, including SD-WAN rules**
```fortios
diagnose firewall proute list
```
Ex.: `diagnose firewall proute list | grep -i -A 10 port1`

### BGP

**Neighbor summary**
```fortios
get router info bgp summary
```
Ex.: `get router info bgp summary | grep 10.2.0.250`

**Neighbor details** (state, timers, capabilities)
```fortios
get router info bgp neighbors [<neighbor-ip>]
```
Ex.: `get router info bgp neighbors 10.2.0.250`

**Advertised routes**
```fortios
get router info bgp neighbors <neighbor-ip> advertised-routes
```
Ex.: `get router info bgp neighbors 10.2.0.250 advertised-routes`

**Received routes** (**needs** `set soft-reconfiguration enable` on the neighbor)
```fortios
get router info bgp neighbors <neighbor-ip> received-routes
```
Ex.: `get router info bgp neighbors 10.2.0.250 received-routes`

**BGP table** (all paths, not just the best ones)
```fortios
get router info bgp network [<prefix>]
```
Ex.: `get router info bgp network 10.10.0.0/16`

**Soft clear, inbound**: relearns routes without dropping the neighbor
```fortios
execute router clear bgp ip <neighbor-ip> soft in
```
Ex.: `execute router clear bgp ip 10.2.0.250 soft in`

### OSPF

**Neighbors and adjacency state**
```fortios
get router info ospf neighbor
```
Ex.: `get router info ospf neighbor | grep -i full`

**LSDB summary**
```fortios
get router info ospf database brief
```
Ex.: `get router info ospf database brief | grep -i router`

**OSPF routes in the RIB**
```fortios
get router info routing-table ospf
```
Ex.: `get router info routing-table ospf | grep 10.20.`

---

## 📇 ARP

**Full ARP table**
```fortios
get system arp
```
Ex.: `get system arp | grep -i port3`

**Filter by IP or range**
```fortios
get system arp | grep <ip-or-network>
```
Ex.: `get system arp | grep 10.11.100.`

**Find an IP anywhere in the config** (`-f` prints the whole block around the hit)
```fortios
show | grep -f <ip>
```
Ex.: `show | grep -f 10.11.100.232`

---

## 🔗 Sessions

### List and stats

**List active sessions**
```fortios
diagnose sys session list
```
Ex.: `diagnose sys session list | grep -c "session info"` counts them.

**Session statistics** (totals, clashes, ephemeral drops)
```fortios
diagnose sys session stat
```
Ex.: `diagnose sys session stat | grep -i clash`

### Filter and clear

🔴 **Clear all sessions**
```fortios
diagnose sys session clear
```
Ex.: only after a filter, see the two blocks below.

> [!CAUTION]
> Without a filter this drops **every session** on the firewall. Set a filter first (see the blocks below) and check it with `diagnose sys session filter`.

🟡 **Delete one session by ID**
```fortios
diagnose sys session delete <session-id>
```
Ex.: `diagnose sys session delete 0001a2b3`

🟡 **Filter by destination IP, then clear** (one IP or a range)
```fortios
diagnose sys session filter clear
diagnose sys session filter dst <first-ip> <last-ip>
diagnose sys session filter
diagnose sys session clear
```
Ex.: `diagnose sys session filter dst 10.3.79.165 10.3.79.185`

Running `filter` with no argument prints the active filter, so you can check it before the `clear`.

🟡 **Filter by source IP, then clear**
```fortios
diagnose sys session filter clear
diagnose sys session filter src <first-ip> <last-ip>
diagnose sys session filter
diagnose sys session clear
```
Ex.: `diagnose sys session filter src 10.3.79.165 10.3.79.185`

---

## 📡 Ping, traceroute and telnet

**Simple ping**
```fortios
execute ping <ip-or-fqdn>
```
Ex.: `execute ping 8.8.8.8`

**Ping from a specific source**: must be an IP that **belongs to the firewall**
```fortios
execute ping-options source <interface-ip>
execute ping <dst-ip>
```
Ex.: `execute ping-options source 10.11.100.232`

**Back to defaults**
```fortios
execute ping-options reset
```
Ex.: `execute ping-options reset`, then `execute ping-options view-settings` to confirm.

**Traceroute**
```fortios
execute traceroute <ip-or-fqdn>
```
Ex.: `execute traceroute 8.8.8.8`

**Traceroute from a specific source**
```fortios
execute traceroute-options source <interface-ip>
execute traceroute <dst-ip>
```
Ex.: `execute traceroute-options source 10.11.100.232`

**Telnet to test a TCP port**
```fortios
execute telnet <ip> <port>
```
Ex.: `execute telnet 10.11.100.232 22`

---

## 🔍 Packet sniffer

Syntax: `diagnose sniffer packet <interface> "<filter>" <verbosity> [count] [timestamp]`. **`Ctrl+C`** stops the capture.

| Verbosity | Shows |
|:---:|---|
| 1 | Headers only |
| 3 | Headers and Ethernet payload |
| 4 | Headers and in/out interface name |
| 6 | Headers, Ethernet payload and interface name (good for Wireshark conversion) |

Count `0` means unlimited. Timestamp `a` is absolute (UTC), `l` is local time; leave it out for relative time.

**By host**
```fortios
diagnose sniffer packet any "host <ip>" 4
```
Ex.: `diagnose sniffer packet any "host 10.211.12.68" 4`

**By host, unlimited, absolute timestamp**
```fortios
diagnose sniffer packet any "host <ip>" 4 0 a
```
Ex.: `diagnose sniffer packet any "host 10.211.12.68" 4 0 a`

**By host with payload** (export to Wireshark)
```fortios
diagnose sniffer packet any "host <ip>" 6 0 a
```
Ex.: `diagnose sniffer packet any "host 10.211.12.68" 6 0 a`

**Host and protocol (ICMP)**
```fortios
diagnose sniffer packet any "host <ip> and icmp" 4
```
Ex.: `diagnose sniffer packet any "host 201.55.34.141 and icmp" 4`

**Between two hosts**
```fortios
diagnose sniffer packet any "host <ip-a> and host <ip-b>" 4
```
Ex.: `diagnose sniffer packet any "host 201.55.34.141 and host 10.211.12.68" 4`

---

## 🐞 Debug flow

Shows the packet's path inside the firewall: chosen route, matching policy, NAT and the drop reason. 
> [!NOTE]
> The number after `trace start` is **how many packets to trace**, not a detail level.

🟡 **Basic debug flow**
```fortios
diagnose debug disable
diagnose debug flow trace stop
diagnose debug flow filter clear
diagnose debug flow filter saddr <src-ip>
diagnose debug flow filter daddr <dst-ip>
diagnose debug flow filter dport <port>
diagnose debug flow show console enable
diagnose debug flow show function-name enable
diagnose debug flow show iprope enable
diagnose debug console timestamp enable
diagnose debug flow trace start 9999
diagnose debug enable
```
Ex.:
```fortios
diagnose debug disable
diagnose debug flow trace stop
diagnose debug flow filter clear
diagnose debug flow filter saddr 10.210.14.139
diagnose debug flow filter daddr 57.144.179.32
diagnose debug flow filter dport 443
diagnose debug flow show console enable
diagnose debug flow show function-name enable
diagnose debug flow show iprope enable
diagnose debug console timestamp enable
diagnose debug flow trace start 9999
diagnose debug enable
```

**Available filters** (`diagnose debug flow filter ?`)

| Filter | What it does | Ex. |
|---|---|---|
| `clear` | Clears every filter | `filter clear` |
| `vd` | VDOM by index | `filter vd 0` |
| `vd-name` | VDOM by name | `filter vd-name root` |
| `proto` | Protocol (1 ICMP, 6 TCP, 17 UDP) | `filter proto 6` |
| `addr` | Source **or** destination IP (accepts a range) | `filter addr 10.2.54.3 10.2.35.5` |
| `saddr` | Source IP | `filter saddr 10.255.254.1` |
| `daddr` | Destination IP | `filter daddr 8.8.8.8` |
| `port` | Source **or** destination port | `filter port 5060` |
| `sport` | Source port | `filter sport 5060` |
| `dport` | Destination port | `filter dport 1521` |
| `negate` | Inverts the next filter | `filter negate saddr 10.44.114.36` |

> Every filter starts with `diagnose debug flow`. Running a filter with no value (e.g. `filter saddr`) removes only that one.

**Stop the debug**
```fortios
diagnose debug flow trace stop
diagnose debug disable
```
Ex.: run it as soon as you have the lines you need.

> [!WARNING]
> A forgotten trace **keeps eating CPU**. Always stop it before closing the session.

🟡 **Port-only flow with iprope** (route and policy lookup)
```fortios
diagnose debug disable
diagnose debug flow trace stop
diagnose debug flow filter clear
diagnose debug flow show iprope enable
diagnose debug flow filter dport <port>
diagnose debug flow show console enable
diagnose debug flow show function-name enable
diagnose debug console timestamp enable
diagnose debug flow trace start 999999
diagnose debug enable
```
Ex.: same block with `diagnose debug flow filter dport 514` to follow syslog traffic.

To test the policy without traffic, see [Iprope lookup](#-iprope-lookup).

---

## 🔄 NAT

**Central SNAT table** (when central NAT is enabled)
```fortios
show firewall central-snat-map [<id>]
```
Ex.: `show firewall central-snat-map 3`

**Virtual IPs (DNAT)**
```fortios
show firewall vip [<name>]
```
Ex.: `show firewall vip "VIP_WEB_443"`

**IPs the firewall answers for through VIPs and IP pools**
```fortios
diagnose firewall iplist list
```
Ex.: `diagnose firewall iplist list | grep 201.55.34.`

**IP pools and their type**
```fortios
diagnose firewall ippool-all list
```
Ex.: `diagnose firewall ippool-all list | grep -i overload`

**Sessions per IP pool**
```fortios
diagnose firewall ippool-all stats [<pool>]
```
Ex.: `diagnose firewall ippool-all stats "POOL_INTERNET"`

---

## 🧱 Firewall policy

**Policies as the kernel sees them** (group 100004 holds the regular IPv4 policies)
```fortios
diagnose firewall iprope list <group>
```
Ex.: `diagnose firewall iprope list 100004 | grep -A 5 "index=956"`

🟡 **Move a rule**
```fortios
config firewall policy
    move <id> before <reference-id>
end
```
Ex.: `move 956 before 905`

---

## 🔐 VPN

### IPsec (site-to-site)

🟡 **Debug IKE negotiation**
```fortios
diagnose debug disable
diagnose debug reset
diagnose vpn ike log filter clear
diagnose vpn ike log filter dst-addr4 <local-ip>
diagnose vpn ike log filter rem-addr4 <peer-ip>
diagnose debug application ike -1
diagnose debug console timestamp enable
diagnose debug enable
```
Ex.:
```fortios
diagnose debug disable
diagnose debug reset
diagnose vpn ike log filter clear
diagnose vpn ike log filter dst-addr4 200.200.200.1
diagnose vpn ike log filter rem-addr4 201.55.34.141
diagnose debug application ike -1
diagnose debug console timestamp enable
diagnose debug enable
```

**Check whether phase 1 is up**
```fortios
diagnose vpn ike gateway list name "<vpn-name>"
```
Ex.: `diagnose vpn ike gateway list name "VPN_DC-01"`

**Tunnel summary** (phase 1 and phase 2)
```fortios
get vpn ipsec tunnel summary | grep -i -f "<vpn-name>"
```
Ex.: `get vpn ipsec tunnel summary | grep -i -f "VPN_DC-01"`

**Phase 2 SAs, selectors and counters**
```fortios
diagnose vpn tunnel list name "<vpn-name>"
```
Ex.: `diagnose vpn tunnel list name "VPN_DC-01"`

**Tunnel count per VDOM**
```fortios
diagnose vpn tunnel stat
```
Ex.: `diagnose vpn tunnel stat` right after a change, to see if the tunnel count dropped.

🔴 **Reset phase 1** (renegotiates the whole tunnel)
```fortios
diagnose vpn ike gateway clear name "<vpn-name>"
```
Ex.: `diagnose vpn ike gateway clear name "VPN_DC-01"`

> [!WARNING]
> **Drops the tunnel.** Traffic over it stops until it renegotiates.

🔴 **Flush phase 2 SAs**
```fortios
diagnose vpn tunnel flush "<vpn-name>"
```
Ex.: `diagnose vpn tunnel flush "VPN_DC-01"`

> [!WARNING]
> **Tears down the phase 2 SAs.** Traffic over the tunnel stops until they are renegotiated.

### SSL-VPN

🟡 **General debug**
```fortios
diagnose debug reset
diagnose debug application fnbamd 255
diagnose debug enable
```
Ex.: add `diagnose vpn ssl debug-filter src-addr4 189.40.12.7` before `enable` to follow one user only.

**List logged-in users**
```fortios
execute vpn sslvpn list
```
Ex.: `execute vpn sslvpn list`, then pick the index to drop with `execute vpn sslvpn del-tunnel <index>`.

🔴 **Drop every SSL-VPN session**
```fortios
execute vpn sslvpn del-all
```
Ex.: `execute vpn sslvpn del-all` after changing the SSL-VPN portal, so users reconnect with the new settings.

> [!WARNING]
> **Disconnects every SSL-VPN user** at once.

**Live monitor** (user, source IP, tunnel IP, duration)
```fortios
get vpn ssl monitor
```
Ex.: `get vpn ssl monitor | grep -i jusantos`

**SSL-VPN statistics**
```fortios
diagnose vpn ssl statistics
```
Ex.: `diagnose vpn ssl statistics | grep -i max`

---

## 👤 Authentication

### Local and SSL-VPN users

🟡 **Local user and password debug**
```fortios
diagnose debug reset
diagnose debug console timestamp enable
diagnose debug application fnbamd -1
diagnose debug application authd -1
diagnose debug application sslvpn -1
diagnose debug enable
```
Ex.: add `diagnose vpn ssl debug-filter src-addr4 <user-ip>` before `enable` to keep only one user in the output.

🟡 **SSL-VPN with SAML**
```fortios
diagnose vpn ssl debug-filter src-addr4 <user-ip>
diagnose debug application samld -1
diagnose debug application sslvpn -1
diagnose debug enable
```
Ex.: `diagnose vpn ssl debug-filter src-addr4 189.40.12.7`

### SAML

🟡 **SAML authentication debug**
```fortios
diagnose debug application httpsd -1
diagnose debug application samld -1
diagnose debug console timestamp enable
diagnose debug enable
```
Ex.: run the block, log in through the IdP once, and look for the `samld` assertion lines.

### FSSO

🟡 **FSSO collector status**
```fortios
diagnose debug enable
diagnose debug authd fsso server-status
```
Ex.: `diagnose debug authd fsso server-status | grep -i connected`

### FortiClient EMS

🟡 **EMS connector debug** (fabric connector to EMS)
```fortios
diagnose debug application fcnacd -1
diagnose debug console timestamp enable
diagnose debug enable
```
Ex.: run it and then hit "Refresh" on the EMS connector in the GUI.

**Verify the EMS certificate and connection**
```fortios
execute fctems verify <ems-name>
```
Ex.: `execute fctems verify EMS01`

---

## 🌐 DNS

> [!NOTE]
> FortiGate has **no `nslookup`**. Resolve names with `execute ping` and check the resolver with the `dnsproxy` commands.

**DNS servers in use**
```fortios
show system dns
```
Ex.: `show system dns | grep primary`

**Test name resolution**
```fortios
execute ping <fqdn>
```
Ex.: `execute ping update.fortiguard.net`

🟡 **DNS proxy debug**
```fortios
diagnose debug application dnsproxy -1
diagnose debug console timestamp enable
diagnose debug enable
```
Ex.: run the block and then `execute ping www.fortinet.com` from another session.

**DNS proxy test menu** (lists every option)
```fortios
diagnose test application dnsproxy <option>
```
Ex.: `diagnose test application dnsproxy 1` clears the DNS cache.

**FQDN address cache**
```fortios
diagnose test application dnsproxy 6
```
Ex.: `diagnose test application dnsproxy 6 | grep -i microsoft`

---

## 📬 DHCP

🟡 **DHCP server debug**
```fortios
diagnose debug reset
diagnose debug application dhcps -1
diagnose debug enable
```
Ex.: run it and then `ipconfig /renew` on the client to watch the DISCOVER/OFFER/REQUEST/ACK exchange.

**Stop the debug**
```fortios
diagnose debug reset
diagnose debug disable
```
Ex.: run it once the client has its lease.

---

## 🚫 Web filter and security profiles

### FortiGuard

**FortiGuard rating servers and connectivity**
```fortios
diagnose debug rating
execute ping service.fortiguard.net
execute ping update.fortiguard.net
execute ping guard.fortinet.net
```
Ex.: `diagnose debug rating | grep -i -A 3 service`

**Web filter service status**
```fortios
get webfilter status
```
Ex.: `get webfilter status | grep -i "license"`

🟡 **Update debug** (signature and database downloads)
```fortios
diagnose debug application update -1
diagnose debug enable
execute update-now
```
Ex.: run the block and watch for `upd_comm` lines that end in an error.

### Web filter logs

**Web filter category in the logs**
```fortios
execute log filter category utm-webfilter
execute log display
```
Ex.: `execute log filter category utm-webfilter` followed by `execute log filter field srcip 10.211.12.68`, then `execute log display`.

### URL filter

**Find an FQDN in firewall objects**
```fortios
diagnose firewall fqdn list | grep -i -f <domain>
```
Ex.: `diagnose firewall fqdn list | grep -i -f youtube`

**Show web filter profiles**
```fortios
show webfilter profile [<profile>]
```
Ex.: `show webfilter profile "default"`

🟡 **Create a URL filter table**

`edit 0` takes the next free ID. Use `set type wildcard` when the URL has `*`; an exact URL keeps the default type.
```fortios
config webfilter urlfilter
    edit 0
        set name "<table-name>"
        config entries
            edit 1
                set url "<url>"
                set type wildcard
            next
        end
    next
end
```
Ex. (RAVPN-WEB-PROF table):
```fortios
config webfilter urlfilter
    edit 0
        set name "RAVPN-WEB-PROF"
        config entries
            edit 1
                set url "*.qualys.com"
                set type wildcard
            next
            edit 2
                set url "*.santandergatewayft.com.br"
                set type wildcard
            next
            edit 3
                set url "download.docker.com"
                set type wildcard
            next
            edit 4
                set url "*.redhat.com"
                set type wildcard
            next
            edit 5
                set url "reinf.receita.fazenda.gov.br"
            next
            edit 6
                set url "preprodefdreinf.receita.fazenda.gov.br"
            next
            edit 7
                set url "maps.googleapis.com"
                set type wildcard
            next
            edit 8
                set url "totvs.logistica.totvs.app"
                set type wildcard
            next
            edit 9
                set url "gefcoindustria.logistica.totvs.app"
                set type wildcard
            next
            edit 10
                set url "admin.rac.totvs.app"
                set type wildcard
            next
            edit 11
                set url "40.121.71.32"
            next
            edit 12
                set url "*argoit.com.br*"
                set type wildcard
            next
            edit 14
                set url "*api.unidas.com.br*"
                set type wildcard
            next
            edit 15
                set url "*multirio.com.br*"
                set type wildcard
            next
        end
    next
end
```

**Check the generated ID**
```fortios
show webfilter urlfilter [<id>]
```
Ex.: `show webfilter urlfilter 3`

🟡 **Attach the table to a profile**
```fortios
config webfilter profile
    edit "<profile>"
        config web
            set urlfilter-table <table-id>
        end
    next
end
```
Ex.: `edit "RAVPN-WEB"` with `set urlfilter-table 3`

### Security profile engines

**AV, IPS and app control engine and signature versions**
```fortios
diagnose autoupdate versions
```
Ex.: `diagnose autoupdate versions | grep -i -A 4 "IPS Attack Engine"`

---

## 📝 Logging

🟡 **Logging daemon debug** (miglogd handles disk, FortiAnalyzer and syslog output)
```fortios
diagnose debug application miglogd -1
diagnose debug enable
```
Ex.: run it while `diagnose log test` fires, to see where each log goes.

**Syslog server settings**
```fortios
get log syslogd setting
```
Ex.: `get log syslogd setting | grep -i server`

**Generate test logs**
```fortios
diagnose log test
```
Ex.: `diagnose log test`, then check that the entries show up on the syslog server or FortiAnalyzer.

**Filter logs by field before displaying**
```fortios
execute log filter field <field> <value>
execute log display
```
Ex.: `execute log filter field dstport 443`

**Reset log filters**
```fortios
execute log filter reset
```
Ex.: `execute log filter reset` before building a new filter, so old fields don't linger.

---

## 🔁 HA (high availability)

**HA status**
```fortios
get system ha status
```
Ex.: `get system ha status | grep -i primary`

**Jump to another cluster member**
```fortios
execute ha manage <member-id> <user>
```
Ex.: `execute ha manage 0 jusantos`

🟡 **Set node priority**
```fortios
execute ha set-priority <serial> <priority>
```
Ex.: `execute ha set-priority FG4H0E5819900156 151`

🔴 **Reset HA uptime** (forces a new election)
```fortios
diagnose sys ha reset-uptime
```
Ex.: `diagnose sys ha reset-uptime` on the primary when `override` is off and you need the other node to take over.

> [!CAUTION]
> **Can trigger a failover.** Do it in a **maintenance window**.

**Config checksum on every member** (out-of-sync check)
```fortios
diagnose sys ha checksum cluster
```
Ex.: `diagnose sys ha checksum cluster | grep -i all`

**Recalculate checksums**
```fortios
diagnose sys ha checksum recalculate
```
Ex.: `diagnose sys ha checksum recalculate`, then run `checksum cluster` again.

**HA state per virtual cluster**
```fortios
diagnose sys ha dump-by vcluster
```
Ex.: `diagnose sys ha dump-by vcluster | grep -i state`

---

## 📶 SD-WAN and link monitor

🟡 **Create a link monitor probe**

`gateway-ip` is the next hop; `server` is the IP being tested.
```fortios
config system link-monitor
    edit "<probe-name>"
        set srcintf <interface>
        set gateway-ip <next-hop>
        set server <target-ip>
    next
end
```
Ex.:
```fortios
config system link-monitor
    edit "ProbeMPLS"
        set srcintf port1
        set gateway-ip 10.112.36.194
        set server 10.41.122.2
    next
end
```

**Link monitor state**
```fortios
diagnose sys link-monitor status [<probe-name>]
```
Ex.: `diagnose sys link-monitor status ProbeMPLS`

**SD-WAN members**
```fortios
diagnose sys sdwan member
```
Ex.: `diagnose sys sdwan member | grep -i port1`

**SD-WAN health checks** (latency, jitter, loss per member)
```fortios
diagnose sys sdwan health-check [<name>]
```
Ex.: `diagnose sys sdwan health-check "Default_DNS"`

**SD-WAN rules and the member each one picked** (**`service` before 7.4.4**)
```fortios
diagnose sys sdwan service4 [<rule-id>]
```
Ex.: `diagnose sys sdwan service4 1`

---

## 🤖 Automation stitch

🟡 **Debug a stitch that doesn't fire**
```fortios
diagnose debug reset
diagnose debug application autod -1
diagnose debug enable
```
Ex.: start the block, then cause the event the trigger waits for (an admin login, for instance).
> Reference: [Fortinet Community: how to check why automation stitch is not working](https://community.fortinet.com/fortigate-3/technical-tip-how-to-check-why-automation-stitch-is-not-working-as-expected-92733)

---

## 🧩 REST API

🔴 **Show an IPsec PSK in plain text (phase1-interface)**

You must be logged in to the GUI in the **same browser**.

> [!CAUTION]
> This **reveals the pre-shared key in clear text**. Don't paste the output into tickets or chats, and clear it from the screen and browser history afterwards.

```
https://<firewall-ip>:<admin-port>/api/v2/cmdb/vpn.ipsec/phase1-interface/<vpn-name>?plain-text-password=1
```
Ex.:
```
https://192.168.253.253:11443/api/v2/cmdb/vpn.ipsec/phase1-interface/VPN_DC-01?plain-text-password=1
https://192.168.1.99:11443/api/v2/cmdb/vpn.ipsec/phase1-interface/VPN_DC-WCS?plain-text-password=1
```
