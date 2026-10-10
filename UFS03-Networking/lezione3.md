# Lezione 3 — Laboratorio pratico: IPv4 e subnetting

**Durata indicativa:** 140 minuti  
**Strumento principale:** Cisco Packet Tracer  
**Obiettivo:** progettare, configurare e verificare un piano IPv4, applicare subnetting/VLSM e svolgere troubleshooting con fault controllati e supporto dell'IA.

---

## Metodo di lavoro

**Progetto → Configuro → Verifico → Rompo → Osservo → Formulo ipotesi → Testo → Risolvo → Verifico nuovamente → Documento**

Durante il troubleshooting: **una modifica alla volta**.

---

# A. Esercizi progressivi di subnetting

## Esercizio 1 — /24

Dato `192.168.10.25/24`, determinare mask, network ID, broadcast, primo/ultimo host e host utilizzabili.

**Soluzione attesa**

- Mask: `255.255.255.0`
- Network: `192.168.10.0`
- Primo host: `192.168.10.1`
- Ultimo host: `192.168.10.254`
- Broadcast: `192.168.10.255`
- Host utilizzabili: `254`

## Esercizio 2 — /26

Dato `192.168.10.77/26`.

Blocchi: `0–63`, `64–127`, `128–191`, `192–255`.

**Soluzione:** Network `192.168.10.64`, primo host `.65`, ultimo host `.126`, broadcast `.127`, mask `255.255.255.192`.

## Esercizio 3 — /27

Dato `192.168.20.101/27`.

Blocchi da 32 indirizzi: `0–31`, `32–63`, `64–95`, `96–127`, ...

**Soluzione:** Network `192.168.20.96`, primo host `.97`, ultimo host `.126`, broadcast `.127`, mask `255.255.255.224`.

## Esercizio 4 — sovrapposizione

Confrontare:

- `192.168.30.64/26`
- `192.168.30.96/27`

La prima copre `.64–.127`, la seconda `.96–.127`: **le subnet si sovrappongono**.

---

# B. Piano di indirizzamento VLSM

Rete disponibile: `192.168.40.0/24`.

| Reparto | Host richiesti |
|---|---:|
| Amministrazione | 50 |
| Produzione | 25 |
| Laboratorio | 12 |
| Management | 6 |

Assegnare dal reparto più grande al più piccolo.

| Reparto | Subnet | Mask | Primo host | Ultimo host | Broadcast | Gateway |
|---|---|---|---|---|---|---|
| Amministrazione | `192.168.40.0/26` | `255.255.255.192` | `.1` | `.62` | `.63` | `.1` |
| Produzione | `192.168.40.64/27` | `255.255.255.224` | `.65` | `.94` | `.95` | `.65` |
| Laboratorio | `192.168.40.96/28` | `255.255.255.240` | `.97` | `.110` | `.111` | `.97` |
| Management | `192.168.40.112/29` | `255.255.255.248` | `.113` | `.118` | `.119` | `.113` |

Verificare: nessuna sovrapposizione, capacità sufficiente, gateway nella subnet, network/broadcast non assegnati agli host.

---

# C. Cisco Packet Tracer — prima LAN

Topologia:

```text
PC0 -------- SW0 -------- PC1
```

| Dispositivo | IP | Mask | Gateway |
|---|---|---|---|
| PC0 | `192.168.10.10` | `255.255.255.0` | — |
| PC1 | `192.168.10.20` | `255.255.255.0` | — |

Attività:

1. Inserire due PC e uno switch Cisco 2960.
2. Collegare i PC allo switch.
3. Configurare gli indirizzi IPv4.
4. Salvare come `Lezione3_IPv4_Subnetting.pkt`.
5. Da PC0: `ping 192.168.10.20`.
6. Ripetere da PC1.
7. Osservare ARP e ICMP in **Simulation Mode**.

Domande: perché comunicano senza router? Qual è il network ID? Qual è il broadcast? Quale ruolo svolge la mask?

---

# D. Due subnet diverse e default gateway

Usare:

- PC0: `192.168.10.10/24`
- PC1: `192.168.20.10/24`

Con il solo switch Layer 2 il traffico tra le due subnet non viene instradato.

Estensione:

```text
PC0 ---- SW0 ---- R0 ---- SW1 ---- PC1
```

Gateway:

- rete `192.168.10.0/24` → `192.168.10.1`
- rete `192.168.20.0/24` → `192.168.20.1`

Configurazione di esempio:

```text
enable
configure terminal
interface gigabitEthernet 0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit
interface gigabitEthernet 0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
```

PC0: IP `192.168.10.10`, mask `255.255.255.0`, gateway `192.168.10.1`.

PC1: IP `192.168.20.10`, mask `255.255.255.0`, gateway `192.168.20.1`.

Test da PC0:

```text
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.20.10
```

> **Nota didattica:** il piano VLSM completo a quattro reparti è stato progettato sopra; l'implementazione multi-reparto completa verrà approfondita nelle lezioni successive con VLAN e segmentazione Layer 2/Layer 3.

---

# E. Fault Injection

Creare una copia funzionante del progetto prima di introdurre i fault.

## Fault 1 — subnet mask errata

- PC0: `192.168.10.10/26`
- PC1: `192.168.10.20/24`

