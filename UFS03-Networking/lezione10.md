# Lezione 10 — Laboratorio: Virtualizzazione, QoS, TFTP/FTP e troubleshooting con AI

**Corso:** UFS 03 — Networking  
**Lezione:** 10  
**Durata laboratorio:** 150 minuti  
**Ambiente:** Cisco Packet Tracer + eventuale ambiente reale/virtualizzato per analisi TFTP/FTP  

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio lo studente sarà in grado di:

- progettare una semplice rete con segmentazione del traffico;
- distinguere traffico utenti, server e voce;
- ragionare sul ruolo di virtual switch e virtual NIC;
- identificare traffico sensibile a latenza e jitter;
- spiegare perché la QoS diventa importante in presenza di congestione;
- osservare gli effetti della congestione sul traffico;
- utilizzare TFTP e FTP per trasferire file/configurazioni in uno scenario controllato;
- riconoscere i principali rischi di TFTP e FTP;
- applicare una metodologia strutturata di troubleshooting;
- utilizzare l'AI come supporto alla diagnosi senza delegare all'AI la decisione tecnica.

> **Nota didattica:** Packet Tracer permette di simulare molti concetti di switching, VLAN e servizi. La virtualizzazione di un hypervisor reale e alcune caratteristiche avanzate di QoS possono invece richiedere un ambiente esterno o una simulazione concettuale. In questo laboratorio distinguiamo quindi ciò che osserviamo direttamente dal simulatore da ciò che analizziamo come modello architetturale.

---

# 2. Scenario aziendale

Una piccola azienda dispone di:

- alcuni server virtualizzati;
- postazioni di lavoro;
- telefoni IP o traffico VoIP simulato;
- una rete Ethernet con VLAN;
- un servizio per il trasferimento di file/configurazioni.

L'obiettivo è progettare una rete nella quale i diversi flussi siano separati e ragionare su quali debbano avere maggiore priorità in presenza di congestione.

### Architettura logica

```text
                         +----------------+
                         |     Router     |
                         |   / Gateway    |
                         +-------+--------+
                                 |
                              Trunk
                                 |
                         +-------+--------+
                         |     Switch     |
                         +---+---+---+----+
                             |   |   |
                             |   |   +---------- Server / servizi
                             |   |
                             |   +-------------- PC utenti
                             |
                             +------------------ VoIP

        VLAN 10 = UTENTI
        VLAN 20 = SERVER
        VLAN 30 = VOICE
        VLAN 99 = MANAGEMENT
```

---

# 3. Piano di indirizzamento

| VLAN | Nome | Rete | Gateway | Funzione |
|---:|---|---|---|---|
| 10 | UTENTI | 192.168.10.0/24 | 192.168.10.1 | PC utenti |
| 20 | SERVER | 192.168.20.0/24 | 192.168.20.1 | Server |
| 30 | VOICE | 192.168.30.0/24 | 192.168.30.1 | VoIP |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 | Gestione apparati |

Esempio di assegnazione:

- PC0 → `192.168.10.10/24`
- Server0 → `192.168.20.10/24`
- Server1 → `192.168.20.20/24`
- dispositivo VoIP → `192.168.30.10/24`
- management switch → `192.168.99.2/24`

---

# 4. Fase 1 — Progettazione della rete virtualizzata

## 4.1 Dal server fisico alla VM

Prima di configurare Packet Tracer, ragioniamo sul modello:

```text
Server fisico
      |
      | NIC fisica
      v
+-------------------+
|     Hypervisor    |
+-------------------+
   |       |       |
   v       v       v
  VM1     VM2     VM3
   |       |       |
 vNIC    vNIC    vNIC
   \       |       /
    +-- vSwitch --+
           |
       NIC fisica
           |
        Switch
```

### Attività

Rispondere alle domande:

1. Qual è la differenza tra NIC fisica e vNIC?
2. Quale funzione svolge il virtual switch?
3. Dove può essere applicata la segmentazione VLAN?
4. Perché una VM non deve necessariamente avere una NIC fisica dedicata?
5. Quali vantaggi porta la virtualizzazione dal punto di vista della gestione?

### Output richiesto

Realizzare un piccolo schema della rete indicando:

- server fisico;
- hypervisor;
- almeno due VM;
- vNIC;
- vSwitch;
- collegamento verso lo switch fisico;
- VLAN associate alle VM.

---

# 5. Fase 2 — Creazione delle VLAN

Configurare sullo switch:

```cisco
enable
configure terminal

vlan 10
 name UTENTI
exit

vlan 20
 name SERVER
exit

vlan 30
 name VOICE
exit

vlan 99
 name MANAGEMENT
exit
```

