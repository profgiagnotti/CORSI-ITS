# Lezione 7 — Laboratorio pratico: Routing, NAT/PAT, WAN e troubleshooting con AI

**Corso:** UFS 03 — Networking  
**Lezione:** 7 — Routing, NAT e WAN  
**Durata indicativa:** 140 minuti  
**Ambiente:** Cisco Packet Tracer  
**Livello:** intermedio  
**Metodo:** Configuro → Verifico → Osservo → Rompo → Formulo ipotesi → Testo → Risolvo → Verifico nuovamente → Documento

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio dovrai essere in grado di:

- costruire una WAN tra due LAN;
- configurare gli indirizzi IPv4 di router e host;
- comprendere il ruolo del default gateway;
- leggere una routing table;
- riconoscere route connected, static e default;
- configurare il routing statico;
- verificare il percorso dei pacchetti;
- configurare NAT;
- configurare PAT;
- distinguere indirizzi inside local e inside global;
- verificare le traduzioni NAT;
- introdurre fault intenzionali;
- diagnosticare problemi di IP, gateway, routing e NAT;
- utilizzare l'AI come supporto al troubleshooting;
- costruire prompt diagnostici basati sulle evidenze;
- documentare correttamente causa, test, soluzione e verifica.

> **Regola del laboratorio:** l'AI non deve inventare output o configurazioni mancanti. Prima si raccolgono le evidenze, poi si formulano le ipotesi, quindi si eseguono i test.

---

# 2. Scenario

Realizzeremo una piccola WAN composta da due LAN collegate tramite due router.

```text
 PC0
  |
 SW0
  |
 G0/0
 R1
 G0/1
  |
 WAN
  |
 G0/1
 R2
 G0/0
  |
 SW1
  |
 PC1
```

La rete sarà organizzata in tre segmenti:

- LAN A;
- collegamento WAN tra R1 e R2;
- LAN B.

---

# 3. Piano di indirizzamento

Utilizza il seguente piano.

| Dispositivo | Interfaccia | Indirizzo IPv4 | Mask |
|---|---|---|---|
| PC0 | NIC | 192.168.10.10 | 255.255.255.0 |
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 |
| R1 | G0/1 | 10.0.0.1 | 255.255.255.252 |
| R2 | G0/1 | 10.0.0.2 | 255.255.255.252 |
| R2 | G0/0 | 192.168.20.1 | 255.255.255.0 |
| PC1 | NIC | 192.168.20.10 | 255.255.255.0 |

Gateway:

```text
PC0 → 192.168.10.1
PC1 → 192.168.20.1
```

---

# 4. Costruzione della topologia

Inserisci in Cisco Packet Tracer:

- 2 router;
- 2 switch;
- 2 PC.

Collega:

```text
PC0 → SW0 → R1 → R2 → SW1 → PC1
```

Per il collegamento tra R1 e R2 utilizza le interfacce disponibili coerenti con il modello scelto.

> **Nota:** se il modello di router utilizza nomi diversi delle interfacce, adatta i comandi alla numerazione presente nel dispositivo.

---

# 5. Configurazione di PC0

Configura:

```text
IP address: 192.168.10.10
Subnet mask: 255.255.255.0
Default gateway: 192.168.10.1
```

Verifica dal prompt:

```text
ipconfig
```

---

# 6. Configurazione di PC1

Configura:

```text
IP address: 192.168.20.10
Subnet mask: 255.255.255.0
Default gateway: 192.168.20.1
```

Verifica:

```text
ipconfig
```

---

# 7. Configurazione di R1

Accedi alla CLI.

```text
enable
configure terminal
hostname R1

interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

interface gigabitEthernet 0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
exit

end
```

Verifica:

```text
show ip interface brief
```

Dovresti vedere le interfacce utilizzate nello stato:

```text
up    up
```

---

# 8. Configurazione di R2

```text
enable
configure terminal
hostname R2

interface gigabitEthernet 0/0
ip address 192.168.20.1 255.255.255.0
no shutdown
exit

interface gigabitEthernet 0/1
ip address 10.0.0.2 255.255.255.252
no shutdown
exit

end
```

Verifica:

```text
show ip interface brief
```

---

# 9. Primo test: connettività locale

Da R1:

```text
ping 192.168.10.10
```

Poi:

```text
ping 10.0.0.2
```

Da R2:

```text
ping 192.168.20.10
```

Poi:

```text
ping 10.0.0.1
```

### Domanda