Osservare ping, ARP e configurazione dei due host. Stabilire quale piano di indirizzamento è atteso.

## Fault 2 — indirizzo IP duplicato

Configurare temporaneamente entrambi con `192.168.10.10/24`.

Osservare ping e comportamento ARP. Cercare l'evidenza che suggerisce un duplicate IP.

## Fault 3 — gateway errato

PC0:

- IP `192.168.10.10`
- Mask `255.255.255.0`
- Gateway errato `192.168.10.254`

Testare:

1. host nella stessa subnet;
2. gateway corretto;
3. host della rete remota.

Distinguere connettività locale, gateway e rete remota.

## Fault 4 — IP fuori piano

Impostare temporaneamente `192.168.50.10/24` quando l'host dovrebbe essere nella rete `192.168.10.0/24`.

Determinare quali dati del piano permettono di individuare il problema.

---

# F. Scheda di indagine diagnostica

| Campo | Osservazione |
|---|---|
| Fault introdotto | |
| Sintomo osservato | |
| IP host | |
| Subnet mask | |
| Gateway | |
| Ping locale | |
| Ping gateway | |
| Ping remoto | |
| Altri test | |
| Ipotesi | |
| Test dell'ipotesi | |
| Risultato | |
| Causa confermata | |
| Correzione | |
| Verifica finale | |

Documentare il **percorso diagnostico**, non soltanto la soluzione.

---

# G. Attività con IA — prompt professionale

```text
Agisci come assistente esperto nel troubleshooting di reti IPv4.

Il tuo compito è aiutarmi a diagnosticare un guasto in una rete
Cisco Packet Tracer.

IMPORTANTE:
- non modificare direttamente la configurazione;
- non fornire immediatamente una soluzione definitiva;
- non inventare dati che non ti ho fornito;
- distingui sempre fatti osservati, ipotesi e conclusioni;
- se manca un'informazione importante, chiedimela;
- proponi una sequenza di test progressiva;
- modifica una sola variabile alla volta;
- considera prima i livelli più bassi e poi quelli superiori.

TOPOLOGIA:
[descrivere dispositivi e collegamenti]

PIANO DI INDIRIZZAMENTO ATTESO:
[riportare subnet, mask, gateway e dispositivi]

CONFIGURAZIONE OSSERVATA:
[riportare IP, mask e gateway effettivi]

SINTOMO:
[descrivere esattamente cosa non funziona]

TEST GIÀ ESEGUITI:
[elencare ping e altri test con relativo risultato]

DATI OSSERVATI:
[riportare gli elementi verificabili]

Voglio che la risposta sia organizzata in:
1. Sintesi del problema
2. Fatti certi
3. Informazioni mancanti
4. Ipotesi diagnostiche ordinate
5. Sequenza di test
6. Per ogni test: cosa verificare, azione/comando, risultato atteso,
   interpretazione
7. Criterio per confermare o escludere ogni ipotesi
8. Correzione solo dopo la conferma della causa
9. Verifica finale
10. Documentazione dell'incidente

Considera in particolare:
- subnet mask errata;
- indirizzo IP duplicato;
- gateway errato;
- IP fuori dalla subnet prevista;
- subnet sovrapposte;
- configurazione incoerente rispetto al piano.

Non assumere che un ping fallito dimostri da solo la causa.
Voglio un'indagine basata su evidenze e test riproducibili.
```

---

# H. Valutare la risposta dell'IA

Verificare se l'IA:

- separa fatti e ipotesi;
- evita di inventare informazioni;
- chiede dati mancanti;
- propone test progressivi;
- spiega come interpretare i risultati;
- evita modifiche simultanee;
- arriva a una causa verificata in Packet Tracer;
- permette di ripetere il test dopo la correzione.

**Regola fondamentale:** la risposta dell'IA è un'ipotesi diagnostica, non una prova. La prova è il risultato di un test riproducibile.

---

# I. Consegna finale

1. file Cisco Packet Tracer `.pkt`;
2. tabella completa VLSM;
3. assegnazione IP;
4. schede diagnostiche di almeno due fault;
5. prompt IA;
6. risposta dell'IA;
7. commento critico sulla risposta;
8. verifica finale della rete.

---

# J. Checklist

- [ ] So determinare network ID e broadcast.
- [ ] So interpretare /24–/30.
- [ ] So calcolare gli host utilizzabili.
- [ ] So riconoscere gli indirizzi privati.
- [ ] So spiegare il default gateway.
- [ ] So progettare VLSM senza sovrapposizioni.
- [ ] So configurare IP, mask e gateway in Packet Tracer.
- [ ] So verificare una LAN locale.
- [ ] So verificare la comunicazione tra subnet.
- [ ] So introdurre un fault controllato.
- [ ] So raccogliere evidenze prima di concludere.
- [ ] So usare l'IA per strutturare un'indagine diagnostica.

## Sintesi operativa

**Progetto → Configuro → Verifico → Rompo → Osservo → Formulo ipotesi → Testo → Risolvo → Verifico nuovamente → Documento**

L'obiettivo non è soltanto far funzionare la rete, ma imparare a spiegare **perché** funziona e **perché** smette di funzionare quando introduciamo un fault.
