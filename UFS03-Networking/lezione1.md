# Lezione 1 — Prima rete, analisi del traffico e troubleshooting

> **Corso:** UFS 03 — Networking  
> **Lezione:** 1 — Parte applicativa  
> **Durata indicativa:** circa 2h40min  
> **Ambiente:** Cisco Packet Tracer  
> **Tema:** costruzione di una rete base, configurazione iniziale, analisi di MAC/ARP/ICMP, ping e traceroute, guasti intenzionali e prima metodologia diagnostica con IA.

---

## 1. Obiettivi dell'attività

Al termine dell'attività dovresti essere in grado di:

- costruire una piccola rete LAN in Cisco Packet Tracer;
- collegare correttamente PC, switch e router;
- configurare gli indirizzi IPv4 degli host;
- configurare le interfacce di un router;
- comprendere il ruolo del default gateway;
- osservare e interpretare gli indirizzi MAC;
- osservare il funzionamento di ARP;
- comprendere che cosa succede quando viene eseguito un `ping`;
- utilizzare `tracert`/`traceroute` per osservare il percorso verso una destinazione;
- introdurre volutamente alcuni guasti;
- distinguere un problema fisico da un problema di configurazione;
- applicare una metodologia ordinata di troubleshooting;
- utilizzare un sistema di IA come supporto alla diagnosi, senza sostituire le verifiche tecniche.

La logica che accompagnerà tutta l'attività è:

> **Progetto → Configuro → Verifico → Rompo → Osservo → Formulo ipotesi → Testo → Risolvo → Verifico nuovamente**

---

# 2. Scenario della rete

Costruiamo una rete molto semplice, ma sufficientemente ricca per osservare diversi fenomeni.

La topologia sarà:

```text
PC0
 |
 |
Switch0
 |
 |
Router0
 |
 |
PC1
```

Avremo quindi **due reti IPv4 differenti**, collegate dal router.

### Rete LAN 1

```text
192.168.10.0/24
```

### Rete LAN 2

```text
192.168.20.0/24
```

Utilizzeremo questa configurazione:

| Dispositivo | Interfaccia | Indirizzo IP | Subnet mask | Gateway |
|---|---|---:|---:|---:|
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Router0 | G0/0 | 192.168.10.1 | 255.255.255.0 | — |
| Router0 | G0/1 | 192.168.20.1 | 255.255.255.0 | — |
| PC1 | FastEthernet0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

> **Nota:** il nome delle interfacce può variare a seconda del modello di router scelto in Packet Tracer. Se il tuo dispositivo presenta `Fa0/0` e `Fa0/1` invece di `G0/0` e `G0/1`, utilizza i nomi effettivamente presenti.

---

# 3. Prima di iniziare: capire cosa stiamo costruendo

Prima di aprire Packet Tracer, fermiamoci un minuto.

PC0 e PC1 appartengono a due reti differenti:

```text
192.168.10.0/24
192.168.20.0/24
```

Per questo motivo PC0 **non può comunicare direttamente con PC1 utilizzando soltanto lo switch**.

Lo switch opera principalmente a livello 2 e inoltra le trame Ethernet utilizzando gli indirizzi MAC.

Il router invece collega le due reti IP.

Dal punto di vista di PC0:

```text
PC0
 |
 | destinazione locale?
 |
 NO
 |
 v
Default Gateway
192.168.10.1
 |
 v
Router0
 |
 v
192.168.20.0/24
 |
 v
PC1
```

Questa semplice topologia ci permetterà di osservare:

- MAC;
- ARP;
- Ethernet;
- IP;
- ICMP;
- routing;
- default gateway;
- ping;
- traceroute;
- troubleshooting.

---

# 4. Costruzione della rete in Packet Tracer

## 4.1 Creare un nuovo progetto

Apri **Cisco Packet Tracer** e crea un nuovo progetto.

Salva subito il file, per esempio come:

```text
Lezione1_Rete_Base.pkt
```

