# Lezione 4 — Laboratorio pratico: IPv6, configurazione IP, dual-stack e troubleshooting con AI

**Corso:** UFS 03 — Networking  
**Lezione:** 4  
**Durata laboratorio:** circa 140 minuti  
**Ambiente:** Cisco Packet Tracer

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio sarai in grado di:

- configurare indirizzi IPv6 su router e host;
- distinguere Global Unicast, Link-Local, Multicast e Loopback;
- utilizzare correttamente i prefissi `/64`;
- verificare la connettività IPv6;
- osservare le informazioni NDP;
- analizzare la tabella di routing IPv6;
- realizzare uno scenario dual-stack IPv4/IPv6;
- confrontare il comportamento di IPv4 e IPv6;
- applicare una metodologia strutturata di troubleshooting;
- introdurre e analizzare fault intenzionali (**Fault Injection**);
- utilizzare l'AI come assistente per formulare ipotesi, scegliere test e interpretare risultati;
- costruire prompt tecnici completi e verificabili.

> **Metodo di lavoro:**  
> **Progetto → Configuro → Verifico → Rompo → Osservo → Formulo ipotesi → Testo → Risolvo → Verifico nuovamente → Documento**

---

# 2. Preparazione dell'ambiente

Apri **Cisco Packet Tracer** e crea un nuovo progetto.

Salva il file come:

```text
Lezione4_IPv6_DualStack.pkt
```

Salva frequentemente il progetto, soprattutto prima e dopo ogni Fault Injection.

---

# 3. Scenario 1 — Prima configurazione IPv6

Realizza:

```text
PC0 -------- SW0 -------- R0
```

### Piano di indirizzamento

Rete:

```text
2001:db8:10::/64
```

| Dispositivo | Interfaccia | IPv6 |
|---|---|---|
| R0 | G0/0 | `2001:db8:10::1/64` |
| PC0 | FastEthernet | `2001:db8:10::10/64` |

---

## 3.1 Configurazione del router

```text
enable
configure terminal
ipv6 unicast-routing
interface gigabitEthernet 0/0
ipv6 address 2001:db8:10::1/64
no shutdown
exit
```

Verifica:

```text
show ipv6 interface brief
```

Controlla che `G0/0` sia attiva e che l'indirizzo IPv6 sia presente.

---

## 3.2 Configurazione del PC

In:

**PC0 → Desktop → IP Configuration**

imposta:

```text
IPv6 Address: 2001:db8:10::10
Prefix Length: 64
IPv6 Gateway: 2001:db8:10::1
```

---

# 4. Prima verifica

Dal PC0:

```text
ping 2001:db8:10::1
```

Poi:

```text
ping ::1
```

### Domande

1. Che cosa rappresenta `::1`?
2. Il ping verso `::1` richiede la rete fisica?
3. Il ping verso `2001:db8:10::1` richiede invece la comunicazione attraverso la rete?

---

# 5. Verifica delle informazioni IPv6

Su R0:

```text
show ipv6 interface
```

Osserva:

- indirizzo Global Unicast;
- indirizzo Link-Local;
- stato dell'interfaccia;
- prefisso;
- informazioni relative a Neighbor Discovery.

Poi:

```text
show ipv6 neighbors
```

### Domanda

Dopo il ping dal PC0, che cosa cambia nella tabella dei vicini IPv6?

---

# 6. Esercizio — Riconoscere i tipi di indirizzo IPv6

Completa:

| Indirizzo | Tipo |
|---|---|
| `2001:db8:10::10` | ? |
| `fe80::1` | ? |
| `ff02::1` | ? |
| `::1` | ? |

Categorie:

- Global Unicast;
- Link-Local;
- Multicast;
- Loopback.

Spiega con una frase il ruolo di ciascuna categoria.

---

# 7. Esercizio — Abbreviazione IPv6

Riduci i seguenti indirizzi.

### 1

```text
2001:0db8:0000:0000:0000:0000:0000:0010
```

Risultato:

```text
2001:db8::10
```

### 2

```text
2001:0db8:0001:0000:0000:0000:0042:0010
```

### 3

```text
fe80:0000:0000:0000:021a:2bff:fe3c:4d5e
```

### 4

```text
2001:0db8:0000:0001:0000:0000:0000:0005
```

---

# 8. Esercizio — Espansione IPv6

Espandi fino alla forma completa:

```text
2001:db8::10
```

```text
2001:db8:1::42:10
```

```text
fe80::1
```

