# Lezione 2 — Laboratorio pratico
## Ethernet, mezzi trasmissivi e switching Layer 2

**Durata:** 140 minuti  
**Strumento:** Cisco Packet Tracer  
**Obiettivo:** verificare direttamente il comportamento di Ethernet e di uno switch Layer 2 attraverso configurazione, osservazione della tabella MAC, Simulation Mode e troubleshooting.

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio dovrai essere in grado di:

- costruire una semplice LAN Ethernet;
- configurare gli indirizzi IPv4 dei due PC;
- configurare e verificare le porte di uno switch Cisco;
- leggere una tabella MAC;
- spiegare il processo di **learning**;
- distinguere **forwarding**, **flooding** e **filtering**;
- osservare ARP ed Ethernet in Simulation Mode;
- capire la differenza tra MAC e IP;
- simulare alcuni guasti Layer 1 e Layer 2;
- applicare una metodologia di troubleshooting;
- utilizzare l'AI come supporto alla diagnosi senza sostituire la raccolta delle evidenze.

> **Principio guida**
>
> **Non cambiare configurazione senza prima aver raccolto evidenze.**

---

## 2. Scenario di laboratorio

Realizziamo una LAN minima:

```text
PC0 -------- SW0 -------- PC1
            Fa0/1        Fa0/2
```

Non utilizziamo un router: vogliamo concentrarci sul comportamento **Layer 2**.

### Indirizzamento

| Dispositivo | Interfaccia | Indirizzo IP | Subnet mask | Gateway |
|---|---|---|---|---|
| PC0 | FastEthernet | 192.168.10.10 | 255.255.255.0 | — |
| PC1 | FastEthernet | 192.168.10.20 | 255.255.255.0 | — |
| SW0 | Fa0/1 | — | — | — |
| SW0 | Fa0/2 | — | — | — |

### Attrezzatura

- 1 switch Cisco 2960
- 2 PC
- 2 collegamenti Ethernet

Salva il progetto con il nome:

```text
Lezione2_Ethernet_Switching_L2.pkt
```

---

## 3. Fase 1 — Costruzione della topologia

### 3.1 Inserisci i dispositivi

In Cisco Packet Tracer:

1. inserisci **2 PC**;
2. inserisci uno **Switch 2960**;
3. collega PC0 allo switch sulla porta **Fa0/1**;
4. collega PC1 allo switch sulla porta **Fa0/2**.

La topologia finale deve essere:

```text
PC0 ---------------- SW0 ---------------- PC1
        Fa0/1                 Fa0/2
```

### Verifica

Prima di procedere controlla che i collegamenti risultino attivi.

> **Suggerimento**
>
> Se un collegamento non risulta attivo, non iniziare ancora a controllare gli indirizzi IP. Parti dal **Layer 1**: cavo, porta e stato dell'interfaccia.

---

## 4. Fase 2 — Configurazione degli indirizzi IP

### 4.1 Configura PC0

Apri:

**PC0 → Desktop → IP Configuration**

Imposta:

```text
IP Address:    192.168.10.10
Subnet Mask:   255.255.255.0
Default Gateway: lasciare vuoto
```

### 4.2 Configura PC1

Apri:

**PC1 → Desktop → IP Configuration**

Imposta:

```text
IP Address:    192.168.10.20
Subnet Mask:   255.255.255.0
Default Gateway: lasciare vuoto
```

### Perché non serve il gateway?

I due PC appartengono alla stessa rete:

```text
192.168.10.0/24
```

Quindi PC0 può comunicare direttamente con PC1 senza passare attraverso un router.

---

## 5. Fase 3 — Configurazione dello switch

Accedi alla CLI di SW0.

### 5.1 Configura Fa0/1

```cisco
enable
configure terminal
interface fastethernet 0/1
description COLLEGAMENTO_PC0
no shutdown
exit
```

### 5.2 Configura Fa0/2

```cisco
interface fastethernet 0/2
description COLLEGAMENTO_PC1
no shutdown
exit
```

### 5.3 Salva la configurazione

```cisco
end
copy running-config startup-config
```