È buona pratica salvare frequentemente il lavoro.

---

## 4.2 Inserire i dispositivi

Dal pannello dei dispositivi inserisci:

- 2 PC;
- 1 switch;
- 1 router.

Per esempio:

```text
PC0
PC1
2960 Switch
Router 1941
```

Il modello preciso non è fondamentale per questa attività, purché il router disponga di due interfacce Ethernet utilizzabili.

Rinomina eventualmente i dispositivi:

```text
PC0
SW0
R0
PC1
```

Questo rende più leggibile la topologia.

---

# 5. Collegare i dispositivi

Utilizza i collegamenti Ethernet appropriati.

La topologia dovrà essere:

```text
PC0 ---- SW0 ---- R0 ---- PC1
```

Per esempio:

```text
PC0 Fa0
   |
   | Ethernet
   |
SW0 Fa0/1
SW0 Fa0/24
   |
   |
R0 G0/0

R0 G0/1
   |
   |
PC1 Fa0
```

> **Nota didattica:** Packet Tracer può consentire anche l'uso di collegamenti automatici. Durante questa attività è comunque utile osservare quali interfacce vengono collegate.

Attendi che i collegamenti diventino operativi.

---

# 6. Configurare PC0

Clicca su **PC0**.

Vai in:

```text
Desktop
→ IP Configuration
```

Imposta:

```text
IP Address:      192.168.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
```

Non è necessario configurare un DNS per questa attività.

La configurazione di PC0 sarà quindi:

```text
IP:       192.168.10.10
Mask:     255.255.255.0
Gateway:  192.168.10.1
```

---

# 7. Configurare PC1

Su PC1:

```text
Desktop
→ IP Configuration
```

imposta:

```text
IP Address:      192.168.20.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.20.1
```

Configurazione:

```text
IP:       192.168.20.10
Mask:     255.255.255.0
Gateway:  192.168.20.1
```

---

# 8. Configurare il router

Questa volta lavoriamo dalla CLI del router.

Apri:

```text
Router0
→ CLI
```

Se Packet Tracer propone la procedura iniziale di configurazione, possiamo saltarla e configurare manualmente.

Entriamo nella modalità privilegiata:

```text
Router> enable
```

Poi nella modalità di configurazione globale:

```text
Router# configure terminal
```

oppure:

```text
Router# conf t
```

Il prompt diventerà:

```text
Router(config)#
```

---

## 8.1 Configurare la prima interfaccia

Supponendo che la prima interfaccia sia `GigabitEthernet0/0`:

```text
Router(config)# interface gigabitEthernet 0/0
```

Impostiamo l'indirizzo:

```text
Router(config-if)# ip address 192.168.10.1 255.255.255.0
```

Attiviamo l'interfaccia:

```text
Router(config-if)# no shutdown
```

Il comando `no shutdown` è fondamentale.

Su molti router Cisco le interfacce sono amministrativamente disabilitate fino a quando non vengono attivate.

Usciamo:

```text
Router(config-if)# exit
```

---

## 8.2 Configurare la seconda interfaccia

Ora:

```text
Router(config)# interface gigabitEthernet 0/1
```

Configurazione:

```text
Router(config-if)# ip address 192.168.20.1 255.255.255.0
```

Attivazione:

```text
Router(config-if)# no shutdown
```

Poi:

```text
Router(config-if)# exit
```

Infine:

```text
Router(config)# end
```

---

# 9. Verificare la configurazione del router

Non dobbiamo mai considerare una configurazione corretta soltanto perché abbiamo digitato i comandi.

Dobbiamo **verificare**.

Usiamo:

```text
Router# show ip interface brief
```

Dovremmo ottenere qualcosa di simile:

```text
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.10.1    YES manual up                    up
GigabitEthernet0/1     192.168.20.1    YES manual up                    up
```

Le parole importanti sono:

```text
up
up
```