> `::` può rappresentare una sequenza di uno o più gruppi consecutivi composti da zeri, ma può comparire una sola volta nello stesso indirizzo.

---

# 9. Scenario 2 — Due LAN IPv6 attraverso un router

Realizza:

```text
PC0 ---- SW0 ---- R0 ---- SW1 ---- PC1
```

## LAN 1

```text
2001:db8:10::/64
```

| Dispositivo | IPv6 |
|---|---|
| R0 G0/0 | `2001:db8:10::1/64` |
| PC0 | `2001:db8:10::10/64` |

## LAN 2

```text
2001:db8:20::/64
```

| Dispositivo | IPv6 |
|---|---|
| R0 G0/1 | `2001:db8:20::1/64` |
| PC1 | `2001:db8:20::10/64` |

---

# 10. Configurazione del router

```text
enable
configure terminal
ipv6 unicast-routing

interface gigabitEthernet 0/0
ipv6 address 2001:db8:10::1/64
no shutdown
exit

interface gigabitEthernet 0/1
ipv6 address 2001:db8:20::1/64
no shutdown
exit
```

Verifica:

```text
show ipv6 interface brief
```

---

# 11. Configurazione degli host

## PC0

```text
IPv6 Address: 2001:db8:10::10
Prefix Length: 64
Gateway: 2001:db8:10::1
```

## PC1

```text
IPv6 Address: 2001:db8:20::10
Prefix Length: 64
Gateway: 2001:db8:20::1
```

---

# 12. Verifica progressiva

Dal PC0:

```text
ping ::1
```

```text
ping 2001:db8:10::1
```

```text
ping 2001:db8:20::1
```

```text
ping 2001:db8:20::10
```

Ripeti dal PC1.

Compila:

| Test | Esito | Interpretazione |
|---|---|---|
| PC0 → `::1` | | |
| PC0 → gateway LAN1 | | |
| PC0 → interfaccia LAN2 | | |
| PC0 → PC1 | | |
| PC1 → `::1` | | |
| PC1 → gateway LAN2 | | |
| PC1 → interfaccia LAN1 | | |
| PC1 → PC0 | | |

### Domanda

Se PC0 raggiunge `2001:db8:10::1` ma non `2001:db8:20::10`, quali livelli della comunicazione sono già stati parzialmente verificati?

---

# 13. Analisi della tabella di routing IPv6

Su R0:

```text
show ipv6 route
```

Individua:

```text
2001:db8:10::/64
```

e:

```text
2001:db8:20::/64
```

### Domande

1. Come sono indicate le reti direttamente connesse?
2. Perché il router conosce entrambe le LAN?
3. È necessaria una rotta statica per queste due reti?

---

# 14. Scenario 3 — Dual-stack IPv4/IPv6

Realizza:

```text
PC0 ---- SW0 ---- R0 ---- SW1 ---- PC1
```

## Piano IPv4

LAN 1:

```text
192.168.10.0/24
```

R0:

```text
192.168.10.1/24
```

PC0:

```text
192.168.10.10/24
```

LAN 2:

```text
192.168.20.0/24
```

R0:

```text
192.168.20.1/24
```

PC1:

```text
192.168.20.10/24
```

## Piano IPv6

LAN 1:

```text
2001:db8:10::/64
```

LAN 2:

```text
2001:db8:20::/64
```

---

# 15. Configurazione dual-stack del router

```text
enable
configure terminal

ipv6 unicast-routing

interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
ipv6 address 2001:db8:10::1/64
no shutdown
exit

interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
ipv6 address 2001:db8:20::1/64
no shutdown
exit
```

Verifica IPv4:

```text
show ip interface brief
```

Verifica IPv6:

```text
show ipv6 interface brief
```

---

# 16. Configurazione dual-stack dei PC

## PC0

IPv4:

```text
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

IPv6:

```text
IPv6 Address: 2001:db8:10::10
Prefix Length: 64
IPv6 Gateway: 2001:db8:10::1
```

## PC1

IPv4:

```text
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
```

IPv6:

```text
IPv6 Address: 2001:db8:20::10
Prefix Length: 64
IPv6 Gateway: 2001:db8:20::1
```

---

# 17. Confronto IPv4 / IPv6

Dal PC0:

```text
ping 192.168.20.10
```

Poi:

```text
ping 2001:db8:20::10
```

Compila:

| Protocollo | Destinazione | Esito | Note |
|---|---|---|---|
| IPv4 | `192.168.20.10` | | |
| IPv6 | `2001:db8:20::10` | | |

### Domande

1. Quale configurazione viene utilizzata dal traffico IPv4?
2. Quale configurazione viene utilizzata dal traffico IPv6?
3. Un problema IPv4 implica necessariamente un problema IPv6?
4. Se IPv4 non funziona ma IPv6 funziona, cosa puoi concludere?

---

# 18. Fault Injection 1 — Prefisso IPv6 errato

Salva il progetto.

Modifica temporaneamente PC0:

```text
IPv6 Address: 2001:db8:20::10
Prefix Length: 64
```

PC0 si trova però nella LAN:

```text
2001:db8:10::/64
```

Prova:

```text
ping 2001:db8:10::1
```

e:

```text
ping 2001:db8:20::10
```

## Troubleshooting

Non correggere subito.

Segui:

1. **Osserva** il sintomo.
2. **Raccogli** le evidenze.
3. **Formula** un'ipotesi.
4. **Scegli** un test.
5. **Esegui** il test.
6. **Interpreta** il risultato.
7. **Conferma o smentisci** l'ipotesi.
8. **Correggi**.
9. **Verifica** nuovamente.
10. **Documenta**.

Ripristina:

```text
2001:db8:10::10/64
```

---

# 19. Fault Injection 2 — Interfaccia disabilitata

Su R0:

```text
configure terminal
interface gigabitEthernet 0/0
shutdown
end
```

Verifica:

```text
show ipv6 interface brief
```

Dal PC0:

```text
ping 2001:db8:10::1
```

```text
ping 2001:db8:20::10
```

### Domanda

Quali evidenze distinguono questo problema da un errore di indirizzamento?

## Ripristino

```text
configure terminal
interface gigabitEthernet 0/0
no shutdown
end
```

---

# 20. Fault Injection 3 — Routing IPv6

Verifica il ruolo di:

```text
ipv6 unicast-routing
```

Se il modello IOS di Packet Tracer lo consente, disabilitalo temporaneamente e osserva il comportamento.

Verifica:

```text
show ipv6 interface brief
```

```text
show ipv6 route
```

Ripeti i test di connettività.

> **Nota:** il comportamento può dipendere dal modello di router e dalla versione IOS simulata da Packet Tracer. Se il comando non è supportato nel modo previsto, documenta il comportamento osservato senza forzare l'esercizio.

Ripristina:

```text
configure terminal
ipv6 unicast-routing
end
```

---

# 21. Fault Injection 4 — Problema IPv4 in una rete dual-stack

Modifica temporaneamente PC0:

```text
IPv4 Address: 192.168.50.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

Mantieni invariata la configurazione IPv6.

Testa:

```text
ping 192.168.20.10
```

e:

```text
ping 2001:db8:20::10
```

### Obiettivo

Confronta i risultati e identifica quale stack presenta il problema.

Non scrivere soltanto:

> La rete non funziona.

Formula invece una conclusione tecnica specifica.

---

# 22. Fault Injection 5 — Osservazione NDP

Ripristina la configurazione corretta.

Su R0:

```text
show ipv6 neighbors
```

Dal PC0:

```text
ping 2001:db8:10::1
```

Poi:

```text
show ipv6 neighbors
```

Osserva:

- indirizzo IPv6;
- indirizzo Link-Local;
- interfaccia;
- stato dell'entry.

### Domande

1. Quale dispositivo compare?
2. Quale indirizzo IPv6 è associato?
3. Quale Link-Local viene utilizzato?
4. Perché questa informazione è utile nel troubleshooting?

---

# 23. Scheda di troubleshooting

Compila per ogni Fault Injection:

| Campo | Risposta |
|---|---|
| Fault osservato | |
| Sintomo | |
| Evidenze raccolte | |
| Ipotesi 1 | |
| Ipotesi 2 | |
| Test scelto | |
| Risultato atteso | |
| Risultato osservato | |
| Ipotesi confermata/smentita | |
| Correzione | |
| Verifica finale | |
| Causa del fault | |

---

# 24. Troubleshooting con AI

L'AI deve essere utilizzata come **assistente al ragionamento**, non come sostituto della diagnosi.

Può aiutarti a:

- organizzare le evidenze;
- formulare ipotesi;
- individuare dati mancanti;
- scegliere test diagnostici;
- interpretare output Cisco;
- confrontare ipotesi;
- documentare la causa.

Prima raccogli i dati, poi usa l'AI.

