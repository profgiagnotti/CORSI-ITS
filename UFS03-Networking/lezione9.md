# Lezione 9 — Laboratorio: Wi-Fi, sicurezza e SSH

**Corso:** UFS 03 — Networking
**Lezione:** 9 di 11
**Durata:** 150 minuti
**Ambiente:** Cisco Packet Tracer
**Modalità:** laboratorio guidato + Fault Injection + troubleshooting con AI

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio lo studente sarà in grado di:

- configurare una rete wireless simulata;
- distinguere SSID, AP, controller/WLC e client wireless;
- associare una rete Wi-Fi a una VLAN dedicata;
- applicare una configurazione di sicurezza wireless di base;
- verificare la connettività tra client wireless e rete cablata;
- configurare l'accesso remoto sicuro tramite SSH a uno switch Cisco;
- distinguere SSH da Telnet dal punto di vista della sicurezza;
- individuare configurazioni deboli o rischiose;
- applicare una metodologia strutturata di troubleshooting;
- utilizzare l'AI come supporto all'analisi dei fault senza delegarle automaticamente la soluzione.

> **Metodo di lavoro del corso**
>
> **OSSERVA → RACCOGLI LE EVIDENZE → FORMULA IPOTESI → TESTA → INTERPRETA → CONFERMA O SMENTISCI → RISOLVI → VERIFICA → DOCUMENTA**

---

# 2. Scenario del laboratorio

L'azienda dispone di una piccola rete con una parte cablata e una parte wireless.

L'obiettivo è separare la rete Wi-Fi degli utenti dalla rete di gestione e configurare l'accesso amministrativo allo switch utilizzando SSH.

Topologia di riferimento:

```text
                         +----------------+
                         |    Router      |
                         |    R1          |
                         +-------+--------+
                                 |
                              trunk
                                 |
                         +-------+--------+
                         |    Switch      |
                         |     SW1       |
                         +---+--------+---+
                             |        |
                         access      access
                         VLAN 10     VLAN 99
                             |        |
                            AP1      PC-Admin
                             |
                         ))  Wi-Fi  ((
                             |
                            PC0
```

> **Nota Packet Tracer**
>
> La disponibilità e il comportamento dei dispositivi wireless possono variare in base alla versione di Cisco Packet Tracer. Se il modello WLC/AP previsto dal laboratorio non è disponibile o presenta un'interfaccia differente, utilizzare i dispositivi wireless equivalenti presenti nella propria versione e mantenere invariati gli obiettivi didattici.

---

# 3. Piano di indirizzamento

| Dispositivo | Interfaccia | VLAN | Indirizzo IPv4 | Subnet mask | Gateway |
|---|---|---:|---|---|---|
| R1 | G0/0.10 | 10 | 192.168.10.1 | 255.255.255.0 | — |
| R1 | G0/0.99 | 99 | 192.168.99.1 | 255.255.255.0 | — |
| SW1 | VLAN 99 | 99 | 192.168.99.2 | 255.255.255.0 | 192.168.99.1 |
| AP1 | Management | 99 | 192.168.99.10 | 255.255.255.0 | 192.168.99.1 |
| PC-Admin | NIC | 99 | 192.168.99.10* | 255.255.255.0 | 192.168.99.1 |
| PC0 | Wireless | 10 | DHCP | /24 | 192.168.10.1 |

> **Attenzione:** nell'esempio sopra AP1 e PC-Admin non devono utilizzare lo stesso indirizzo IP. Assegnare, ad esempio, **AP1 = 192.168.99.10** e **PC-Admin = 192.168.99.20**.

### Configurazione definitiva consigliata

- VLAN 10 → `WIFI-USERS` → `192.168.10.0/24`
- VLAN 99 → `MGMT` → `192.168.99.0/24`
- R1 VLAN 10 → `192.168.10.1/24`
- R1 VLAN 99 → `192.168.99.1/24`
- SW1 management → `192.168.99.2/24`
- AP1 management → `192.168.99.10/24`
- PC-Admin → `192.168.99.20/24`
- PC0 wireless → DHCP sulla VLAN 10

---

# 4. Fase 1 — Creazione delle VLAN

## 4.1 Obiettivo

Creare una VLAN dedicata agli utenti Wi-Fi e una VLAN separata per la gestione dei dispositivi.

Configurare su SW1:

```text
VLAN 10  → WIFI-USERS
VLAN 99  → MGMT
```

### Comandi