Se questi ping funzionano, significa che la WAN tra le due LAN è già configurata?

**No.**

Abbiamo verificato soltanto alcuni segmenti direttamente connessi.

---

# 10. Analizzare la routing table

Su R1:

```text
show ip route
```

Su R2:

```text
show ip route
```

Identifica le reti:

- direttamente connesse;
- non ancora conosciute.

Osserva le sigle:

```text
C
L
S
```

In particolare:

- `C` = Connected;
- `L` = Local;
- `S` = Static.

---

# 11. Configurazione del routing statico

R1 deve sapere come raggiungere:

```text
192.168.20.0/24
```

Configura:

```text
configure terminal
ip route 192.168.20.0 255.255.255.0 10.0.0.2
end
```

R2 deve sapere come raggiungere:

```text
192.168.10.0/24
```

Configura:

```text
configure terminal
ip route 192.168.10.0 255.255.255.0 10.0.0.1
end
```

---

# 12. Verificare le route statiche

Su R1:

```text
show ip route
```

Su R2:

```text
show ip route
```

Cerca le route:

```text
S    192.168.20.0/24
```

e:

```text
S    192.168.10.0/24
```

---

# 13. Test end-to-end

Da PC0:

```text
ping 192.168.20.10
```

Poi:

```text
tracert 192.168.20.10
```

Da PC1:

```text
ping 192.168.10.10
```

Poi:

```text
tracert 192.168.10.10
```

### Domande

1. Quali router attraversa il traffico?
2. Perché PC0 invia il traffico a R1?
3. Perché R1 inoltra il traffico verso R2?
4. Perché R2 sa raggiungere la LAN B?
5. Perché è necessaria una route anche sul percorso di ritorno?

---

# 14. Osservare il percorso con traceroute

Utilizza:

```text
tracert 192.168.20.10
```

Confronta il risultato con:

```text
show ip route
```

### Obiettivo

Collegare il comportamento osservato dall'host alla routing table dei router.

Non memorizzare soltanto il comando: chiediti sempre:

> "Quale decisione di routing sta prendendo il router in questo momento?"

---

# 15. Introduzione al NAT

Ora introduciamo il NAT.

L'obiettivo è osservare come un indirizzo privato della LAN può essere tradotto in un indirizzo utilizzato verso l'esterno.

Nel nostro laboratorio utilizzeremo R1 come router NAT.

Per rendere l'esercizio più chiaro, considereremo:

- interfaccia LAN di R1 = `inside`;
- interfaccia verso la WAN = `outside`.

Su R1:

```text
configure terminal

interface gigabitEthernet 0/0
ip nat inside
exit

interface gigabitEthernet 0/1
ip nat outside
exit
```

---

# 16. NAT statico — esercizio guidato

Per comprendere il concetto di traduzione, possiamo associare un indirizzo interno a un indirizzo esterno.

Esempio:

```text
ip nat inside source static 192.168.10.10 10.0.0.10
```

> **Attenzione didattica:** questo esempio serve per osservare il meccanismo NAT. L'indirizzo `10.0.0.10` è un indirizzo di laboratorio e non rappresenta un vero indirizzo pubblico Internet.

Verifica:

```text
show ip nat translations
```

e:

```text
show ip nat statistics
```

---

# 17. NAT dinamico e PAT

Nel laboratorio principale utilizzeremo soprattutto PAT, perché permette a più host interni di condividere un singolo indirizzo attraverso l'uso delle porte.

Prima rimuovi, se necessario, la configurazione NAT statica precedente:

```text
configure terminal
no ip nat inside source static 192.168.10.10 10.0.0.10
end
```

Configura un ACL per identificare la rete interna:

```text
configure terminal
access-list 1 permit 192.168.10.0 0.0.0.255
```

Configura PAT utilizzando l'interfaccia WAN di R1:

```text
ip nat inside source list 1 interface gigabitEthernet 0/1 overload
end
```

Verifica:

```text
show ip nat translations
```

---

# 18. Generare traffico per osservare PAT

Da PC0 genera traffico verso una destinazione raggiungibile attraverso R1.

Per esempio:

```text
ping 10.0.0.2
```

Poi controlla:

```text
show ip nat translations
```

Puoi anche verificare:

```text
show ip nat statistics
```

### Domande

1. Quale indirizzo interno è stato tradotto?
2. Quale indirizzo viene utilizzato come indirizzo di traduzione?
3. Perché PAT è utile?
4. Cosa cambia rispetto al NAT statico?