Evita richieste generiche come:

```text
IPv6 non funziona, cosa devo fare?
```

---

# 25. Prompt base per troubleshooting IPv6

```text
Agisci come assistente esperto nel troubleshooting di reti IPv6 e dual-stack.

Il tuo compito non è fornirmi immediatamente la soluzione, ma aiutarmi a seguire una metodologia diagnostica strutturata.

Regole:
1. Non inventare informazioni che non ti ho fornito.
2. Se mancano dati importanti, chiedimi quali evidenze raccogliere.
3. Distingui sempre tra fatti osservati, ipotesi e conclusioni.
4. Non modificare direttamente la configurazione.
5. Non fornire subito la soluzione definitiva.
6. Proponi un test alla volta.
7. Per ogni test indica:
   - comando o azione;
   - risultato atteso;
   - interpretazione;
   - cosa confermerebbe;
   - cosa smentirebbe.
8. Spiega perché il test è utile.
9. Dopo ogni risultato rivaluta le ipotesi.
10. Considera il ping fallito come un sintomo e non come una diagnosi.
11. Al termine identifica:
   - causa probabile;
   - evidenze;
   - correzione;
   - verifica finale;
   - documentazione.

TOPOLOGIA:
[descrivi la topologia]

PIANO IPv6:
[riporta prefissi e indirizzi]

PIANO IPv4:
[riporta gli indirizzi se presenti]

CONFIGURAZIONE OSSERVATA:
[riporta la configurazione]

OUTPUT RACCOLTI:

show ipv6 interface brief
[incolla output]

show ipv6 interface
[incolla output]

show ipv6 neighbors
[incolla output]

show ipv6 route
[incolla output]

SINTOMO:
[descrivi esattamente cosa non funziona]

TEST GIÀ ESEGUITI:
[elenca test e risultati]

Aiutami a costruire una sequenza di troubleshooting basata esclusivamente sulle evidenze disponibili.
```

---

# 26. Attività AI 1 — Tre ipotesi

Per uno dei Fault Injection chiedi:

```text
Per questo problema proponi tre ipotesi indipendenti.

Per ogni ipotesi indica:
- evidenze che la supportano;
- evidenze che la indeboliscono;
- test da eseguire;
- risultato che la confermerebbe;
- risultato che la smentirebbe.

Non fornire ancora la soluzione.
```

Obiettivo: evitare di fissarsi sulla prima spiegazione plausibile.

---

# 27. Attività AI 2 — Scegliere il test migliore

```text
Tra i test proposti, scegli quello con il miglior rapporto tra:
- informazione ottenuta;
- semplicità;
- rischio di modifica della configurazione.

Spiega perché lo eseguiresti prima degli altri.

Non modificare la configurazione.
```

Confronta la risposta dell'AI con la tua scelta.

---

# 28. Attività AI 3 — Progettare un Fault Injection

```text
Sto realizzando un laboratorio didattico di troubleshooting IPv6.

Voglio introdurre un solo fault alla volta.

La metodologia deve essere:

OSSERVA
→ FORMULA IPOTESI
→ TESTA
→ INTERPRETA
→ CONFERMA O SMENTISCI
→ RISOLVI
→ VERIFICA
→ DOCUMENTA

Proponimi un fault coerente con una rete IPv6 dual-stack.

Il fault deve essere:
- riproducibile;
- sicuro;
- reversibile;
- diagnosticabile con comandi standard Cisco;
- realistico.

Non indicarmi immediatamente la soluzione.

Prepara:
1. modifica da introdurre;
2. sintomo atteso;
3. evidenze da raccogliere;
4. possibili ipotesi;
5. test progressivi;
6. criteri di conferma o smentita;
7. correzione;
8. verifica finale;
9. documentazione.
```

---

# 29. Attività AI 4 — Documentazione finale

```text
Analizza la seguente scheda di troubleshooting.

Non modificare i fatti tecnici.

Trasformala in una documentazione professionale composta da:

1. Sintomo
2. Contesto
3. Evidenze raccolte
4. Ipotesi iniziali
5. Test eseguiti
6. Risultati
7. Ipotesi confermata
8. Causa del problema
9. Correzione applicata
10. Verifica finale
11. Lezione appresa
12. Possibile prevenzione futura

Se una conclusione non è supportata dalle evidenze, segnalalo esplicitamente.
```

---

# 30. Challenge finale

Ricostruisci autonomamente:

```text
PC0 ---- SW0 ---- R0 ---- SW1 ---- PC1
```

