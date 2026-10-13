# Lezione 5 — Laboratorio pratico: VLAN, Trunk, EtherChannel, CDP, LLDP e Troubleshooting con AI

**Corso:** UFS 03 — Networking  
**Lezione:** 5  
**Durata consigliata:** 140 minuti  
**Ambiente:** Cisco Packet Tracer  
**Livello:** progressivo, con attività di Fault Injection e supporto dell'Intelligenza Artificiale

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio dovrai essere in grado di:

- creare e nominare VLAN su uno switch Cisco;
- configurare porte **access** e assegnarle alla VLAN corretta;
- configurare un collegamento **trunk 802.1Q** tra switch;
- verificare VLAN e trunk con i principali comandi diagnostici;
- comprendere il ruolo della **native VLAN**;
- configurare un **EtherChannel** utilizzando LACP;
- verificare lo stato del Port-Channel;
- utilizzare **CDP** e **LLDP** per osservare i dispositivi vicini;
- individuare errori introdotti volutamente nella configurazione;
- utilizzare l'AI come supporto metodologico al troubleshooting, senza delegare all'AI la diagnosi;
- documentare fatti, ipotesi, test, risultati e soluzione finale.

---

## 2. Metodo di lavoro

Durante il laboratorio utilizza sempre questa sequenza:

> **PROGETTO → CONFIGURO → VERIFICO → ROMPO → OSSERVO → FORMULO IPOTESI → TESTO → RISOLVO → VERIFICO NUOVAMENTE → DOCUMENTO**

L'obiettivo non è semplicemente ottenere il `ping` funzionante. Devi imparare a capire **perché** una configurazione funziona o non funziona.

### Regola importante

Non modificare una configurazione alla cieca.

Prima raccogli evidenze, poi formula un'ipotesi, quindi esegui un test mirato.

---

# 3. Scenario 1 — Creazione delle VLAN e porte access

## 3.1 Topologia

Realizza questa topologia:

```text
PC0 -------- SW0 -------- PC1
             |
             |
            PC2
```

Utilizza:

- 1 switch Cisco 2960;
- 3 PC;
- cavi Ethernet Copper Straight-Through.

Collega:

```text
PC0 -> SW0 Fa0/1
PC1 -> SW0 Fa0/2
PC2 -> SW0 Fa0/3
```

---

## 3.2 Piano delle VLAN

| VLAN | Nome | Porta | Dispositivo |
|---:|---|---|---|
| 10 | STUDENTI | Fa0/1 | PC0 |
| 20 | DOCENTI | Fa0/2 | PC1 |
| 10 | STUDENTI | Fa0/3 | PC2 |

In questa prima attività PC0 e PC2 appartengono alla stessa VLAN, mentre PC1 appartiene a una VLAN differente.

---

## 3.3 Configurazione delle VLAN

Accedi alla CLI dello switch:

```text
enable
configure terminal
vlan 10
 name STUDENTI
exit
vlan 20
 name DOCENTI
exit
```

Verifica:

```text
show vlan brief
```

### Cosa devi osservare?

Devi verificare che:

- VLAN 10 esista;
- VLAN 20 esista;
- i nomi siano corretti;
- inizialmente le porte possano ancora risultare nella VLAN predefinita.

---

## 3.4 Configurazione delle porte access

Configura Fa0/1 e Fa0/3 nella VLAN 10:

```text
configure terminal
interface range fastEthernet 0/1 - 3
 switchport mode access
exit
interface fastEthernet 0/1
 switchport access vlan 10
exit
interface fastEthernet 0/2
 switchport access vlan 20
exit
interface fastEthernet 0/3
 switchport access vlan 10
exit
end
```

Verifica nuovamente:

```text
show vlan brief
```

---

## 3.5 Domande di verifica

Rispondi sul tuo file di laboratorio:

1. Quali porte appartengono alla VLAN 10?
2. Quale porta appartiene alla VLAN 20?
3. PC0 e PC2 si trovano nello stesso broadcast domain?
4. PC0 e PC1 si trovano nello stesso broadcast domain?
5. Perché una porta collegata a un PC viene normalmente configurata come access?

