# Lezione 11 — Laboratorio finale: Network Security, Fault Injection, Troubleshooting e IA

**Corso:** UFS 03 — Networking  
**Lezione:** 11  
**Durata laboratorio:** 170 minuti  
**Ambiente consigliato:** Cisco Packet Tracer + strumento AI disponibile agli studenti

---

## 1. Obiettivo del laboratorio

Questo laboratorio rappresenta la sintesi del percorso svolto durante il corso.

L'obiettivo non è più semplicemente configurare una rete, ma **ragionare come un tecnico di rete** davanti a un problema reale.

La rete viene consegnata funzionante e successivamente vengono introdotti più guasti contemporaneamente.

Gli studenti devono:

- osservare i sintomi;
- raccogliere evidenze;
- individuare il livello o il servizio coinvolto;
- formulare ipotesi;
- verificare le ipotesi con test mirati;
- correggere i problemi;
- verificare nuovamente la connettività;
- documentare ciò che è stato trovato e risolto;
- utilizzare l'IA come supporto al ragionamento diagnostico.

> **Regola fondamentale:** l'IA non deve essere utilizzata come semplice generatore di comandi.
>
> Lo studente deve prima raccogliere informazioni e formulare una propria ipotesi.

---

# 2. Metodologia professionale

Durante tutto il laboratorio utilizzare questa sequenza:

```text
1. Definire il problema
2. Raccogliere informazioni
3. Isolare il livello interessato
4. Formulare ipotesi
5. Verificare l'ipotesi
6. Applicare la soluzione
7. Verificare nuovamente
8. Documentare
```

Una versione più operativa, utilizzata nelle lezioni precedenti, è:

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

---

# 3. Scenario aziendale

Immagina di essere il tecnico responsabile della rete di una piccola azienda.

La rete comprende:

- VLAN 10 — AMMINISTRAZIONE
- VLAN 20 — PRODUZIONE
- VLAN 30 — SERVER
- VLAN 40 — GUEST
- VLAN 99 — MANAGEMENT

Sono presenti:

- uno switch principale;
- un router per l'inter-VLAN routing;
- PC appartenenti a VLAN differenti;
- un server aziendale;
- servizi DHCP e DNS;
- accesso Internet simulato;
- NAT/PAT;
- collegamenti trunk tra apparati.

La topologia di riferimento può essere:

```text
 PC VLAN 10 ----+
                |
 PC VLAN 20 ----+---- SW1 ===== trunk ===== R1 ===== WAN/Internet
                |
 PC VLAN 30 ----+          |
                |          +---- Server
 PC VLAN 40 ----+

              VLAN 99
             Management
```

`=====` indica un collegamento trunk.

---

# 4. Piano di indirizzamento di riferimento

| VLAN | Nome | Rete | Gateway |
|---|---|---|---|
| 10 | AMMINISTRAZIONE | 192.168.10.0/24 | 192.168.10.1 |
| 20 | PRODUZIONE | 192.168.20.0/24 | 192.168.20.1 |
| 30 | SERVER | 192.168.30.0/24 | 192.168.30.1 |
| 40 | GUEST | 192.168.40.0/24 | 192.168.40.1 |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 |

Esempio:

```text
PC-ADMIN   192.168.10.10/24
PC-PROD    192.168.20.10/24
SERVER     192.168.30.10/24
PC-GUEST   192.168.40.10/24
```

---

# 5. Fase 1 — Verifica della rete funzionante

Prima di introdurre i guasti, verificare che la rete funzioni correttamente.

## 5.1 Verifica degli host

Da ogni PC controllare:

```text
ipconfig /all
```

Verificare:

- indirizzo IP;
- subnet mask;
- default gateway;
- DNS.

## 5.2 Verifica Layer 2

Sullo switch:

```text
show vlan brief
show interfaces trunk
show mac address-table
show interfaces status
```

Domande:

1. Le VLAN esistono?
2. Le porte degli host appartengono alla VLAN corretta?
3. Il trunk è attivo?
4. Lo switch sta imparando gli indirizzi MAC?

## 5.3 Verifica Layer 3

Sul router:

```text
show ip interface brief
show ip route
```

Verificare:

- interfacce attive;
- gateway delle VLAN;
- reti direttamente connesse;
- eventuali route statiche.

## 5.4 Test di connettività

Eseguire progressivamente:

```text
ping 127.0.0.1
ping <proprio-gateway>
ping <gateway-altra-VLAN>
ping <server>
```

Non partire direttamente dal ping al server: il test deve aiutare a **localizzare il punto di rottura**.

---

# 6. Fase 2 — Fault Injection multipla

Il docente introduce più guasti contemporaneamente.

Gli studenti **non devono sapere in anticipo quali siano**.

Possibili fault:

### Fault A — VLAN errata

Una porta destinata alla VLAN 20 viene assegnata alla VLAN 10.

### Fault B — Trunk errato

Una VLAN viene esclusa dal trunk oppure la porta non è più configurata come trunk.

### Fault C — Gateway errato

Un PC della VLAN 20 riceve un gateway non appartenente alla propria rete.

Esempio:

```text
IP:      192.168.20.10
Mask:    255.255.255.0
Gateway: 192.168.10.1
```

### Fault D — DHCP errato

Il pool DHCP della VLAN 20 contiene un gateway o una rete errata.

### Fault E — DNS non funzionante

Il client possiede un indirizzo IP corretto ma non riesce a risolvere il nome del server.

### Fault F — Routing errato

Una route necessaria è assente o configurata in modo non corretto.

### Fault G — NAT/PAT errato

La rete interna raggiunge il router ma non riesce a raggiungere la rete esterna simulata.

> Il docente può scegliere 3–5 fault contemporaneamente in funzione del livello della classe.

---

# 7. Regola del laboratorio

Quando viene rilevato un problema, **non modificare immediatamente la configurazione**.

Prima compilare:

```text
SINTOMO:

EVIDENZE:

LIVELLO/SERVIZIO INTERESSATO:

IPOTESI:

TEST DA ESEGUIRE:

RISULTATO DEL TEST:

CONCLUSIONE:

AZIONE CORRETTIVA:

VERIFICA FINALE:
```

---

# 8. Fase 3 — Troubleshooting senza IA

Prima dell'utilizzo dell'IA ogni gruppo deve affrontare autonomamente almeno il primo problema.

## Domande guida

1. Quale host non funziona?
2. Quale destinazione non è raggiungibile?
3. Il problema riguarda un solo host o più host?
4. Il problema riguarda una sola VLAN?
5. Il gateway risponde?
6. Il server risponde tramite IP?
7. Il server risponde tramite nome?
8. Il routing è corretto?
9. Il problema è locale oppure attraversa più dispositivi?
10. Quale evidenza conferma l'ipotesi?

---

# 9. Fase 4 — Utilizzo dell'IA

A questo punto l'IA può essere utilizzata come **assistente diagnostico**.

## Regola fondamentale

Non utilizzare prompt come:

```text
Qual è il comando per risolvere questo problema?
```

oppure:

```text
Dimmi come configurare il router.
```

Questi prompt saltano completamente il processo diagnostico.

---

# 10. Prompt base per il troubleshooting

Utilizzare questo modello:

```text
Agisci come un network engineer senior e come supporto al troubleshooting.

Non modificare direttamente la configurazione e non fornire subito una soluzione finale.

Segui questa metodologia:

OSSERVA → RACCOGLI LE EVIDENZE → FORMULA IPOTESI → TESTA → INTERPRETA → CONFERMA O SMENTISCI → RISOLVI → VERIFICA → DOCUMENTA

SCENARIO:
[descrizione della rete]

TOPOLOGIA:
[descrizione della topologia]

SINTOMO:
[descrizione del problema]

OUTPUT RACCOLTI:
[incollare gli output dei comandi]

MIA IPOTESI:
[ipotesi formulata dallo studente]

Analizza esclusivamente le informazioni fornite.

1. Se mancano dati importanti, indicami quali dati raccogliere.
2. Valuta la mia ipotesi.
3. Proponi le possibili cause ordinate per probabilità.
4. Indica quale verifica eseguire per prima e perché.
5. Per ogni test indica:
   - comando o azione;
   - risultato atteso;
   - interpretazione del risultato.
6. Non inventare output non forniti.
7. Non dare per confermata un'ipotesi senza una verifica.
8. Dopo ogni test attendi il nuovo risultato prima di procedere.
9. Solo dopo aver identificato la causa proponi l'azione correttiva.
10. Concludi con una verifica finale e una breve documentazione del problema risolto.
```

---