```cisco
enable
configure terminal

vlan 10
 name WIFI-USERS
exit

vlan 99
 name MGMT
exit

end
show vlan brief
```

### Verifica

Controllare che entrambe le VLAN siano presenti.

**Domande:**

1. Perché è preferibile non utilizzare la VLAN 1 per gli utenti wireless?
2. Qual è la differenza tra VLAN utenti e VLAN di management?
3. La VLAN, da sola, rappresenta una misura di sicurezza completa?

---

# 5. Fase 2 — Configurazione delle porte dello switch

Supponiamo che:

- `Fa0/1` colleghi l'AP1;
- `Fa0/2` colleghi il PC-Admin;
- `Gi0/1` sia il collegamento verso R1.

## 5.1 Porta verso AP1

Se l'AP utilizzato nel laboratorio opera come dispositivo access per una singola WLAN:

```cisco
configure terminal
interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
exit
end
```

> Se il modello AP/WLC utilizzato richiede una configurazione differente, non applicare automaticamente questi comandi. Prima osservare il modello e la modalità di collegamento prevista.

## 5.2 Porta PC-Admin

```cisco
configure terminal
interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast
 no shutdown
exit
end
```

## 5.3 Verifica

```cisco
show vlan brief
show interfaces status
```

Annotare:

- porta;
- VLAN;
- stato;
- eventuali anomalie.

---

# 6. Fase 3 — Trunk tra Switch e Router

Il collegamento tra SW1 e R1 deve trasportare le VLAN necessarie.

Sul collegamento `Gi0/1` dello switch:

```cisco
configure terminal
interface gigabitEthernet 0/1
 switchport mode trunk
 no shutdown
exit
end
```

Verificare:

```cisco
show interfaces trunk
```

### Domande

- Perché una porta access non è sufficiente per trasportare più VLAN?
- Che cosa significa che una porta è in modalità trunk?
- Quale problema può provocare un trunk configurato come access?

---

# 7. Fase 4 — Router-on-a-Stick

Per consentire al router di gestire le due VLAN, configurare subinterface sul router R1.

```cisco
enable
configure terminal

interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
exit

end
```

Verificare:

```cisco
show ip interface brief
```

Poi verificare la connettività:

```text
PC-Admin → 192.168.99.1
```

---

# 8. Fase 5 — Configurazione della rete Wi-Fi

Configurare sul dispositivo wireless disponibile nel laboratorio:

```text
SSID: ITS-Azienda
Sicurezza: WPA2/WPA3, se disponibile
Password: utilizzare una password di laboratorio robusta
Rete: VLAN 10
```

Per il laboratorio **non utilizzare** la password debole del challenge finale (`azienda123`) come configurazione definitiva.

### Obiettivo didattico

Gli studenti devono capire che:

```text
SSID ≠ sicurezza

Password ≠ unica misura di sicurezza

VLAN ≠ crittografia
```

Una WLAN correttamente progettata richiede più livelli di protezione.

---

# 9. Fase 6 — DHCP per i client wireless

Configurare sul router un pool DHCP per la VLAN 10.

```cisco
configure terminal

ip dhcp excluded-address 192.168.10.1 192.168.10.20

ip dhcp pool WIFI-USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
exit

end
```

Verificare:

```cisco
show ip dhcp binding
show ip dhcp pool
```

Sul client wireless verificare che l'indirizzo sia stato ricevuto automaticamente.

---

# 10. Fase 7 — Verifica della connettività Wi-Fi

Dal client wireless verificare progressivamente:

### Test 1 — indirizzo IP

Controllare che il client abbia ricevuto un indirizzo `192.168.10.x`.

### Test 2 — gateway

```text
ping 192.168.10.1
```

### Test 3 — management network

```text
ping 192.168.99.2
```

### Test 4 — PC amministrativo

```text
ping 192.168.99.20
```

Per ogni test compilare:

| Test | Risultato | Evidenza | Ipotesi |
|---|---|---|---|
| DHCP | | | |
| Gateway | | | |
| VLAN 99 | | | |
| PC-Admin | | | |

---

# 11. Fase 8 — Configurazione SSH su SW1

Ora trasformiamo lo switch in un dispositivo amministrabile in modo sicuro.

## 11.1 Configurare hostname

```cisco
enable
configure terminal
hostname SW1
```

## 11.2 Configurare dominio

```cisco
ip domain-name its-network.local
```

## 11.3 Creare un utente locale