Verificare:

```cisco
show vlan brief
```

### Domande

- Quale problema risolve la segmentazione?
- Una VLAN rappresenta una rete IP oppure un dominio Layer 2?
- Perché può essere utile separare server e utenti?
- Perché il traffico voce può essere separato dal traffico dati?

---

# 6. Fase 3 — Configurazione delle porte

Configurare alcune porte come access port.

Esempio:

```cisco
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
exit

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
exit
```

Per un dispositivo VoIP, se il modello Packet Tracer utilizzato lo supporta, osservare la possibilità di associare una VLAN voce alla porta.

Verificare:

```cisco
show vlan brief
show interfaces status
```

> **Attenzione:** la disponibilità e la sintassi di alcune funzioni VoIP possono cambiare in base al dispositivo scelto in Packet Tracer. Non assumere che ogni modello supporti esattamente gli stessi comandi.

---

# 7. Fase 4 — Trunk e inter-VLAN

Configurare il collegamento verso il router come trunk:

```cisco
interface gigabitEthernet 0/1
 switchport mode trunk
exit
```

Verificare:

```cisco
show interfaces trunk
```

Se si utilizza Router-on-a-Stick, configurare le subinterface sul router.

Esempio:

```cisco
interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit

interface gigabitEthernet 0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
exit
```

Verificare:

```cisco
show ip interface brief
show ip route
```

---

# 8. Fase 5 — Verifica della segmentazione

Configurare gli host con gli indirizzi previsti e verificare:

```text
PC0 → Gateway VLAN 10
PC0 → Server0 VLAN 20
PC0 → Server1 VLAN 20
PC0 → dispositivo VoIP VLAN 30
```

Eseguire i test uno alla volta.

### Domande

1. Il PC riesce a raggiungere il proprio gateway?
2. Riesce a raggiungere il server?
3. Attraverso quale dispositivo passa il traffico tra VLAN diverse?
4. Cosa succede se il trunk viene configurato in modo errato?

---

# 9. Fase 6 — QoS: ragionare sul traffico

Consideriamo tre flussi:

| Flusso | Esempio | Sensibilità |
|---|---|---|
| Dati | download di un file | soprattutto banda/tempo |
| VoIP | chiamata audio | latenza + jitter + perdita |
| Video | videoconferenza | banda + latenza + perdita |

### Attività

Ordinare i flussi in base alla priorità che assegnereste in una rete congestionata e motivare la scelta.

> **Tip del docente:** non confondere "traffico importante" con "traffico che deve sempre avere priorità massima". La QoS deve partire dai requisiti dell'applicazione e dalle policy dell'organizzazione.

---

# 10. Fase 7 — Simulazione della congestione

Packet Tracer non rappresenta tutte le funzioni QoS di un apparato enterprise reale. Per questo esercizio l'obiettivo principale è **osservare e ragionare sugli effetti della congestione**, non costruire una policy QoS completa da produzione.

## Attività A — Generazione del traffico

Creare più flussi contemporanei tra host e server.

Utilizzare, dove disponibile, la modalità **Simulation** di Packet Tracer per osservare i pacchetti.

Confrontare:

- traffico ICMP;
- traffico TCP;
- traffico UDP;
- frequenza dei pacchetti;
- comportamento in presenza di ritardi o perdita simulata.

## Attività B — Analisi

Rispondere:

1. Quale flusso risente maggiormente del jitter?
2. Perché una perdita di pacchetti può essere percepita diversamente da un download rispetto a una chiamata VoIP?
3. Cosa significa prioritizzare un traffico?
4. Quali informazioni servirebbero prima di configurare una vera policy QoS?

---

# 11. Fase 8 — TFTP

Il TFTP viene spesso utilizzato in contesti di rete per trasferimenti semplici di file, ad esempio configurazioni o immagini, quando l'ambiente e il processo lo prevedono.

> **Nota di sicurezza:** TFTP non deve essere considerato equivalente a un trasferimento sicuro autenticato. È un protocollo semplice e va utilizzato solo in contesti appropriati e controllati.

## Scenario

Supponiamo di avere:

```text
PC amministratore
      |
    Switch
      |
   Router
      |
   Server TFTP
```

Sul dispositivo Cisco, verificare la possibilità di salvare una configurazione su un server TFTP presente nello scenario.

Esempio:

```cisco
copy running-config tftp:
```

Il dispositivo richiederà l'indirizzo del server e il nome del file.

### Verifica

Controllare che il file sia effettivamente presente sul server TFTP.