---

# 19. Verificare il ruolo del gateway

Il default gateway permette a un host di inviare traffico destinato a una rete diversa dalla propria.

Per PC0:

```text
IP: 192.168.10.10
Mask: 255.255.255.0
Gateway: 192.168.10.1
```

Esegui:

```text
ping 192.168.10.1
```

Poi:

```text
ping 192.168.20.10
```

Confronta i risultati.

---

# 20. Fault Injection 1 — Indirizzo IP errato

Modifica temporaneamente PC0:

```text
IP address: 192.168.30.10
Mask: 255.255.255.0
Gateway: 192.168.10.1
```

Test:

```text
ping 192.168.10.1
```

Poi:

```text
ping 192.168.20.10
```

### Obiettivo

Capire perché un gateway appartenente a una rete diversa dall'host può impedire il corretto inoltro del traffico.

### Ripristino

Riporta PC0 a:

```text
192.168.10.10/24
gateway 192.168.10.1
```

---

# 21. Fault Injection 2 — Gateway errato

Configura PC0 con:

```text
IP: 192.168.10.10
Mask: 255.255.255.0
Gateway: 192.168.10.254
```

Esegui:

```text
ping 192.168.10.1
```

e:

```text
ping 192.168.20.10
```

### Domanda

Perché il ping verso il router può avere un comportamento diverso dal ping verso una rete remota?

Documenta il risultato.

---

# 22. Fault Injection 3 — Route statica errata

Su R1 modifica intenzionalmente la route:

```text
configure terminal
no ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.20.0 255.255.255.0 10.0.0.9
end
```

Verifica:

```text
show ip route
```

Poi:

```text
ping 192.168.20.10
```

e:

```text
traceroute 192.168.20.10
```

### Obiettivo

Distinguere:

- destinazione conosciuta;
- next-hop configurato;
- next-hop raggiungibile;
- percorso realmente utilizzabile.

### Ripristino

```text
configure terminal
no ip route 192.168.20.0 255.255.255.0 10.0.0.9
ip route 192.168.20.0 255.255.255.0 10.0.0.2
end
```

---

# 23. Fault Injection 4 — NAT non funzionante

Su R1 rimuovi temporaneamente la configurazione PAT:

```text
configure terminal
no ip nat inside source list 1 interface gigabitEthernet 0/1 overload
end
```

Genera traffico e osserva:

```text
show ip nat translations
```

e:

```text
show ip nat statistics
```

### Domande

- le traduzioni vengono create?
- il problema è di routing o di NAT?
- quali evidenze lo dimostrano?

### Ripristino

```text
configure terminal
ip nat inside source list 1 interface gigabitEthernet 0/1 overload
end
```

---

# 24. Fault Injection 5 — Inside/Outside invertiti

Modifica intenzionalmente il ruolo delle interfacce NAT.

Esempio:

```text
configure terminal

interface gigabitEthernet 0/0
no ip nat inside
ip nat outside
exit

interface gigabitEthernet 0/1
no ip nat outside
ip nat inside
exit

end
```

Testa la comunicazione e osserva:

```text
show ip nat translations
show ip nat statistics
```

### Obiettivo

Comprendere che NAT non è soltanto una regola di traduzione: anche la corretta identificazione delle interfacce inside/outside è fondamentale.

### Ripristino

```text
configure terminal

interface gigabitEthernet 0/0
no ip nat outside
ip nat inside
exit

interface gigabitEthernet 0/1
no ip nat inside
ip nat outside
exit

end
```

---

# 25. Fault Injection 6 — Interfaccia WAN down

Su R1:

```text
configure terminal
interface gigabitEthernet 0/1
shutdown
end
```

Test:

```text
show ip interface brief
show ip route
ping 10.0.0.2
```

Poi verifica dal PC:

```text
ping 192.168.20.10
```

### Obiettivo

Capire come un problema fisico o amministrativo su una interfaccia può produrre un sintomo apparentemente "di routing".

### Ripristino

```text
configure terminal
interface gigabitEthernet 0/1
no shutdown
end
```

---

# 26. Metodologia di troubleshooting

Per ogni fault applica:

```text
OSSERVA
   ↓
RACCOGLI LE EVIDENZE
   ↓
FORMULA IPOTESI
   ↓
SCEGLI IL TEST PIÙ UTILE
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

### Regola

Non saltare direttamente a:

> "Devo modificare la route."

Prima chiediti:

> "Quale evidenza dimostra che il problema è nella routing table?"

---

# 27. Comandi diagnostici

## Interfacce

```text
show ip interface brief
```

## Routing table

```text
show ip route
```

## Configurazione

```text
show running-config
```

## NAT

```text
show ip nat translations
show ip nat statistics
```

## ACL

```text
show access-lists
```

## Ping

```text
ping <indirizzo>
```

## Traceroute

```text
traceroute <indirizzo>
```

Su PC:

```text
tracert <indirizzo>
```

---

# 28. Troubleshooting con AI

L'AI deve aiutarti a ragionare sulla base delle evidenze.

Non incollare semplicemente:

> "Non funziona, cosa faccio?"

Fornisci invece:

- topologia;
- indirizzamento;
- sintomo;
- output;
- configurazione rilevante;
- obiettivo;
- metodologia diagnostica.

## Prompt base

```text
Sto lavorando in Cisco Packet Tracer su una WAN composta da
due LAN collegate tramite due router.

Devo diagnosticare un problema di connettività.

Topologia:
[descrizione]

Piano di indirizzamento:
[tabella]

Sintomo:
[descrizione precisa]

Output disponibili:

show ip interface brief
[output]

show ip route
[output]

show ip nat translations
[output]

show ip nat statistics
[output]

show access-lists
[output]

ping:
[risultati]

traceroute/tracert:
[risultati]

Procedi con questa metodologia:

OSSERVA
FORMULA IPOTESI
TESTA
INTERPRETA
CONFERMA O SMENTISCI
RISOLVI
VERIFICA
DOCUMENTA

Non inventare dati mancanti.
Non fornire immediatamente la soluzione.
Non modificare la configurazione senza spiegare quale
ipotesi stiamo verificando.

Per ogni ipotesi indica:
- evidenze a favore;
- evidenze contro;
- test;
- comando;
- risultato atteso;
- interpretazione.

Procedi un test alla volta.
```

---

# 29. Prompt avanzato per Fault Injection

```text
Sto svolgendo un'attività di troubleshooting in Cisco Packet Tracer.

La topologia contiene:
- due LAN;
- due router;
- un collegamento WAN;
- routing statico;
- NAT/PAT.

È stato introdotto intenzionalmente uno o più fault.

NON devi indovinare subito il problema.

Devi identificare la causa utilizzando esclusivamente
le evidenze che ti fornirò.

Considera almeno queste categorie:

1. indirizzamento IP;
2. subnet mask;
3. default gateway;
4. stato delle interfacce;
5. collegamento WAN;
6. routing table;
7. route statiche;
8. next-hop;
9. NAT inside/outside;
10. ACL utilizzate dal NAT;
11. traduzioni NAT;
12. PAT.

METODO OBBLIGATORIO:

OSSERVA
↓
ELENCA I FATTI
↓
FORMULA MASSIMO 3 IPOTESI
↓
ORDINA LE IPOTESI
↓
PROPONI UN SOLO TEST
↓
INDICA IL COMANDO
↓
INDICA IL RISULTATO ATTESO
↓
INTERPRETA IL RISULTATO
↓
CONFERMA O SMENTISCI
↓
PROPONI LA CORREZIONE
↓
VERIFICA
↓
DOCUMENTA

Non inventare output.
Se manca un'informazione, chiedimela.
Non assumere automaticamente che il problema sia NAT.
Non assumere automaticamente che il problema sia routing.