La prima indica lo stato dell'interfaccia; la seconda lo stato del protocollo di linea.

Se troviamo:

```text
administratively down
```

è molto probabile che manchi:

```text
no shutdown
```

---

# 10. Verificare la tabella di routing

Sempre sul router:

```text
Router# show ip route
```

Dovremmo vedere le due reti direttamente connesse.

Concettualmente:

```text
192.168.10.0/24 → direttamente connessa
192.168.20.0/24 → direttamente connessa
```

Questo è un passaggio molto importante.

Il router **conosce già entrambe le reti** perché sono direttamente collegate alle sue interfacce.

Non abbiamo quindi bisogno di configurare una rotta statica per questo scenario.

---

# 11. Prima verifica: PC0 → gateway

Torniamo su PC0.

Apriamo:

```text
Desktop
→ Command Prompt
```

Eseguiamo:

```text
ping 192.168.10.1
```

Se tutto è configurato correttamente, dovremmo ricevere risposte.

Il test verifica principalmente che:

- PC0 abbia una configurazione IP coerente;
- il collegamento locale funzioni;
- PC0 riesca a raggiungere il proprio gateway;
- il router risponda.

---

# 12. Seconda verifica: PC1 → gateway

Su PC1:

```text
ping 192.168.20.1
```

Anche questo test dovrebbe avere successo.

Ora abbiamo verificato entrambi i segmenti separatamente.

---

# 13. Terza verifica: PC0 → PC1

Da PC0:

```text
ping 192.168.20.10
```

Questa volta la comunicazione attraversa il router.

Il percorso logico è:

```text
PC0
 ↓
SW0
 ↓
R0 G0/0
 ↓
R0 G0/1
 ↓
PC1
```

Questo è il primo momento nel quale possiamo osservare concretamente la differenza tra:

- comunicazione locale;
- comunicazione verso un'altra rete.

---

# 14. Che cosa succede realmente durante un ping?

Il comando:

```text
ping 192.168.20.10
```

non significa semplicemente “manda un pacchetto”.

Dietro le quinte avvengono diverse operazioni.

PC0 deve prima capire se `192.168.20.10` appartiene alla propria rete.

Con la configurazione:

```text
PC0 = 192.168.10.10/24
```

la rete locale è:

```text
192.168.10.0/24
```

La destinazione:

```text
192.168.20.10
```

non appartiene alla stessa rete.

PC0 quindi deve utilizzare il **default gateway**:

```text
192.168.10.1
```

Ma per costruire la trama Ethernet deve conoscere anche il MAC del gateway.

Qui entra in gioco ARP.

---

# 15. Analizzare ARP

Su PC0:

```text
Desktop
→ Command Prompt
```

esegui:

```text
arp -a
```

Potremmo vedere una voce associata al gateway:

```text
192.168.10.1    xx-xx-xx-xx-xx-xx
```

Il MAC visualizzato dipende dal dispositivo e dall'istanza di Packet Tracer.

La cosa importante è capire il processo.

PC0 conosce:

```text
IP gateway = 192.168.10.1
```

ma per inviare una trama Ethernet sulla LAN deve conoscere:

```text
MAC gateway = ?
```

ARP serve proprio a risolvere:

```text
IP → MAC
```

---

# 16. Osservare ARP in modo più evidente

Prima di procedere possiamo svuotare la cache ARP.

Su alcuni sistemi/comandi di Packet Tracer è possibile utilizzare:

```text
arp -d
```

Se il comando non viene accettato dalla versione di Packet Tracer utilizzata, non è un problema: possiamo semplicemente osservare la prima comunicazione in **Simulation Mode**.

Dopo aver eseguito un ping verso il gateway, controlliamo nuovamente:

```text
arp -a
```

L'idea da osservare è:

```text
Prima:
IP del gateway → MAC sconosciuto

ARP:
"Chi ha 192.168.10.1?"

Risposta:
"192.168.10.1 è associato a MAC XX-XX-XX-XX-XX-XX"

Dopo:
IP → MAC memorizzato nella cache
```