# 11. Esempio di interazione corretta con l'IA

### Studente

```text
Il PC della VLAN 20 non riesce a raggiungere il server 192.168.30.10.

Il PC ha:
IP: 192.168.20.10
Mask: 255.255.255.0
Gateway: 192.168.20.1

Il ping al gateway funziona.
Il ping al server non funziona.

La mia ipotesi è che il problema possa essere nel routing.
```

### Richiesta all'IA

```text
Valuta la mia ipotesi e indicami quale verifica eseguire per prima.
Non fornire ancora la soluzione.
```

L'obiettivo è costringere l'IA a **ragionare sulle evidenze**, non a sostituirsi allo studente.

---

# 12. Fase 5 — Analisi multilivello

Per ogni problema gli studenti devono tentare di identificare il livello coinvolto.

| Livello | Domanda diagnostica | Esempi |
|---|---|---|
| L1 | Il collegamento fisico funziona? | porta down, cavo |
| L2 | La comunicazione locale è corretta? | VLAN, trunk, MAC |
| L3 | L'indirizzamento e il routing sono corretti? | IP, gateway, route |
| Servizi | I servizi funzionano? | DHCP, DNS, NAT |
| Sicurezza | Esistono restrizioni o configurazioni non sicure? | VLAN, accesso remoto |
| Applicazione | Il servizio richiesto risponde? | server, web, FTP |

---

# 13. Caso speciale — VLAN 20 non raggiunge il server

Il docente presenta questo scenario:

> **Sei il responsabile della rete. Il PC della VLAN 20 non riesce a raggiungere il server, mentre i PC della VLAN 10 funzionano correttamente.**

Gli studenti devono procedere autonomamente.

### Vincoli

Non viene fornita immediatamente la causa.

Lo studente deve determinare se il problema riguarda:

- VLAN;
- porta access;
- trunk;
- gateway;
- inter-VLAN routing;
- routing;
- server;
- DNS;
- ACL, se presenti.

### Sequenza minima consigliata

```text
PC → IP configuration
↓
Gateway VLAN 20
↓
Switch VLAN
↓
Trunk
↓
Gateway/router VLAN 20
↓
Routing
↓
Server
↓
DNS
```

---

# 14. Ultimi 30 minuti — Simulazione di guasto generata dall'IA

L'IA genera lo scenario di troubleshooting.

## Prompt per generare il fault

```text
Agisci come responsabile di una rete aziendale e come generatore di scenari di troubleshooting per studenti ITS Networking.

Genera uno scenario realistico di guasto di rete.

Requisiti:
- utilizza una rete con almeno 3 VLAN;
- inserisci uno o più servizi tra DHCP, DNS e NAT;
- inserisci almeno un problema Layer 2 e/o Layer 3;
- il problema deve essere diagnosticabile attraverso evidenze e comandi Cisco IOS;
- non rivelare immediatamente la causa;
- descrivi soltanto i sintomi iniziali;
- prepara internamente la soluzione corretta per la successiva verifica del docente.

Fornisci allo studente:
1. topologia;
2. piano IP;
3. sintomi osservabili;
4. test iniziali consentiti.

Non fornire la soluzione.
```

---

# 15. Simulazione assegnata

Esempio di scenario:

> **Sei il responsabile della rete. Il PC della VLAN 20 non riesce a raggiungere il server, mentre i PC della VLAN 10 funzionano correttamente.**

Lo studente deve:

1. definire il problema;
2. raccogliere le informazioni;
3. formulare una prima ipotesi;
4. eseguire i test;
5. raccogliere gli output;
6. utilizzare l'IA fornendo le evidenze;
7. confermare o smentire l'ipotesi;
8. correggere il problema;
9. verificare nuovamente la rete;
10. documentare la soluzione.

---

# 16. Fault Injection finale — due guasti contemporanei

Per aumentare la difficoltà il docente può introdurre due fault contemporaneamente.

Esempio:

```text
FAULT 1:
Porta PC VLAN 20 assegnata alla VLAN 10

FAULT 2:
DNS configurato con indirizzo errato
```

In questo caso lo studente deve evitare di concludere che il primo problema trovato sia necessariamente **l'unico problema**.

Dopo ogni correzione deve essere ripetuta la verifica completa.

---

# 17. Scheda di troubleshooting