> **Perché usare `description`?**
>
> Non modifica il comportamento dello switching. Serve a documentare la funzione della porta e rende più leggibile la configurazione in un ambiente reale.

---

## 6. Fase 4 — Verifica delle porte

Sullo switch esegui:

```cisco
show interfaces status
```

Osserva:

- porta;
- stato;
- VLAN;
- duplex;
- velocità;
- tipo di interfaccia.

Per una porta correttamente collegata dovresti osservare uno stato compatibile con:

```text
connected
```

### 6.1 Analizza una singola interfaccia

```cisco
show interfaces fastethernet 0/1
```

Osserva:

- stato dell'interfaccia;
- line protocol;
- indirizzo MAC;
- velocità;
- duplex;
- contatori;
- eventuali errori.

---

## 7. Fase 5 — Prima osservazione della tabella MAC

Esegui:

```cisco
show mac address-table
```

La tabella potrebbe non contenere ancora le associazioni che ci interessano.

Questo è normale.

Lo switch costruisce dinamicamente la propria tabella osservando i frame che attraversano le porte.

> **Domanda**
>
> Come fa lo switch a sapere che un determinato MAC si trova su una determinata porta?
>
> Attraverso il **MAC learning**: osserva il MAC sorgente dei frame ricevuti e lo associa alla porta di ingresso.

---

## 8. Fase 6 — Generiamo traffico

Dal PC0 apri:

**Desktop → Command Prompt**

Esegui:

```text
ping 192.168.10.20
```

Dovresti ottenere risposte da PC1.

### 8.1 Ripeti la verifica della tabella MAC

Torna su SW0:

```cisco
show mac address-table
```

Dovresti trovare concettualmente:

```text
MAC PC0 → Fa0/1
MAC PC1 → Fa0/2
```

I MAC effettivi saranno diversi.

### Domanda fondamentale

**Come ha fatto lo switch a sapere che il MAC di PC0 si trova su Fa0/1?**

Perché quando un frame è entrato da Fa0/1, lo switch ha letto il **MAC sorgente** e ha associato quel MAC alla porta di ingresso.

Questo è:

```text
LEARNING
```

---

## 9. Fase 7 — Capire il forwarding

Supponiamo che lo switch abbia appreso:

```text
AAAA.AAAA.AAAA → Fa0/1
BBBB.BBBB.BBBB → Fa0/2
```

Se arriva un frame:

```text
MAC source      = AAAA.AAAA.AAAA
MAC destination = BBBB.BBBB.BBBB
```

lo switch consulta la tabella e trova:

```text
BBBB.BBBB.BBBB → Fa0/2
```

Quindi inoltra il frame verso:

```text
Fa0/2
```

Questo comportamento è:

> **FORWARDING**

Lo switch non deve inviare il frame verso tutte le porte perché conosce già la destinazione.

---

## 10. Fase 8 — Osservare ARP

Il ping può richiedere prima una risoluzione:

```text
IP → MAC
```

con ARP.

Dal PC0:

```text
arp -a
```

Dovresti trovare un'associazione tra:

```text
192.168.10.20
```

e il MAC di PC1.

Una richiesta ARP utilizza come destinazione Ethernet:

```text
FF:FF:FF:FF:FF:FF
```

quindi è un **broadcast**.

---

## 11. Fase 9 — Simulation Mode

Passa da:

```text
Realtime
```

a:

```text
Simulation
```

Genera nuovamente il traffico e usa:

```text
Capture / Forward
```

per procedere passo dopo passo.

Osserva:

1. ARP;
2. Ethernet;
3. IP;
4. ICMP.

L'obiettivo è ricostruire la sequenza:

```text
PC0
 ↓
ARP
 ↓
SW0
 ↓
PC1
 ↓
ARP Reply
 ↓
ICMP
```

Non limitarti a verificare se il ping funziona: osserva **come** funziona.

---

## 12. Fase 10 — Esperimento sul flooding

Una richiesta ARP è un esempio di broadcast.

Destinazione Ethernet:

```text
FF:FF:FF:FF:FF:FF
```

Lo switch diffonde il frame sulle altre porte della VLAN, esclusa la porta di ingresso.