---

# 17. Analizzare la tabella MAC dello switch

Ora spostiamoci su SW0.

Apriamo la CLI:

```text
SW0
→ CLI
```

Entriamo nella modalità privilegiata:

```text
Switch> enable
```

Poi:

```text
Switch# show mac address-table
```

Possiamo ottenere una tabella simile:

```text
Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type       Ports
----    -----------       --------   -----
  1     xxxx.xxxx.xxxx    DYNAMIC    Fa0/1
  1     yyyy.yyyy.yyyy    DYNAMIC    Fa0/24
```

I MAC e le porte saranno diversi nella tua simulazione.

---

# 18. Come impara lo switch?

Questo è un concetto fondamentale.

Quando una trama arriva su una porta, lo switch osserva il:

```text
MAC sorgente
```

e associa quel MAC alla porta dalla quale la trama è arrivata.

Per esempio:

```text
MAC PC0 → Fa0/1
MAC Router → Fa0/24
```

Lo switch costruisce quindi progressivamente una tabella.

Possiamo rappresentarla così:

```text
+-------------------+--------+
| MAC               | Porta  |
+-------------------+--------+
| MAC-PC0           | Fa0/1  |
| MAC-Router        | Fa0/24 |
+-------------------+--------+
```

Questa tabella permette allo switch di decidere dove inoltrare le trame.

---

# 19. Analizzare ICMP

Il comando:

```text
ping
```

utilizza normalmente **ICMP**.

ICMP non è un protocollo di trasporto come TCP o UDP.

È un protocollo utilizzato all'interno della suite IP per messaggi di controllo, diagnostica e segnalazione.

Nel caso del ping osserviamo principalmente:

```text
ICMP Echo Request
```

seguito da:

```text
ICMP Echo Reply
```

Concettualmente:

```text
PC0
 |
 | ICMP Echo Request
 v
PC1
 |
 | ICMP Echo Reply
 v
PC0
```

Quando vediamo:

```text
Reply from 192.168.20.10
```

stiamo osservando la risposta al test ICMP.

---

# 20. Usare la modalità Simulation

La modalità **Simulation** di Packet Tracer è particolarmente utile in questa lezione.

Passa da:

```text
Realtime
```

a:

```text
Simulation
```

Poi genera un ping:

```text
ping 192.168.20.10
```

Utilizza i controlli della simulazione per avanzare evento per evento.

L'obiettivo non è semplicemente vedere “il pacchetto che si muove”.

Prova a riconoscere:

1. ARP;
2. Ethernet;
3. IP;
4. ICMP;
5. passaggio attraverso lo switch;
6. passaggio attraverso il router;
7. risposta ICMP.

Questa osservazione collega direttamente la teoria dei livelli alla pratica.

---

# 21. Analizzare il percorso con traceroute

Da PC0 utilizziamo:

```text
tracert 192.168.20.10
```

In alcuni sistemi il comando è:

```text
traceroute 192.168.20.10
```

In Packet Tracer, sui PC, il comando tipicamente utilizzato è:

```text
tracert
```

Lo scopo è osservare i dispositivi di livello 3 attraversati dal traffico.

Nel nostro esempio ci aspettiamo un percorso concettualmente simile:

```text
PC0
 |
 | 1
 v
192.168.10.1
 |
 | 2
 v
192.168.20.10
```

Il numero preciso di hop e la visualizzazione possono dipendere dalla simulazione.

---

# 22. Ping e traceroute non sono la stessa cosa

È importante distinguere i due strumenti.

### Ping

Domanda principalmente:

> "La destinazione risponde?"

### Traceroute / tracert

Domanda principalmente:

> "Quale percorso viene seguito per raggiungere la destinazione?"

Quindi:

```text
ping
→ connettività / raggiungibilità

tracert
→ percorso
```

Durante il troubleshooting questa distinzione è molto utile.

