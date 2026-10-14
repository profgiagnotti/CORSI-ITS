# Lezione 6 — Laboratorio pratico: Spanning Tree, RSTP e troubleshooting Layer 2 con AI

**Corso:** UFS 03 — Networking  
**Lezione:** 6 — Spanning Tree e troubleshooting Layer 2  
**Durata indicativa:** 140 minuti  
**Ambiente:** Cisco Packet Tracer  
**Livello:** intermedio  
**Metodo:** Configuro → Verifico → Osservo → Rompo → Formulo ipotesi → Testo → Risolvo → Verifico nuovamente → Documento

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio dovrai essere in grado di:

- costruire una topologia Layer 2 ridondata;
- spiegare perché una rete Ethernet con collegamenti ridondanti può generare loop;
- osservare il comportamento di STP;
- identificare il **Root Bridge**;
- distinguere **Root Port**, **Designated Port** e porta bloccata/non forwarding;
- verificare il funzionamento di STP con i comandi Cisco IOS;
- comprendere il ruolo delle BPDU;
- modificare la selezione del Root Bridge in modo controllato;
- osservare il comportamento di RSTP;
- introdurre fault intenzionali;
- applicare una metodologia strutturata di troubleshooting;
- utilizzare l'AI come supporto all'analisi, senza sostituire osservazione e verifica;
- costruire prompt diagnostici completi per i Fault Injection;
- documentare fatti, ipotesi, test, risultati e soluzione.

> **Regola del laboratorio:** non considerare valida una diagnosi solo perché l'AI la propone. L'AI deve aiutarti a formulare e testare ipotesi; la conferma deve arrivare dalle evidenze raccolte in Packet Tracer.

---

# 2. Scenario di laboratorio

Realizzeremo una rete Layer 2 ridondata composta da tre switch:

```text
                 +-------------+
                 |     SW1     |
                 |             |
                 +------+------+ 
                    /        \
                   /          \
                  /            \
        +--------+              +--------+
        |  SW2   |--------------|  SW3   |
        |        |              |        |
        +---+----+              +----+---+
            |                        |
           PC0                      PC1
```

I tre switch formano un triangolo.

Questa scelta è intenzionale: se tutti i collegamenti Layer 2 fossero contemporaneamente utilizzati per inoltrare traffico, si potrebbe creare un loop.

**STP interviene proprio per evitare questo problema**, costruendo una topologia logica priva di loop pur mantenendo la ridondanza fisica.

---

# 3. Costruzione della topologia

## 3.1 Dispositivi

In Cisco Packet Tracer inserisci:

- 3 switch Cisco 2960:
  - SW1
  - SW2
  - SW3
- 2 PC:
  - PC0
  - PC1

Collega gli switch in triangolo:

- SW1 ↔ SW2
- SW2 ↔ SW3
- SW3 ↔ SW1

Collega inoltre:

- PC0 → SW2
- PC1 → SW3

Puoi utilizzare le porte FastEthernet per i collegamenti tra switch e le porte FastEthernet disponibili per i PC.

> **Nota:** la numerazione delle porte può variare in base alla disposizione scelta in Packet Tracer. Adatta i comandi alla porta realmente utilizzata.

---

# 4. Configurazione iniziale

Prima di osservare STP, assegna un indirizzo IP ai due PC.

## PC0

```text
IP address: 192.168.10.10
Subnet mask: 255.255.255.0
```

## PC1

```text
IP address: 192.168.10.20
Subnet mask: 255.255.255.0
```

Non è necessario configurare un gateway perché i due PC appartengono alla stessa rete Layer 2.

Verifica:

```text
PC0> ping 192.168.10.20
```

Il ping dovrebbe avere esito positivo.

---

# 5. Prima osservazione: STP senza modifiche

Accedi alla CLI di SW1.

```text
enable
show spanning-tree
```

Ripeti sui tre switch:

```text
show spanning-tree
```

Osserva attentamente:

- Root ID;
- Bridge ID;
- ruolo delle porte;
- stato delle porte;
- cost;
- eventuali porte in forwarding;
- eventuali porte non forwarding.

### Domande

