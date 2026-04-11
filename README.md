# Guida Completa: Configurazione VLAN su GL.iNet Flint 2 (OpenWrt)

> Guida passo-passo per configurare reti separate (VLAN) su un router GL.iNet Flint 2,
> incluse reti wireless dedicate e integrazione con WireGuard VPN.
>
> (C) Antonio Maulucci, 2026-04-11 20:51:27

---

## Indice

- [Guida Completa: Configurazione VLAN su GL.iNet Flint 2 (OpenWrt)](#guida-completa-configurazione-vlan-su-glinet-flint-2-openwrt)
  - [Indice](#indice)
  - [1. Concetti Fondamentali](#1-concetti-fondamentali)
    - [1.1 Cos'è una VLAN e perché usarla](#11-cosè-una-vlan-e-perché-usarla)
    - [1.2 Come funziona il tagging 802.1Q](#12-come-funziona-il-tagging-8021q)
    - [1.3 Porte Tagged e Untagged](#13-porte-tagged-e-untagged)
    - [1.4 Cos'è un Bridge e il VLAN Filtering](#14-cosè-un-bridge-e-il-vlan-filtering)
    - [1.5 DSA: la tecnologia moderna di OpenWrt](#15-dsa-la-tecnologia-moderna-di-openwrt)
    - [1.6 Zone Firewall in OpenWrt](#16-zone-firewall-in-openwrt)
  - [2. Hardware: GL.iNet Flint 2](#2-hardware-glinet-flint-2)
    - [2.1 Architettura interna](#21-architettura-interna)
    - [2.2 Nomi delle interfacce in OpenWrt](#22-nomi-delle-interfacce-in-openwrt)
  - [3. Pianificazione della Rete](#3-pianificazione-della-rete)
    - [3.1 Le VLAN che vogliamo creare](#31-le-vlan-che-vogliamo-creare)
    - [3.2 Schema degli indirizzi IP](#32-schema-degli-indirizzi-ip)
    - [3.3 Politiche di accesso tra VLAN](#33-politiche-di-accesso-tra-vlan)
    - [3.4 Assegnazione delle porte fisiche](#34-assegnazione-delle-porte-fisiche)
  - [4. Configurazione tramite LuCI (Interfaccia Web)](#4-configurazione-tramite-luci-interfaccia-web)
    - [4.1 Preparazione e backup](#41-preparazione-e-backup)
    - [4.2 Passo 1 — Abilitare il VLAN Filtering sul bridge](#42-passo-1--abilitare-il-vlan-filtering-sul-bridge)
    - [4.3 Passo 2 — Riassegnare l'interfaccia LAN principale](#43-passo-2--riassegnare-linterfaccia-lan-principale)
    - [4.4 Passo 3 — Creare le interfacce per le nuove VLAN](#44-passo-3--creare-le-interfacce-per-le-nuove-vlan)
    - [4.5 Passo 4 — Configurare le zone Firewall](#45-passo-4--configurare-le-zone-firewall)
    - [4.6 Passo 5 — Configurare le reti Wireless](#46-passo-5--configurare-le-reti-wireless)
  - [5. File di Configurazione Completi](#5-file-di-configurazione-completi)
    - [5.1 /etc/config/network](#51-etcconfignetwork)
    - [5.2 /etc/config/wireless](#52-etcconfigwireless)
    - [5.3 /etc/config/firewall](#53-etcconfigfirewall)
    - [5.4 /etc/config/dhcp](#54-etcconfigdhcp)
  - [6. Integrazione con WireGuard VPN](#6-integrazione-con-wireguard-vpn)
    - [6.1 Come WireGuard si integra con le VLAN](#61-come-wireguard-si-integra-con-le-vlan)
    - [6.2 Configurazione della zona VPN nel Firewall](#62-configurazione-della-zona-vpn-nel-firewall)
    - [6.3 Aggiornamento della configurazione del client WireGuard](#63-aggiornamento-della-configurazione-del-client-wireguard)
  - [7. Verifica e Risoluzione dei Problemi](#7-verifica-e-risoluzione-dei-problemi)
    - [7.1 Comandi di verifica via SSH](#71-comandi-di-verifica-via-ssh)
    - [7.2 Problemi comuni e soluzioni](#72-problemi-comuni-e-soluzioni)
  - [Fonti](#fonti)

---

## 1. Concetti Fondamentali

### 1.1 Cos'è una VLAN e perché usarla

Immagina di avere una casa con molte stanze. Normalmente, tutti i dispositivi connessi al tuo router si trovano nella stessa "stanza virtuale": possono vedersi e comunicare tra loro liberamente. Questo va bene per una rete semplice, ma presenta dei rischi:

- Una telecamera di sicurezza con un software vulnerabile potrebbe essere usata da un attaccante per entrare nel tuo PC.
- Un ospite connesso alla tua rete WiFi può navigare nei tuoi file condivisi.
- Un dispositivo IoT (lampadina smart, termostato) compromesso potrebbe spiare il traffico di tutta la rete.

Una **VLAN** (Virtual Local Area Network — Rete Locale Virtuale) risolve questo problema creando "stanze separate" virtuali all'interno della stessa infrastruttura fisica (gli stessi cavi, lo stesso router). I dispositivi in VLAN diverse non possono comunicare tra loro, a meno che tu non lo consenta esplicitamente attraverso regole firewall.

Il grande vantaggio è che non hai bisogno di router e switch separati per ogni rete: tutto funziona sullo stesso hardware, separato a livello software.

**Esempi pratici di utilizzo:**
- **IoT separato**: le lampadine smart e i termostati sono isolati dal PC con i tuoi documenti.
- **Ospiti sicuri**: chi viene a casa può usare internet, ma non accedere al tuo NAS.
- **Server protetti**: il tuo NAS o server home è accessibile solo da chi deve accedervi.

---

### 1.2 Come funziona il tagging 802.1Q

Per permettere a più VLAN di viaggiare sullo stesso cavo fisico, si usa uno standard chiamato **802.1Q**. Il meccanismo è semplice: ogni pacchetto di dati che viaggia sulla rete viene "etichettato" con un numero identificativo (il **VLAN ID**, o VID) che dice a quale rete virtuale appartiene.

Questa etichetta è un piccolo header di 4 byte inserito nel pacchetto Ethernet:

```
┌─────────────────┬──────────┬─────────────────────────────────────┐
│  Header Ethernet│ Tag VLAN │          Dati del pacchetto          │
│  (14 byte)      │ (4 byte) │                                      │
└─────────────────┴──────────┴─────────────────────────────────────┘
                       │
                       └─ Contiene il VLAN ID (1-4094)
```

Quando uno switch riceve un pacchetto con il tag VLAN 30, sa che appartiene alla VLAN 30 e lo consegna solo alle porte associate a quella VLAN.

---

### 1.3 Porte Tagged e Untagged

Esistono due tipi di porte su uno switch che supporta le VLAN:

**Porta Untagged (o Access Port)**
- Rimuove il tag VLAN quando invia il pacchetto al dispositivo finale.
- Aggiunge il tag della VLAN assegnata quando riceve un pacchetto non etichettato.
- Si usa per collegare dispositivi normali (PC, smartphone, stampanti) che non capiscono le VLAN.
- Il dispositivo finale non sa nemmeno di essere su una VLAN specifica.

**Porta Tagged (o Trunk Port)**
- Lascia il tag VLAN intatto sui pacchetti.
- Permette a più VLAN di viaggiare sullo stesso cavo.
- Si usa per collegare switch, access point, o altri router che "capiscono" le VLAN.

**Esempio visivo:**

```
Router
   │
   ├── Porta Trunk (tagged 10,20,30) ──── Managed Switch ──┬── Porta Access VLAN 10 ── PC
   │                                                        ├── Porta Access VLAN 20 ── Telecamera
   │                                                        └── Porta Access VLAN 30 ── Raspberry Pi
   │
   └── Porta Access VLAN 10 (untagged) ── PC di gestione
```

**PVID (Primary VLAN ID)**: è la VLAN di default assegnata a una porta. Quando un dispositivo invia un pacchetto senza tag su quella porta, lo switch aggiunge automaticamente il tag del PVID. Tipicamente si imposta su porte untagged.

---

### 1.4 Cos'è un Bridge e il VLAN Filtering

In Linux (e quindi OpenWrt), un **bridge** è un componente software che si comporta come uno switch virtuale. Il bridge principale di OpenWrt si chiama `br-lan` e collega tutte le porte LAN fisiche tra loro.

Il **VLAN Filtering** è la funzionalità che, quando abilitata sul bridge, permette al bridge stesso di separare il traffico in base ai tag VLAN — esattamente come farebbe uno switch fisico gestito. Senza questa funzionalità, tutte le porte del bridge si vedono liberamente.

Quando abilitiamo il VLAN filtering su `br-lan`, possiamo poi creare **interfacce virtuali** come `br-lan.10`, `br-lan.20`, ecc. Ogni interfaccia virtuale vede solo il traffico della sua VLAN.

---

### 1.5 DSA: la tecnologia moderna di OpenWrt

Il GL.iNet Flint 2 usa OpenWrt con un'architettura chiamata **DSA (Distributed Switch Architecture)**. È la modalità moderna introdotta con OpenWrt 21.02.

In pratica, DSA espone ogni porta fisica dello switch come un'interfaccia di rete separata:
- `lan1` → porta LAN 1
- `lan2` → porta LAN 2
- `lan3` → porta LAN 3
- `lan4` → porta LAN 4
- `eth1` → porta WAN

Prima di DSA (sistema "swconfig"), c'era una sola interfaccia `eth0` per tutto lo switch, e le VLAN si configuravano diversamente. **Non usare le guide che mostrano `eth0.1`, `eth0.2`, `config switch_vlan`**: quelle sono per il vecchio sistema e non funzionano sul Flint 2.

Con DSA, la configurazione VLAN si fa tramite **bridge-vlan** nella configurazione di rete.

---

### 1.6 Zone Firewall in OpenWrt

OpenWrt usa un sistema firewall basato su **zone**. Ogni zona è un gruppo di una o più interfacce di rete, e si configurano le politiche di accesso tra zone (non tra singole interfacce).

**Come funziona:**
1. Ogni interfaccia di rete viene assegnata a una zona.
2. Si configura la politica di default della zona (accetta/rifiuta traffico in entrata, uscita, inoltro).
3. Si aggiungono regole di **forwarding** per permettere il traffico tra zone specifiche.
4. Si aggiungono regole specifiche per eccezioni (es: permettere DHCP e DNS da una zona isolata).

**Politiche tipiche:**
- Zona `lan`: tutto permesso (rete fidata)
- Zona `wan`: tutto bloccato in entrata, NAT in uscita
- Zone VLAN isolate: input DROP, forward DROP, con eccezioni solo per DHCP/DNS

---

## 2. Hardware: GL.iNet Flint 2

### 2.1 Architettura interna

Il **GL.iNet Flint 2 (GL-MT6000)** è un router WiFi 6 di fascia alta basato sul chip **MediaTek MT7986AV (Filogic 830)**. Internamente ha questa architettura:

```
┌──────────────────────────────────────────────────────┐
│                  CPU MT7986AV                        │
│                                                      │
│  ┌──────────────┐    ┌──────────────────────────┐   │
│  │   eth1       │    │   eth0 (CPU port)         │   │
│  │ (2.5 GbE     │    │         │                 │   │
│  │  diretto)    │    │   Switch MT7531AE          │   │
│  └──────┬───────┘    │   ┌────┴──────────────┐   │   │
│         │            │   │ lan1 lan2 lan3 lan4│   │   │
│         │            │   └────────────────────┘   │   │
│         │            └──────────────────────────┘   │
│         │                                            │
│  Radio 2.4 GHz (MT7976GN) — WiFi 6, 4x4 MIMO       │
│  Radio 5 GHz   (MT7976AN) — WiFi 6, 4x4 MIMO       │
└──────────────────────────────────────────────────────┘
         │                   │
       PORTA WAN           PORTE LAN 1-4
      (2.5 GbE)            (1 GbE ciascuna)
```

**Punto chiave**: tutte e 4 le porte LAN fisiche sono collegate tramite lo switch interno, che comunica con la CPU attraverso `eth0`. Questo significa che tutto il traffico tra le porte LAN e la CPU passa attraverso questa connessione interna, e il VLAN tagging è il meccanismo che separa i diversi flussi.

**Specifiche tecniche:**
| Componente | Dettaglio |
|---|---|
| CPU | MediaTek MT7986AV, quad-core ARM Cortex-A53 @ 2 GHz |
| RAM | 1 GB DDR4 |
| Storage | 8 GB eMMC |
| Switch interno | MediaTek MT7531AE |
| Porta WAN | 2.5 GbE (Realtek RTL8221B) |
| Porte LAN | 4x 1 GbE |
| WiFi 2.4 GHz | 802.11ax (WiFi 6), 4x4 MIMO |
| WiFi 5 GHz | 802.11ax (WiFi 6), 4x4 MIMO |

---

### 2.2 Nomi delle interfacce in OpenWrt

In OpenWrt sul Flint 2, i nomi delle interfacce sono:

| Nome OpenWrt | Cosa rappresenta |
|---|---|
| `eth1` | Porta WAN fisica (2.5 GbE) |
| `lan1` | Porta LAN 1 (1 GbE) |
| `lan2` | Porta LAN 2 (1 GbE) |
| `lan3` | Porta LAN 3 (1 GbE) |
| `lan4` | Porta LAN 4 (1 GbE) |
| `br-lan` | Bridge virtuale che raggruppa lan1–lan4 |
| `br-lan.10` | Interfaccia virtuale per VLAN 10 sul bridge |
| `radio0` | Radio WiFi 2.4 GHz |
| `radio1` | Radio WiFi 5 GHz |
| `wg0` | Interfaccia WireGuard VPN |

---

## 3. Pianificazione della Rete

Prima di toccare qualsiasi impostazione, è fondamentale pianificare la rete su carta. Modifiche non pianificate possono isolarti dal router.

### 3.1 Le VLAN che vogliamo creare

| VLAN ID | Nome | Scopo | Dispositivi tipici |
|---|---|---|---|
| **10** | **Untagged / Management** | Rete principale, accesso all'admin del router | PC personale, dispositivi fidati, pannello LuCI |
| **20** | **Home** | Rete domestica generale | Smartphone, PC, tablet, smart TV |
| **30** | **IoT** | Dispositivi smart home | Lampadine smart, termostati, assistenti vocali, prese smart |
| **40** | **Security** | Videosorveglianza e allarmi | Telecamere IP, NVR, sensori allarme |
| **50** | **Server** | Server e storage | NAS, Raspberry Pi server, Home Assistant, media server |
| **60** | **Guest** | Ospiti, accesso solo internet | Smartphone e PC di visitatori |

> **Perché VLAN 10 come "untagged"?**
> La VLAN 10 sarà quella "nativa" sulla porta fisica da cui gestiamo il router. Quando colleghi il tuo PC a quella porta con un cavo normale, il traffico non ha tag e viene automaticamente associato alla VLAN 10 (che è il Management). Le altre VLAN richiedono un dispositivo che capisce i tag VLAN (come un access point o uno switch managed).

---

### 3.2 Schema degli indirizzi IP

| VLAN | Rete | Gateway (IP router) | Range DHCP | Broadcast |
|---|---|---|---|---|
| 10 – Management | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.100 – .250 | 192.168.10.255 |
| 20 – Home | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.100 – .250 | 192.168.20.255 |
| 30 – IoT | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.100 – .250 | 192.168.30.255 |
| 40 – Security | 192.168.40.0/24 | 192.168.40.1 | 192.168.40.100 – .200 | 192.168.40.255 |
| 50 – Server | 192.168.50.0/24 | 192.168.50.1 | 192.168.50.10 – .50 | 192.168.50.255 |
| 60 – Guest | 192.168.60.0/24 | 192.168.60.1 | 192.168.60.100 – .200 | 192.168.60.255 |
| VPN WireGuard | 10.100.0.0/24 | 10.100.0.1 | Assegnati manualmente | 10.100.0.255 |

> **Nota sulla subnet `/24`**: significa che la rete ha 254 indirizzi disponibili per i dispositivi (da .1 a .254, dove .1 è il gateway e .255 è il broadcast). È la scelta più comune per reti domestiche.

---

### 3.3 Politiche di accesso tra VLAN

Questa tabella mostra chi può comunicare con chi. Leggi le righe come "sorgente" e le colonne come "destinazione":

| Da \ Verso | WAN | Mgmt (10) | Home (20) | IoT (30) | Security (40) | Server (50) | Guest (60) |
|---|---|---|---|---|---|---|---|
| **Mgmt (10)** | ✅ Internet | — | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Home (20)** | ✅ Internet | ❌ | — | ❌ | ❌ | ✅ Solo server | ❌ |
| **IoT (30)** | ✅ Internet | ❌ | ❌ | — | ❌ | ❌ | ❌ |
| **Security (40)** | ❌ | ❌ | ❌ | ❌ | — | ✅ NVR/storage | ❌ |
| **Server (50)** | ❌ | ❌ | ❌ | ❌ | ❌ | — | ❌ |
| **Guest (60)** | ✅ Internet | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| **VPN (wg0)** | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ Accesso completo | ❌ |

**Spiegazione delle scelte:**
- **Management** ha accesso a tutto: è la rete fidata del'amministratore.
- **Home** può accedere ai server (NAS, media server) ma non alle altre VLAN.
- **IoT** può accedere a internet (per aggiornamenti, cloud) ma è completamente isolata dalla rete domestica.
- **Security** non ha accesso a internet (le telecamere non devono "chiamare casa") ma può raggiungere un eventuale NVR/storage nella VLAN Server.
- **Server** non ha accesso a nulla in uscita: i server rispondono a chi li chiama, non iniziano connessioni.
- **Guest** ha solo internet.
- **VPN** può accedere solo alla VLAN Server: chi si connette da remoto tramite VPN può usare i server ma non entra nella rete domestica.

> **Queste politiche sono esempi** e possono essere adattate alle tue esigenze. La configurazione firewall nella sezione 5 implementa esattamente questo schema.

---

### 3.4 Assegnazione delle porte fisiche

Nella configurazione che creeremo, tutte le porte LAN fisiche saranno **trunk port** (tagged per tutte le VLAN). Questo le rende universali: puoi collegare uno switch managed o un access point a qualsiasi porta e gestire tutte le VLAN.

Se vuoi invece una porta "access" (untagged) per un dispositivo specifico, la tabella seguente mostra come personalizzarla — ma nella configurazione base le lasceremo tutte come trunk:

| Porta fisica | Modalità suggerita | Uso tipico |
|---|---|---|
| LAN 1 | Trunk (tutte le VLAN tagged) | Switch managed, access point aggiuntivo |
| LAN 2 | Trunk (tutte le VLAN tagged) | Switch managed, access point aggiuntivo |
| LAN 3 | Trunk o Access VLAN 10 | PC di gestione (se vuoi accesso diretto senza trunk) |
| LAN 4 | Trunk o Access VLAN 50 | NAS o server diretto (accesso senza switch managed) |
| WAN | Solo WAN | Collegamento al modem/internet |

---

## 4. Configurazione tramite LuCI (Interfaccia Web)

LuCI è l'interfaccia web di OpenWrt. Si accede aprendo un browser e andando su `http://192.168.1.1` (o l'IP attuale del router).

> **ATTENZIONE CRITICA**: Alcune modifiche possono farti perdere l'accesso al router. Leggi le avvertenze in ogni passo prima di procedere. OpenWrt ha un meccanismo di sicurezza: se clicchi "Salva e Applica", hai circa **90 secondi** per confermare le modifiche. Se non confermi (o perdi la connessione), le modifiche vengono **automaticamente annullate**. Questo è il tuo salvavita.

### 4.1 Preparazione e backup

**Prima di qualsiasi modifica, esegui un backup della configurazione attuale.**

1. Vai su **System → Backup / Flash Firmware**
2. Clicca su **Generate archive** per scaricare un file `.tar.gz` con tutta la configurazione
3. Salvalo in un posto sicuro sul tuo PC

**Installa anche `diffutils`** (utile per confrontare configurazioni):

Vai su **System → Software**, poi nella barra di ricerca cerca `diffutils` e installalo. Oppure via SSH:
```bash
opkg update
opkg install diffutils
```

**Connettiti al router con un cavo fisico** dalla porta che vuoi usare come management (es. LAN 3). Questo è il dispositivo con cui stai facendo la configurazione: assicurati che rimanga accessibile durante tutto il processo.

---

### 4.2 Passo 1 — Abilitare il VLAN Filtering sul bridge

Questo è il passo fondamentale che "accende" la capacità del bridge di separare il traffico per VLAN.

1. Vai su **Network → Devices**

   Vedrai la lista dei dispositivi di rete. Trova `br-lan` e clicca su **Configure**.

2. Si apre una finestra di configurazione del bridge. Cerca la tab o sezione **"Bridge VLAN filtering"**.

3. Metti il segno di spunta su **"Enable VLAN filtering"**.

4. Ora devi aggiungere le 6 VLAN. Per ognuna clicca **"Add"** e inserisci le seguenti configurazioni:

   **VLAN 10 (Management/Untagged):**
   - VLAN ID: `10`
   - Per ogni porta (lan1, lan2, lan3, lan4):
     - Se è la porta da cui stai configurando (es. lan3): imposta **Egress untagged** e **PVID** (Primary VLAN ID). Questo la rende una porta "access" per la VLAN 10.
     - Per le altre porte: imposta **Egress tagged** (porta trunk).
   - Lascia la casella **local** spuntata (permette al router stesso di far parte di questa VLAN).

   **VLAN 20, 30, 40, 50, 60:**
   - Per ciascuna, imposta tutte le porte come **Egress tagged** (tutte trunk).
   - La casella **local** deve essere spuntata (il router deve poter "parlare" su queste VLAN per fare da gateway DHCP).

   > **Cosa significa "Egress"?**
   > "Egress" significa "in uscita". Quando un pacchetto lascia il bridge verso una porta, l'opzione egress determina se il tag VLAN viene mantenuto (tagged) o rimosso (untagged).

5. **Non cliccare ancora "Save & Apply"**. Prima completa il passo successivo.

---

### 4.3 Passo 2 — Riassegnare l'interfaccia LAN principale

L'interfaccia `lan` attuale usa `br-lan` (il bridge senza VLAN). Dobbiamo spostarla su `br-lan.10` (il bridge con VLAN 10).

> **QUESTO PASSO È CRITICO.** Se non fai questa modifica prima di applicare le impostazioni VLAN, perdi l'accesso al router perché il traffico non taggato non saprà più a quale VLAN appartiene.

1. Vai su **Network → Interfaces**
2. Trova l'interfaccia `LAN` e clicca su **Edit**
3. Nella sezione **"Device"**, cambia `br-lan` con `br-lan.10`
4. Lascia tutto il resto uguale (IP 192.168.10.1, netmask 255.255.255.0)

   > **Nota**: Stai solo spostando la gestione della rete principale dalla VLAN "piatta" a una VLAN specifica (VLAN 10). Il tuo PC collegato alla porta configurata come PVID 10 continuerà a vedere il router allo stesso indirizzo.

5. Clicca **Save** (non Save & Apply ancora).

---

### 4.4 Passo 3 — Creare le interfacce per le nuove VLAN

Per ciascuna delle 5 VLAN rimanenti (Home, IoT, Security, Server, Guest), crea una nuova interfaccia:

1. Vai su **Network → Interfaces**
2. Clicca su **"Add new interface..."**

**Interfaccia VLAN 20 (Home):**
- **Name**: `home`
- **Protocol**: Static address
- **Device**: Seleziona "Custom device..." e digita `br-lan.20`
- Clicca **Create interface**
- Nella scheda **General Settings**:
  - IPv4 address: `192.168.20.1`
  - IPv4 netmask: `255.255.255.0`
- Nella scheda **DHCP Server**: clicca **"Set up DHCP Server"**
  - Start: `100`
  - Limit: `150`
  - Leasetime: `12h`
- Nella scheda **Firewall Settings**: seleziona o crea una zona `home`
- Clicca **Save**

**Interfaccia VLAN 30 (IoT):**
- **Name**: `iot`
- **Device**: `br-lan.30`
- IPv4 address: `192.168.30.1`
- Netmask: `255.255.255.0`
- DHCP: Start `100`, Limit `150`, Leasetime `1h` (lease brevi per dispositivi IoT)
- Firewall zone: `iot`

**Interfaccia VLAN 40 (Security):**
- **Name**: `security`
- **Device**: `br-lan.40`
- IPv4 address: `192.168.40.1`
- Netmask: `255.255.255.0`
- DHCP: Start `100`, Limit `100`, Leasetime `12h`
- Firewall zone: `security`

**Interfaccia VLAN 50 (Server):**
- **Name**: `server`
- **Device**: `br-lan.50`
- IPv4 address: `192.168.50.1`
- Netmask: `255.255.255.0`
- DHCP: Start `10`, Limit `40`, Leasetime `24h` (lease lunghi: i server non cambiano IP spesso)
- Firewall zone: `server`

   > **Consiglio**: Per i server, usa IP statici o DHCP con IP fisso (static lease). Vai in **Network → DHCP and DNS → Static Leases** e assegna sempre lo stesso IP al MAC address del tuo server.

**Interfaccia VLAN 60 (Guest):**
- **Name**: `guest`
- **Device**: `br-lan.60`
- IPv4 address: `192.168.60.1`
- Netmask: `255.255.255.0`
- DHCP: Start `100`, Limit `100`, Leasetime `30m` (lease brevissimi per ospiti)
- Firewall zone: `guest`

3. Dopo aver creato tutte le interfacce, puoi cliccare **"Save & Apply"**.

   Quando appare la finestra di conferma con il countdown di 90 secondi, verifica che il tuo PC abbia ancora accesso al router (apri `http://192.168.10.1`). Se funziona, clicca **"Apply and keep settings"**.

---

### 4.5 Passo 4 — Configurare le zone Firewall

Vai su **Network → Firewall**.

Vedrai le zone esistenti (`lan`, `wan`). Devi:
1. Modificare la zona `lan` per assicurarti che sia assegnata solo all'interfaccia `lan` (VLAN 10)
2. Creare le zone per le nuove VLAN
3. Configurare le regole di forwarding e le regole di traffico

**Creare una nuova zona (esempio per IoT):**

1. Nella sezione "Zones", clicca **Add**
2. Inserisci:
   - **Name**: `iot`
   - **Input**: DROP (il router non risponde alle richieste dall'IoT)
   - **Output**: ACCEPT
   - **Forward**: DROP
   - **Covered networks**: seleziona `iot`
3. Salva

Ripeti per `home`, `security`, `server`, `guest`:
- `home`: Input ACCEPT, Forward ACCEPT (rete fidata secondaria)
- `iot`: Input DROP, Forward DROP
- `security`: Input DROP, Forward DROP
- `server`: Input DROP, Forward DROP
- `guest`: Input DROP, Forward DROP

**Configurare le regole di Forwarding:**

Nella sezione "Inter-Zone Forwarding" o "Forwarding rules":

| Sorgente | Destinazione | Motivo |
|---|---|---|
| `lan` | `wan` | Management accede a internet |
| `lan` | `home` | Management accede a home |
| `lan` | `iot` | Management gestisce IoT |
| `lan` | `security` | Management gestisce Security |
| `lan` | `server` | Management accede ai server |
| `lan` | `guest` | Management gestisce Guest |
| `home` | `wan` | Home accede a internet |
| `home` | `server` | Home accede ai server |
| `iot` | `wan` | IoT aggiorna firmware |
| `security` | `server` | Telecamere scrivono su NAS |
| `guest` | `wan` | Ospiti usano internet |

**Aggiungere regole per DHCP e DNS dalle zone isolate:**

Le zone con `Input DROP` non possono fare richieste DHCP e DNS al router, il che bloccherebbe i dispositivi dall'ottenere un indirizzo IP. Devi aggiungere eccezioni:

Per ogni zona isolata (`iot`, `security`, `server`, `guest`), aggiungi una regola:

1. Nella sezione "Traffic Rules", clicca **Add**
2. **Name**: `Allow-DHCP-DNS-IoT`
3. **Protocol**: TCP+UDP
4. **Source zone**: `iot`
5. **Destination port**: `53 67 68`
6. **Action**: ACCEPT

Porta 53 = DNS, porta 67/68 = DHCP. Queste devono essere aperte affinché i dispositivi possano ottenere un indirizzo IP e risolvere i nomi di dominio.

> **Nota su DHCP**: in realtà il DHCP è broadcast (non va al router "direttamente"), ma OpenWrt gestisce questa eccezione internamente quando `dnsmasq` è in ascolto sull'interfaccia. La regola importante è quella per la porta 53 (DNS).

---

### 4.6 Passo 5 — Configurare le reti Wireless

Vai su **Network → Wireless**.

Vedrai i due radio (`radio0` per 2.4 GHz, `radio1` per 5 GHz) con il loro SSID principale. Devi aggiungere nuovi SSID virtuali per ogni VLAN.

**Come aggiungere un SSID per una VLAN:**

1. Accanto a `radio0` o `radio1`, clicca **Add**
2. Si apre la configurazione del nuovo access point virtuale

**Scheda "Device Configuration"** (lascia i valori di default del radio):

**Scheda "Interface Configuration":**
- **Mode**: Access Point
- **SSID**: il nome della rete (es. `Casa-Home`, `Casa-IoT`, ecc.)
- **Network**: qui selezioni l'interfaccia VLAN (es. `home`, `iot`, ecc.) — **questo è il collegamento tra WiFi e VLAN**
- **Encryption**: WPA2-PSK o WPA3 (raccomandato WPA3 o WPA2/WPA3 misto per compatibilità)
- **Key**: la password della rete

**Configurazione suggerita per ogni SSID:**

| SSID | Radio | VLAN (Network) | Sicurezza | Note |
|---|---|---|---|---|
| `Casa` | 5 GHz (radio1) | `lan` (VLAN 10) | WPA3 | Management principale |
| `Casa-Home` | 5 GHz (radio1) | `home` | WPA3 | Dispositivi di casa |
| `Casa-Home-2G` | 2.4 GHz (radio0) | `home` | WPA2/WPA3 | Dispositivi più vecchi |
| `Casa-IoT` | 2.4 GHz (radio0) | `iot` | WPA2 | IoT usa quasi sempre 2.4 GHz |
| `Casa-Security` | 2.4 GHz (radio0) | `security` | WPA2 | Telecamere spesso 2.4 GHz |
| `Casa-Server` | — | — | — | Solitamente no WiFi, solo cavo |
| `Casa-Guest` | 5 GHz (radio1) | `guest` | WPA2 | Ospiti |

**Opzione "Client isolation"**: per le reti IoT e Guest, considera di abilitare `option isolate '1'` nella configurazione (o cercare "Client isolation" nell'interfaccia). Questa opzione impedisce ai dispositivi sulla stessa rete WiFi di comunicare direttamente tra loro — un dispositivo IoT non può "attaccare" un altro dispositivo IoT sulla stessa rete.

**Come funziona il collegamento WiFi-VLAN:**

Quando imposti `Network: iot` su un'interfaccia WiFi virtuale, OpenWrt crea un'interfaccia virtuale (es. `wlan0-1`) e la aggiunge al bridge `br-lan` come membro della VLAN 30. I pacchetti WiFi vengono taggati automaticamente con la VLAN corretta prima di entrare nel bridge.

---

## 5. File di Configurazione Completi

Questa sezione mostra i file di configurazione completi che puoi applicare direttamente via SSH/terminale. Sono l'equivalente testuale di tutto quello che hai configurato tramite LuCI.

> **Accedere al router via SSH:**
> ```bash
> ssh root@192.168.10.1
> ```
> La password è quella del pannello LuCI.

### 5.1 /etc/config/network

Questo file definisce tutte le interfacce di rete, il bridge con VLAN filtering, e le VLAN.

```uci
# /etc/config/network

config interface 'loopback'
    option device 'lo'
    option proto 'static'
    option ipaddr '127.0.0.1'
    option netmask '255.0.0.0'

config globals 'globals'
    option ula_prefix 'fd00::/48'

# =========================================================
# BRIDGE br-lan con VLAN Filtering abilitato
# Raggruppa tutte le porte LAN fisiche
# =========================================================
config device
    option name 'br-lan'
    option type 'bridge'
    option vlan_filtering '1'
    list ports 'lan1'
    list ports 'lan2'
    list ports 'lan3'
    list ports 'lan4'

# =========================================================
# VLAN 10 — Management / Untagged
# lan3 = porta di gestione (untagged, PVID=10)
# altre porte = trunk (tagged)
# Modifica 'lan3' con la porta che vuoi usare per il management
# =========================================================
config bridge-vlan
    option device 'br-lan'
    option vlan '10'
    list ports 'lan1:t'
    list ports 'lan2:t'
    list ports 'lan3:u*'
    list ports 'lan4:t'

# =========================================================
# VLAN 20 — Home
# Tutte le porte sono trunk (tagged), accesso solo via WiFi
# o switch managed con porte access configurate
# =========================================================
config bridge-vlan
    option device 'br-lan'
    option vlan '20'
    list ports 'lan1:t'
    list ports 'lan2:t'
    list ports 'lan3:t'
    list ports 'lan4:t'

# VLAN 30 — IoT
config bridge-vlan
    option device 'br-lan'
    option vlan '30'
    list ports 'lan1:t'
    list ports 'lan2:t'
    list ports 'lan3:t'
    list ports 'lan4:t'

# VLAN 40 — Security
config bridge-vlan
    option device 'br-lan'
    option vlan '40'
    list ports 'lan1:t'
    list ports 'lan2:t'
    list ports 'lan3:t'
    list ports 'lan4:t'

# VLAN 50 — Server
config bridge-vlan
    option device 'br-lan'
    option vlan '50'
    list ports 'lan1:t'
    list ports 'lan2:t'
    list ports 'lan3:t'
    list ports 'lan4:t'

# VLAN 60 — Guest
config bridge-vlan
    option device 'br-lan'
    option vlan '60'
    list ports 'lan1:t'
    list ports 'lan2:t'
    list ports 'lan3:t'
    list ports 'lan4:t'

# =========================================================
# INTERFACCE LOGICHE
# Ogni interfaccia usa br-lan.VLANID come device
# Il router si comporta da gateway per quella sottorete
# =========================================================

# VLAN 10 — Management (interfaccia principale, ex "lan")
config interface 'lan'
    option device 'br-lan.10'
    option proto 'static'
    option ipaddr '192.168.10.1'
    option netmask '255.255.255.0'
    option ip6assign '60'

# VLAN 20 — Home
config interface 'home'
    option device 'br-lan.20'
    option proto 'static'
    option ipaddr '192.168.20.1'
    option netmask '255.255.255.0'

# VLAN 30 — IoT
config interface 'iot'
    option device 'br-lan.30'
    option proto 'static'
    option ipaddr '192.168.30.1'
    option netmask '255.255.255.0'

# VLAN 40 — Security
config interface 'security'
    option device 'br-lan.40'
    option proto 'static'
    option ipaddr '192.168.40.1'
    option netmask '255.255.255.0'

# VLAN 50 — Server
config interface 'server'
    option device 'br-lan.50'
    option proto 'static'
    option ipaddr '192.168.50.1'
    option netmask '255.255.255.0'

# VLAN 60 — Guest
config interface 'guest'
    option device 'br-lan.60'
    option proto 'static'
    option ipaddr '192.168.60.1'
    option netmask '255.255.255.0'

# =========================================================
# INTERFACCIA WAN
# eth1 = porta WAN fisica 2.5 GbE (connessa al modem)
# =========================================================
config interface 'wan'
    option device 'eth1'
    option proto 'dhcp'

config interface 'wan6'
    option device 'eth1'
    option proto 'dhcpv6'

# =========================================================
# WIREGUARD VPN
# Interfaccia tunnel per i client VPN remoti
# =========================================================
config interface 'wg0'
    option proto 'wireguard'
    option private_key 'INSERISCI_QUI_LA_TUA_CHIAVE_PRIVATA_SERVER'
    option listen_port '51820'
    list addresses '10.100.0.1/24'

# Un blocco per ogni client VPN autorizzato:
config wireguard_wg0
    option description 'Laptop remoto'
    option public_key 'CHIAVE_PUBBLICA_DEL_CLIENT'
    option route_allowed_ips '1'
    list allowed_ips '10.100.0.2/32'
    option persistent_keepalive '25'
```

**Come applicare le modifiche:**
```bash
# Dopo aver modificato il file:
service network reload
# oppure:
uci commit network && reload_config
```

---

### 5.2 /etc/config/wireless

Questo file configura tutte le reti WiFi. I radio fisici sono definiti nella sezione `wifi-device`, mentre ogni SSID virtuale è un blocco `wifi-iface`.

```uci
# /etc/config/wireless

# =========================================================
# RADIO FISICI
# Non modificare questi valori a meno che non sai cosa fai
# =========================================================

# Radio 2.4 GHz
config wifi-device 'radio0'
    option type 'mac80211'
    option band '2g'
    option htmode 'HE40'
    option channel 'auto'
    option country 'IT'
    option cell_density '0'

# Radio 5 GHz
config wifi-device 'radio1'
    option type 'mac80211'
    option band '5g'
    option htmode 'HE80'
    option channel 'auto'
    option country 'IT'
    option cell_density '0'

# =========================================================
# SSID MANAGEMENT (VLAN 10)
# Solo 5 GHz per il pannello di gestione
# Password robusta — usato solo per amministrare il router
# =========================================================
config wifi-iface 'wifi_mgmt'
    option device 'radio1'
    option mode 'ap'
    option network 'lan'
    option ssid 'Casa-Management'
    option encryption 'sae'
    option key 'PasswordMoltoSicura!'

# =========================================================
# SSID HOME (VLAN 20)
# Rete principale per dispositivi fidati di casa
# Doppio radio: 5 GHz per velocità, 2.4 GHz per compatibilità
# =========================================================
config wifi-iface 'wifi_home_5g'
    option device 'radio1'
    option mode 'ap'
    option network 'home'
    option ssid 'Casa'
    option encryption 'sae-mixed'
    option key 'PasswordCasa!'

config wifi-iface 'wifi_home_2g'
    option device 'radio0'
    option mode 'ap'
    option network 'home'
    option ssid 'Casa-2G'
    option encryption 'sae-mixed'
    option key 'PasswordCasa!'

# =========================================================
# SSID IOT (VLAN 30)
# Solo 2.4 GHz: quasi tutti i dispositivi IoT usano questa banda
# Isolamento client attivo per sicurezza
# =========================================================
config wifi-iface 'wifi_iot'
    option device 'radio0'
    option mode 'ap'
    option network 'iot'
    option ssid 'Casa-IoT'
    option encryption 'psk2'
    option key 'PasswordIoT!'
    option isolate '1'

# =========================================================
# SSID SECURITY (VLAN 40)
# Per telecamere e sensori — quasi sempre 2.4 GHz
# Isolamento client attivo
# =========================================================
config wifi-iface 'wifi_security'
    option device 'radio0'
    option mode 'ap'
    option network 'security'
    option ssid 'Casa-Security'
    option encryption 'psk2'
    option key 'PasswordSecurity!'
    option isolate '1'

# =========================================================
# SSID SERVER (VLAN 50)
# Generalmente i server usano cavi — ometti se non necessario
# Se hai qualche server WiFi, aggiungilo qui
# =========================================================
# config wifi-iface 'wifi_server'
#     option device 'radio1'
#     option mode 'ap'
#     option network 'server'
#     option ssid 'Casa-Server'
#     option encryption 'sae'
#     option key 'PasswordServer!'

# =========================================================
# SSID GUEST (VLAN 60)
# Rete ospiti — solo internet
# Isolamento client: gli ospiti non si vedono tra loro
# =========================================================
config wifi-iface 'wifi_guest'
    option device 'radio1'
    option mode 'ap'
    option network 'guest'
    option ssid 'Ospiti'
    option encryption 'psk2'
    option key 'PasswordOspiti!'
    option isolate '1'
```

---

### 5.3 /etc/config/firewall

Questo è il file più complesso. Implementa le politiche di sicurezza definite nella sezione 3.3.

```uci
# /etc/config/firewall

# =========================================================
# IMPOSTAZIONI GLOBALI
# =========================================================
config defaults
    option syn_flood '1'
    option input 'DROP'
    option output 'ACCEPT'
    option forward 'DROP'

# =========================================================
# ZONA WAN — Internet
# Input DROP: nessuno può entrare dall'esterno
# masq: NAT (tutti i dispositivi usano l'IP pubblico del router)
# mtu_fix: necessario per PPPoE e alcune connessioni
# =========================================================
config zone
    option name 'wan'
    option input 'DROP'
    option output 'ACCEPT'
    option forward 'DROP'
    option masq '1'
    option mtu_fix '1'
    list network 'wan'
    list network 'wan6'

# =========================================================
# ZONA LAN (Management VLAN 10)
# Input ACCEPT: si può accedere al pannello del router
# Rete completamente fidata
# =========================================================
config zone
    option name 'lan'
    option input 'ACCEPT'
    option output 'ACCEPT'
    option forward 'ACCEPT'
    list network 'lan'

# =========================================================
# ZONA HOME (VLAN 20)
# Rete fidata secondaria
# =========================================================
config zone
    option name 'home'
    option input 'ACCEPT'
    option output 'ACCEPT'
    option forward 'ACCEPT'
    list network 'home'

# =========================================================
# ZONA IOT (VLAN 30)
# Input DROP: i dispositivi IoT non accedono al router
# Forward DROP: non possono raggiungere altre VLAN
# =========================================================
config zone
    option name 'iot'
    option input 'DROP'
    option output 'ACCEPT'
    option forward 'DROP'
    list network 'iot'

# =========================================================
# ZONA SECURITY (VLAN 40)
# Stessa politica restrittiva di IoT
# =========================================================
config zone
    option name 'security'
    option input 'DROP'
    option output 'ACCEPT'
    option forward 'DROP'
    list network 'security'

# =========================================================
# ZONA SERVER (VLAN 50)
# Input DROP: i server non devono essere "raggiungibili" dal router
# direttamente per gestione (solo dalle altre VLAN autorizzate)
# =========================================================
config zone
    option name 'server'
    option input 'DROP'
    option output 'ACCEPT'
    option forward 'DROP'
    list network 'server'

# =========================================================
# ZONA GUEST (VLAN 60)
# Solo internet — niente altro
# =========================================================
config zone
    option name 'guest'
    option input 'DROP'
    option output 'ACCEPT'
    option forward 'DROP'
    list network 'guest'

# =========================================================
# ZONA VPN (WireGuard)
# I client VPN remoti hanno accesso controllato
# =========================================================
config zone
    option name 'vpn'
    option input 'ACCEPT'
    option output 'ACCEPT'
    option forward 'DROP'
    list network 'wg0'

# =========================================================
# REGOLE DI FORWARDING TRA ZONE
# Ogni riga = "la zona sorgente può raggiungere la destinazione"
# Il traffico di ritorno è gestito automaticamente dal firewall
# tramite connection tracking (stateful firewall)
# =========================================================

# Management accede ovunque
config forwarding
    option src 'lan'
    option dest 'wan'

config forwarding
    option src 'lan'
    option dest 'home'

config forwarding
    option src 'lan'
    option dest 'iot'

config forwarding
    option src 'lan'
    option dest 'security'

config forwarding
    option src 'lan'
    option dest 'server'

config forwarding
    option src 'lan'
    option dest 'guest'

config forwarding
    option src 'lan'
    option dest 'vpn'

# Home → Internet e Server
config forwarding
    option src 'home'
    option dest 'wan'

config forwarding
    option src 'home'
    option dest 'server'

# IoT → Internet (per aggiornamenti firmware, cloud)
# Rimuovi se vuoi isolare completamente l'IoT da internet
config forwarding
    option src 'iot'
    option dest 'wan'

# Security → Server (telecamere scrivono su NAS)
config forwarding
    option src 'security'
    option dest 'server'

# Guest → Internet
config forwarding
    option src 'guest'
    option dest 'wan'

# VPN → Server (client VPN remoti accedono ai server)
config forwarding
    option src 'vpn'
    option dest 'server'

# =========================================================
# REGOLA WAN: Permettere il traffico WireGuard in entrata
# Senza questa regola, i client VPN non possono connettersi
# =========================================================
config rule
    option name 'Permetti-WireGuard-WAN'
    option src 'wan'
    option dest_port '51820'
    list proto 'udp'
    option target 'ACCEPT'

# =========================================================
# REGOLE DNS E DHCP PER ZONE ISOLATE
# Senza queste regole, i dispositivi non ottengono IP
# e non risolvono i nomi di dominio
# =========================================================
config rule
    option name 'IoT-DNS-DHCP'
    option src 'iot'
    option dest_port '53 67 68'
    list proto 'tcp'
    list proto 'udp'
    option target 'ACCEPT'

config rule
    option name 'Security-DNS-DHCP'
    option src 'security'
    option dest_port '53 67 68'
    list proto 'tcp'
    list proto 'udp'
    option target 'ACCEPT'

config rule
    option name 'Server-DNS-DHCP'
    option src 'server'
    option dest_port '53 67 68'
    list proto 'tcp'
    list proto 'udp'
    option target 'ACCEPT'

config rule
    option name 'Guest-DNS-DHCP'
    option src 'guest'
    option dest_port '53 67 68'
    list proto 'tcp'
    list proto 'udp'
    option target 'ACCEPT'

# =========================================================
# REGOLE DI BLOCCO ESPLICITO (Defense in Depth)
# Queste regole bloccano esplicitamente alcuni percorsi
# anche se le politiche "forward DROP" dovrebbero già farlo
# Utile come doppio livello di sicurezza
# =========================================================

# IoT non può raggiungere Management né Home
config rule
    option name 'Blocca-IoT-verso-LAN'
    option src 'iot'
    option dest 'lan'
    option target 'REJECT'

config rule
    option name 'Blocca-IoT-verso-Home'
    option src 'iot'
    option dest 'home'
    option target 'REJECT'

# Guest non può raggiungere reti interne
config rule
    option name 'Blocca-Guest-verso-LAN'
    option src 'guest'
    option dest 'lan'
    option target 'REJECT'

config rule
    option name 'Blocca-Guest-verso-Home'
    option src 'guest'
    option dest 'home'
    option target 'REJECT'

# VPN non raggiunge Management né Home (solo Server)
config rule
    option name 'Blocca-VPN-verso-LAN'
    option src 'vpn'
    option dest 'lan'
    option target 'REJECT'

config rule
    option name 'Blocca-VPN-verso-Home'
    option src 'vpn'
    option dest 'home'
    option target 'REJECT'

config rule
    option name 'Blocca-VPN-verso-IoT'
    option src 'vpn'
    option dest 'iot'
    option target 'REJECT'

config rule
    option name 'Blocca-VPN-verso-Guest'
    option src 'vpn'
    option dest 'guest'
    option target 'REJECT'

# =========================================================
# REGOLE STANDARD (non modificare)
# =========================================================
config rule
    option name 'Allow-DHCP-Renew'
    option src 'wan'
    option proto 'udp'
    option dest_port '68'
    option target 'ACCEPT'

config rule
    option name 'Allow-Ping'
    option src 'wan'
    list proto 'icmp'
    option icmp_type 'echo-request'
    option family 'ipv4'
    option target 'ACCEPT'

config rule
    option name 'Allow-ICMPv6-Input'
    option src 'wan'
    list proto 'icmp'
    option family 'ipv6'
    option target 'ACCEPT'
```

---

### 5.4 /etc/config/dhcp

Questo file configura il server DHCP (gestito da `dnsmasq`), che assegna automaticamente gli indirizzi IP ai dispositivi su ogni VLAN.

```uci
# /etc/config/dhcp

config dnsmasq
    option domainneeded '1'
    option boguspriv '1'
    option localise_queries '1'
    option rebind_protection '1'
    option rebind_localhost '1'
    option local '/lan/'
    option domain 'lan'
    option expandhosts '1'
    option cachesize '1000'
    option authoritative '1'
    option readethers '1'
    option leasefile '/tmp/dhcp.leases'
    option resolvfile '/tmp/resolv.conf.d/resolv.conf.auto'
    option localservice '1'
    option ednspacket_max '1232'

# =========================================================
# DHCP per VLAN 10 — Management
# =========================================================
config dhcp 'lan'
    option interface 'lan'
    option start '100'
    option limit '150'
    option leasetime '12h'
    option ra 'server'
    option dhcpv6 'server'

# =========================================================
# DHCP per VLAN 20 — Home
# =========================================================
config dhcp 'home'
    option interface 'home'
    option start '100'
    option limit '150'
    option leasetime '12h'

# =========================================================
# DHCP per VLAN 30 — IoT
# Lease brevi: i dispositivi IoT si rinnovano frequentemente
# =========================================================
config dhcp 'iot'
    option interface 'iot'
    option start '100'
    option limit '150'
    option leasetime '1h'
    list dhcp_option '6,192.168.30.1'

# =========================================================
# DHCP per VLAN 40 — Security
# =========================================================
config dhcp 'security'
    option interface 'security'
    option start '100'
    option limit '100'
    option leasetime '12h'
    list dhcp_option '6,192.168.40.1'

# =========================================================
# DHCP per VLAN 50 — Server
# Lease lunghi: i server non cambiano IP spesso
# Range piccolo: ci sono pochi server, e preferibilmente
# usiamo IP statici (vedere Static Leases)
# =========================================================
config dhcp 'server'
    option interface 'server'
    option start '10'
    option limit '40'
    option leasetime '24h'
    list dhcp_option '6,192.168.50.1'

# =========================================================
# DHCP per VLAN 60 — Guest
# Lease brevissimi: gli ospiti si connettono e disconnettono
# =========================================================
config dhcp 'guest'
    option interface 'guest'
    option start '100'
    option limit '100'
    option leasetime '30m'
    list dhcp_option '6,192.168.60.1'

# =========================================================
# Disabilita DHCP su WAN (non rispondere a richieste WAN)
# =========================================================
config dhcp 'wan'
    option interface 'wan'
    option ignore '1'

# =========================================================
# STATIC LEASES — IP fissi per server (esempio)
# I server devono sempre avere lo stesso IP!
# Vai su Network → DHCP and DNS → Static Leases in LuCI
# per aggiungerne altri graficamente
# =========================================================
# config host
#     option name 'nas'
#     option mac '00:11:22:33:44:55'
#     option ip '192.168.50.10'
#     option leasetime 'infinite'
```

---

## 6. Integrazione con WireGuard VPN

### 6.1 Come WireGuard si integra con le VLAN

WireGuard in OpenWrt crea un'interfaccia di rete virtuale chiamata `wg0`. Questa interfaccia non è una porta fisica, ma un "tunnel" attraverso internet: i dispositivi che si connettono tramite WireGuard "entrano" nella rete del router come se fossero collegati localmente.

La domanda è: in quale VLAN li mettiamo?

La risposta è: **in nessuna VLAN fisica, ma in una zona firewall separata** (`vpn`). I client VPN hanno il loro subnet dedicato (`10.100.0.0/24`) e tramite le regole firewall decidiamo a cosa possono accedere.

**Schema di funzionamento:**

```
Client remoto (es. laptop)
        │
        │ Internet (traffico cifrato WireGuard)
        │
    Router Flint 2
        │
        ├── Interfaccia wg0 (zona vpn: 10.100.0.0/24)
        │         │
        │    Firewall: vpn → server PERMESSO
        │    Firewall: vpn → lan BLOCCATO
        │    Firewall: vpn → home BLOCCATO
        │
        └── VLAN 50 Server (192.168.50.0/24) ← il client VPN può raggiungere qui
```

**Subnet routing**: quando un client VPN vuole raggiungere `192.168.50.10` (il tuo NAS), il pacchetto:
1. Viaggia cifrato da internet al router sulla porta 51820 UDP
2. WireGuard decifra il pacchetto
3. Il pacchetto entra nell'interfaccia `wg0`
4. Il firewall verifica: `vpn → server` è permesso
5. Il pacchetto viene inoltrato all'interfaccia `br-lan.50` (VLAN 50)
6. Il NAS riceve la richiesta e risponde normalmente

---

### 6.2 Configurazione della zona VPN nel Firewall

La configurazione del firewall per WireGuard è già inclusa nel file `/etc/config/firewall` sopra. Riepiloghiamo i punti chiave:

**Zona VPN:**
```uci
config zone
    option name 'vpn'
    option input 'ACCEPT'    # Il router risponde ai client VPN (DNS, ecc.)
    option output 'ACCEPT'
    option forward 'DROP'    # Non possono andare ovunque, solo dove permesso
    list network 'wg0'
```

**Forwarding VPN → Server:**
```uci
config forwarding
    option src 'vpn'
    option dest 'server'
```

**Apertura porta WireGuard sulla WAN:**
```uci
config rule
    option name 'Permetti-WireGuard-WAN'
    option src 'wan'
    option dest_port '51820'
    list proto 'udp'
    option target 'ACCEPT'
```

> **Perché `input 'ACCEPT'` sulla zona VPN?**
> I client VPN hanno bisogno di risolvere nomi DNS e di ricevere risposte dal router. Se metti `input 'DROP'`, i client VPN non potranno nemmeno fare una query DNS. Il parametro `input` controlla il traffico *verso il router stesso* (non verso le altre VLAN, che è gestito da `forward`).

> **Importante — NO masquerading sulla VPN:**
> Non abilitare `option masq '1'` sulla zona VPN. Il NAT (masquerading) nasconde l'IP originale del client VPN, il che crea problemi con il routing di ritorno (il server non saprebbe a chi rispondere). Solo la zona WAN deve avere il NAT abilitato.

---

### 6.3 Aggiornamento della configurazione del client WireGuard

Il file di configurazione del client WireGuard (sul laptop/telefono remoto) deve essere aggiornato per includere il subnet della VLAN Server negli `AllowedIPs`.

**Configurazione client WireGuard (split tunnel — solo Server VLAN):**

```ini
[Interface]
PrivateKey = CHIAVE_PRIVATA_DEL_CLIENT
Address = 10.100.0.2/32
DNS = 192.168.50.1   # Usa il gateway VLAN Server come DNS

[Peer]
PublicKey = CHIAVE_PUBBLICA_DEL_SERVER
Endpoint = tuo-dominio-o-ip-pubblico:51820
# Split tunnel: solo il traffico verso Server VLAN e subnet VPN
# passa attraverso il tunnel. Il resto usa internet normale.
AllowedIPs = 10.100.0.0/24, 192.168.50.0/24
PersistentKeepalive = 25
```

**Configurazione client WireGuard (full tunnel — tutto il traffico via VPN):**

```ini
[Interface]
PrivateKey = CHIAVE_PRIVATA_DEL_CLIENT
Address = 10.100.0.2/32
DNS = 10.100.0.1   # Router come DNS

[Peer]
PublicKey = CHIAVE_PUBBLICA_DEL_SERVER
Endpoint = tuo-dominio-o-ip-pubblico:51820
# Full tunnel: TUTTO il traffico internet passa attraverso casa
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

**Quale scegliere?**
- **Split tunnel** (`AllowedIPs = 10.100.0.0/24, 192.168.50.0/24`): solo il traffico verso i server di casa usa la VPN. Il resto di internet usa la connessione diretta. Più veloce, meno carico sul router.
- **Full tunnel** (`AllowedIPs = 0.0.0.0/0`): tutto il traffico passa dal tuo router di casa. Utile per sicurezza in reti WiFi pubbliche, ma usa più banda della connessione di casa.

---

## 7. Verifica e Risoluzione dei Problemi

### 7.1 Comandi di verifica via SSH

Connettiti al router via SSH (`ssh root@192.168.10.1`) ed esegui questi comandi per verificare che tutto funzioni correttamente.

**Verificare le interfacce di rete:**
```bash
ip addr show
```
Dovresti vedere: `br-lan`, `br-lan.10`, `br-lan.20`, `br-lan.30`, `br-lan.40`, `br-lan.50`, `br-lan.60`, `wg0`

**Verificare il VLAN filtering del bridge:**
```bash
bridge vlan show
```
Output atteso (esempio parziale):
```
port              vlan-id
lan1              10 tagged
                  20 tagged
                  30 tagged
                  40 tagged
                  50 tagged
                  60 tagged
lan3              10 PVID Egress Untagged
                  20 tagged
                  ...
br-lan            10 self
                  20 self
                  ...
```

**Verificare le tabelle del bridge (dispositivi connessi):**
```bash
bridge fdb show
```

**Verificare i lease DHCP attivi:**
```bash
cat /tmp/dhcp.leases
```

**Verificare le regole firewall attive:**
```bash
nft list ruleset
# oppure su versioni più vecchie:
iptables -L -n -v
```

**Verificare lo stato di WireGuard:**
```bash
wg show
```

**Testare connettività tra VLAN:**
```bash
# Dal router, testa se riesci a pingare un dispositivo in una VLAN
ping -I br-lan.20 192.168.20.100

# Verifica che da IoT NON si raggiunge Home (deve fallire)
# Questo lo fai da un dispositivo nella VLAN IoT
ping 192.168.20.1   # Deve dare "Network unreachable" o timeout
```

**Diagnosticare il traffico in tempo reale:**
```bash
# Monitora i log del firewall
logread -f | grep -i "firewall\|drop\|reject"
```

---

### 7.2 Problemi comuni e soluzioni

**Problema: Ho perso accesso al router dopo aver applicato le modifiche**

OpenWrt aspetta automaticamente 90 secondi prima di annullare le modifiche. Se non hai confermato entro quel tempo, le impostazioni precedenti vengono ripristinate automaticamente.

Se invece hai confermato le modifiche e ora sei bloccato:
1. Connettiti fisicamente alla porta che hai impostato come PVID per la VLAN 10 (porta di management)
2. Assicurati che il tuo PC sia impostato su DHCP
3. Prova ad accedere a `http://192.168.10.1`

Se non funziona ancora, devi fare un reset del router: tieni premuto il tasto reset per 10 secondi.

---

**Problema: I dispositivi nella VLAN IoT non ricevono un indirizzo IP**

Cause possibili:
1. La regola firewall per DHCP/DNS non è configurata → aggiungi la regola `IoT-DNS-DHCP`
2. Il server DHCP non è abilitato per quell'interfaccia → vai su **Network → DHCP and DNS** e verifica
3. L'interfaccia `iot` non è correttamente assegnata alla VLAN 30 → verifica `bridge vlan show`

---

**Problema: Le reti WiFi delle VLAN non funzionano**

Verifica che:
1. Il campo `option network 'iot'` in `/etc/config/wireless` corrisponda esattamente al nome dell'interfaccia in `/etc/config/network`
2. Il radio non sia disabilitato (`option disabled '0'` o assenza della riga)
3. Riavvia il wireless: `wifi reload`

---

**Problema: I client VPN non riescono a raggiungere i server**

Verifica che:
1. La porta 51820 UDP sia aperta sulla WAN (`config rule 'Permetti-WireGuard-WAN'`)
2. Il forwarding `vpn → server` sia configurato
3. Il client WireGuard abbia `192.168.50.0/24` negli `AllowedIPs`
4. Non ci sia NAT (masq) sulla zona VPN
5. Verifica con: `wg show` — deve mostrare il client come connected con `latest handshake` recente

---

**Problema: Un dispositivo IoT non riesce a raggiungere internet**

1. Verifica che il forwarding `iot → wan` sia presente e attivo
2. Verifica che la zona `iot` abbia `option masq '1'` — **aspetta**, il masquerading deve essere sulla zona WAN, non IoT. OpenWrt gestisce il NAT automaticamente: tutto il traffico che attraverso la zona WAN viene nattato. Non serve masq sulle zone interne.
3. Controlla i log: `logread -f | grep DROP` mentre il dispositivo prova a connettersi

---

**Problema: La VLAN Security non riesce a scrivere sul NAS nella VLAN Server**

1. Verifica il forwarding `security → server`
2. Sul NAS, verifica che accetti connessioni dalla subnet 192.168.40.0/24
3. Se il NAS ha un proprio firewall (es. Synology DSM), aggiungilo come rete fidata
4. Testa manualmente: dal router, `ping -I br-lan.40 192.168.50.10`

---

> **Nota finale sulla sicurezza**: La separazione in VLAN è un ottimo livello di protezione, ma non è infallibile. Un dispositivo molto sofisticato potrebbe tentare VLAN hopping in alcune configurazioni. Per la massima sicurezza, mantieni sempre aggiornato il firmware del router e dei dispositivi, usa password robuste, e considera di disabilitare l'accesso internet per le zone IoT e Security se quei dispositivi non ne hanno bisogno.

---

## Fonti

- [OpenWrt Documentation — DSA VLAN Configuration](https://openwrt.org/docs/guide-user/network/vlan/dsa/start)
- [OpenWrt Forum — GL-MT6000 DSA Config](https://forum.openwrt.org/t/gl-mt6000-dsa-config/193307)
- [OpenWrt Forum — Flint 2 VLAN Setup](https://forum.openwrt.org/t/gl-inet-flint-2-gl-mt6000-vlan-setup/213037)
- [GL.iNet Flint 2 Official Docs](https://docs.gl-inet.com/router/en/4/user_guide/gl-mt6000/)
- [GL.iNet Forum — Configuring VLANs on Flint 2](https://forum.gl-inet.com/t/configuring-vlans-on-flint-2/49291)
- [OpenWrt Forum — WireGuard Firewall Configuration](https://forum.openwrt.org/t/firewall-configuration-for-wireguard-vpn/159225)
- [Fabian Lee — Bridge VLAN Filtering for OpenWrt 21.x with DSA](https://fabianlee.org/2023/01/22/openwrt-bridge-vlan-filtering-for-openwrt-21-x-with-dsa-isolated-guest-wi-fi/)
- [Video YouTube originale](https://www.youtube.com/watch?v=qeuZqRqH-ug)