---

# 23. Prima metodologia di troubleshooting

A questo punto introduciamo volutamente alcuni problemi.

La regola principale è:

> **Non correggere immediatamente il problema. Prima osservalo e cerca di diagnosticarlo.**

Per ogni guasto dobbiamo seguire una sequenza:

```text
1. Osserva il problema
2. Raccogli informazioni
3. Formula un'ipotesi
4. Esegui un test
5. Conferma o smentisci l'ipotesi
6. Applica la correzione
7. Verifica nuovamente
8. Documenta la causa
```

Questo sarà il nostro primo vero esercizio di troubleshooting.

---

# 24. Guasto 1 — Cavo scollegato

## 24.1 Creare il guasto

In Packet Tracer rimuovi oppure scollega il collegamento tra:

```text
PC0 ↔ SW0
```

Ora prova:

```text
ping 192.168.10.1
```

Il ping dovrebbe fallire.

---

## 24.2 Non guardare subito la soluzione

Prima chiediti:

> Qual è il livello più basso che posso controllare?

La prima ipotesi deve riguardare la connettività fisica.

Controlla:

- collegamento presente?
- porta corretta?
- stato del link?
- interfaccia attiva?
- cavo corretto?

In una rete reale controlleremmo anche:

- connettore;
- patch cord;
- presa;
- patch panel;
- porta dello switch;
- LED di collegamento.

---

## 24.3 Ripristinare il collegamento

Ricollega:

```text
PC0 ↔ SW0
```

Attendi che il collegamento diventi operativo.

Ripeti:

```text
ping 192.168.10.1
```

Poi:

```text
ping 192.168.20.10
```

Il test dovrebbe tornare a funzionare.

---

# 25. Guasto 2 — Indirizzo IP errato

Ora creiamo un problema diverso.

Su PC0 modifica temporaneamente l'indirizzo:

```text
IP Address:
192.168.30.10
```

mantenendo:

```text
Subnet Mask:
255.255.255.0

Default Gateway:
192.168.10.1
```

Ora prova:

```text
ping 192.168.10.1
```

e:

```text
ping 192.168.20.10
```

---

# 26. Diagnosticare il guasto IP

Prima di correggere la configurazione, chiediamoci:

```text
PC0 appartiene alla rete corretta?
```

Controlliamo:

```text
ipconfig
```

Dovremmo vedere:

```text
IP Address
Subnet Mask
Default Gateway
```

Confrontiamo i valori con il progetto.

La configurazione prevista era:

```text
IP:
192.168.10.10

Mask:
255.255.255.0

Gateway:
192.168.10.1
```

Abbiamo quindi individuato una differenza tra:

```text
configurazione attuale
```

e:

```text
configurazione prevista
```

Questo è già troubleshooting.

---

# 27. Correggere il guasto IP

Ripristina:

```text
IP Address:
192.168.10.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.10.1
```

Poi verifica progressivamente:

```text
ping 192.168.10.1
```

e:

```text
ping 192.168.20.10
```

Non fermarti al primo ping riuscito.

La verifica deve arrivare fino alla destinazione finale.

---

# 28. Guasto 3 — Default gateway errato

Ora lasciamo corretto l'indirizzo IP:

```text
192.168.10.10
```

ma modifichiamo il gateway:

```text
Default Gateway:
192.168.10.254
```

Questo indirizzo non corrisponde al router configurato nella nostra rete.

Proviamo:

```text
ping 192.168.10.1
```

Il gateway reale sarà comunque raggiungibile perché appartiene alla stessa rete locale.

Questo è un punto importante.

Il fatto che:

```text
ping 192.168.10.1
```

funzioni **non dimostra** che il default gateway configurato sia corretto.

Il vero test è:

```text
ping 192.168.20.10
```

Se la configurazione del gateway è errata, PC0 non dispone del next hop corretto per il traffico destinato alla rete remota.

---