> **Attenzione:** il supporto e la modalità di utilizzo del servizio TFTP dipendono dal dispositivo/server scelto in Packet Tracer.

---

# 12. Fase 9 — FTP

FTP è orientato al trasferimento di file tra client e server.

Creare o utilizzare un server FTP disponibile in Packet Tracer, se presente nella versione utilizzata.

Configurare:

- servizio FTP;
- almeno un utente;
- password di laboratorio;
- file di test.

Dal client, verificare la connessione FTP utilizzando gli strumenti disponibili nel dispositivo scelto.

### Domande

1. Qual è la differenza concettuale tra TFTP e FTP?
2. FTP offre autenticazione?
3. FTP deve essere considerato automaticamente sicuro?
4. Perché oggi, in molti contesti, si preferiscono protocolli più moderni e sicuri per il trasferimento dei file?

---

# 13. Fault Injection — Introduciamo i problemi

Ora la rete funziona. È il momento di romperla intenzionalmente.

La regola del laboratorio è:

> **non correggere immediatamente il problema. Prima raccogli le evidenze.**

---

## Fault 1 — VLAN errata

Modificare la VLAN della porta del server.

Esempio:

```cisco
interface fastEthernet 0/2
 switchport access vlan 10
```

invece della VLAN 20 prevista.

### Obiettivo

Determinare perché il server non è più raggiungibile correttamente.

Comandi utili:

```cisco
show vlan brief
show interfaces status
show mac address-table
```

---

## Fault 2 — Trunk errato

Rimuovere o alterare la configurazione trunk del collegamento verso il router.

Verificare:

```cisco
show interfaces trunk
```

### Domanda

Quale differenza osservi tra un problema di porta access e un problema di trunk?

---

## Fault 3 — Gateway errato

Modificare il gateway di un host.

Esempio errato:

```text
IP:      192.168.20.10
Mask:    255.255.255.0
Gateway: 192.168.20.254
```

### Obiettivo

Capire perché alcuni test locali possono funzionare mentre la comunicazione verso altre reti fallisce.

---

## Fault 4 — Servizio TFTP non disponibile

Disabilitare il servizio TFTP oppure utilizzare temporaneamente l'indirizzo IP errato del server.

### Verificare

- connettività IP;
- indirizzo del server;
- servizio attivo;
- eventuale errore durante il trasferimento.

---

## Fault 5 — Servizio FTP non disponibile

Disabilitare FTP o modificare le credenziali di laboratorio.

### Obiettivo

Distinguere:

```text
problema di rete
        ↓
problema di servizio
        ↓
problema di autenticazione
```

---

# 14. Metodo di troubleshooting

Utilizzare sempre questa sequenza:

```text
OSSERVA
   ↓
RACCOGLI LE EVIDENZE
   ↓
FORMULA IPOTESI
   ↓
TESTA
   ↓
INTERPRETA
   ↓
CONFERMA O SMENTISCI
   ↓
RISOLVI
   ↓
VERIFICA
   ↓
DOCUMENTA
```

### Regola fondamentale

Non partire dal comando che "forse risolve".

Partire dalla domanda:

> **Quale evidenza mi permetterebbe di distinguere questa ipotesi dalle altre?**

---

# 15. Troubleshooting con AI

L'AI deve essere utilizzata come **assistente al ragionamento diagnostico**.

Non bisogna scrivere:

> "La rete non funziona, risolvila."

Il prompt deve fornire:

- topologia;
- indirizzamento;
- configurazione rilevante;
- sintomo;
- comandi già eseguiti;
- output ottenuti;
- vincoli dell'ambiente;
- eventuali modifiche introdotte con Fault Injection.

## Prompt base

```text
Agisci come tutor di troubleshooting di rete.

Scenario:
[descrivi la topologia]

Indirizzamento:
[riporta IP, mask, gateway e VLAN]

Servizi:
[indica TFTP, FTP o altri servizi coinvolti]

Sintomo osservato:
[descrivi esattamente cosa non funziona]

Evidenze raccolte:
[incolla gli output dei comandi]

Fault Injection introdotto:
[indica il fault, se noto]

Procedi secondo questa metodologia:
OSSERVA → RACCOGLI LE EVIDENZE → FORMULA IPOTESI → TESTA →
INTERPRETA → CONFERMA O SMENTISCI → RISOLVI → VERIFICA → DOCUMENTA

Non inventare dati mancanti.
Distingui fatti, ipotesi e conclusioni.
Proponi un solo test diagnostico alla volta.
Per ogni test indica:
1. comando/azione;
2. risultato atteso;
3. interpretazione del risultato.
Non proporre subito la soluzione definitiva.
```