```cisco
username admin privilege 15 secret <PASSWORD_DI_LABORATORIO>
```

> Non pubblicare password reali nella repository GitHub. Utilizzare un placeholder o una password specifica per il laboratorio.

## 11.4 Generare le chiavi RSA

```cisco
crypto key generate rsa modulus 1024
```

Se la versione di Packet Tracer/dispositivo richiede una procedura diversa, seguire il prompt del dispositivo.

## 11.5 Abilitare SSH sulle linee VTY

```cisco
line vty 0 4
 login local
 transport input ssh
exit
```

## 11.6 Impostare SSH version 2

```cisco
ip ssh version 2
end
```

Verificare:

```cisco
show ip ssh
show running-config | section line vty
```

---

# 12. Fase 9 — Configurazione dell'IP di management dello switch

Configurare la SVI della VLAN 99:

```cisco
configure terminal
interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
exit

ip default-gateway 192.168.99.1
end
```

Verificare:

```cisco
show ip interface brief
```

Il risultato atteso è che `Vlan99` sia operativa.

---

# 13. Fase 10 — Connessione SSH dal PC-Admin

Dal PC-Admin utilizzare il client SSH disponibile nella versione di Packet Tracer.

Schema:

```text
PC-Admin
   |
   | SSH
   v
 SW1
192.168.99.2
```

Utilizzare:

```text
IP: 192.168.99.2
Username: admin
Password: <PASSWORD_DI_LABORATORIO>
Protocollo: SSH
```

### Obiettivo

Dimostrare che l'amministrazione remota può essere effettuata senza utilizzare Telnet.

---

# 14. SSH vs Telnet

Compilare la tabella:

| Caratteristica | Telnet | SSH |
|---|---|---|
| Accesso remoto | Sì | Sì |
| Cifratura della sessione | No | Sì |
| Protezione delle credenziali | Debole/non cifrata | Protetta tramite SSH |
| Utilizzo consigliato per management | No | Sì |
| Porta TCP predefinita | 23 | 22 |

### Domanda

Perché non è sufficiente dire “SSH è migliore perché è più moderno”?

La risposta deve riferirsi al **modello di minaccia** e alla protezione della comunicazione.

---

# 15. Challenge 1 — Individuare le criticità

Il docente presenta agli studenti questa configurazione:

```text
SSID: Azienda
Password: azienda123
VLAN: 1
Management: Telnet
```

## Consegna

Individuare **tutte le criticità di sicurezza**.

Per ogni criticità indicare:

1. problema;
2. rischio;
3. evidenza/configurazione che lo dimostra;
4. possibile correzione;
5. verifica successiva.

### Possibili aree da analizzare

- password debole;
- utilizzo della VLAN 1;
- assenza di una VLAN dedicata agli utenti wireless;
- management tramite Telnet;
- assenza di SSH;
- assenza di separazione tra rete utenti e rete management;
- eventuale assenza di autenticazione robusta;
- eventuale assenza di una politica di accesso amministrativo;
- esposizione non necessaria delle interfacce di management.

> **Regola:** non limitarsi a elencare problemi. Per ogni problema bisogna spiegare **perché** rappresenta un rischio.

---

# 16. Challenge 2 — Progettare la configurazione corretta

Partendo dalla rete precedente, proporre una nuova configurazione con almeno:

```text
SSID: ITS-Azienda
VLAN utenti: 10
VLAN management: 99
Sicurezza Wi-Fi: WPA2/WPA3 se disponibile
Management: SSH
Telnet: disabilitato
Password: robusta
```

Gli studenti devono produrre:

- schema logico;
- tabella VLAN;
- piano IP;
- configurazione essenziale;
- verifiche;
- motivazione delle scelte.

---

# 17. Fault Injection — Wi-Fi

Il docente introduce progressivamente un fault.

## Fault 1 — SSID errato

Modificare l'SSID configurato sull'AP rispetto a quello atteso dal client.

### Sintomo

Il client non riesce ad associarsi alla rete prevista.

### Evidenze da raccogliere

- SSID configurato;
- SSID cercato dal client;
- stato dell'associazione;
- configurazione AP.

### Domanda AI

> Qual è il test più semplice per distinguere un problema di SSID da un problema di autenticazione?

---

# 18. Fault Injection — autenticazione Wi-Fi

Cambiare intenzionalmente la password sul dispositivo wireless senza aggiornare il client.

### Obiettivo

Distinguere:

```text
rete visibile
        ↓
associazione
        ↓
autenticazione
        ↓
indirizzo IP
        ↓
connettività
```

Il fatto che il client “veda la rete” non significa che sia già connesso correttamente.

---

# 19. Fault Injection — VLAN errata

Modificare la porta dello switch collegata all'AP:

```cisco
switchport access vlan 99
```

quando dovrebbe appartenere alla VLAN 10.

### Obiettivo

Osservare la differenza tra:

- client associato al Wi-Fi;
- client che riceve un IP;
- client che raggiunge il gateway corretto.

### Verifiche

```cisco
show vlan brief
show interfaces status
```

---

# 20. Fault Injection — trunk errato

Modificare il collegamento SW1–R1 in modo che non trasporti correttamente le VLAN.

Esempio didattico:

```cisco
interface gigabitEthernet 0/1
 switchport mode access
```

### Obiettivo

Osservare l'impatto della modifica sul traffico tra VLAN e sul gateway wireless.

Verificare:

```cisco
show interfaces trunk
show vlan brief
show ip interface brief
```

---

# 21. Fault Injection — SSH non disponibile

Introdurre uno dei seguenti problemi:

- `transport input ssh` rimosso;
- `login local` rimosso;
- indirizzo SVI errato;
- gateway di management errato;
- VLAN 99 non attiva;
- username non corretto;
- chiavi RSA non presenti.

## Metodo di diagnosi

Non cambiare immediatamente la configurazione.

Prima verificare:

```cisco
show ip interface brief
show running-config | section line vty
show ip ssh
show running-config | include username
```

Poi formulare una prima ipotesi.

---

# 22. Fault Injection — Telnet ancora abilitato

Configurare intenzionalmente:

```cisco
line vty 0 4
 transport input telnet ssh
```

### Obiettivo

Individuare perché questa configurazione non rispetta il requisito di utilizzare esclusivamente SSH per la gestione remota.

Correggere con:

```cisco
line vty 0 4
 transport input ssh
```

Verificare nuovamente la configurazione.

---

# 23. Troubleshooting strutturato

Quando il problema viene introdotto, seguire sempre questo ordine:

### 1. OSSERVA

Che cosa non funziona?

### 2. RACCOGLI LE EVIDENZE

Quali informazioni sono realmente disponibili?

### 3. FORMULA IPOTESI

Quali sono le possibili cause?

### 4. TESTA

Quale test permette di distinguere le ipotesi?

### 5. INTERPRETA

Il risultato conferma o smentisce l'ipotesi?

### 6. RISOLVI

Modificare solo ciò che è necessario.

### 7. VERIFICA

Il servizio ora funziona?

### 8. DOCUMENTA

Qual era il fault? Come è stato individuato? Quale test lo ha confermato?

---

# 24. Utilizzo dell'AI nel troubleshooting

L'AI non deve ricevere una richiesta generica come:

> “Il Wi-Fi non funziona, risolvilo.”

Il prompt deve contenere:

- topologia;
- configurazione;
- sintomo;
- evidenze raccolte;
- comandi eseguiti;
- risultati;
- ipotesi già considerate;
- vincoli;
- richiesta di procedere per test successivi.

## Prompt base

```text
Sto analizzando un problema di rete Wi-Fi in Cisco Packet Tracer.

TOPOLOGIA:
PC wireless → AP → SW1 → R1

VLAN:
VLAN 10 = WIFI-USERS
VLAN 99 = MGMT

SINTOMO:
Il client wireless non riesce a raggiungere il gateway.

EVIDENZE RACCOLTE:
[incolla qui le evidenze]

COMANDI E RISULTATI:
[incolla qui i comandi e l'output]

IPOTESI ATTUALI:
[elenco delle ipotesi]

Agisci come supporto al troubleshooting.
Non inventare dati mancanti.
Se mancano informazioni, chiedimi quali evidenze raccogliere.
Non fornire subito la soluzione finale.

Per ogni ipotesi:
1. spiega perché è plausibile;
2. proponi un solo test alla volta;
3. indica il comando o la verifica da eseguire;
4. indica il risultato atteso;
5. spiega come interpretare il risultato;
6. aggiorna la probabilità dell'ipotesi;
7. solo dopo la conferma proponi la correzione;
8. concludi con una verifica finale.
```

---

# 25. Prompt AI per il challenge sicurezza

