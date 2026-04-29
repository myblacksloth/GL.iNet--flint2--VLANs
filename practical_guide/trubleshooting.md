- [Comandi utili](#comandi-utili)
    - [DHCP](#dhcp)
    - [Check regole firewall](#check-regole-firewall)
    - [Aggiunta regole via ssh](#aggiunta-regole-via-ssh)
    - [Fix regole via ssh](#fix-regole-via-ssh)


## Firewall

> [!CAUTION]
> Le regole vengono interpretate in ordine, dalla prima all'ultima. Quindi un *allow* specifico deve essere posto prima di un *deny* generico

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Le 2 VLAN non comunicano --> aggiunta di regole manuali |  ![](stuff/i/SCR-20260420-mdsk.png) |
|   |  ![](stuff/i/SCR-20260420-mdxz.png) |
|   |  ![](stuff/i/SCR-20260420-mgkr.png) |
<!-- |   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) | -->

## Traffico intra-vlan non funziona

Se il traffico tra zone non funziona, allora (esempio):

1. forwarding da server a iot  (ABILITARE FORWARD DA ZONA A ZONA nelle regole della zona)
2. rule ACCEPT from 192.168.8.30 to 192.168.30.21  (permette la connessione tra i due device dalle traffic rules)
3. rule ACCEPT from 192.168.30.21 to 192.168.8.30  (permette il ritorno dei pacchtti tramite le traffic rules)
4. rule REJECT from server to iot  (blocca tutto il resto dalle traffic rules)
5. rule REJECT from iot to server  (blocca tutto il resto dalle traffic rules)

Fonti usate per giungere a questa conclusione:

- [https://oldwiki.archive.openwrt.org/doc/uci/firewall](https://oldwiki.archive.openwrt.org/doc/uci/firewall "https://oldwiki.archive.openwrt.org/doc/uci/firewall")
- [https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples](http://https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples "https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples")
- [https://forum.openwrt.org/t/firewall-zones-forwards-and-rules/25197](http://https://forum.openwrt.org/t/firewall-zones-forwards-and-rules/25197 "https://forum.openwrt.org/t/firewall-zones-forwards-and-rules/25197")
[https://forum.openwrt.org/t/firewall-traffic-rules-override-zone-settings/150238](http://https://forum.openwrt.org/t/firewall-traffic-rules-override-zone-settings/150238 "https://forum.openwrt.org/t/firewall-traffic-rules-override-zone-settings/150238")
[https://forum.openwrt.org/t/how-to-set-up-firewall-rules-zones-correctly-for-vlans/105686](http://https://forum.openwrt.org/t/how-to-set-up-firewall-rules-zones-correctly-for-vlans/105686 "https://forum.openwrt.org/t/how-to-set-up-firewall-rules-zones-correctly-for-vlans/105686")
[https://forum.archive.openwrt.org/viewtopic.php?id=72338](https://forum.archive.openwrt.org/viewtopic.php?id=72338http:// "https://forum.archive.openwrt.org/viewtopic.php?id=72338")

## (luci rotto) Aggiungere regola per server->iot

```bash
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='server'
uci set firewall.@forwarding[-1].dest='iot'
uci commit firewall
/etc/init.d/firewall restart
```

## Luci corrotto

```
/usr/lib/lua/luci/dispatcher.lua:230: /etc/config/luci seems to be corrupt, unable to find section 'main'
stack traceback:
	[C]: in function 'assert'
	/usr/lib/lua/luci/dispatcher.lua:230: in function 'dispatch'
	/usr/lib/lua/luci/dispatcher.lua:127: in function </usr/lib/lua/luci/dispatcher.lua:126>
```

soluzione (da ssh sul router):

```bash
ls /etc/init.d/ | grep -E "nginx|httpd|luci"
rm -f /etc/config/luci
# /etc/init.d/uhttpd restart
/etc/init.d/nginx restart
```
```bash
cp /etc/config/luci /etc/config/luci.bak 2>/dev/null

cat > /etc/config/luci <<'EOF'
config core main
	option lang auto
	option mediaurlbase /luci-static/bootstrap
	option resourcebase /luci-static/resources
	option ubuspath /ubus/

config extern flash_keep
	option uci "/etc/config/"
	option dropbear "/etc/dropbear/"
	option openvpn "/etc/openvpn/"
	option passwd "/etc/passwd"
	option opkg "/etc/opkg.conf"
	option firewall "/etc/firewall.user"
	option uploads "/lib/uci/upload/"

config internal languages

config internal sauth
	option sessionpath "/tmp/luci-sessions"
	option sessiontime 3600

config internal ccache
	option enable 1

config internal themes

config internal apply
	option rollback 90
	option holdoff 4
	option timeout 5
	option display 1.5
EOF
```

```bash
/etc/init.d/uhttpd restart
rm -f /tmp/luci-indexcache /tmp/luci-modulecache/*

rm -f /tmp/luci-indexcache
rm -rf /tmp/luci-modulecache/*
```

```bash
uci show luci
uci show luci.main
ubus call uci get '{"config":"luci","section":"main"}'
```

```bash
/etc/init.d/ubus restart
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```

# Comandi utili

### DHCP

```bash
# Vedi tutti i lease DHCP attivi
cat /tmp/dhcp.leases

# Oppure con più dettagli (MAC, IP, hostname)
cat /tmp/dhcp.leases | awk '{print $3, $4, $2}'

# Vedi anche i dispositivi che rispondono sulla rete
arp -a
```

### Check regole firewall

```bash
# Vedi tutte le regole firewall con nome
uci show firewall | grep -E "name|src_ip|dest_ip|target"
```

Visualizzare solo le regole di forwarding:

```bash
uci show firewall | grep -n "forwarding"
```

### Aggiunta regole via ssh

```bash
# Da server (Pi5) verso iot (cam)
uci add firewall rule
uci set firewall.@rule[-1].name='server-to-iot'
uci set firewall.@rule[-1].src='server'
uci set firewall.@rule[-1].dest='iot'
uci set firewall.@rule[-1].src_ip='192.168.8.30'
uci set firewall.@rule[-1].dest_ip='192.168.30.21'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='any'

# Da iot (cam) verso server (Pi5)
uci add firewall rule
uci set firewall.@rule[-1].name='iot-to-server'
uci set firewall.@rule[-1].src='iot'
uci set firewall.@rule[-1].dest='server'
uci set firewall.@rule[-1].src_ip='192.168.30.21'
uci set firewall.@rule[-1].dest_ip='192.168.8.30'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='any'

# Salva e applica
uci commit firewall
/etc/init.d/firewall restart
```

### Fix regole via ssh

```bash
uci set firewall.@rule[21].dest_ip='192.168.30.21'
uci commit firewall
/etc/init.d/firewall restart
```
