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

In OpenWRT esiste gia' anche un concetto fondamentale: il **bridge**. Un bridge e' uno "switch software" interno al router che unisce porte fisiche, VLAN e interfacce logiche in un unico punto di gestione. Per questo nella guida compaiono nomi come `br-lan`, `br-home` o `br-iot`.

Questi bridge sono spesso **gia' preconfigurati** perche' OpenWRT, soprattutto sui dispositivi moderni con DSA, crea di default un bridge LAN di base (`br-lan`) per far funzionare subito le porte LAN come un'unica rete locale. Non e' un dettaglio opzionale: e' parte del modo in cui OpenWRT organizza internamente la rete.

Per questo motivo i bridge **non vanno creati a mano alla cieca**: nella maggior parte dei casi bisogna partire da quelli gia' presenti e modificarli o associarli correttamente alle VLAN. Creare bridge manuali inutili o duplicati puo' portare a configurazioni incoerenti, perdita di connettivita', porte finite nel bridge sbagliato o conflitti con `br-lan` gia' esistente.

## Perche' i passaggi della guida vanno fatti in quest'ordine?

### 1. Prima si configura lo switch fisico (VLAN sui dispositivi fisici)

Lo switch integrato nel router deve sapere quali porte appartengono a quale VLAN. Questo e' il livello piu' basso: hardware. Senza questo passaggio, i pacchetti non vengono instradati correttamente a livello fisico.

> Se salti questo passaggio: i dispositivi collegati via cavo non vengono assegnati alla VLAN giusta.

### 2. Poi si creano le interfacce logiche

OpenWRT deve avere un'**interfaccia software** per ciascuna VLAN: e' come dire al sistema operativo "questa VLAN esiste, le assegno l'indirizzo IP X.X.X.1, gestisco il suo DHCP da qui". Ogni interfaccia corrisponde a un bridge (es. `br-home`, `br-iot`) che fa da "punto di accesso" del router verso quella VLAN.

In pratica non si sta "inventando da zero" la rete del router: si stanno collegando correttamente le VLAN ai bridge che OpenWRT usa gia' come struttura interna. Ecco perche' in questa guida si rimuove prima la LAN dal bridge di default e poi si riassegna tutto in modo ordinato, invece di creare bridge manuali senza criterio.

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