```text
Sto analizzando la sicurezza di una piccola rete aziendale.

Configurazione iniziale:
SSID: Azienda
Password: azienda123
VLAN: 1
Management: Telnet

Obiettivo:
individuare le criticità e progettare una configurazione più sicura.

Non limitarti a elencare problemi.
Per ogni criticità indica:
- configurazione osservata;
- minaccia o rischio associato;
- motivo per cui la configurazione è debole;
- correzione proposta;
- verifica necessaria dopo la correzione.

Considera almeno:
- autenticazione wireless;
- password;
- segmentazione VLAN;
- rete di management;
- accesso remoto;
- SSH/Telnet;
- superficie di esposizione.

Distingui chiaramente:
FATTI → RISCHI → IPOTESI → CORREZIONI → VERIFICHE.
Non inventare caratteristiche del dispositivo che non sono state fornite.
```

---

# 26. Scheda di troubleshooting

| Campo | Risposta |
|---|---|
| Fault osservato | |
| Sintomo | |
| Evidenza 1 | |
| Evidenza 2 | |
| Evidenza 3 | |
| Ipotesi 1 | |
| Ipotesi 2 | |
| Test scelto | |
| Risultato | |
| Ipotesi confermata/smentita | |
| Correzione | |
| Verifica finale | |
| Documentazione | |

---

# 27. Esercizio finale — Scenario aziendale

Progettare e configurare una rete per una piccola azienda con:

```text
WLAN utenti
VLAN utenti = 10
VLAN management = 99
SSID = ITS-Azienda
Sicurezza wireless = WPA2/WPA3, se disponibile
Management = SSH
Telnet = disabilitato
```

## Requisiti

Il client wireless deve:

- associarsi alla WLAN;
- ricevere un indirizzo IP;
- raggiungere il gateway;
- raggiungere la rete di management secondo quanto previsto dallo scenario;
- permettere al PC-Admin di raggiungere SW1 tramite SSH.

## Fault finale

Il docente introduce **due fault contemporaneamente**.

Gli studenti devono:

1. non modificare immediatamente la configurazione;
2. raccogliere evidenze;
3. formulare almeno due ipotesi;
4. utilizzare l'AI per progettare i test;
5. eseguire un test alla volta;
6. individuare entrambi i fault;
7. correggerli;
8. verificare nuovamente l'intera rete;
9. documentare il procedimento.

---

# 28. Checklist finale

Prima di considerare concluso il laboratorio verificare:

- [ ] VLAN 10 configurata
- [ ] VLAN 99 configurata
- [ ] porta AP configurata correttamente
- [ ] trunk configurato e verificato
- [ ] gateway VLAN configurati
- [ ] client wireless associato
- [ ] sicurezza wireless configurata
- [ ] DHCP funzionante
- [ ] IP del client verificato
- [ ] gateway raggiungibile
- [ ] management SVI funzionante
- [ ] username locale configurato
- [ ] chiavi SSH generate
- [ ] SSH version 2 configurato
- [ ] Telnet disabilitato
- [ ] accesso SSH verificato
- [ ] almeno un Fault Injection risolto
- [ ] evidenze raccolte
- [ ] prompt AI utilizzato correttamente
- [ ] troubleshooting documentato

---

# 29. Consegne dello studente

Ogni studente o gruppo deve consegnare:

1. file `.pkt` della topologia;
2. tabella VLAN e indirizzamento;
3. configurazione essenziale dei dispositivi;
4. scheda del troubleshooting;
5. prompt AI utilizzato;
6. risultato dell'analisi AI;
7. descrizione dei fault individuati;
8. verifica finale della rete.

---

# 30. Criteri di valutazione

| Competenza | Peso indicativo |
|---|---:|
| Configurazione VLAN/Wi-Fi | 20% |
| Configurazione e verifica SSH | 20% |
| Comprensione dei rischi di sicurezza | 15% |
| Troubleshooting tecnico | 20% |
| Utilizzo consapevole dell'AI | 15% |
| Documentazione e metodo | 10% |

---

# 31. Conclusione

In questo laboratorio la configurazione Wi-Fi e SSH è solo il punto di partenza.

L'obiettivo principale è imparare a ragionare sulla sicurezza e sul comportamento della rete.

Una rete correttamente configurata non è soltanto una rete che “funziona”: deve essere progettata in modo coerente, verificabile e difendibile.

Il troubleshooting deve quindi partire dalle evidenze e non dalle supposizioni.

L'AI può aiutare a formulare ipotesi, scegliere test e interpretare risultati, ma la responsabilità della verifica rimane del tecnico.