| Campo | Risposta |
|---|---|
| Gruppo | |
| Data | |
| Scenario | |
| Sintomo iniziale | |
| Host coinvolto | |
| Destinazione | |
| Livello sospetto | |
| Evidenza 1 | |
| Evidenza 2 | |
| Ipotesi | |
| Test | |
| Risultato | |
| Ipotesi confermata? | |
| Causa | |
| Azione correttiva | |
| Verifica finale | |
| Utilizzo dell'IA | |
| Prompt utilizzato | |
| Output utile dell'IA | |
| Eventuale errore dell'IA | |

---

# 18. Regole per l'utilizzo responsabile dell'IA

L'IA può:

- aiutare a organizzare le evidenze;
- suggerire ipotesi;
- proporre test diagnostici;
- spiegare output Cisco IOS;
- aiutare a confrontare ipotesi;
- aiutare nella documentazione finale.

L'IA non deve:

- inventare output;
- essere utilizzata senza fornire evidenze;
- sostituire il ragionamento dello studente;
- essere utilizzata per copiare una configurazione senza comprenderla;
- essere considerata automaticamente corretta.

> **Principio:** l'output dell'IA è un'ipotesi o un supporto all'analisi, non una prova.

---

# 19. Consegna finale

Ogni gruppo deve consegnare:

1. file Packet Tracer `.pkt`;
2. piano IP aggiornato;
3. tabella dei fault individuati;
4. scheda di troubleshooting compilata;
5. prompt utilizzato con l'IA;
6. evidenze raccolte;
7. configurazioni corrette;
8. breve relazione finale.

La relazione deve rispondere a:

```text
Qual era il problema?

Quali sintomi abbiamo osservato?

Quali ipotesi abbiamo formulato?

Quali test abbiamo eseguito?

Quale evidenza ha confermato la causa?

Come abbiamo risolto il problema?

Come abbiamo verificato la soluzione?

In che modo l'IA ci ha aiutato?

L'IA ha commesso errori o fornito suggerimenti non corretti?

Come abbiamo verificato autonomamente le sue indicazioni?
```

---

# 20. Checklist finale

Prima della consegna verificare:

- [ ] tutti gli host hanno una configurazione IP coerente;
- [ ] le VLAN sono corrette;
- [ ] le porte access sono corrette;
- [ ] i trunk funzionano;
- [ ] i gateway sono corretti;
- [ ] l'inter-VLAN routing funziona;
- [ ] il routing è corretto;
- [ ] DHCP funziona;
- [ ] DNS funziona;
- [ ] NAT/PAT funziona quando previsto;
- [ ] il server è raggiungibile;
- [ ] tutti i fault sono stati identificati;
- [ ] dopo ogni correzione è stata eseguita una nuova verifica;
- [ ] il troubleshooting è stato documentato;
- [ ] l'IA è stata utilizzata sulla base di evidenze;
- [ ] gli output dell'IA sono stati verificati autonomamente.

---

# 21. Criteri di valutazione

| Area | Valutazione |
|---|---|
| Analisi del problema | Capacità di definire correttamente il sintomo |
| Raccolta evidenze | Qualità e pertinenza dei dati raccolti |
| Troubleshooting | Metodo e sequenza delle verifiche |
| Configurazione | Correttezza delle modifiche |
| Verifica | Capacità di dimostrare che il problema è risolto |
| Documentazione | Chiarezza della relazione tecnica |
| Utilizzo IA | Qualità dei prompt e capacità di verificare le risposte |
| Autonomia | Capacità di ragionare senza dipendere dall'IA |

---

# 22. Conclusione

Questo laboratorio conclude il percorso operativo del modulo Networking.

L'obiettivo finale non è ricordare un elenco di comandi, ma sviluppare un metodo professionale:

```text
VEDO UN PROBLEMA
↓
LO DEFINISCO
↓
RACCOLGO EVIDENZE
↓
FORMULO IPOTESI
↓
TESTO
↓
INTERPRETO
↓
RISOLVO
↓
VERIFICO
↓
DOCUMENTO
```

L'IA entra nel processo come **strumento di supporto al ragionamento tecnico**.

Il tecnico rimane responsabile delle decisioni, delle verifiche e della soluzione finale.

> **La competenza non consiste nel sapere quale comando digitare. Consiste nel sapere perché eseguirlo, cosa aspettarsi e come interpretare il risultato.**