---

# 4. Scenario 2 — Verifica del comportamento delle VLAN

Assegna ai PC questi indirizzi:

| PC | IP | Mask |
|---|---|---|
| PC0 | 192.168.10.10 | 255.255.255.0 |
| PC1 | 192.168.20.10 | 255.255.255.0 |
| PC2 | 192.168.10.20 | 255.255.255.0 |

Per questa attività non è necessario configurare un gateway.

## 4.1 Test di connettività

Da PC0 esegui:

```text
ping 192.168.10.20
```

Poi:

```text
ping 192.168.20.10
```

### Risultato atteso

- PC0 → PC2: **deve funzionare**, perché appartengono alla VLAN 10 e alla stessa rete IP.
- PC0 → PC1: **non deve funzionare**, perché appartengono a VLAN differenti e non hai configurato alcun dispositivo Layer 3 per l'inter-VLAN routing.

### Domanda

Perché avere indirizzi IP nella stessa classe o reti apparentemente simili non significa automaticamente che due host possano comunicare a livello Layer 2?

---

# 5. Scenario 3 — Collegamento trunk tra due switch

## 5.1 Nuova topologia

Aggiungi un secondo switch:

```text
PC0 ---- SW0 ===== SW1 ---- PC2
          TRUNK
            |
           PC1
```

Collega:

```text
SW0 Gi0/1 <----> SW1 Gi0/1
```

Mantieni le VLAN 10 e 20 su entrambi gli switch.

---

## 5.2 Configurazione VLAN su SW1

```text
enable
configure terminal
vlan 10
 name STUDENTI
exit
vlan 20
 name DOCENTI
exit
end
```

Verifica:

```text
show vlan brief
```

---

## 5.3 Configurazione del trunk

Su SW0:

```text
enable
configure terminal
interface gigabitEthernet 0/1
 switchport mode trunk
end
```

Su SW1:

```text
enable
configure terminal
interface gigabitEthernet 0/1
 switchport mode trunk
end
```

Verifica su entrambi:

```text
show interfaces trunk
```

---

## 5.4 Perché serve il trunk?

Un trunk permette di trasportare il traffico appartenente a più VLAN attraverso lo stesso collegamento fisico tra dispositivi di rete.

Nel laboratorio immagina il collegamento tra SW0 e SW1 come una strada condivisa da più "corsie logiche": VLAN 10 e VLAN 20 possono attraversare lo stesso collegamento fisico mantenendo la propria separazione logica.

---

# 6. Scenario 4 — Verifica della propagazione delle VLAN

Collega:

```text
PC0 -> SW0 Fa0/1 -> VLAN 10
PC2 -> SW1 Fa0/1 -> VLAN 10
```

Configura la porta di PC2:

```text
configure terminal
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
end
```

Assegna a PC2:

```text
192.168.10.20 /24
```

PC0:

```text
192.168.10.10 /24
```

Test:

```text
ping 192.168.10.20
```

### Risultato atteso

Il ping deve funzionare se:

- VLAN 10 esiste su entrambi gli switch;
- le porte degli host sono nella VLAN 10;
- il collegamento tra gli switch è trunk;
- il cablaggio è corretto;
- gli indirizzi IP sono coerenti.

---

# 7. Scenario 5 — Native VLAN

La **native VLAN** è la VLAN associata al traffico non taggato su un trunk 802.1Q.

Osserva la configurazione del trunk:

```text
show interfaces trunk
```

Individua la voce relativa alla native VLAN.

### Attività

1. Qual è la native VLAN attuale?
2. È uguale sui due estremi del trunk?
3. Cosa potrebbe accadere se i due estremi fossero configurati con native VLAN differenti?

> **Nota didattica:** in una rete reale la native VLAN va gestita in modo coerente e, per motivi di sicurezza, è buona pratica evitare configurazioni casuali o incoerenti.

---

# 8. Scenario 6 — EtherChannel con LACP

## 8.1 Nuova topologia

Realizza due collegamenti fisici tra SW0 e SW1:

```text
             Gi0/1 ===== Gi0/1
SW0                              SW1
             Gi0/2 ===== Gi0/2

             EtherChannel
```

