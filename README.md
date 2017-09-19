# dhcpv6

## Apuntes sobre dhcpv6

En el server: `/etc/radvd.conf`


```bash
interface eth2 {
  AdvSendAdvert on;
  MaxRtrAdvInterval 30;
  AdvManagedFlag on;
  AdvOtherConfigFlag on;

	prefix 2001:470:ccba:1::/64  
	{
	       AdvAutonomous on;
                AdvOnLink on;
		AdvRouterAddr on;

	};
};
```

En `/etc/default/isc-dhcp-server`:

```bash
....
INTERFACESv6="eth2"
```

En `dhcpv6.conf`:

```bash
subnet6 2001:470:ccba:1::/64 {
	range6 2001:470:ccba:1::10 2001:470:ccba:1::11;
	range6 2001:470:ccba:1:: temporary;
	prefix6 2001:470:ccba:1000:: 2001:470:ccba:2000:: /64;
	option dhcp6.name-servers 3ffe:501:ffff:100:200:ff:fe00:3f3e;
	option dhcp6.domain-search "test.example.com","example.com";
}
```

En `/etc/network/interfaces`:

```bash
iface eth2 inet6 static
	address 2001:470:ccba:1::1
	netmask 64
```

En el dhcpv6-pd

En `/etc/network/interfaces`

```bash
iface eth2 inet6 dhcp
```

En `/etc/nibbler/client.conf`

```bash
script "/etc/dibbler/client-notify.sh"

...
face eth1 {
# ask for address
ia
pd

.....
```

En client-notify.sh:

```bash
#!/bin/bash

# for each downlink interface...
ifn=0
for iface in $DOWNLINK_PREFIX_IFACES; do
    ifn=$(( $ifn + 1 ))
    prefix=`printenv PREFIX$ifn`
    prefixlen=`printenv PREFIX${ifn}LEN`

    # remove all global ipv6 addresses on the iface
    ip -6 addr flush dev $iface scope global

    if [ "$1" == "add" -o "$1" == "update" ]; then
        # set the first IP from the prefix
        ip addr add "${prefix}1/${prefixlen}" dev $iface
    fi
done
```
---
```bash
sysctl net.ipv6.conf.eth1.accept_ra=2
sysctl -w net.ipv6.conf.all.forwarding=1
```
