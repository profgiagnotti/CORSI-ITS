# Lezione 8 — Laboratorio pratico: DHCP, DNS, TCP, UDP e analisi del traffico con AI

**Corso:** UFS 03 — Networking  
**Lezione:** 8  
**Durata indicativa:** 150 minuti  
**Ambiente principale:** Cisco Packet Tracer  
**Analisi reale del traffico:** Wireshark / file PCAP fornito dal docente

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio lo studente sarà in grado di:

- configurare un servizio DHCP;
- ottenere automaticamente la configurazione IP di un PC;
- configurare un servizio DNS locale;
- associare un nome DNS all'indirizzo IP di un server;
- verificare la comunicazione tra PC, DNS e server;
- distinguere il ruolo di DHCP, DNS, TCP e UDP;
- osservare DHCP, DNS, TCP e UDP in Cisco Packet Tracer Simulation Mode;
- leggere una cattura Wireshark e ricostruire ciò che sta accadendo nella rete;
- applicare una metodologia strutturata di troubleshooting;
- utilizzare l'AI come supporto all'indagine, senza delegarle la diagnosi;
- creare e analizzare Fault Injection controllati.

> **Metodo di lavoro:** OSSERVA → RACCOGLI LE EVIDENZE → FORMULA IPOTESI → TESTA → INTERPRETA → CONFERMA O SMENTISCI → RISOLVI → VERIFICA → DOCUMENTA

---

# 2. Scenario del laboratorio

Realizzeremo una piccola rete nella quale un PC non riceve manualmente tutta la configurazione, ma utilizza i servizi di rete messi a disposizione dai server.

Schema logico:

```text
                    +-------------------+
                    |      SERVER       |
                    | DHCP + DNS + HTTP |
                    | 192.168.10.10     |
                    +---------+---------+
                              |
                              |
                        +-----+-----+
                        |    SW0    |
                        +-----+-----+
                              |
                              |
                         +----+----+
                         |   PC0   |
                         |  DHCP   |
                         +---------+
```

## Piano di indirizzamento

| Dispositivo | Interfaccia | Indirizzo IPv4 | Maschera | Gateway | DNS |
|---|---|---|---|---|---|
| Server | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1* | 192.168.10.10 |
| PC0 | Fa0 | DHCP | /24 | DHCP | DHCP |
| Gateway | — | 192.168.10.1* | /24 | — | — |

> **Nota:** se si utilizza una rete composta soltanto da switch, il gateway non è indispensabile per le verifiche locali. Se si inserisce un router, configurare `192.168.10.1/24` sull'interfaccia della LAN.

---

# 3. Fase 1 — Costruzione della topologia

## 3.1 Dispositivi

Inserire in Cisco Packet Tracer:

- 1 PC;
- 1 Switch 2960;
- 1 Server.

Collegare:

```text
PC0 Fa0  ----  SW0 Fa0/1
SERVER Fa0 ---- SW0 Fa0/2
```

Utilizzare cavi Copper Straight-Through dove richiesto dalla versione di Packet Tracer.

## 3.2 Prima verifica

Controllare che i link diventino operativi.

Non procedere immediatamente con la configurazione: osservare prima la topologia.

### Domande

1. Quale dispositivo fornirà l'indirizzo IP al PC?
2. Quale dispositivo risponderà alle richieste DNS?
3. Il PC conosce già l'indirizzo del server?
4. Quali informazioni dovrà ottenere tramite DHCP?

---

# 4. Fase 2 — Configurazione del Server

Selezionare il Server e aprire:

**Desktop → IP Configuration**

Configurare:

```text
IP Address:      192.168.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1   (se presente un router)
DNS Server:      192.168.10.10
```

Se non è presente un router, il campo Default Gateway può essere lasciato vuoto per le verifiche locali.

---

# 5. Fase 3 — Configurazione del servizio DHCP

Sul Server:

**Services → DHCP**

Attivare il servizio DHCP.

Creare un pool, ad esempio:

```text
Pool Name:       LAN10
Default Gateway: 192.168.10.1
DNS Server:      192.168.10.10
Start IP Address: 192.168.10.100
Subnet Mask:      255.255.255.0
Maximum Users:    50
```

Se non è presente un router nella topologia, lasciare il gateway vuoto o non utilizzato, secondo la versione di Packet Tracer.

Salvare/attivare il pool.

## 5.1 Configurare PC0 come client DHCP

Aprire:

**PC0 → Desktop → IP Configuration**

Selezionare:

**DHCP**

Attendere qualche secondo.

Il PC dovrebbe ricevere automaticamente:

- indirizzo IP;
- subnet mask;
- gateway, se configurato;
- DNS server.

## 5.2 Verifica

Aprire il Command Prompt del PC.

Usare:

```text
ipconfig
```

Annotare:

```text
IP address: __________________
Subnet mask: __________________
Default gateway: ______________
DNS server: ___________________
```

### Domande

- L'indirizzo appartiene alla rete `192.168.10.0/24`?
- È stato assegnato l'indirizzo previsto dal pool?
- Il DNS ricevuto è `192.168.10.10`?
- Quale fase della sequenza DHCP ha permesso al client di ottenere la configurazione?

---

# 6. Fase 4 — Osservare DHCP con Simulation Mode

Passare a **Simulation Mode**.

Impostare, se utile, i filtri in modo da osservare il traffico DHCP.

Rinnovare la configurazione IP del PC oppure ripetere l'attivazione DHCP.

Osservare la sequenza:

```text
DHCP Discover
      ↓
DHCP Offer
      ↓
DHCP Request
      ↓
DHCP ACK
```

## Esercizio di osservazione

Compilare la tabella:

| Messaggio | Chi lo invia? | A chi? | Scopo |
|---|---|---|---|
| Discover | | | |
| Offer | | | |
| Request | | | |
| ACK | | | |

### Domanda chiave

**Perché il client deve prima cercare un server DHCP invece di comunicare direttamente con un indirizzo IP già noto?**

---

# 7. Fase 5 — Configurazione DNS locale

Sul Server:

**Services → DNS**

Attivare il servizio DNS.

Creare un record di tipo **A**.

Esempio:

```text
Name:    server.its.local
Address: 192.168.10.10
Type:    A
```

Il significato è:

```text
server.its.local  →  192.168.10.10
```

## 7.1 Verifica dal PC

Controllare che PC0 abbia ricevuto:

```text
DNS Server: 192.168.10.10
```

Se la versione di Packet Tracer in uso supporta `nslookup`, provare:

```text
nslookup server.its.local
```

Se `nslookup` non è disponibile nella versione utilizzata, effettuare la verifica attraverso gli strumenti DNS/Simulation Mode disponibili nel simulatore.

### Risultato atteso

Il nome dovrebbe essere risolto nell'indirizzo:

```text
192.168.10.10
```

---

# 8. Fase 6 — Verifica ping e DNS

Eseguire:

```text
ping 192.168.10.10
```

Poi, quando supportato:

```text
ping server.its.local
```

Confrontare i due test.

## Domande

1. Il ping verso l'indirizzo IP funziona?
2. La risoluzione DNS funziona?
3. Se il ping all'IP funziona ma il ping al nome fallisce, dove cercheresti il problema?
4. Il problema sarebbe necessariamente un problema di connettività Layer 2?

> **Errore comune:** pensare che DNS e ping siano la stessa cosa. Il ping può verificare la raggiungibilità IP, mentre DNS si occupa della risoluzione dei nomi.

---

# 9. Fase 7 — Dal nome al servizio

Aggiungere, se disponibile nella versione di Packet Tracer, un servizio HTTP sul Server.

Aprire:

**Services → HTTP**

Attivare HTTP.

Dal PC utilizzare il browser e provare a raggiungere:

```text
http://192.168.10.10
```

Poi, se la risoluzione DNS è correttamente disponibile:

```text
http://server.its.local
```

Il flusso concettuale diventa:

```text
PC
 │
 ├── DHCP → ottiene configurazione IP
 │
 ├── DNS  → risolve server.its.local
 │
 └── HTTP → comunica con il server
```

Questo è il punto centrale del laboratorio: **i servizi di rete lavorano insieme**.

---

# 10. Fase 8 — Osservare TCP e UDP

In Simulation Mode osservare almeno due tipi di traffico.

## 10.1 TCP

Individuare una comunicazione TCP, ad esempio verso il servizio HTTP.

Cercare la sequenza iniziale:

```text
SYN
SYN-ACK
ACK
```

Questa sequenza rappresenta il **three-way handshake**.

### Domande

- Chi inizia la connessione?
- Quale host risponde?
- Quale messaggio completa l'handshake?
- Perché TCP utilizza una fase di instaurazione della connessione?

## 10.2 UDP

Osservare il traffico DHCP e/o DNS.

Verificare che UDP non utilizzi lo stesso handshake TCP.

### Domanda