### Domande

1. Perché PC1 riceve la richiesta?
2. Perché lo switch non invia il frame nuovamente verso la porta di ingresso?
3. Cosa succederebbe con molti altri dispositivi nella LAN?

Annota le risposte.

---

## 13. Fase 11 — Esperimento sul forwarding

Dopo che lo switch ha imparato i MAC, ripeti:

```text
ping 192.168.10.20
```

Osserva nuovamente il traffico in Simulation Mode.

Confronta:

```text
Broadcast / Flooding
```

con:

```text
Unicast / Forwarding
```

---

## 14. Fase 12 — Spostiamo PC1

Sposta PC1 dalla porta:

```text
Fa0/2
```

alla porta:

```text
Fa0/3
```

Genera nuovamente traffico da PC0 verso PC1.

Poi:

```cisco
show mac address-table
```

L'associazione dovrebbe riflettere la nuova porta dopo che lo switch ha osservato nuovo traffico:

```text
Prima:

MAC_PC1 → Fa0/2

Dopo:

MAC_PC1 → Fa0/3
```

Questo è un esempio di **MAC learning dinamico**.

---

## 15. Fase 13 — Cambiamo solo l'IP

Riporta PC1 su Fa0/2.

Modifica soltanto l'indirizzo IP:

```text
192.168.10.20
```

in:

```text
192.168.10.30
```

Lascia invariato il collegamento fisico.

### Osservazione

Il MAC della scheda di rete non cambia semplicemente perché cambia l'IP.

Quindi:

```text
IP ≠ MAC
```

L'esperimento rende concreta la differenza tra indirizzamento logico e identificazione Layer 2.

---

## 16. Fase 14 — Scollegamento del cavo

Scollega il collegamento tra PC1 e SW0.

Da PC0:

```text
ping 192.168.10.30
```

Il ping dovrebbe fallire.

### 16.1 Controlla lo stato della porta

```cisco
show interfaces status
```

Poi:

```cisco
show interfaces fastethernet 0/2
```

### 16.2 Controlla la tabella MAC

```cisco
show mac address-table
```

Una voce dinamica può rimanere temporaneamente nella tabella prima di essere rimossa tramite **aging**.

> **Domanda**
>
> Una voce MAC presente nella tabella significa necessariamente che il dispositivo è attualmente raggiungibile?
>
> **No.** Le informazioni dinamiche possono diventare obsolete fino alla loro rimozione.

---

## 17. Fase 15 — Speed, duplex e contatori

Esegui:

```cisco
show interfaces fastethernet 0/1
```

Osserva:

- speed;
- duplex;
- input packets;
- output packets;
- errori;
- altri contatori disponibili.

`show interfaces` non serve soltanto a sapere se una porta è attiva: fornisce informazioni utili per il troubleshooting.

---

## 18. Fase 16 — Mini troubleshooting Layer 1 → Layer 2 → Layer 3

Problema:

> PC0 non riesce a raggiungere PC1.

Non cambiare subito gli indirizzi IP.

### Step 1 — Layer 1

```cisco
show interfaces status
```

Controlla:

- porta `connected`;
- cavo;
- porta utilizzata;
- stato del link.

### Step 2 — Layer 2

```cisco
show mac address-table
```

Controlla:

- MAC di PC0;
- MAC di PC1;
- associazioni MAC/porta;
- corrispondenza tra tabella e topologia.

### Step 3 — Layer 3

Sui PC:

```text
ipconfig
```

Verifica:

```text
PC0 → 192.168.10.10/24
PC1 → 192.168.10.20/24
```

Poi:

```text
ping 192.168.10.20
```

---

## 19. Fase 17 — Utilizzare l'AI per il troubleshooting

L'AI deve essere utilizzata come **strumento di supporto alla diagnosi**, non come sostituto dell'osservazione della rete.

Usa questo prompt:

```text
Sto facendo troubleshooting di una piccola LAN Ethernet in Cisco Packet Tracer.

Topologia:
PC0 ---- SW0 ---- PC1

Configurazione prevista:

PC0:
IP 192.168.10.10/24

PC1:
IP 192.168.10.20/24

SW0:
PC0 collegato a Fa0/1
PC1 collegato a Fa0/2

Problema:
PC0 non riesce a raggiungere PC1.

Dati raccolti:

ping 192.168.10.20:
[incollare risultato]

ipconfig su PC0:
[incollare risultato]

ipconfig su PC1:
[incollare risultato]

show interfaces status:
[incollare risultato]

show mac address-table:
[incollare risultato]

Non darmi subito la soluzione.

Costruisci una sequenza diagnostica Layer 1 → Layer 2 → Layer 3.

Per ogni verifica indica:

1. cosa controllare;
2. comando o strumento;
3. risultato atteso;
4. possibile anomalia;
5. quale ipotesi confermerebbe o smentirebbe.

Non inventare risultati che non ti ho fornito.

Se mancano dati, indica quali informazioni devo raccogliere.
```

### Uso corretto dell'AI

Prima:

```text
Raccolgo evidenze
```

poi:

```text
Formulo ipotesi
```

poi:

```text
Eseguo test
```

infine:

```text
Confermo o smentisco l'ipotesi
```

L'AI può aiutare a ordinare i test e formulare ipotesi, ma non deve inventare risultati che non sono stati raccolti.

---

## 20. Sfida finale

### Domanda 1

Quando un frame entra nello switch, quale MAC viene utilizzato per il learning?

**Risposta:** il MAC sorgente.

### Domanda 2

Se il MAC destination è noto, cosa fa lo switch?

**Risposta:** inoltra il frame verso la porta associata alla destinazione.

### Domanda 3

Se il MAC destination non è noto?

**Risposta:** flooding sulle altre porte della VLAN, esclusa quella di ingresso.

### Domanda 4

Qual è il MAC di destinazione di una trasmissione broadcast?

```text
FF:FF:FF:FF:FF:FF
```

### Domanda 5

Se cambio l'IP di PC1, il MAC cambia automaticamente?

**Risposta:** no.

### Domanda 6

Se sposto PC1 da Fa0/2 a Fa0/3?

**Risposta:** lo switch deve apprendere nuovamente l'associazione tra il MAC di PC1 e la nuova porta.

---

## 21. Checklist finale

- [ ] Ho costruito la topologia PC0 — SW0 — PC1.
- [ ] Ho configurato PC0 con 192.168.10.10/24.
- [ ] Ho configurato PC1 con 192.168.10.20/24.
- [ ] Ho configurato Fa0/1 e Fa0/2.
- [ ] Ho utilizzato `description`.
- [ ] Ho verificato `show interfaces status`.
- [ ] Ho verificato `show interfaces`.
- [ ] Ho osservato `show mac address-table`.
- [ ] Ho eseguito un ping.
- [ ] Ho osservato ARP.
- [ ] Ho utilizzato Simulation Mode.
- [ ] Ho osservato il flooding.
- [ ] Ho osservato il forwarding.
- [ ] Ho spostato PC1 su un'altra porta.
- [ ] Ho osservato il nuovo MAC learning.
- [ ] Ho cambiato l'IP senza cambiare il MAC.
- [ ] Ho simulato un guasto fisico.
- [ ] Ho seguito la sequenza Layer 1 → Layer 2 → Layer 3.
- [ ] Ho utilizzato l'AI fornendo dati reali.
- [ ] Ho verificato nuovamente la soluzione dopo la diagnosi.

---

## 22. Conclusione

Il laboratorio segue il metodo:

```text
PROGETTO
   ↓
CONFIGURO
   ↓
VERIFICO
   ↓
ROMPO
   ↓
OSSERVO
   ↓
FORMULO IPOTESI
   ↓
TESTO
   ↓
RISOLVO
   ↓
VERIFICO NUOVAMENTE
   ↓
DOCUMENTO
```

La competenza professionale non consiste nel conoscere a memoria un comando.

Consiste nel saper **osservare una rete, raccogliere evidenze, formulare ipotesi e verificare le proprie conclusioni**.

> **Un buon tecnico non cambia configurazione alla cieca: prima osserva, poi ragiona, infine interviene.**