1. Quale switch è il Root Bridge?
2. Quali porte sono Root Port?
3. Quali porte sono Designated Port?
4. Quale collegamento contiene una porta che non inoltra traffico?
5. Perché STP non mantiene tutti i collegamenti contemporaneamente in forwarding?

Annota le risposte nel report.

---

# 6. Comprendere la selezione del Root Bridge

STP deve scegliere un punto di riferimento logico: il **Root Bridge**.

La scelta dipende dal Bridge ID.

In questa fase non modificare ancora la configurazione.

Utilizza:

```text
show spanning-tree
```

e confronta i dati dei tre switch.

Osserva in particolare:

```text
Bridge ID
Priority
Address
```

e:

```text
Root ID
Priority
Address
```

### Attività

Completa questa tabella:

| Switch | Bridge Priority | MAC | È Root Bridge? |
|---|---:|---|---|
| SW1 | | | |
| SW2 | | | |
| SW3 | | | |

---

# 7. Selezionare manualmente il Root Bridge

Ora vogliamo rendere SW1 il Root Bridge in modo esplicito.

Su SW1:

```text
enable
configure terminal
spanning-tree vlan 1 priority 24576
end
```

Verifica:

```text
show spanning-tree
```

Poi controlla anche SW2 e SW3:

```text
show spanning-tree
```

### Osserva

- è cambiato il Root Bridge?
- quali porte hanno cambiato ruolo?
- è cambiato il percorso logico?
- quale porta è stata messa in stato non forwarding?

> **Tip del laboratorio:** quando modifichi il Root Bridge non limitarti a controllare "chi è diventato root". Osserva anche cosa è successo alle porte degli altri switch.

---

# 8. Analizzare Root Port e Designated Port

Per ogni switch, individua:

- Root Port;
- Designated Port;
- porta non forwarding/bloccata.

Costruisci una tabella:

| Switch | Porta | Ruolo | Stato | Collegamento |
|---|---|---|---|---|
| SW1 | | | | |
| SW1 | | | | |
| SW2 | | | | |
| SW2 | | | | |
| SW3 | | | | |

### Domande

- Perché SW1, essendo Root Bridge, non ha una Root Port?
- Come viene scelto il percorso verso il Root Bridge?
- Perché una porta può essere fisicamente collegata ma non inoltrare traffico?
- Cosa accadrebbe se STP non esistesse?

---

# 9. Osservare le BPDU

STP utilizza le **BPDU (Bridge Protocol Data Units)** per permettere agli switch di scambiarsi informazioni relative alla topologia.

In questa fase utilizza la modalità **Simulation** di Packet Tracer.

## Procedura

1. Passa a **Simulation Mode**.
2. Se necessario, limita gli eventi visualizzati a STP.
3. Osserva il traffico tra gli switch.
4. Individua i pacchetti BPDU.
5. Segui il percorso di una BPDU.

### Domande

1. Quale dispositivo invia le BPDU?
2. Quale informazione trasportano?
3. Perché le BPDU sono importanti?
4. Cosa succede alla topologia quando cambia il percorso disponibile?

---

# 10. Cambiare il collegamento disponibile

Ora simuleremo un guasto fisico.

Scegli uno dei collegamenti tra switch e scollega il cavo in Packet Tracer.

Ad esempio:

```text
SW1 ↔ SW2
```

Prima del guasto verifica:

```text
show spanning-tree
```

Poi scollega il collegamento e attendi la convergenza.

Ripeti:

```text
show spanning-tree
```

e verifica la connettività:

```text
PC0> ping 192.168.10.20
```

### Osserva

- quale collegamento è rimasto disponibile?
- quale porta ha cambiato ruolo/stato?
- il ping continua a funzionare?
- la rete ha mantenuto la connettività grazie alla ridondanza?

---

# 11. STP e RSTP

STP tradizionale utilizza un processo di convergenza più lento rispetto a RSTP.

Verifica quale modalità è configurata:

```text
show spanning-tree summary
```

Puoi verificare anche la configurazione globale:

```text
show running-config
```

Per impostare RSTP tramite PVST+ su uno switch Cisco:

```text
configure terminal
spanning-tree mode rapid-pvst
end
```

Ripeti sugli altri switch della topologia.

Verifica:

```text
show spanning-tree summary
```

e:

```text
show spanning-tree
```