# 29. Diagnosticare il default gateway

Utilizziamo:

```text
ipconfig
```

Controlliamo:

```text
Default Gateway
```

e confrontiamolo con la configurazione del router:

```text
R0 G0/0 = 192.168.10.1
```

La configurazione corretta di PC0 deve quindi essere:

```text
IP:
192.168.10.10

Mask:
255.255.255.0

Gateway:
192.168.10.1
```

Ripristiniamo il gateway e ripetiamo:

```text
ping 192.168.20.10
```

---

# 30. Costruire una tabella di troubleshooting

Durante le attività conviene documentare ogni guasto.

| Problema | Sintomo | Ipotesi | Test | Risultato | Correzione |
|---|---|---|---|---|---|
| Cavo scollegato | Ping fallisce | Problema fisico | Controllo link | Confermata | Ricollegare cavo |
| IP errato | Comunicazione assente | Configurazione IP | `ipconfig` | Confermata | Correggere IP |
| Gateway errato | Rete locale OK, rete remota KO | Gateway errato | `ipconfig` + ping remoto | Confermata | Correggere gateway |

Questa tabella introduce un principio importante:

> **Una diagnosi professionale deve lasciare una traccia del ragionamento.**

---

# 31. Prima metodologia diagnostica con l'IA

Ora introduciamo l'intelligenza artificiale.

L'obiettivo **non** è chiedere:

> "Perché il ping non funziona?"

e accettare la prima risposta.

L'obiettivo è utilizzare l'IA come **assistente al ragionamento diagnostico**.

La domanda corretta deve contenere:

- topologia;
- configurazione;
- sintomo;
- test già eseguiti;
- risultati;
- eventuali vincoli.

---

# 32. Prompt iniziale per l'IA

Possiamo utilizzare un prompt come questo:

```text
Sto facendo troubleshooting di una rete IPv4 in Cisco Packet Tracer.

Topologia:

PC0 — Switch0 — Router0 — PC1

Reti:

LAN 1: 192.168.10.0/24
LAN 2: 192.168.20.0/24

Configurazione prevista:

PC0:
IP 192.168.10.10/24
Gateway 192.168.10.1

Router0:
G0/0 192.168.10.1/24
G0/1 192.168.20.1/24

PC1:
IP 192.168.20.10/24
Gateway 192.168.20.1

Problema:
PC0 non riesce a raggiungere PC1.

Test già eseguiti:
- ping 192.168.10.1 → [inserire risultato]
- ping 192.168.20.10 → [inserire risultato]
- ipconfig su PC0 → [inserire risultato]
- show ip interface brief su Router0 → [inserire risultato]
- show ip route su Router0 → [inserire risultato]

Non darmi subito la soluzione.
Costruisci invece una sequenza di troubleshooting dal livello più basso al più alto.
Per ogni verifica indica:
1. cosa devo controllare;
2. quale comando o strumento usare;
3. quale risultato mi aspetto;
4. come interpretare un risultato anomalo.
```

---

# 33. Perché il prompt è costruito così?

Stiamo insegnando all'IA la nostra metodologia:

```text
Sintomo
 ↓
Raccolta dati
 ↓
Verifica fisica
 ↓
Verifica locale
 ↓
Verifica IP
 ↓
Verifica gateway
 ↓
Verifica routing
 ↓
Verifica destinazione
 ↓
Ipotesi
 ↓
Test
 ↓
Correzione
 ↓
Nuova verifica
```

L'IA non deve quindi diventare un "oracolo".

Deve diventare un **secondo punto di vista**.

---

# 34. Esercizio: confrontare la diagnosi umana con quella dell'IA

Dividiamo la classe in piccoli gruppi.

Ogni gruppo riceve un problema.

Esempi:

### Scenario A

```text
PC0 non raggiunge il gateway.
```

### Scenario B

```text
PC0 raggiunge il gateway ma non PC1.
```

### Scenario C