L'obiettivo è aggregare i due link in un unico **Port-Channel**.

> **Attenzione:** usa porte compatibili con il modello dello switch presente nel tuo Packet Tracer. Se il modello utilizza una nomenclatura diversa, adatta i comandi alla porta effettivamente disponibile.

---

## 8.2 Configurazione LACP su SW0

```text
enable
configure terminal
interface range gigabitEthernet 0/1 - 2
 channel-group 1 mode active
exit
interface port-channel 1
 switchport mode trunk
end
```

---

## 8.3 Configurazione LACP su SW1

```text
enable
configure terminal
interface range gigabitEthernet 0/1 - 2
 channel-group 1 mode active
exit
interface port-channel 1
 switchport mode trunk
end
```

---

## 8.4 Verifica EtherChannel

Esegui su entrambi gli switch:

```text
show etherchannel summary
```

Cerca il **Port-Channel 1** e verifica lo stato dei membri.

Esegui anche:

```text
show interfaces trunk
```

### Domande

1. Quali interfacce fisiche appartengono al Port-Channel?
2. Il Port-Channel è operativo?
3. Il traffico delle VLAN attraversa il Port-Channel?
4. Cosa succederebbe se i parametri dei due membri fossero incompatibili?

---

# 9. Scenario 7 — CDP

CDP consente di raccogliere informazioni sui dispositivi Cisco direttamente collegati.

Esegui:

```text
show cdp neighbors
```

Poi:

```text
show cdp neighbors detail
```

Annota:

- dispositivo vicino;
- porta locale;
- porta remota;
- Platform;
- Capability, se disponibile.

### Domanda

In una rete reale, perché CDP può essere utile durante una fase di troubleshooting?

---

# 10. Scenario 8 — LLDP

LLDP è un protocollo di discovery standardizzato e non limitato ai dispositivi Cisco.

Verifica se è disponibile e abilitato nel tuo dispositivo Packet Tracer.

```text
show lldp neighbors
```

Poi:

```text
show lldp neighbors detail
```

Se necessario, abilitalo globalmente:

```text
configure terminal
lldp run
end
```

Ripeti:

```text
show lldp neighbors
```

### Confronto

| Protocollo | Caratteristica |
|---|---|
| CDP | Discovery proprietario Cisco |
| LLDP | Discovery standardizzato e multivendor |

---

# 11. Checklist diagnostica Layer 2

Quando una VLAN o un trunk non funzionano, procedi dal livello più semplice a quello più specifico.

### 11.1 Stato fisico

```text
show interfaces status
```

Controlla:

- porta connected/notconnect;
- VLAN assegnata;
- velocità;
- duplex, quando disponibile.

### 11.2 VLAN

```text
show vlan brief
```

Controlla:

- VLAN esistente;
- nome;
- porte corrette.

### 11.3 Trunk

```text
show interfaces trunk
```

Controlla:

- interfaccia trunk;
- native VLAN;
- VLAN consentite;
- VLAN attive sul trunk.

### 11.4 EtherChannel

```text
show etherchannel summary
```

Controlla:

- Port-Channel presente;
- membri correttamente aggregati;
- stato operativo.

### 11.5 Discovery

```text
show cdp neighbors
show lldp neighbors
```

Usa queste informazioni per confrontare la topologia reale con quella progettata.

---

# 12. Fault Injection 1 — Porta nella VLAN sbagliata

Ora devi **rompere intenzionalmente** la rete.

Sposta PC2 dalla VLAN 10 alla VLAN 20:

```text
configure terminal
interface fastEthernet 0/1
 switchport access vlan 20
end
```

Verifica:

```text
show vlan brief
```

Da PC0:

```text
ping 192.168.10.20
```

## Obiettivo

Non correggere subito il problema.

Segui il metodo:

```text
OSSERVA
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

### Domanda

Quale comando ti permette di verificare rapidamente l'appartenenza della porta alla VLAN?

---

# 13. Fault Injection 2 — Trunk non configurato

Rimuovi temporaneamente il trunk da uno dei due switch.

Esempio:

```text
configure terminal
interface gigabitEthernet 0/1
 switchport mode access