### Attenzione

Non confondere:

- STP;
- RSTP;
- Rapid PVST+;
- il ruolo delle porte;
- lo stato operativo della singola interfaccia.

L'obiettivo del laboratorio non è memorizzare solamente i nomi, ma osservare come cambia il comportamento della rete.

---

# 12. Fault Injection 1 — Root Bridge inatteso

## Obiettivo

Creare una situazione nella quale il Root Bridge non è quello previsto dal progetto.

### Fault

Imposta una priorità più bassa su SW3:

```text
enable
configure terminal
spanning-tree vlan 1 priority 4096
end
```

### Verifica

Su tutti gli switch:

```text
show spanning-tree
```

### Domande

1. Chi è diventato Root Bridge?
2. Era quello previsto?
3. Quale parametro ha determinato la scelta?
4. Quali porte hanno cambiato ruolo?

### Ripristino

Riporta SW3 a una priorità coerente con il progetto e verifica nuovamente la topologia.

---

# 13. Fault Injection 2 — Collegamento Layer 2 interrotto

## Fault

Scollega uno dei collegamenti tra switch.

Prima:

```text
show spanning-tree
```

Dopo il fault:

```text
show spanning-tree
```

Verifica anche:

```text
show interfaces status
```

### Obiettivo diagnostico

Distinguere tra:

- problema STP;
- problema fisico;
- porta down;
- percorso alternativo correttamente attivato.

### Domanda chiave

> Se la rete continua a funzionare dopo la rottura del cavo, significa che il fault non esiste?

No.

Il fault esiste, ma la ridondanza della topologia permette alla rete di mantenere la connettività.

---

# 14. Fault Injection 3 — Porta in stato inatteso

Individua una porta coinvolta nella topologia STP e modifica intenzionalmente la configurazione.

Esempio:

```text
configure terminal
interface fastEthernet 0/1
shutdown
end
```

Verifica:

```text
show interfaces status
```

e:

```text
show spanning-tree
```

### Obiettivo

Capire se il problema osservato è:

- amministrativo;
- fisico;
- STP;
- di configurazione.

---

# 15. Fault Injection 4 — Priorità STP incoerente

Configura su uno switch una priorità che lo renda Root Bridge quando non dovrebbe esserlo.

Esempio:

```text
configure terminal
spanning-tree vlan 1 priority 0
end
```

Poi:

```text
show spanning-tree
```

### Analisi

Documenta:

- Root Bridge previsto;
- Root Bridge osservato;
- configurazione responsabile;
- test utilizzato;
- correzione applicata;
- verifica finale.

---

# 16. Fault Injection 5 — Due problemi contemporaneamente

Questa è la prova più importante.

Introduci contemporaneamente:

1. un collegamento fisico interrotto;
2. una priorità STP errata.

Non comunicare immediatamente all'AI quali fault hai introdotto.

L'obiettivo è verificare se riesci a distinguere:

- sintomi;
- cause possibili;
- evidenze;
- cause confermate.

---

# 17. Metodologia strutturata di troubleshooting

Durante il troubleshooting utilizza sempre questa sequenza:

```text
OSSERVA
   ↓
RACCOGLI LE EVIDENZE
   ↓
FORMULA IPOTESI
   ↓
SCEGLI UN TEST
   ↓
ESEGUI IL TEST
   ↓
INTERPRETA IL RISULTATO
   ↓
CONFERMA O SMENTISCI L'IPOTESI
   ↓
RISOLVI
   ↓
VERIFICA NUOVAMENTE
   ↓
DOCUMENTA
```

## Regola fondamentale

Non partire dalla configurazione.

Parti dal comportamento osservato.

Esempio:

> "PC0 non raggiunge PC1"

non significa automaticamente:

> "STP è configurato male".

Potrebbero esserci:

- cavo scollegato;
- porta shutdown;
- VLAN errata;
- interfaccia down;
- percorso Layer 2 interrotto;
- Root Bridge inatteso;
- problema di convergenza;
- problema sul PC.

---

# 18. Comandi diagnostici principali

## STP

```text
show spanning-tree
```

## Sintesi STP

```text
show spanning-tree summary
```

## Interfacce

```text
show interfaces status
```

## Configurazione

```text
show running-config
```

