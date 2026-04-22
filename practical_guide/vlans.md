
- [Il perche' di questa guida](#il-perche-di-questa-guida)
- [Passaggi che verranno illustrati](#passaggi-che-verranno-illustrati)
- [Condizioni preliminari](#condizioni-preliminari)
  - [Connessione VLAN](#connessione-vlan)
  - [Modifica indirizzo di rete di default](#modifica-indirizzo-di-rete-di-default)
- [Configurazione delle VLAN per i dispositivi fisici](#configurazione-delle-vlan-per-i-dispositivi-fisici)
  - [Rimuovere la LAN principale da br-lan di default e associarla alla VLAN principale](#rimuovere-la-lan-principale-da-br-lan-di-default-e-associarla-alla-vlan-principale)
  - [Definire le interfacce per tutte le VLAN](#definire-le-interfacce-per-tutte-le-vlan)
    - [Definizione di tutte le interfacce](#definizione-di-tutte-le-interfacce)
  - [Consentire la gestione del router dall'interfaccia home (via firewall)](#consentire-la-gestione-del-router-dallinterfaccia-home-via-firewall)
- [Configurare le reti wireless](#configurare-le-reti-wireless)
- [Regole firewall](#regole-firewall)
  - [Tutte le zone devono poter raggiungere la WAN (accesso a internet)](#tutte-le-zone-devono-poter-raggiungere-la-wan-accesso-a-internet)
  - [Abilitare DNS per tutte le zone](#abilitare-dns-per-tutte-le-zone)
  - [Avilitare DHCP per le zone](#avilitare-dhcp-per-le-zone)
  - [Regole di forwarding intra-zones](#regole-di-forwarding-intra-zones)
  - [Regole firewall custom](#regole-firewall-custom)
- [Configurazione di Avhai per mDNS intra-zones](#configurazione-di-avhai-per-mdns-intra-zones)
  - [Configurazione di Avahi](#configurazione-di-avahi)
- [Trubleshooting](#trubleshooting)
  - [Firewall](#firewall)
  - [Vecchi OpenWRT (o anche nuovi???) traffico non funziona (da confermare)](#vecchi-openwrt-o-anche-nuovi-traffico-non-funziona-da-confermare)
  - [(luci rotto) Aggiungere regola per server-\>iot](#luci-rotto-aggiungere-regola-per-server-iot)
  - [Luci corrotto](#luci-corrotto)
  - [Avahi non funziona (ancora da fixare)](#avahi-non-funziona-ancora-da-fixare)
- [Test della rete](#test-della-rete)
  - [Associazione manuale dei mac address agli indirizzi ip](#associazione-manuale-dei-mac-address-agli-indirizzi-ip)
  - [Aggiornamento delle regole firewall per ambiente realistico](#aggiornamento-delle-regole-firewall-per-ambiente-realistico)
  - [Comandi utili](#comandi-utili)
    - [DHCP](#dhcp)
    - [Check regole firewall](#check-regole-firewall)
    - [Aggiunta regole via ssh](#aggiunta-regole-via-ssh)
    - [Fix regole via ssh](#fix-regole-via-ssh)
  - [Test dei path](#test-dei-path)
  - [Test Avahi](#test-avahi)
- [Spiegazioni per i nOObs](#spiegazioni-per-i-noobs)
  - [Cos'e' una rete locale (LAN)?](#cose-una-rete-locale-lan)
  - [Cos'e' uno switch?](#cose-uno-switch)
  - [Cos'e' una VLAN?](#cose-una-vlan)
  - [Cos'e' OpenWRT e LuCI?](#cose-openwrt-e-luci)
  - [Perche' i passaggi della guida vanno fatti in quest'ordine?](#perche-i-passaggi-della-guida-vanno-fatti-in-questordine)
    - [1. Prima si configura lo switch fisico (VLAN sui dispositivi fisici)](#1-prima-si-configura-lo-switch-fisico-vlan-sui-dispositivi-fisici)
    - [2. Poi si creano le interfacce logiche](#2-poi-si-creano-le-interfacce-logiche)
    - [3. Poi si configurano le reti wireless](#3-poi-si-configurano-le-reti-wireless)
    - [4. Poi si configura il firewall](#4-poi-si-configura-il-firewall)
    - [5. Infine si configura Avahi per mDNS](#5-infine-si-configura-avahi-per-mdns)
  - [Riepilogo visivo](#riepilogo-visivo)
  - [Glossario rapido](#glossario-rapido)

# Il perche' di questa guida

Questa guida nasce dall'esigenza di avere una guida tecnico-pratica per la definizione delle VLAN sui router GL.iNet via interfaccia LuCi (direttamente via OpenWRT).
Il principale materiale a disposizione e' su YouTube ma non vi sono, al momento, guide (scritte) pratiche in lingua italiana.

> [!NOTE]  
> Nell'era dell'AI questa guida NON e' stata scritta facendo uso dell'AI.
>
> Le funzionalita' non marcate come *non funzionanti*, *incomplete* o *dubbie* sono state testate manualmente
> 
> Qualora vi dovessero essere, in futuro, paragrafi realizzati grazie all'aiuto dell'AI, questo verra' indicato esplicitamente.

# Passaggi che verranno illustrati

1. definizione delle VLAN per le interfacce fisiche
2. definizione delle VLAN tramite interfacce
3. regole firewall per il traffico intra-zones
4. configurazione di Avahi per mDNS intra-zones (nei test effettuati non ha funzionato)
5. Troubleshooting
6. Test

# Condizioni preliminari

> [!IMPORTANT]  
> Salvare senza applicare, applicare le modifiche in blocco solo alla fine di ciascun blocco
>
> La rete di gestione che si utilizza in questa fase e' quella 5GHz di default

## Connessione VLAN

SCR-20260420-ksqv

## Modifica indirizzo di rete di default

SCR-20260420-kthu

# Configurazione delle VLAN per i dispositivi fisici

|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
|  ![](stuff/i/SCR-20260420-ktwh.png) |   |
| ![](stuff/i/SCR-20260420-kvuw.png)  |   |
| ![](stuff/i/SCR-20260420-kwgs.png)  |   |

## Rimuovere la LAN principale da br-lan di default e associarla alla VLAN principale

> [!IMPORTANT]  
> Passaggio fondamentale per evitare problemi di connessione in questa fase

|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kygj.png)  | ![](stuff/i/.png)  |

## Definire le interfacce per tutte le VLAN

|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kyut.png)  | ![](stuff/i/.png)  |
| ![](stuff/i/.png)  | ![](stuff/i/.png)  |


### Definizione di tutte le interfacce

**Ripetere il procedimento per ciascuna VLAN**

| Dettagli  | Screenshot  |
| ------------ | ------------ |
| Creazione dell'interfaccia home | ![](stuff/i/SCR-20260420-kzof.png)  |
| Impostazione DHCP per l'interfaccia (eseguire prima questo perche' l'interfaccia grafica ha un bug sulle versioni vecchie) | ![](stuff/i/SCR-20260420-kzyw.png)  |
| Assegnazione IP statico per la VLAN | ![](stuff/i/SCR-20260420-laje.png)  |
| Creare nuova zona firewall per la VLAN | ![](stuff/i/SCR-20260420-lapt.png)  |
|  Risultato | ![](stuff/i/SCR-20260420-layo.png)  |

## Consentire la gestione del router dall'interfaccia home (via firewall)

> [!NOTE]  
> Ignorare le regole firewall preconfigurate e considerare solo quella in esame

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Consentire regola input da zona home |  ![](stuff/i/SCR-20260420-lcgy.png) |
|   |  ![](stuff/i/.png) |

> [!TIP]
> A questo punto e' possibile applicare le modifiche

# Configurare le reti wireless

> [!TIP]
> Il router verra' gestito dalla VLAN home quindi non creo una wlan per la VLAN di gestione (management)

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Associare la rete 5GHz di default alla VLAN home |  ![](stuff/i/SCR-20260420-ldar.png) |
| Eliminare la rete 2.4GHz di default per evitare problemi di avvio dell'interfaccia  |  ![](stuff/i/SCR-20260420-ldlv.png) |
| Creazione delle reti wireless per le VLAN  |  ![](stuff/i/SCR-20260420-ldzg.png) |
| Impostare password e tipo di sicurezza della rete  |  ![](stuff/i/SCR-20260420-lefk.png) |

> [!NOTE]  
> All'occorrenza, configurare anche le reti 5Ghz

# Regole firewall

> [!TIP]
> Considerare il rapporto ZONE > impostazioni zona (input, output, reject) > Traffic rules
>
> info aggiuntive di seguito

> [!IMPORTANT]  
> Nella sezione "*Vecchi OpenWRT (o anche nuovi???) traffico non funziona (da confermare)*" si evincono delle informazioni sulla configurazione delle zone reperite nella documentazione ufficiale e sui forum.
>
> I contenuti illustrati sono in attesa di conferma
>
> Visionare anche quella sezione per avere una panormica piu' chiara

> [!NOTE]  
> input, output, forward si riferiscono al router stesso
>
> input = dalla zona verso il router
> output = dal router verso la zona
> forward = da una zona all'altra attraversando il router

## Tutte le zone devono poter raggiungere la WAN (accesso a internet)

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Nozione preliminare: la confiurazione di base funziona per zona (zona + regole standard) - in fase inziale ragionare per colonna |  ![](stuff/i/SCR-20260420-lfdm.png) |
|  Permettere il forwarding da tutte le zone alla zona WAN |  ![](stuff/i/SCR-20260420-lfkl.png) |

## Abilitare DNS per tutte le zone
| Dettagli  | Screenshot  |
| ------------ | ------------ |
| Scheda traffic rules  |  ![](stuff/i/SCR-20260420-lgdh.png) |
| Dove creare nuove regole  |  ![](stuff/i/SCR-20260420-lghx.png) |
|  Abilitare regole DNS per tutte le zone (altrimenti DNS non funziona) |  ![](stuff/i/SCR-20260420-lgsp.png) |
| Regole DNS definite |  ![](stuff/i/SCR-20260420-lhej.png) |

## Avilitare DHCP per le zone

> [!IMPORTANT]  
> Configurare per tutte le zone VLAN

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Configurazione della regola |  ![](stuff/i/SCR-20260420-lzcm.png) |


## Regole di forwarding intra-zones

> [!TIP]
> Tenere sempre presente che il firewall di OpenWRT funziona per zona!
> 
> Quindi modificare per singola zona! Ad esempio, da iot scegliere verso chi si puo' effettuare forward
>
> Riptere la configurazione per tutte le zone fino ad ottenere il risultato desiderato

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  regole tra le zone |  ![](stuff/i/SCR-20260420-lifd.png) |
| Esempio di come effettuare le modifiche  |  ![](stuff/i/SCR-20260420-lipx.png) |

## Regole firewall custom

> [!NOTE]  
> Configurare in base alle proprie esigenze personali

> [!TIP]
> Se la confiurazione avviene per indirizzo IP e non per MAC Address allora assicurarsi che gli indirizzi IP coinvolti siano configurati in modo statico (associazione statica tra IP e MAC)

> [!IMPORTANT]  
> Per i dispositivi che devono comunicare intra-zones via mDNS bisogna configurare un servizio apposito (preferibilmente tramite Avhai)

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Dispositivo IoT che ha bisogno di comunicare con un server |  ![](stuff/i/SCR-20260420-ljsx.png) |
| Dispositivi multimediali che si trovano in IoT ma devono comunicare con home (mDNS) ... questo da solo non basta!  |  ![](stuff/i/SCR-20260420-lkky.png) |


# Configurazione di Avhai per mDNS intra-zones

> [!TIP]
> Se la connessione ssh non funziona provare con il seguente comando:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.10.1
```

| Dettagli  | Screenshot  |
| ------------ | ------------ |
| Abilitare accesso SSH da VLAN home  |  ![](stuff/i/SCR-20260420-llvs.png) |
|  Testare connessione SSH |  ![](stuff/i/SCR-20260420-lmsf.png) |
|  Cercare pacchetti Avahi |  ![](stuff/i/SCR-20260420-loat.png) |
|  Installazione di Avahi |  ![](stuff/i/SCR-20260420-logg.png) |
|  Pacchetti installati |  ![](stuff/i/SCR-20260420-lpav.png) |

## Configurazione di Avahi

File di configurazione:

`/etc/avahi/avahi-daemon.conf`

Modificare in base alle proprie zone

```ini
[server]
use-ipv4=yes
use-ipv6=no
allow-interfaces=br-home,br-iot   # le tue VLAN/bridge
ratelimit-interval-usec=1000000
ratelimit-burst=1000

[reflector]
enable-reflector=yes
reflect-ipv=no
```

Riavviare il servizio:

```bash
/etc/init.d/avahi-daemon enable
/etc/init.d/avahi-daemon restart
#
# check
ps | grep avahi
logread | grep -i avahi
```

> [!WARNING]  
> Come indicato nella sezione firewall, per il corretto funzionamento di mDNS occorre anche configurare le regole firewall!

# Trubleshooting

## Firewall

> [!CAUTION]
> Le regole vengono interpretate in ordine, dalla prima all'ultima. Quindi un *allow* specifico deve essere posto prima di un *deny* generico

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Le 2 VLAN non comunicano --> aggiunta di regole manuali |  ![](stuff/i/SCR-20260420-mdsk.png) |
|   |  ![](stuff/i/SCR-20260420-mdxz.png) |
|   |  ![](stuff/i/SCR-20260420-mgkr.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |

## Vecchi OpenWRT (o anche nuovi???) traffico non funziona (da confermare)

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

## Avahi non funziona (ancora da fixare)

```bash
# Abilita multicast su br-iot se non presente
ip link set br-iot multicast on

# Riavvia avahi
/etc/init.d/avahi-daemon restart
```

# Test della rete

## Associazione manuale dei mac address agli indirizzi ip

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Associazione manuale |  ![](stuff/i/SCR-20260420-mbvj.png) |

## Aggiornamento delle regole firewall per ambiente realistico

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Server 2 cam |  ![](stuff/i/SCR-20260420-mcvf.png) |

## Comandi utili

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

## Test dei path

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  da home a server |  ![](stuff/i/SCR-20260420-mtnf.png) |
|  da server a iot |  ![](stuff/i/SCR-20260420-mtlm.png) |
<!-- |   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) | -->

## Test Avahi

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  Regola firewall per forward |  ![](stuff/i/SCR-20260420-mvpn.png) |
| Regole firewall specifiche  |  ![](stuff/i/SCR-20260420-mver.png) |
|  Test |  ![](stuff/i/.png) |

<!--
| ![](stuff/i/.png)  | ![](stuff/i/.png)  |
|   | ![](stuff/i/.png)  |
-->

<!--
|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
| ![](stuff/i/.png)  | ![](stuff/i/.png)  |


| Dettagli  | Screenshot  |
| ------------ | ------------ |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |


-->


# Spiegazioni per i nOObs

> [!WARNING]  
> Qusta sezione e' stata realizzata grazie all'aiuto dell'AI e NON verificata dall'autore

## Cos'e' una rete locale (LAN)?

Quando colleghi piu' dispositivi (PC, telefono, smart TV, telecamere) allo stesso router, questi si trovano tutti nella stessa **rete locale (LAN)**. Per default, tutti si "vedono" tra loro: il tuo PC puo' parlare con la telecamera IoT, la smart TV puo' tentare connessioni verso il tuo NAS, etc.

Questo va bene per una rete semplice, ma e' un problema di sicurezza: se un dispositivo IoT (tipicamente poco sicuro) viene compromesso, l'attaccante ha accesso a tutta la rete.

## Cos'e' uno switch?

Uno **switch** e' il "centralino" fisico della tua rete: e' il componente (spesso integrato nel router) che riceve i dati da una porta e li smista verso la porta di destinazione corretta. Ogni dispositivo collegato via cavo ethernet e' connesso a una porta dello switch.

Senza VLAN, uno switch manda il traffico di broadcast (annunci di rete, richieste DHCP, mDNS, etc.) a **tutte** le porte: tutti i dispositivi ricevono tutto.

## Cos'e' una VLAN?

Una **VLAN (Virtual LAN)** e' una rete locale virtuale: permette di dividere logicamente uno switch fisico in piu' switch "virtuali" isolati, senza bisogno di hardware separato.

```
Senza VLAN:
[PC] [Telefono] [Telecamera IoT] [NAS]  <-- tutti nella stessa rete, si vedono tutti

Con VLAN:
VLAN home:    [PC] [Telefono]
VLAN iot:     [Telecamera IoT]
VLAN server:  [NAS]
              ^--- isolate tra loro, il firewall decide chi puo' parlare con chi
```

Ogni VLAN ha un numero identificativo (VLAN ID) che viene "taggato" sui pacchetti ethernet, cosi' lo switch sa a quale rete virtuale appartiene ciascun pacchetto.

## Cos'e' OpenWRT e LuCI?

**OpenWRT** e' un sistema operativo Linux alternativo per router, molto piu' flessibile del firmware di fabbrica. I router GL.iNet lo supportano nativamente.

**LuCI** e' l'interfaccia web grafica di OpenWRT: ti permette di configurare tutto dal browser invece di usare la riga di comando.

## Perche' i passaggi della guida vanno fatti in quest'ordine?

### 1. Prima si configura lo switch fisico (VLAN sui dispositivi fisici)

Lo switch integrato nel router deve sapere quali porte appartengono a quale VLAN. Questo e' il livello piu' basso: hardware. Senza questo passaggio, i pacchetti non vengono instradati correttamente a livello fisico.

> Se salti questo passaggio: i dispositivi collegati via cavo non vengono assegnati alla VLAN giusta.

### 2. Poi si creano le interfacce logiche

OpenWRT deve avere un'**interfaccia software** per ciascuna VLAN: e' come dire al sistema operativo "questa VLAN esiste, le assegno l'indirizzo IP X.X.X.1, gestisco il suo DHCP da qui". Ogni interfaccia corrisponde a un bridge (es. `br-home`, `br-iot`) che fa da "punto di accesso" del router verso quella VLAN.

> Se salti questo passaggio: il router non sa come gestire il traffico di quella VLAN, i dispositivi non ricevono un IP.

### 3. Poi si configurano le reti wireless

Le reti Wi-Fi (SSID) devono essere **associate** a una specifica VLAN/interfaccia. Altrimenti, tutti i dispositivi wireless finiscono nella stessa rete indipendentemente a quale Wi-Fi si connettono.

> Se salti questo passaggio: creare SSID separati non serve a niente, tutto il traffico wireless va comunque sulla stessa rete.

### 4. Poi si configura il firewall

Per default OpenWRT, una volta create le VLAN, le **isola completamente**: nessuna zona puo' comunicare con le altre. Il firewall deve essere configurato esplicitamente per:

- **Permettere l'accesso a internet (WAN)** da tutte le zone — senza questa regola i dispositivi IoT non hanno internet
- **Abilitare DNS** — il DNS e' il servizio che traduce `google.com` in un indirizzo IP. Se non e' abilitato esplicitamente per ogni zona, i siti non si aprono anche se la connessione c'e'
- **Abilitare DHCP** — il DHCP e' il servizio che assegna automaticamente un indirizzo IP ai dispositivi quando si connettono. Senza, i dispositivi rimangono senza IP
- **Definire i forwarding intra-zone** — se vuoi che il tuo PC (in `home`) possa vedere il NAS (in `server`), devi dirlo esplicitamente al firewall

> Se salti questo passaggio: internet non funziona sulle nuove VLAN, oppure alcune VLAN non possono comunicare con quelle che dovrebbero.

> [!CAUTION]
> Le regole firewall vengono lette dall'alto verso il basso. Una regola `ALLOW` specifica deve stare **prima** di un `DENY` generico, altrimenti viene ignorata e il blocco generico ha la precedenza.

### 5. Infine si configura Avahi per mDNS

**mDNS (multicast DNS)** e' il protocollo usato da dispositivi Apple (AirPlay, AirPrint), Chromecast, stampanti di rete etc. per "annunciarsi" sulla rete locale senza bisogno di un server DNS centrale.

Il problema: mDNS usa pacchetti **multicast** che per definizione non attraversano i confini di rete. Se la tua Apple TV e' nella VLAN `iot` e il tuo Mac e' nella VLAN `home`, non si vedono.

**Avahi** e' un servizio che fa da "traduttore": ascolta gli annunci mDNS su una VLAN e li "riflette" sulle altre. Deve essere configurato dopo il firewall perche' ha bisogno che le interfacce esistano gia' e che le regole firewall per il multicast siano attive.

> Se salti questo passaggio: Chromecast, AirPlay, stampanti di rete non funzionano tra VLAN diverse.

## Riepilogo visivo

```
[Dispositivo fisico / Wi-Fi]
        |
        v
[Switch fisico con VLAN tag]   <-- Passo 1: configurazione VLAN fisiche
        |
        v
[Interfaccia logica OpenWRT]   <-- Passo 2: interfacce + DHCP + IP
        |
        v
[Rete wireless associata]      <-- Passo 3: SSID -> VLAN
        |
        v
[Firewall / Zone]              <-- Passo 4: chi puo' parlare con chi
        |
        v
[Avahi per mDNS]               <-- Passo 5: servizi locali cross-VLAN
```

## Glossario rapido

| Termine | Spiegazione semplice |
|---------|----------------------|
| **LAN** | La tua rete di casa |
| **WAN** | Internet (la rete "esterna") |
| **VLAN** | Sottorete virtuale isolata logicamente |
| **Switch** | Il "centralino" fisico che smista i cavi ethernet |
| **Bridge** | Componente software che unisce piu' interfacce di rete |
| **DHCP** | Il servizio che da' automaticamente un IP ai dispositivi |
| **DNS** | Il servizio che traduce nomi (google.com) in indirizzi IP |
| **Firewall** | Il "guardiano" che decide quali connessioni sono permesse |
| **Zona (firewall)** | Un gruppo di interfacce con le stesse regole di sicurezza |
| **Forward** | Permettere al traffico di passare da una zona a un'altra |
| **mDNS** | Protocollo per scoprire dispositivi locali (Chromecast, AirPlay) |
| **Avahi** | Servizio che propaga mDNS tra VLAN diverse |
| **LuCI** | L'interfaccia web grafica di OpenWRT |
| **OpenWRT** | Sistema operativo Linux per router |


<!--

> [!NOTE]  
> Highlights information that users should take into account, even when skimming.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]  
> Crucial information necessary for users to succeed.

> [!WARNING]  
> Critical content demanding immediate user attention due to potential risks.

> [!CAUTION]
> Negative potential consequences of an action.
> 
-->