```text
PC0 raggiunge PC1 ma il percorso non è quello atteso.
```

Ogni gruppo deve produrre:

1. sintomo;
2. ipotesi iniziale;
3. test;
4. risultato;
5. nuova ipotesi;
6. correzione;
7. verifica finale.

Poi si chiede all'IA di produrre una metodologia diagnostica.

Infine si confrontano:

```text
Diagnosi degli studenti
        ↕
Diagnosi proposta dall'IA
```

La domanda finale non è:

> "Chi ha ragione?"

ma:

> **"Quali verifiche ci permettono di stabilire quale ipotesi è supportata dalle evidenze?"**

---

# 35. Regola fondamentale nell'uso dell'IA

L'IA può:

- proporre ipotesi;
- suggerire comandi;
- spiegare un output;
- proporre una sequenza diagnostica;
- aiutare a confrontare più cause possibili.

L'IA non deve:

- sostituire le verifiche;
- inventare risultati non osservati;
- essere considerata una fonte infallibile;
- portare a modificare la configurazione senza aver prima compreso il problema.

La regola che suggerisco agli studenti è:

> **Prima osserva, poi chiedi, poi verifica.**

Non:

> **Chiedi, copia e spera.**

---

# 36. Checklist finale della lezione

## Topologia

- [ ] PC0 inserito
- [ ] PC1 inserito
- [ ] Switch inserito
- [ ] Router inserito
- [ ] Collegamenti realizzati

## Configurazione

- [ ] PC0 configurato
- [ ] PC1 configurato
- [ ] G0/0 configurata
- [ ] G0/1 configurata
- [ ] Interfacce attive
- [ ] Tabella di routing verificata

## Analisi

- [ ] `ipconfig`
- [ ] `arp -a`
- [ ] `show mac address-table`
- [ ] `show ip interface brief`
- [ ] `show ip route`
- [ ] `ping`
- [ ] `tracert`
- [ ] Simulation Mode

## Troubleshooting

- [ ] Cavo scollegato
- [ ] IP errato
- [ ] Gateway errato
- [ ] Sintomo osservato
- [ ] Ipotesi formulata
- [ ] Test eseguito
- [ ] Problema corretto
- [ ] Verifica finale eseguita

## IA

- [ ] Prompt diagnostico utilizzato
- [ ] Metodologia proposta dall'IA analizzata
- [ ] Risultati reali confrontati con le ipotesi
- [ ] Nessuna modifica applicata senza verifica

---

# 37. Conclusione

Questa prima attività può sembrare molto semplice: quattro dispositivi, due reti e qualche comando.

In realtà contiene già molti dei concetti sui quali costruiremo il resto del corso.

Abbiamo visto che una comunicazione non è semplicemente:

```text
PC → PC
```

ma può essere descritta come:

```text
Applicazione
   ↓
TCP/UDP
   ↓
IP
   ↓
Ethernet/Wi-Fi
   ↓
Switch
   ↓
Router
   ↓
Altra rete
   ↓
Destinazione
```

Abbiamo poi trasformato un errore in un'opportunità didattica.

Un cavo scollegato, un indirizzo IP errato o un gateway sbagliato non devono essere considerati semplicemente "problemi da sistemare". Sono occasioni per imparare a ragionare.

Il troubleshooting professionale non consiste nel provare comandi a caso fino a quando qualcosa torna a funzionare.

Consiste nel costruire una sequenza logica:

> **Osservo → raccolgo dati → formulo un'ipotesi → eseguo un test → interpreto il risultato → correggo → verifico.**

Ed è proprio qui che introduciamo l'IA.

Un buon tecnico non utilizza l'IA per evitare di pensare. La utilizza per pensare meglio, confrontare ipotesi e organizzare le verifiche.

La competenza che vogliamo iniziare a costruire già dalla prima lezione è quindi molto semplice da enunciare:

> **Non cercare subito la soluzione. Cerca prima di capire perché il problema si manifesta.**