end
```

Verifica:

```text
show interfaces trunk
```

Ripeti il ping tra host della stessa VLAN ma collocati su switch differenti.

### Obiettivo

Capire che un problema di trunk può manifestarsi come un problema di connettività VLAN.

Non limitarti a dire:

> "Il ping non funziona."

Devi arrivare a una conclusione più precisa, ad esempio:

> "La VLAN locale è corretta, ma il collegamento tra gli switch non sta trasportando correttamente la VLAN interessata."

---

# 14. Fault Injection 3 — Native VLAN incoerente

Se il modello e la configurazione del laboratorio lo consentono, modifica la native VLAN su un solo estremo del trunk.

Esempio concettuale:

```text
interface gigabitEthernet 0/1
 switchport trunk native vlan 99
```

Mantieni sull'altro switch una native VLAN differente.

Verifica:

```text
show interfaces trunk
```

### Obiettivo

Osservare che una configurazione apparentemente "quasi corretta" può generare un'incoerenza tra i due estremi del collegamento.

Ripristina poi una configurazione coerente.

---

# 15. Fault Injection 4 — EtherChannel incoerente

Introduci volutamente un'incoerenza in uno dei membri del Port-Channel.

Per esempio, verifica la configurazione delle interfacce:

```text
show running-config interface gigabitEthernet 0/1
show running-config interface gigabitEthernet 0/2
```

Confronta i due switch.

Puoi anche rimuovere temporaneamente un membro dal channel-group:

```text
configure terminal
interface gigabitEthernet 0/2
 no channel-group 1
end
```

Verifica:

```text
show etherchannel summary
```

### Domande

1. Il Port-Channel è ancora operativo?
2. Quali porte risultano membri?
3. Cosa è cambiato rispetto alla situazione iniziale?
4. Il problema riguarda VLAN, trunk o aggregazione?

---

# 16. Fault Injection 5 — Discovery e topologia inattesa

Confronta la topologia disegnata con ciò che i dispositivi dichiarano tramite:

```text
show cdp neighbors
show lldp neighbors
```

### Attività

Immagina che il progetto preveda:

```text
SW0 Gi0/1 <----> SW1 Gi0/1
```

ma il comando mostri una porta remota diversa.

Formula almeno due ipotesi:

1. cablaggio fisico differente da quello previsto;
2. numerazione delle interfacce o topologia modificata.

Indica quale comando o osservazione useresti per discriminare le due ipotesi.

---

# 17. Troubleshooting con AI

L'AI deve essere utilizzata come **assistente al ragionamento**, non come sostituto delle verifiche sul dispositivo.

Il principio da seguire è:

> **prima le evidenze, poi le ipotesi, poi i test.**

Non fornire all'AI solamente:

> "Il ping non funziona, risolvi."

Un prompt efficace deve contenere:

- topologia;
- obiettivo della rete;
- configurazione prevista;
- configurazione osservata;
- output dei comandi diagnostici;
- comportamento osservato;
- test già eseguiti;
- vincoli per l'AI.

---

## 17.1 Prompt base per il troubleshooting

Copia il seguente prompt e completalo con i dati reali del tuo laboratorio:

```text
Sto lavorando su un laboratorio Cisco Packet Tracer relativo a VLAN,
trunk ed EtherChannel.

Obiettivo della rete:
[descrivi cosa dovrebbe funzionare]

Topologia:
[descrivi dispositivi e collegamenti]

VLAN configurate:
[elenca VLAN, nomi e porte]

Trunk:
[indica interfacce e configurazione prevista]

EtherChannel:
[indica Port-Channel e membri]

Comportamento osservato:
[descrivi esattamente cosa non funziona]

Test già eseguiti:
[elenca ping e altri test]

Output diagnostici:
show vlan brief
[incolla output]

show interfaces trunk
[incolla output]

show etherchannel summary
[incolla output]

show interfaces status
[incolla output]

show cdp neighbors
[incolla output]

show lldp neighbors
[incolla output]