## VLAN

```text
show vlan brief
```

## MAC address table

```text
show mac address-table
```

## Informazioni sulle interfacce

```text
show interfaces
```

---

# 19. Troubleshooting con AI

L'AI deve essere utilizzata come **assistente diagnostico**, non come sostituto dell'attività tecnica.

Un buon prompt deve fornire:

1. topologia;
2. obiettivo;
3. sintomo;
4. configurazione disponibile;
5. output dei comandi;
6. fault eventualmente introdotti, se l'attività lo prevede;
7. vincoli;
8. metodologia richiesta.

## Prompt base

Puoi utilizzare questo modello:

```text
Sto lavorando in Cisco Packet Tracer su una topologia Layer 2
ridondata con tre switch collegati a triangolo.

Obiettivo:
analizzare un problema relativo a Spanning Tree.

Sintomo osservato:
[descrivi il sintomo]

Topologia:
[descrivi i collegamenti]

Root Bridge previsto:
[SW...]

Output disponibili:

show spanning-tree
[incolla output]

show spanning-tree summary
[incolla output]

show interfaces status
[incolla output]

show vlan brief
[incolla output]

Voglio che tu proceda secondo questa metodologia:

OSSERVA
FORMULA IPOTESI
TESTA
INTERPRETA
CONFERMA O SMENTISCI
RISOLVI
VERIFICA
DOCUMENTA

Non inventare informazioni mancanti.
Non fornire subito la soluzione finale.
Non modificare la configurazione senza spiegare quale
ipotesi stai verificando.

Per ogni ipotesi indica:
- evidenze a favore;
- evidenze contro;
- comando/test da eseguire;
- risultato atteso;
- interpretazione del risultato.

Procedi una verifica alla volta.
```

---

# 20. Prompt avanzato per Fault Injection

Quando il docente introduce un fault senza comunicarlo allo studente, puoi utilizzare:

```text
Sto svolgendo un'attività di troubleshooting in Cisco Packet Tracer.

La rete contiene tre switch Layer 2 collegati in una topologia
ridondata.

È stato introdotto intenzionalmente uno o più fault.

Il tuo compito NON è indovinare immediatamente il fault.

Devi aiutarmi a identificarlo utilizzando esclusivamente
le evidenze che ti fornisco.

METODO OBBLIGATORIO:

1. OSSERVA
2. ELENCA I FATTI
3. FORMULA MASSIMO 3 IPOTESI
4. ORDINA LE IPOTESI per probabilità
5. PROPONI UN SOLO TEST ALLA VOLTA
6. INDICA IL COMANDO DA ESEGUIRE
7. INDICA COSA TI ASPETTI DI OSSERVARE
8. SPIEGA come interpretare il risultato
9. CONFERMA O SMENTISCI L'IPOTESI
10. SOLO DOPO LA CONFERMA proponi la correzione
11. VERIFICA nuovamente la connettività
12. DOCUMENTA causa, soluzione e verifica finale

Non inventare output.
Se manca un'informazione, chiedimela.
Non assumere che il problema sia STP solo perché la topologia
è ridondata.
Considera anche:
- cavi;
- stato delle interfacce;
- VLAN;
- configurazione delle porte;
- Root Bridge;
- Root Port;
- Designated Port;
- priorità STP;
- modalità STP/RSTP;
- MAC address;
- connettività degli host.

Al termine produci una breve relazione tecnica.
```

---

# 21. Attività AI — Confrontare le ipotesi

Prendi uno dei Fault Injection precedenti.

Prima di eseguire i comandi, chiedi all'AI di formulare tre ipotesi.

Esempio:

```text
Sintomo:
PC0 non riesce a raggiungere PC1.

Formula tre ipotesi tecniche differenti.
Per ciascuna indica:
- perché è plausibile;
- quale evidenza la supporterebbe;
- quale evidenza la smentirebbe;
- qual è il test più economico e sicuro da eseguire.
```

Poi esegui realmente i test in Packet Tracer.

### Obiettivo

Imparare che una buona diagnosi non consiste nel trovare subito una risposta, ma nel **ridurre progressivamente l'incertezza**.

---

# 22. Scheda di troubleshooting

Compila questa scheda per ogni Fault Injection.

