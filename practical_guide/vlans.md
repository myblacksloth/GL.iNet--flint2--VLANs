
- [Il perche' di questa guida](#il-perche-di-questa-guida)
- [Passaggi che verranno illustrati](#passaggi-che-verranno-illustrati)
- [Condizioni preliminari](#condizioni-preliminari)
  - [Connessione VLAN](#connessione-vlan)
  - [Modifica indirizzo di rete di default](#modifica-indirizzo-di-rete-di-default)
- [Configurazione delle VLAN per i dispositivi fisici](#configurazione-delle-vlan-per-i-dispositivi-fisici)
  - [Native VLAN](#native-vlan)
    - [Uso della native VLAN per la configurazione di eventuali server Proxmox](#uso-della-native-vlan-per-la-configurazione-di-eventuali-server-proxmox)
  - [Rimuovere la LAN principale da br-lan di default e associarla alla VLAN principale](#rimuovere-la-lan-principale-da-br-lan-di-default-e-associarla-alla-vlan-principale)
  - [Definire le interfacce per tutte le VLAN](#definire-le-interfacce-per-tutte-le-vlan)
    - [Definizione di tutte le interfacce](#definizione-di-tutte-le-interfacce)
      - [Versioni piu' vecchie](#versioni-piu-vecchie)
      - [Versioni nuove (es.21...)](#versioni-nuove-es21)
  - [Consentire la gestione del router dall'interfaccia home (via firewall)](#consentire-la-gestione-del-router-dallinterfaccia-home-via-firewall)
- [Configurare le reti wireless](#configurare-le-reti-wireless)
- [Regole firewall](#regole-firewall)
  - [Le regole della zona potrebbero avere la precedenza assoluta sulle trffic rules](#le-regole-della-zona-potrebbero-avere-la-precedenza-assoluta-sulle-trffic-rules)
  - [Tutte le zone devono poter raggiungere la WAN (accesso a internet)](#tutte-le-zone-devono-poter-raggiungere-la-wan-accesso-a-internet)
  - [Abilitare DNS per tutte le zone](#abilitare-dns-per-tutte-le-zone)
  - [Abilitare DHCP per le zone](#abilitare-dhcp-per-le-zone)
  - [Regole di forwarding intra-zones](#regole-di-forwarding-intra-zones)
  - [Regole firewall custom](#regole-firewall-custom)
- [Configurazione di Avhai per mDNS intra-zones](#configurazione-di-avhai-per-mdns-intra-zones)
  - [Configurazione di Avahi](#configurazione-di-avahi)
    - [Configurazione](#configurazione)
    - [Regole firewall per il funzionamento](#regole-firewall-per-il-funzionamento)
    - [Attivazione](#attivazione)
- [Riepilogo configurazione basilare](#riepilogo-configurazione-basilare)
  - [Eventuali regole per IoT-\>home](#eventuali-regole-per-iot-home)
- [Trubleshooting](#trubleshooting)
- [Test della rete](#test-della-rete)
  - [Test dei path](#test-dei-path)
- [Spiegazioni per i nOObs](#spiegazioni-per-i-noobs)


> [!CAUTION]
> L'autore di questa guida NON si assume alcuna responsabilita' sui danni a cose e/o persone derivanti dalla consultazione della stessa e dalla messa in opera del materiale in essa rappresentato.
> L'utente, seguendo i passaggi di seguito indicati, si assume la piena responsabilita' di eventuali danni apportati alle apparecchiature hardware sulle quali esegue modifiche rispetto allo stato originale del software

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

> [!TIP]
> Se si hanno dubbi su quello che si sta facendo, provare prima a configurare una singola VLAN
>
> (sezione [Riepilogo configurazione basilare](#riepilogo-configurazione-basilare))
>
> e poi tutte le altre.
> 
> La sezione indicata rappresenta un'implementazione basilare e funzionante di configurazione completa ma non implementa tutte le regole firewall necessarie per ottenere il livello di sicurezza desiderato

# Condizioni preliminari

> [!IMPORTANT]  
> Salvare senza applicare, applicare le modifiche in blocco solo alla fine di ciascun blocco
>
> La rete di gestione che si utilizza in questa fase e' quella 5GHz di default

## Connessione VLAN

SCR-20260420-ksqv

## Modifica indirizzo di rete di default

![](./stuff/i/SCR-20260420-kthu.png)

# Configurazione delle VLAN per i dispositivi fisici

|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
|  ![](stuff/i/SCR-20260420-ktwh.png) |  ![](stuff/i/SCR-20260422-rjkv.png) |
| | ![](stuff/i/SCR-20260422-rjwx.png) |
| ![](stuff/i/SCR-20260420-kvuw.png)  |  ![](stuff/i/SCR-20260422-rkde.png) |
| ![](stuff/i/SCR-20260420-kwgs.png)  |   |

<!--
![](stuff/i/.png)
-->

SCR-20260422-rkde

## Native VLAN

> [!WARNING]  
> Per evitare di essere escluso dalla rete, effettuare la prima configurazione, impostando una porta LAN del router come Native VLAN (`primary VLAN ID`) sulla VLAN di management

![](stuff/i/SCR-20260429-hxev.png)

### Uso della native VLAN per la configurazione di eventuali server Proxmox

Proxmox supporta molto bene le VLAN. Configurare come segue:

1. usare una porta tagged (trunk) con native vlan per installare proxmox
   1. N.B.: proxmox non supporta il tagging durante la fase di installazione
2. Attivare la funzionalita' `vlan aware`
3. configurare proxmox per utilizzare la VLAN desiderata (`server`)
4. quando si creano i container/vm specificare la vlan desiderata durante la configurazione della rete


## Rimuovere la LAN principale da br-lan di default e associarla alla VLAN principale

> [!IMPORTANT]  
> Passaggio fondamentale per evitare problemi di connessione in questa fase

![](stuff/i/SCR-20260422-rlcy.png)

|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kygj.png)  | ![](stuff/i/SCR-20260422-rkqo.png)  |

## Definire le interfacce per tutte le VLAN

|  Interfaccia vecchia | Interfaccia nuova  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kyut.png)  | ![](stuff/i/SCR-20260422-rljz.png)  |
<!-- | ![](stuff/i/.png)  |   | -->


### Definizione di tutte le interfacce

**Ripetere il procedimento per ciascuna VLAN**

#### Versioni piu' vecchie

| Dettagli  | Screenshot  |
| ------------ | ------------ |
| Creazione dell'interfaccia home | ![](stuff/i/SCR-20260420-kzof.png)  |
| Impostazione DHCP per l'interfaccia (eseguire prima questo perche' l'interfaccia grafica ha un bug sulle versioni vecchie) | ![](stuff/i/SCR-20260420-kzyw.png)  |
| Assegnazione IP statico per la VLAN | ![](stuff/i/SCR-20260420-laje.png)  |
| Creare nuova zona firewall per la VLAN | ![](stuff/i/SCR-20260420-lapt.png)  |
|  Risultato | ![](stuff/i/SCR-20260420-layo.png)  |

#### Versioni nuove (es.21...)

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  definzione dell'interfaccia |  ![](stuff/i/SCR-20260422-ronl.png) |
|   |  ![](stuff/i/SCR-20260422-rmqs.png) |
|   |  ![](stuff/i/SCR-20260422-rnue.png) |
|   |  ![](stuff/i/SCR-20260422-rnzm.png) |
|   |  ![](stuff/i/SCR-20260422-rnqe.png) |
|   |  ![](stuff/i/.png) |

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
> Considerare che il firewall di OpenWrt e' stateful
>
> info aggiuntive di seguito

> [!NOTE]  
> input, output, forward si riferiscono al router stesso
>
> input = dalla zona verso il router
> output = dal router verso la zona
> forward = da una zona all'altra attraversando il router

## Le regole della zona potrebbero avere la precedenza assoluta sulle trffic rules

> [!CAUTION]
> Su alcune versioni di OpenWRT, il sistema da la precedenza alle regole di zona!
>
> Provare a configurare le regole nelle traffic rules. Se il collegamento non funziona, seguire i passaggi seguenti:

1. abilitare forwarding dalla zona A alla zona B
2. specificare le regole specifiche (per dispositivi / traffico permesso) nelle traffic rules
3. bloccare tutto il resto del traffico (da A a B) nelle traffic rules

> Dai test effettuati, su OpenWrt 18.06
>
> senza la regola esplicita di inoltro da zona A a zona B
>
> le regole
> 
> (1) default policy forward per la zona A
> 
> (2) traffic rule specifica per device X nella vlan A verso device Y nella vlan B
>
> non sono risultate essere sufficienti per il funzionamento del traffico da A a B

Nella sezione "[Riepilogo configurazione basilare](#riepilogo-configurazione-basilare)" viene mostrata la configurazione funzionante

## Tutte le zone devono poter raggiungere la WAN (accesso a internet)

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  **Nozione preliminare**: la confiurazione di base funziona per zona (zona + regole standard) - in fase inziale ragionare per colonna |  ![](stuff/i/SCR-20260420-lfdm.png) |
|  Permettere il forwarding da tutte le zone alla zona WAN |  ![](stuff/i/SCR-20260420-lfkl.png) |

## Abilitare DNS per tutte le zone
| Dettagli  | Screenshot  |
| ------------ | ------------ |
| Scheda traffic rules  |  ![](stuff/i/SCR-20260420-lgdh.png) |
| Dove creare nuove regole  |  ![](stuff/i/SCR-20260420-lghx.png) |
|  Abilitare regole DNS per tutte le zone (altrimenti DNS non funziona) |  ![](stuff/i/SCR-20260420-lgsp.png) |
| Regole DNS definite |  ![](stuff/i/SCR-20260420-lhej.png) |

## Abilitare DHCP per le zone

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
> Su alcuni router e' necessario usare questo comando SSH:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.10.1

scp -O -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.10.1:/root/file ./
```

| Dettagli  | Screenshot  |
| ------------ | ------------ |
| Abilitare accesso SSH da VLAN home  |  ![](stuff/i/SCR-20260420-llvs.png) |
|  Testare connessione SSH |  ![](stuff/i/SCR-20260420-lmsf.png) |
|  Cercare pacchetti Avahi |  ![](stuff/i/SCR-20260420-loat.png) |
|  Installazione di Avahi (**usare versione DBUS**) |  ![](stuff/i/SCR-20260420-logg.png) |
|  Pacchetti installati |  ![](stuff/i/SCR-20260426-lbgo.png) |

## Configurazione di Avahi

> [!WARNING]  
> Come indicato nella sezione firewall, per il corretto funzionamento di mDNS occorre anche configurare le regole firewall come indicato di seguito!

### Configurazione

File di configurazione:

`/etc/avahi/avahi-daemon.conf`

Modificare in base alle proprie zone

```ini
[server]
#host-name=foo
#domain-name=local
use-ipv4=yes
use-ipv6=yes
check-response-ttl=no
use-iff-running=no
allow-interfaces=br-home,br-iot

[publish]
publish-addresses=yes
publish-hinfo=yes
publish-workstation=no
publish-domain=yes
#publish-dns-servers=192.168.1.1
#publish-resolv-conf-dns-servers=yes

[reflector]
enable-reflector=yes
reflect-ipv=no

[rlimits]
#rlimit-as=
rlimit-core=0
rlimit-data=4194304
rlimit-fsize=0
rlimit-nofile=30
rlimit-stack=4194304
rlimit-nproc=3
```

**Prestare massima attenzione a**

```ini
allow-interfaces=br-home,br-iot
enable-reflector=yes
```

### Regole firewall per il funzionamento

|  Descrizione | Screenshot  |
| ------------ | ------------ |
| Creare regola per permettere il traffico mDNS dalla zona iot al router  |  ![](./stuff/i/SCR-20260426-lcmr.png) |
| Risultato  |  ![](./stuff/i/SCR-20260426-lcwf.png) |
<!-- |   |  ![](./stuff/i/.png) |
|   |  ![](./stuff/i/.png) | -->

> [!TIP]
> Sarebbe opportuno abilitare mDNS per tutte le zone!

### Attivazione

Riavviare il servizio:

```bash
/etc/init.d/avahi-daemon enable
/etc/init.d/avahi-daemon restart
#
# check
ps | grep avahi
logread | grep -i avahi
```

# Riepilogo configurazione basilare

> [!NOTE]  
> Questa sezione rappresenta una configurazione funzionante basilare che costituisce un breve riepilogo di quanto visto finora

> [!IMPORTANT]  
> Questo e' solo un esempio, non implementa tutte le regole di sicurezza necessarie

> [!TIP]  
> Si consiglia di configurare le regole firewall come segue:
> 
> per prima cosa abilitare regole generiche di forwarding tra zone:
> per sicurezza disattivare input e forward (che verra' configurato, se ove necessario, tramite traffic rules)
> 
> 1. abilitare DNS per tutte le zone
> 2. abilitare DHCP per tutte le zone
> 3. abilitare regole specifiche per il traffico desiderato
> 4. abilitare mDNS dove necessario
> 5. disabilitare tutto il traffico da una certa zona verso le altre


|  Descrizione | Screenshot  |
| ------------ | ------------ |
| Configurazione dello switch fisico  |  ![](./stuff/i/r1/SCR-20260426-lemh.png) |
| Configurazione delle interfacce  |  ![](./stuff/i/r1/SCR-20260426-leve.png) |
| Reti wireless  |  ![](./stuff/i/r1/SCR-20260426-lfgq.png) |
| Zone firewall  |  ![](./stuff/i/r1/SCR-20260426-lftw.png) |
| Traffic rules fiewall (il ban iot->home è tecnicamente **SUPERFLUO** in quanto la zona iot già prevede il blocco del forwarding dalla configurazione delle zone firewall)  |  ![](./stuff/i/r1/SCR-20260426-lgoq.png) |
| Avahi per mDNS  |  ![](./stuff/i/r1/SCR-20260426-licz.png) |
<!-- |   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) | -->

## Eventuali regole per IoT->home

- Il traffico da home ad IoT e' abilitato di default verso ogni zona (forwarding di zona abilitato + traffico intra-vlan configurato dalle regole della zona)
- il traffico da IoT ad home non e' configurato nelle regole di zona e pertanto verra' scartato dal router (comportamento desiderato)

![](./stuff/i/SCR-20260426-qqiw.png)

In caso di problemi di connessione, considerare quanto illustrato nella sezione "[Le regole della zona potrebbero avere la precedenza assoluta sulle trffic rules](#le-regole-della-zona-potrebbero-avere-la-precedenza-assoluta-sulle-trffic-rules)" andando a definire (1) inoltro tra zone (2) regole di `allow` (3) regola di `deny`. N.B.: **questo comportamento dipende dalla versione di OpenWRT** e pertanto effettuare dei test di rete prima di definire regole superflue e/o fuorvianti

# Trubleshooting

Visionare il file `trubleshooting.md`

# Test della rete

## Test dei path

| Dettagli  | Screenshot  |
| ------------ | ------------ |
|  da home a server |  ![](stuff/i/SCR-20260420-mtnf.png) |
|  da server a iot |  ![](stuff/i/SCR-20260420-mtlm.png) |

<!-- |   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) | -->

<!-- |  Test |  ![](stuff/i/.png) | -->

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

Visionare `noobs.md`

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