---

# 16. Attività AI — Confronto tra ipotesi

Ogni gruppo deve produrre almeno **tre ipotesi** per un fault assegnato.

Esempio:

> Il PC non raggiunge il server FTP.

Possibili ipotesi:

1. IP/gateway errato;
2. VLAN o trunk errati;
3. servizio FTP non attivo;
4. credenziali errate;
5. server non raggiungibile.

Per ogni ipotesi indicare:

| Ipotesi | Evidenza necessaria | Test | Risultato | Esito |
|---|---|---|---|---|
| H1 | ... | ... | ... | confermata/smentita |
| H2 | ... | ... | ... | confermata/smentita |
| H3 | ... | ... | ... | confermata/smentita |

---

# 17. Challenge finale — Piccola azienda virtualizzata

La rete finale deve rappresentare una piccola azienda con:

- VLAN utenti;
- VLAN server;
- VLAN voce;
- VLAN management;
- almeno un server TFTP;
- almeno un servizio FTP;
- traffico dati e traffico voce simulato;
- documentazione della rete virtualizzata.

## Scenario iniziale

Il docente introduce **due fault contemporaneamente**.

Gli studenti non conoscono in anticipo quali siano.

Esempi possibili:

- VLAN errata + servizio FTP disattivato;
- trunk errato + gateway errato;
- TFTP non raggiungibile + porta configurata nella VLAN sbagliata.

### Obiettivo

Individuare entrambi i fault senza procedere per tentativi casuali.

---

# 18. Scheda di troubleshooting

Compilare durante il challenge.

```text
SINTOMO:

____________________________________________

EVIDENZE RACCOLTE:

____________________________________________

____________________________________________

IPOTESI 1:

____________________________________________

TEST:

____________________________________________

RISULTATO:

____________________________________________

IPOTESI 2:

____________________________________________

TEST:

____________________________________________

RISULTATO:

____________________________________________

CAUSA CONFERMATA:

____________________________________________

CORREZIONE:

____________________________________________

VERIFICA FINALE:

____________________________________________

LEZIONE APPRESA:

____________________________________________
```

---

# 19. Consegna finale

Ogni gruppo deve consegnare:

1. schema della rete;
2. piano VLAN e indirizzamento;
3. configurazione essenziale degli apparati;
4. evidenze delle verifiche;
5. tabella di troubleshooting;
6. analisi dei Fault Injection;
7. prompt AI utilizzato;
8. risposta dell'AI con relativa valutazione critica;
9. breve conclusione tecnica.

---

# 20. Checklist dello studente

- [ ] Ho progettato la separazione tra utenti, server, voce e management.
- [ ] So spiegare hypervisor, VM, vNIC e virtual switch.
- [ ] Ho configurato le VLAN necessarie.
- [ ] Ho verificato il trunk.
- [ ] Ho verificato la connettività inter-VLAN.
- [ ] So spiegare latenza, jitter, packet loss e bandwidth.
- [ ] Ho identificato il traffico più sensibile ai problemi di qualità.
- [ ] Ho osservato il traffico in Simulation Mode dove possibile.
- [ ] Ho configurato/verificato un servizio TFTP.
- [ ] Ho configurato/verificato un servizio FTP.
- [ ] Ho introdotto almeno un Fault Injection.
- [ ] Ho raccolto evidenze prima di modificare la configurazione.
- [ ] Ho utilizzato l'AI con un prompt strutturato.
- [ ] Ho verificato autonomamente le conclusioni dell'AI.
- [ ] Ho documentato la soluzione.

---

# 21. Criteri di valutazione

| Area | Peso indicativo |
|---|---:|
| Progettazione e segmentazione | 20% |
| Configurazione VLAN/trunk | 20% |
| Comprensione virtualizzazione e QoS | 15% |
| TFTP/FTP | 15% |
| Troubleshooting | 15% |
| Utilizzo consapevole dell'AI | 10% |
| Documentazione tecnica | 5% |

---

# Conclusione

In questa esercitazione non ci interessa soltanto "far funzionare" la rete.

L'obiettivo è imparare a ragionare su una rete nella quale convivono **infrastruttura virtualizzata, traffico dati, traffico sensibile alla qualità e servizi di trasferimento**.

Il passaggio fondamentale è quello già utilizzato nelle lezioni precedenti:

> **non indovinare la causa: raccogli evidenze, formula ipotesi, testa e verifica.**

L'AI può accelerare questo processo, ma la responsabilità della diagnosi e della verifica rimane dello studente.
