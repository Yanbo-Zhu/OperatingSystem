Paketfilter 设置好了可以实现 Firewall 的功能 


Der Kernel eines modernen Unix-Systems enthält Funktionen zur flexibel konfigurierbaren **Paketfilterung**.

IP-Pakete können damit nicht nur nach verschiedenen Kriterien gefiltert, sondern auch deren Header modifiziert werden. So können unter anderem Firewalls, Port-Weiterleitungen und [NAT](https://en.wikipedia.org/wiki/Network_address_translation) realisiert werden.

# 1 iptables und nfttables 
这些命令 是不用来看 routing table 的 


Der Linux-Kernel verwendet dafür das [**netfilter**](https://en.wikipedia.org/wiki/Netfilter)-Framework. Es stellt zwei verschiedene Backends zur Definition von Tabellen mit Filterregeln bereit:
1. Das ältere **iptables**, in dem für IPv4 und IPv6 getrennte Regel-Tabellen geführt werden.
2. Das neuere und flexiblere **nftables**, in dem Filterregeln in einen Bytecode für eine kleine interne VM übersetzt werden und keine Duplizierung von Regeln für IPv4 und IPv6 mehr erforderlich ist.

==Die Filterregeln werden mit den Kommandos `iptables` (IPv4), `ip6tables` (IPv6) und `nft` (für nftables) administriert.==
通过 这几个命令 去查看 Filterregeln 

==Aktuell existieren (noch) beide Backends im Kernel und können prinzipiell gleichzeitig verwendet werden: Ein Paket durchläuft zuerst die iptables-Regeln, dann die nftables-Regeln.== Da dies zu unerwünschten Effekten führen kann und schwer zu debuggen ist, wird davon dringend abgeraten.

Comparison: `iptables` vs `nftables`
- **Performance**: `nftables` has better performance due to consolidated rule evaluation.
- **Ease of Use**: More concise and human-readable syntax.
- **Flexibility**: Unified handling of IPv4, IPv6, ARP, and Ethernet rules in a single ruleset.


## 1.1 iptables

View Existing Rules:
- **`-L`**: Lists the rules in all chains (INPUT, OUTPUT, FORWARD).
- **`-v`**: Shows detailed information (verbose).
- **`-n`**: Displays IP addresses and ports numerically (no DNS resolution).
```
yzh@BNB23350:~$ sudo iptables -L -v -n
[sudo] password for yzh:
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

---
**Add a Rule**: Add a rule to allow traffic on a specific port (e.g., port 80 for HTTP):
`sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`
- **`-A INPUT`**: Appends the rule to the INPUT chain.
- **`-p tcp`**: Specifies the protocol (TCP in this case).
- **`--dport 80`**: Matches traffic on port 80.
- **`-j ACCEPT`**: Specifies the action (accept the traffic).

---
Delete a Rule: To delete a specific rule:
`sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT`

---

Flush All Rules: Clear all rules from all chains:
`sudo iptables -F`

---

Save Rules: Ensure the rules persist after a reboot (distribution-dependent):
`sudo iptables-save > /etc/iptables/rules.v4`


## 1.2 nftables

nftables is a powerful and modern framework for managing Linux packet filtering and firewall configurations. It replaces the older iptables framework while maintaining backward compatibility.

View Current Rules:
`sudo nft list ruleset`
This displays all the rules currently configured in nftables.

Start with a Blank Ruleset: Clear all existing rules:
`sudo nft flush ruleset`


# 2 Firewall-Tools: 设置  Firewall 

Die direkte Definition von Filterregeln mit `iptables` bzw. `nft` ist sehr flexibel, aber recht komplex und für Anwender gewöhnungsbedürftig. Daher gibt es zahlreiche **Firewall-Tools**, die dem Anwender eine einfachere Oberfläche zum Management von Firewalls bereitstellen und diese intern mit iptables- oder nftables-Regeln realisieren. In Ubuntu gibt es als Standard-Firewall-Tool [**UFW** (Uncomplicated Firewall)](https://help.ubuntu.com/community/UFW), das intern nftables verwendet.

BSD-Systeme haben andere, vom Prinzip her aber ähnliche, Paketfilter-Konzepte und Firewall-Tools, die wir hier nicht weiter betrachten.