Configura:

- IPv4 sulle due LAN;
- IPv6 sulle due LAN;
- routing IPv4;
- routing IPv6;
- verifica della connettività;
- tabelle IPv4 e IPv6.

Poi:

1. introduci un solo fault;
2. salva il progetto;
3. scambialo con un compagno;
4. il compagno diagnostica il problema;
5. può utilizzare l'AI fornendo le evidenze raccolte;
6. confrontate ipotesi, test e risultati;
7. invertite i ruoli.

---

# 31. Checklist finale

## IPv6

- [ ] Ho configurato un indirizzo IPv6.
- [ ] So interpretare un prefisso `/64`.
- [ ] So distinguere Global Unicast e Link-Local.
- [ ] So riconoscere Multicast e Loopback.
- [ ] So abbreviare un indirizzo IPv6.
- [ ] So espandere un indirizzo IPv6.
- [ ] So verificare un'interfaccia IPv6.
- [ ] So leggere `show ipv6 neighbors`.
- [ ] So leggere `show ipv6 route`.

## Dual-stack

- [ ] Ho configurato IPv4 e IPv6 contemporaneamente.
- [ ] Ho verificato IPv4.
- [ ] Ho verificato IPv6.
- [ ] Ho confrontato i due protocolli.
- [ ] Ho analizzato un fault IPv4.
- [ ] Ho verificato il comportamento indipendente dei due stack.

## Troubleshooting

- [ ] Ho introdotto almeno un Fault Injection.
- [ ] Ho raccolto evidenze prima di modificare la configurazione.
- [ ] Ho formulato almeno due ipotesi.
- [ ] Ho scelto test mirati.
- [ ] Ho confermato o smentito le ipotesi.
- [ ] Ho applicato una correzione.
- [ ] Ho verificato nuovamente il funzionamento.

## AI

- [ ] Ho utilizzato un prompt strutturato.
- [ ] Ho fornito dati reali.
- [ ] Ho distinto fatti, ipotesi e conclusioni.
- [ ] Ho chiesto test progressivi.
- [ ] Ho verificato personalmente i suggerimenti dell'AI.
- [ ] Ho documentato il Fault Injection.

---

# 32. Deliverable

Devi consegnare:

### File Packet Tracer

```text
Lezione4_IPv6_DualStack.pkt
```

### Scheda di troubleshooting

Almeno un Fault Injection completamente analizzato.

### Prompt AI

Almeno un prompt completo utilizzato durante l'indagine.

### Documentazione finale

Deve contenere:

- sintomo;
- evidenze;
- ipotesi;
- test;
- risultati;
- causa;
- correzione;
- verifica finale.

### Tabella IPv4 / IPv6

Completa e commentata.

---

# 33. Criteri di valutazione

| Criterio | Valutazione |
|---|---|
| Configurazione IPv6 | Corretta / Parziale / Da rivedere |
| Verifica connettività | Corretta / Parziale / Da rivedere |
| Comprensione prefissi | Corretta / Parziale / Da rivedere |
| Uso comandi di verifica | Corretta / Parziale / Da rivedere |
| Troubleshooting metodologico | Corretta / Parziale / Da rivedere |
| Qualità delle ipotesi | Corretta / Parziale / Da rivedere |
| Qualità dei test | Corretta / Parziale / Da rivedere |
| Uso dell'AI | Critico / Assistito / Superficiale |
| Documentazione | Completa / Parziale / Insufficiente |

> Una buona soluzione non è soltanto "ho corretto l'indirizzo".  
> Una buona soluzione dimostra **come hai capito che l'indirizzo era la causa del problema**.

---

# 34. Conclusione

In questa esercitazione hai lavorato su IPv6 dalla configurazione di base fino allo scenario dual-stack.

Il punto centrale rimane il metodo:

```text
OSSERVARE
↓
RACCOGLIERE EVIDENZE
↓
FORMULARE IPOTESI
↓
SCEGLIERE UN TEST
↓
TESTARE
↓
INTERPRETARE
↓
CONFERMARE O SMENTIRE
↓
CORREGGERE
↓
VERIFICARE
↓
DOCUMENTARE
```

L'AI può supportare il ragionamento: organizzare le evidenze, confrontare ipotesi e progettare test diagnostici.

La responsabilità della verifica rimane però tua.

**Non devi chiedere all'AI di sostituire il troubleshooting: devi imparare a usarla per fare troubleshooting in modo più strutturato.**