Voglio che tu segua questa metodologia:
OSSERVA → FORMULA IPOTESI → TESTA → INTERPRETA →
CONFERMA O SMENTISCI → RISOLVI → VERIFICA → DOCUMENTA.

Non inventare informazioni che non ti ho fornito.
Distingui sempre tra FATTI, IPOTESI e CONCLUSIONI.
Non propormi subito una soluzione definitiva.
Per ogni ipotesi indica:
1. perché è plausibile;
2. quale comando o test eseguire;
3. quale risultato aspettarsi;
4. come interpretare il risultato.

Procedi con un solo test alla volta e attendi il risultato prima di
formulare la conclusione successiva.
```

---

# 18. Attività AI — Diagnosi guidata di un Fault Injection

Il docente introduce uno dei fault precedenti senza comunicare agli studenti quale sia.

Gli studenti ricevono soltanto:

- topologia;
- obiettivo della rete;
- sintomo osservato;
- output iniziali dei comandi diagnostici.

### Fase 1 — Osserva

Raccogli almeno:

```text
show vlan brief
show interfaces trunk
show etherchannel summary
show interfaces status
```

Se utile:

```text
show cdp neighbors
show lldp neighbors
```

### Fase 2 — Formula almeno tre ipotesi

Esempio:

| Ipotesi | Evidenza a favore | Test |
|---|---|---|
| Porta nella VLAN errata | VLAN inattesa su una porta | `show vlan brief` |
| Trunk assente | VLAN non attraversa il collegamento | `show interfaces trunk` |
| EtherChannel non operativo | membri non aggregati | `show etherchannel summary` |

### Fase 3 — Usa l'AI

Invia all'AI il prompt base completato con gli output reali.

Chiedi all'AI di aiutarti a scegliere **il test con maggiore potere discriminante** tra le ipotesi.

### Fase 4 — Esegui il test

Non modificare la configurazione prima di avere un'evidenza sufficiente.

### Fase 5 — Aggiorna il ragionamento

Indica:

```text
Ipotesi confermata:
[...]

Ipotesi smentite:
[...]

Nuova evidenza:
[...]
```

### Fase 6 — Risolvi

Applica la modifica minima necessaria.

### Fase 7 — Verifica nuovamente

Ripeti il test che inizialmente falliva.

Poi verifica che la soluzione non abbia introdotto altri problemi.

### Fase 8 — Documenta

Produci una breve relazione:

```text
SINTOMO
↓
EVIDENZE
↓
IPOTESI
↓
TEST
↓
RISULTATO
↓
CAUSA
↓
CORREZIONE
↓
VERIFICA FINALE
```

---

# 19. Attività avanzata — Chiedere all'AI di non saltare alle conclusioni

Prova questo secondo prompt:

```text
Agisci come un tutor di networking e non come un generatore automatico
di configurazioni.

Il mio compito è diagnosticare un fault injection in una rete Cisco.

Non voglio la soluzione immediata.

Voglio che tu mi guidi attraverso:

1. raccolta delle evidenze;
2. classificazione del problema;
3. formulazione di almeno tre ipotesi;
4. scelta del test più utile;
5. interpretazione del risultato;
6. aggiornamento delle ipotesi;
7. identificazione della causa;
8. correzione;
9. verifica finale;
10. documentazione.

Quando ti fornisco un output Cisco:
- non inventare ciò che non compare nell'output;
- separa FATTI, IPOTESI e CONCLUSIONI;
- indica cosa possiamo realmente dedurre;
- se manca un dato, chiedimi quale comando eseguire;
- proponi un solo test alla volta;
- spiega cosa ci aspettiamo di osservare;
- spiega come interpretare sia un risultato positivo sia uno negativo.

La rete riguarda VLAN, trunk, EtherChannel, CDP e LLDP.