Voglio distinguere chiaramente:
FATTO → IPOTESI → TEST → RISULTATO → CONCLUSIONE.
```

---

# 30. Attività AI — Diagnosi a livelli

Scegli un Fault Injection e chiedi all'AI di aiutarti a diagnosticare il problema procedendo dal livello più semplice al più complesso.

Sequenza suggerita:

```text
1. Host
2. IP
3. Gateway
4. Interfaccia router
5. WAN
6. Routing table
7. Next-hop
8. NAT
9. ACL
10. Verifica end-to-end
```

### Obiettivo

Evitare di cercare subito problemi sofisticati quando potrebbe esserci un semplice errore di configurazione IP.

---

# 31. Scheda di troubleshooting

Compila per ogni Fault Injection:

| Campo | Risposta |
|---|---|
| Fault introdotto | |
| Sintomo | |
| Fatti osservati | |
| IPotesi 1 | |
| Ipotesi 2 | |
| Ipotesi 3 | |
| Test scelto | |
| Comando | |
| Risultato | |
| Interpretazione | |
| Ipotesi confermata | |
| Causa reale | |
| Correzione | |
| Verifica finale | |
| Output conclusivo | |

---

# 32. Challenge finale

Costruisci nuovamente:

```text
LAN A → R1 → WAN → R2 → LAN B
```

Configura:

- indirizzamento IPv4;
- gateway;
- routing statico;
- NAT/PAT;
- verifica end-to-end.

Poi introduci **due fault contemporaneamente**.

Esempi:

- gateway errato + route errata;
- interfaccia WAN down + NAT non configurato;
- ACL NAT errata + route corretta;
- inside/outside errati + problema di routing.

### Regole

Non comunicare all'AI quali fault hai introdotto.

Raccogli prima:

```text
show ip interface brief
show ip route
show running-config
show ip nat translations
show ip nat statistics
show access-lists
```

Poi prepara il prompt diagnostico.

L'AI dovrà aiutarti a:

1. distinguere i sintomi;
2. separare fatti e ipotesi;
3. scegliere i test;
4. interpretare i risultati;
5. individuare le cause;
6. proporre le correzioni;
7. verificare la rete dopo la correzione.

---

# 33. Verifica finale

- [ ] Le due LAN sono configurate.
- [ ] La WAN tra R1 e R2 funziona.
- [ ] Gli host hanno IP e gateway corretti.
- [ ] Le interfacce router sono `up/up`.
- [ ] La routing table è stata analizzata.
- [ ] Sono presenti route connected.
- [ ] Sono state configurate route statiche.
- [ ] La comunicazione end-to-end funziona.
- [ ] È stato osservato il percorso con traceroute/tracert.
- [ ] NAT è stato configurato.
- [ ] PAT è stato configurato.
- [ ] Sono state osservate le traduzioni NAT.
- [ ] È stato eseguito almeno un Fault Injection IP/gateway.
- [ ] È stato eseguito almeno un Fault Injection di routing.
- [ ] È stato eseguito almeno un Fault Injection NAT.
- [ ] È stata applicata la metodologia strutturata.
- [ ] È stato utilizzato un prompt AI diagnostico.
- [ ] Le ipotesi sono state verificate con evidenze.
- [ ] La soluzione è stata verificata dopo la correzione.
- [ ] Il troubleshooting è stato documentato.

---

# 34. Consegna

Consegna:

1. file `.pkt`;
2. piano di indirizzamento;
3. routing table prima e dopo il routing statico;
4. configurazione NAT/PAT;
5. output `show ip nat translations`;
6. output `show ip nat statistics`;
7. Fault Injection realizzati;
8. schede di troubleshooting;
9. prompt AI utilizzati;
10. risultati delle verifiche;
11. breve relazione tecnica finale.

---

# 35. Criteri di valutazione

| Competenza | Evidenza |
|---|---|
| Indirizzamento | configurazione corretta |
| WAN | collegamento R1-R2 funzionante |
| Routing | route statiche corrette |
| Gateway | scelta e verifica corretta |
| NAT/PAT | traduzioni funzionanti |
| Diagnostica | uso appropriato dei comandi |
| Fault Injection | capacità di isolare le cause |
| AI | prompt strutturato e uso critico |
| Verifica | conferma della soluzione |
| Documentazione | relazione tecnica completa |

---

# 36. Conclusione

In questo laboratorio il routing viene affrontato come un processo decisionale.

Un router non "manda semplicemente avanti" un pacchetto: consulta la propria routing table, individua la destinazione e decide quale percorso utilizzare.

NAT e PAT aggiungono un ulteriore livello di elaborazione, perché gli indirizzi possono essere tradotti prima che il traffico attraversi un determinato confine della rete.

Durante il troubleshooting bisogna quindi evitare diagnosi premature.

La domanda non deve essere:

> "Quale comando devo usare per sistemarlo?"

ma:

> "Quale evidenza mi permette di capire dove si trova il problema?"

Il percorso di lavoro rimane:

```text
OSSERVA
   ↓
RACCOGLI EVIDENZE
   ↓
FORMULA IPOTESI
   ↓
TESTA
   ↓
INTERPRETA
   ↓
CONFERMA / SMENTISCI
   ↓
RISOLVI
   ↓
VERIFICA
   ↓
DOCUMENTA
```

L'AI diventa particolarmente utile quando viene utilizzata per confrontare ipotesi, suggerire test mirati e aiutare a interpretare gli output, mantenendo però sempre il controllo tecnico dell'attività da parte dello studente.