| Campo | Risposta |
|---|---|
| Sintomo | |
| Topologia coinvolta | |
| Fatti osservati | |
| Ipotesi 1 | |
| Ipotesi 2 | |
| Ipotesi 3 | |
| Test scelto | |
| Comando utilizzato | |
| Risultato | |
| Interpretazione | |
| Ipotesi confermata/smentita | |
| Causa reale | |
| Correzione | |
| Verifica finale | |
| Evidenza finale | |

---

# 23. Challenge finale

Costruisci nuovamente la topologia triangolare.

Configura:

- SW1 come Root Bridge;
- RSTP/Rapid PVST+;
- PC0;
- PC1;
- collegamenti ridondanti.

Verifica:

```text
show spanning-tree
```

Poi introduci **due fault contemporaneamente**.

### Regole

Non chiedere subito all'AI la soluzione.

Prima raccogli almeno:

```text
show spanning-tree
show spanning-tree summary
show interfaces status
show vlan brief
show mac address-table
```

Poi prepara il prompt diagnostico.

L'AI dovrà:

1. distinguere i sintomi;
2. formulare ipotesi;
3. indicare i test;
4. interpretare i risultati;
5. identificare le cause;
6. proporre la correzione;
7. chiederti di verificare nuovamente la rete.

---

# 24. Verifica finale

Prima di concludere il laboratorio verifica:

- [ ] La topologia Layer 2 è ridondata.
- [ ] È stato identificato il Root Bridge.
- [ ] Sono state individuate le Root Port.
- [ ] Sono state individuate le Designated Port.
- [ ] È stata osservata almeno una porta non forwarding.
- [ ] Sono state osservate le BPDU.
- [ ] È stata modificata la selezione del Root Bridge.
- [ ] È stato osservato un cambiamento della topologia dopo un fault.
- [ ] È stato configurato/verificato RSTP.
- [ ] È stato eseguito almeno un Fault Injection.
- [ ] È stata utilizzata una metodologia strutturata.
- [ ] È stato utilizzato l'AI troubleshooting prompt.
- [ ] Ogni ipotesi è stata verificata con un test.
- [ ] La soluzione è stata verificata dopo la correzione.
- [ ] Il troubleshooting è stato documentato.

---

# 25. Consegna

Il laboratorio deve produrre:

1. file `.pkt` della topologia finale;
2. tabella Root Bridge / Root Port / Designated Port;
3. output principali dei comandi diagnostici;
4. scheda di troubleshooting;
5. prompt AI utilizzato;
6. risposta dell'AI utilizzata durante l'indagine;
7. descrizione dei fault introdotti;
8. causa dei fault;
9. correzioni applicate;
10. verifica finale della connettività.

---

# 26. Criteri di valutazione

| Competenza | Evidenza |
|---|---|
| Realizzazione topologia | rete ridondata funzionante |
| Comprensione STP | identificazione corretta dei ruoli |
| Root Bridge | selezione e verifica |
| Diagnostica | uso corretto dei comandi |
| Fault Injection | capacità di osservare gli effetti |
| Troubleshooting | metodo strutturato |
| AI | prompt completo e uso critico |
| Verifica | conferma della soluzione |
| Documentazione | relazione tecnica chiara |

---

# 27. Conclusione

In questo laboratorio la ridondanza non è stata considerata semplicemente come "avere più cavi".

Il punto fondamentale è capire che una rete Layer 2 deve essere progettata in modo da poter mantenere la connettività anche quando un percorso viene meno, evitando contemporaneamente i loop.

STP e RSTP permettono di trasformare una topologia fisicamente ridondata in una topologia logicamente controllata.

Il troubleshooting aggiunge un secondo livello di competenza: non basta sapere quale comando utilizzare, bisogna sapere **perché** lo si utilizza e come interpretarne il risultato.

L'AI entra in questo processo come supporto al ragionamento tecnico:

```text
EVIDENZA
   ↓
IPOTESI
   ↓
TEST
   ↓
RISULTATO
   ↓
INTERPRETAZIONE
   ↓
DECISIONE
   ↓
VERIFICA
   ↓
DOCUMENTAZIONE
```

Questo è il comportamento che dovrai applicare anche nei laboratori successivi.