Ecco il primo sintomo:
[INSERISCI QUI IL SINTOMO]
```

---

# 20. Scheda di troubleshooting

Compila questa tabella per ogni Fault Injection.

| Campo | Risposta |
|---|---|
| Sintomo osservato | |
| Topologia interessata | |
| VLAN coinvolte | |
| Test iniziale | |
| Risultato | |
| Ipotesi 1 | |
| Test ipotesi 1 | |
| Risultato | |
| Ipotesi 2 | |
| Test ipotesi 2 | |
| Risultato | |
| Ipotesi 3 | |
| Test ipotesi 3 | |
| Risultato | |
| Causa individuata | |
| Configurazione corretta | |
| Verifica finale | |
| Supporto utilizzato dell'AI | |
| Cosa ho imparato | |

---

# 21. Challenge finale

Realizza una piccola rete composta da:

- 2 switch;
- almeno 2 VLAN;
- almeno un trunk;
- un EtherChannel tra gli switch;
- CDP e/o LLDP per osservare la topologia.

Configura la rete in modo che almeno due host della stessa VLAN possano comunicare anche quando sono collegati a switch differenti.

Successivamente il docente introduce **due fault contemporaneamente**.

Esempio:

- una porta assegnata alla VLAN sbagliata;
- un problema sul trunk o sull'EtherChannel.

Il tuo compito è:

1. raccogliere le evidenze;
2. non modificare immediatamente la configurazione;
3. formulare le ipotesi;
4. usare l'AI per confrontare le ipotesi;
5. eseguire test mirati;
6. identificare i due problemi;
7. correggerli;
8. verificare nuovamente la rete;
9. documentare il percorso diagnostico.

---

# 22. Checklist finale

Prima di concludere il laboratorio verifica di saper rispondere **senza copiare i comandi**:

- [ ] Cos'è una VLAN?
- [ ] Qual è la differenza tra access e trunk?
- [ ] Perché un trunk può trasportare più VLAN?
- [ ] Cos'è la native VLAN?
- [ ] Perché è importante che i due estremi del trunk siano coerenti?
- [ ] Cos'è un EtherChannel?
- [ ] Qual è il ruolo di LACP?
- [ ] Come verifico lo stato di un EtherChannel?
- [ ] A cosa serve CDP?
- [ ] A cosa serve LLDP?
- [ ] Qual è la differenza principale tra CDP e LLDP?
- [ ] Quale comando uso per verificare le VLAN?
- [ ] Quale comando uso per verificare i trunk?
- [ ] Quale comando uso per verificare EtherChannel?
- [ ] Come distinguo un fatto da un'ipotesi durante il troubleshooting?
- [ ] Come posso utilizzare l'AI senza delegarle la diagnosi?

---

# 23. Deliverable da consegnare

Al termine del laboratorio devi avere:

1. il file Packet Tracer `.pkt` con la configurazione funzionante;
2. le risposte alle domande degli esercizi;
3. almeno una scheda completa di Fault Injection;
4. il prompt AI utilizzato;
5. gli output diagnostici più significativi;
6. una breve documentazione finale della diagnosi;
7. una riflessione personale su quale evidenza è stata decisiva per individuare il problema.

---

# 24. Criteri di valutazione

| Area | Cosa viene osservato |
|---|---|
| Configurazione | VLAN, access, trunk ed EtherChannel correttamente configurati |
| Verifica | Uso appropriato dei comandi diagnostici |
| Troubleshooting | Capacità di procedere per evidenze e ipotesi |
| Fault Injection | Capacità di individuare e correggere errori introdotti |
| AI | Qualità del prompt e capacità di mantenere il controllo della diagnosi |
| Documentazione | Chiarezza del percorso problema → test → soluzione → verifica |

---

## Conclusione

In questo laboratorio VLAN, trunk ed EtherChannel non devono essere considerati semplicemente come una sequenza di comandi Cisco da memorizzare.

L'obiettivo è imparare a leggere il comportamento della rete.

Quando qualcosa non funziona, la domanda non deve essere soltanto:

> "Quale comando devo inserire?"

La domanda corretta diventa:

> "Quale evidenza mi permette di capire dove si trova il problema?"

L'AI può aiutarti a organizzare le ipotesi, scegliere test più efficaci e interpretare gli output. La decisione tecnica, però, deve rimanere basata sulle evidenze raccolte nella rete.

**Il risultato finale non è solo una rete che funziona: è la capacità di spiegare perché funziona e di dimostrare come hai risolto un guasto.**