Perché un protocollo come DHCP utilizza UDP nella fase iniziale della comunicazione?

---

# 11. Fase 9 — Analisi con Wireshark

> **Importante:** Cisco Packet Tracer dispone di Simulation Mode, ma non è equivalente a una cattura Wireshark reale. Per questa parte utilizzare Wireshark su una rete reale/virtuale oppure un file `.pcap` fornito dal docente.

Aprire la cattura assegnata dal docente.

## 11.1 DHCP

Utilizzare il filtro:

```text
dhcp
```

Individuare:

- Discover;
- Offer;
- Request;
- ACK.

Rispondere:

1. Chi è il client?
2. Quale server risponde?
3. Quale indirizzo viene proposto?
4. Quale indirizzo viene effettivamente assegnato?
5. Quali parametri aggiuntivi vengono comunicati?

---

# 12. Analisi DNS con Wireshark

Filtro:

```text
dns
```

Individuare una query DNS e la relativa risposta.

Compilare:

| Campo | Valore osservato |
|---|---|
| Client | |
| DNS server | |
| Nome richiesto | |
| Tipo record | |
| Risposta | |
| Tempo risposta | |

### Domanda

**Che cosa succede nella rete tra la digitazione di un nome e la ricezione dell'indirizzo IP?**

---

# 13. Analisi TCP con Wireshark

Filtro:

```text
tcp
```

Per una connessione TCP individuare:

```text
SYN
SYN, ACK
ACK
```

Poi cercare i segmenti successivi della comunicazione.

## Esercizio

Scrivere con parole proprie:

> “Che cosa sta succedendo nella rete?”

Non limitarsi a elencare i pacchetti. Ricostruire la sequenza temporale.

---

# 14. Analisi UDP con Wireshark

Filtro:

```text
udp
```

Individuare almeno una comunicazione UDP.

Confrontarla con una comunicazione TCP.

| Caratteristica | TCP | UDP |
|---|---|---|
| Connessione | | |
| Handshake iniziale | | |
| Affidabilità integrata | | |
| Esempio osservato | | |
| Porte | | |

> **Attenzione:** non descrivere TCP semplicemente come “lento” e UDP come “veloce”. La differenza fondamentale riguarda il modello di comunicazione e le garanzie offerte dal protocollo.

---

# 15. Esercizio principale — “Che cosa sta succedendo nella rete?”

Il docente fornisce agli studenti una cattura Wireshark senza spiegazioni preliminari.

L'obiettivo non è riconoscere singolarmente i protocolli, ma **ricostruire una storia della comunicazione**.

## Consegna

Rispondere alle seguenti domande:

1. Quale host avvia la comunicazione?
2. Quale protocollo compare per primo?
3. È presente DHCP?
4. È presente DNS?
5. È presente TCP?
6. È presente UDP?
7. Quale servizio applicativo sembra essere utilizzato?
8. Quali indirizzi IP comunicano?
9. Quali porte sono coinvolte?
10. È possibile individuare una richiesta e una risposta?
11. Ci sono ritrasmissioni o errori evidenti?
12. Qual è la sequenza temporale più probabile?
13. Quale ipotesi formularesti sul comportamento dell'applicazione?
14. Quale ulteriore evidenza cercheresti per confermare l'ipotesi?

### Regola

Non scrivere solamente:

> “C'è un pacchetto DNS.”

Scrivere invece:

> “Il client invia una query DNS al resolver; la risposta restituisce l'indirizzo associato al nome richiesto. Questo permette di ipotizzare che la successiva comunicazione applicativa utilizzi quell'indirizzo.”

---

# 16. Fault Injection — DHCP

Il docente introduce intenzionalmente un errore.

### Fault 1 — DHCP disabilitato

Disattivare temporaneamente il servizio DHCP sul Server.

Dal PC rinnovare la configurazione.

## Osservare

- il PC riceve un indirizzo?
- compare una sequenza DHCP completa?
- quale fase manca?

## Prima di modificare

Scrivere:

```text
Fatto osservato:
________________________________________

Ipotesi:
________________________________________

Test da eseguire:
________________________________________

Risultato atteso:
________________________________________
```

---

# 17. Fault Injection — DNS

### Fault 2 — DNS errato sul client

Impostare temporaneamente sul PC un DNS errato, ad esempio:

```text
192.168.10.200
```

Lasciare invariato l'indirizzo IP del server.

## Test

```text
ping 192.168.10.10
```

Poi provare la risoluzione del nome, se disponibile:

```text
nslookup server.its.local
```

## Obiettivo

Dimostrare sperimentalmente la differenza tra:

```text
connettività IP
```

e

```text
risoluzione DNS
```

---

# 18. Fault Injection — TCP/servizio

### Fault 3 — servizio HTTP non disponibile

Disabilitare temporaneamente HTTP sul Server.

Verificare:

```text
ping 192.168.10.10
```

Poi provare ad aprire il servizio HTTP.

### Domanda

Se il ping funziona ma il servizio HTTP non risponde, è corretto concludere che “la rete non funziona”?

Motivare la risposta.

---

# 19. Fault Injection — combinato

Il docente può introdurre due fault contemporaneamente.

Esempio:

- DNS errato sul PC;
- servizio HTTP disabilitato.

Lo studente deve evitare di cambiare configurazioni casualmente.

### Obiettivo

Separare i problemi:

```text
Problema 1 → DNS
Problema 2 → servizio applicativo
```

Per ciascun problema devono essere raccolte prove indipendenti.

---

# 20. Troubleshooting strutturato

Per ogni fault utilizzare questa sequenza:

### 1. OSSERVA
Che cosa non funziona?

### 2. RACCOGLI LE EVIDENZE
Utilizzare:

- `ipconfig`;
- ping;
- configurazione DHCP;
- configurazione DNS;
- Simulation Mode;
- Wireshark/PCAP;
- eventuali log.

### 3. FORMULA IPOTESI
Esempio:

> “Il client possiede un IP corretto, ma potrebbe avere un DNS errato.”

### 4. TESTA
Eseguire un solo test mirato.

### 5. INTERPRETA
Il risultato conferma o smentisce l'ipotesi?

### 6. CONFERMA O SMENTISCI
Non modificare la configurazione finché non esiste un'evidenza sufficiente.

### 7. RISOLVI
Applicare la correzione minima necessaria.

### 8. VERIFICA
Ripetere il test iniziale.

### 9. DOCUMENTA
Scrivere:

```text
Sintomo:
Causa:
Evidenza:
Test:
Correzione:
Verifica finale:
```

---

# 21. Utilizzare l'AI nel troubleshooting

L'AI non deve essere utilizzata come un “generatore di comandi”.

Il suo compito nel laboratorio è aiutare lo studente a:

- organizzare le evidenze;
- formulare ipotesi;
- proporre test discriminanti;
- interpretare risultati;
- evitare conclusioni premature;
- documentare la diagnosi.

## Prompt base

```text
Sto analizzando un problema di rete in un laboratorio didattico.

Scenario:
[descrivi topologia e servizi]

Sintomo osservato:
[descrivi esattamente cosa non funziona]

Configurazione disponibile:
[IP, subnet mask, gateway, DNS, servizi]

Evidenze raccolte:
[output dei comandi, ping, Simulation Mode o Wireshark]

Voglio che tu lavori come assistente di troubleshooting.

NON inventare dati mancanti.
NON modificare direttamente la configurazione.
NON fornire subito la soluzione finale.

Segui questa metodologia:
OSSERVA → FORMULA IPOTESI → TESTA → INTERPRETA → CONFERMA O SMENTISCI → RISOLVI → VERIFICA → DOCUMENTA

Per ogni ipotesi indicami:
1. quale evidenza la supporta;
2. quale evidenza la smentirebbe;
3. un solo test da eseguire;
4. il risultato atteso;
5. come interpretare il risultato.
```

---

# 22. Prompt AI per analizzare un PCAP

Utilizzare il seguente prompt quando il docente fornisce una cattura Wireshark:

```text
Analizza questa cattura di rete come un tecnico di networking.

Obiettivo: ricostruire cosa sta succedendo nella rete senza inventare informazioni.

Per ogni comunicazione significativa identifica:
- host sorgente;
- host destinazione;
- protocollo;
- porta sorgente;
- porta destinazione;
- direzione della comunicazione;
- sequenza temporale;
- richiesta e risposta, quando presenti.

Cerca in particolare:
- DHCP Discover/Offer/Request/ACK;
- query e response DNS;
- TCP SYN/SYN-ACK/ACK;
- comunicazioni UDP;
- eventuali ritrasmissioni, errori o anomalie.

Se una conclusione non è dimostrabile dalla cattura, dichiaralo esplicitamente.

Organizza la risposta in:
1. fatti osservati;
2. ipotesi;
3. evidenze a supporto;
4. evidenze mancanti;
5. test consigliati;
6. conclusione provvisoria.
```

---

# 23. Attività AI — confronto tra ipotesi

Per uno dei Fault Injection assegnati, chiedere all'AI di produrre **almeno tre ipotesi alternative**.

Esempio:

```text
Il PC riceve un indirizzo IP ma non riesce a raggiungere
server.its.local.
```

Possibili categorie:

1. problema DNS;
2. problema di connettività verso il server;
3. servizio DNS non attivo;
4. configurazione errata del record;
5. problema applicativo.

Lo studente deve poi scegliere il test più efficace per distinguere le ipotesi.

---

# 24. Scheda di troubleshooting

| Campo | Risposta |
|---|---|
| Fault assegnato | |
| Sintomo | |
| Evidenza 1 | |
| Evidenza 2 | |
| Ipotesi 1 | |
| Ipotesi 2 | |
| Ipotesi 3 | |
| Test scelto | |
| Risultato | |
| Ipotesi confermata | |
| Correzione | |
| Verifica finale | |
| Evidenza Wireshark/Simulation Mode | |
| Supporto AI utilizzato | |
| Conclusione | |

---

# 25. Challenge finale

Il docente prepara una rete funzionante e introduce **due fault differenti**.

Lo studente deve:

1. individuare il sintomo;
2. verificare la configurazione IP;
3. verificare DHCP;
4. verificare DNS;
5. verificare la raggiungibilità IP;
6. osservare il traffico;
7. formulare almeno due ipotesi;
8. scegliere test discriminanti;
9. correggere i problemi;
10. verificare nuovamente il funzionamento;
11. documentare la soluzione;
12. utilizzare l'AI esclusivamente come supporto metodologico e interpretativo.

## Consegna finale

Produrre un breve report:

```text
Scenario
--------

Fault rilevati
--------------

Evidenze
--------

Ipotesi
-------

Test eseguiti
-------------

Diagnosi
--------

Correzione
----------

Verifica finale
---------------

Analisi del traffico
--------------------

Supporto dell'AI
----------------

Conclusione
-----------
```

---

# 26. Checklist dello studente

Prima di consegnare verificare:

- [ ] Il PC riceve correttamente un indirizzo tramite DHCP.
- [ ] Il PC riceve subnet mask e DNS corretti.
- [ ] Il servizio DNS è attivo.
- [ ] Il record A è configurato correttamente.
- [ ] Il nome `server.its.local` viene risolto correttamente, quando il comando è disponibile.
- [ ] Il server risponde al ping IP.
- [ ] Il servizio applicativo è raggiungibile quando attivo.
- [ ] Ho osservato DHCP in Simulation Mode.
- [ ] Ho osservato DNS in Simulation Mode o Wireshark.
- [ ] Ho osservato un three-way handshake TCP.
- [ ] Ho osservato traffico UDP.
- [ ] Ho analizzato una cattura Wireshark.
- [ ] Ho eseguito almeno un Fault Injection.
- [ ] Ho seguito una metodologia strutturata di troubleshooting.
- [ ] Ho utilizzato l'AI senza inventare dati.
- [ ] Ho documentato fatti, ipotesi, test e conclusioni.

---

# 27. Criteri di valutazione

| Competenza | Evidenza |
|---|---|
| Configurazione DHCP | Client configurato e lease verificato |
| Configurazione DNS | Record e risoluzione verificati |
| Comprensione TCP/UDP | Analisi corretta del traffico |
| Lettura del traffico | Ricostruzione coerente della sequenza |
| Troubleshooting | Metodo strutturato e test pertinenti |
| Fault Injection | Capacità di individuare la causa |
| Uso dell'AI | Prompt, evidenze e interpretazione corretti |
| Documentazione | Report tecnico chiaro e riproducibile |

---

# 28. Conclusione

In questo laboratorio non abbiamo semplicemente configurato DHCP e DNS.

Abbiamo costruito una piccola catena di servizi:

```text
DHCP
 ↓
configurazione del client
 ↓
DNS
 ↓
risoluzione del nome
 ↓
TCP/UDP
 ↓
servizio applicativo
 ↓
analisi del traffico
 ↓
troubleshooting
```

L'obiettivo professionale è imparare a passare dalla configurazione alla **lettura delle evidenze**.

Quando qualcosa non funziona, la domanda non deve essere soltanto:

> “Quale comando devo dare?”

ma:

> **“Quale evidenza mi permette di capire cosa sta realmente succedendo nella rete?”**
