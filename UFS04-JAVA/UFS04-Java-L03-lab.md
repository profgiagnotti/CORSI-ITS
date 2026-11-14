# UFS 04 — Lezione 03 — Laboratorio
## Classi, oggetti e incapsulamento

**Durata:** 2h 30 min  
**Tecnologie:** Java 17/21, IntelliJ IDEA  
**Prerequisiti:** variabili, tipi primitivi, `String`, operatori, condizioni, cicli, metodi, parametri, valori di ritorno, array  
**Repository:** GitHub  
**Obiettivo:** passare dalla gestione di dati e funzioni alla modellazione di entità reali attraverso classi, oggetti, stato, comportamento e incapsulamento.

---

## 1. Obiettivi del laboratorio

Al termine del laboratorio lo studente deve essere in grado di:

- creare una classe Java;
- definire attributi e metodi;
- distinguere **classe** e **oggetto**;
- creare oggetti con `new`;
- utilizzare costruttori;
- comprendere e utilizzare `this`;
- distinguere membri di istanza e membri `static`;
- utilizzare `final` per valori che non devono cambiare;
- applicare l'incapsulamento tramite attributi `private`;
- creare getter e setter quando appropriato;
- inserire regole di validazione all'interno della classe;
- utilizzare oggetti all'interno di un programma;
- ragionare sul modello del problema prima di scrivere il codice;
- utilizzare il debugger di IntelliJ per osservare stato e comportamento degli oggetti;
- utilizzare un assistente AI come supporto al debugging, verificando criticamente le risposte.

---

# 2. Scenario professionale

Immagina di lavorare come sviluppatore junior presso un'azienda che gestisce una piccola piattaforma per il monitoraggio di una linea produttiva.

Nelle prime esercitazioni hai lavorato principalmente con:

- variabili;
- array;
- metodi;
- condizioni;
- cicli.

Ora il programma deve gestire entità più complesse.

Per esempio, una macchina industriale potrebbe avere:

- un codice identificativo;
- un nome;
- una temperatura;
- uno stato operativo;
- una soglia di allarme.

E potrebbe essere in grado di:

- avviarsi;
- fermarsi;
- aggiornare la temperatura;
- verificare una condizione di allarme;
- restituire informazioni sul proprio stato.

Gestire tutti questi dati con variabili separate diventerebbe rapidamente difficile.

L'obiettivo del laboratorio è quindi imparare a **modellare una macchina come oggetto software**.

> **Tip del docente**
>
> Non pensare subito al codice.
>
> Prima chiediti:
>
> **"Qual è l'entità reale che sto cercando di rappresentare?"**
>
> Poi individua:
>
> 1. quali dati deve conoscere;
> 2. quali operazioni deve saper eseguire;
> 3. quali regole devono essere rispettate.
>
> Questo ragionamento è alla base della programmazione a oggetti.

---

# 3. Esercizio 1 — Creare la prima classe

## Obiettivo

Creare una classe `Macchina` con alcuni attributi e un metodo.

Crea un progetto Java denominato:

```text
UFS04-L03-Lab
```

All'interno del progetto crea:

```text
src/
└── Macchina.java
```

Inserisci una prima versione della classe:

```java
public class Macchina {

    String codice;
    String nome;
    double temperatura;

    void stampaStato() {
        System.out.println("Macchina: " + nome);
        System.out.println("Codice: " + codice);
        System.out.println("Temperatura: " + temperatura + " °C");
    }
}
```

Poi crea una classe `Main`:

```java
public class Main {

    public static void main(String[] args) {

        Macchina macchina = new Macchina();

        macchina.codice = "M001";
        macchina.nome = "Pressa A";
        macchina.temperatura = 72.5;

        macchina.stampaStato();
    }
}
```

### Domande

Prima di procedere, rispondi nei commenti del codice:

1. Che cosa rappresenta `Macchina`?
2. Che cosa rappresenta `macchina`?
3. Qual è la differenza tra `Macchina` e `macchina`?
4. Che cosa fa `new Macchina()`?
5. Quali dati rappresentano lo stato dell'oggetto?

---

# 4. Esercizio 2 — Attributi e stato

Modifica la classe aggiungendo:

```text
stato
numeroCicli
```

La macchina dovrà quindi possedere almeno:

```java
String codice;
String nome;
double temperatura;
String stato;
int numeroCicli;
```

Nel `main` crea due macchine:

```text
M001 → Pressa A
M002 → Robot B
```

Assegna valori differenti alle due macchine.

Poi stampa lo stato di entrambe.

### Obiettivo

Comprendere che due oggetti della stessa classe possono avere **stati differenti**.

Per esempio:

```text
Pressa A → temperatura 72.5 → ATTIVA
Robot B  → temperatura 35.2 → FERMO
```

La classe descrive la struttura.

Gli oggetti contengono valori concreti.

---

# 5. Esercizio 3 — Costruttori

La versione precedente presenta un problema.

Per creare una macchina bisogna scrivere:

```java
Macchina macchina = new Macchina();

macchina.codice = "M001";
macchina.nome = "Pressa A";
macchina.temperatura = 72.5;
```

È possibile inizializzare l'oggetto direttamente attraverso un **costruttore**.

Modifica la classe:

```java
public class Macchina {

    String codice;
    String nome;
    double temperatura;

    Macchina(String codice, String nome, double temperatura) {
        this.codice = codice;
        this.nome = nome;
        this.temperatura = temperatura;
    }

    void stampaStato() {
        System.out.println("Macchina: " + nome);
        System.out.println("Codice: " + codice);
        System.out.println("Temperatura: " + temperatura + " °C");
    }
}
```

Ora nel `main` puoi scrivere:

```java
Macchina macchina = new Macchina(
        "M001",
        "Pressa A",
        72.5
);
```

### Verifica

Crea tre macchine:

```text
M001 → Pressa A → 72.5 °C
M002 → Robot B → 35.2 °C
M003 → Forno C → 145.0 °C
```

Stampa lo stato di tutte.

---

# 6. Esercizio 4 — Comprendere `this`

Osserva:

```java
Macchina(String codice, String nome, double temperatura) {
    this.codice = codice;
    this.nome = nome;
    this.temperatura = temperatura;
}
```

Nel costruttore abbiamo parametri con lo stesso nome degli attributi.

La parola chiave:

```java
this
```

indica l'oggetto corrente.

Quindi:

```java
this.codice = codice;
```

significa:

> assegna all'attributo `codice` dell'oggetto corrente il valore ricevuto nel parametro `codice`.

### Esercizio

Inserisci un commento nel codice che spieghi il significato di:

```java
this.codice
```

e di:

```java
codice
```

---

# 7. Esercizio 5 — Aggiungere comportamento

Una classe non deve contenere solamente dati.

Aggiungi i seguenti metodi:

```java
void avvia() {
    stato = "ATTIVA";
}

void ferma() {
    stato = "FERMA";
}

void incrementaCicli() {
    numeroCicli++;
}
```

Completa la classe con:

```java
int numeroCicli;
String stato;
```

Nel `main` esegui:

```java
Macchina macchina = new Macchina(
        "M001",
        "Pressa A",
        72.5
);

macchina.avvia();
macchina.incrementaCicli();
macchina.incrementaCicli();

macchina.stampaStato();
```

### Domande

1. Qual è lo stato iniziale della macchina?
2. Quale metodo modifica lo stato?
3. Quale metodo modifica il numero di cicli?
4. Chi è responsabile della modifica dello stato: `Main` o `Macchina`?

> **Tip del docente**
>
> Un buon oggetto non dovrebbe essere solamente un contenitore di dati.
>
> Dovrebbe avere anche le operazioni che hanno senso per l'entità che rappresenta.

---

# 8. Esercizio 6 — Incapsulamento

La versione precedente permette di scrivere direttamente:

```java
macchina.temperatura = -500;
```

oppure:

```java
macchina.numeroCicli = -100;
```

Dal punto di vista del modello non sono valori sensati.

Rendi quindi gli attributi `private`:

```java
private String codice;
private String nome;
private double temperatura;
private String stato;
private int numeroCicli;
```

A questo punto il codice esterno non può più accedere direttamente agli attributi.

---

# 9. Esercizio 7 — Getter

Aggiungi i getter necessari:

```java
public String getCodice() {
    return codice;
}

public String getNome() {
    return nome;
}

public double getTemperatura() {
    return temperatura;
}

public String getStato() {
    return stato;
}

public int getNumeroCicli() {
    return numeroCicli;
}
```

Nel `main` prova:

```java
System.out.println(macchina.getNome());
System.out.println(macchina.getTemperatura());
System.out.println(macchina.getStato());
```

### Domanda

Perché è preferibile utilizzare:

```java
macchina.getTemperatura()
```

invece di rendere direttamente pubblico:

```java
macchina.temperatura
```

---

# 10. Esercizio 8 — Setter e validazione

Aggiungi un metodo per modificare la temperatura:

```java
public void setTemperatura(double temperatura) {
    this.temperatura = temperatura;
}
```

Poi migliora il metodo introducendo una regola.

Una macchina industriale non può avere una temperatura inferiore a `-50 °C` o superiore a `200 °C`.

Scrivi:

```java
public void setTemperatura(double temperatura) {

    if (temperatura < -50 || temperatura > 200) {
        System.out.println("Temperatura non valida.");
        return;
    }

    this.temperatura = temperatura;
}
```

### Test

Prova:

```java
macchina.setTemperatura(80);
macchina.setTemperatura(250);
macchina.setTemperatura(-100);
```

Verifica che solamente il valore valido venga accettato.

> **Tip del docente**
>
> Questo è uno dei motivi principali per cui l'incapsulamento è importante.
>
> Non si tratta semplicemente di nascondere una variabile.
>
> Significa poter controllare **come lo stato interno dell'oggetto viene modificato**.

---

# 11. Esercizio 9 — Regole di business

Migliora ulteriormente il metodo `avvia()`.

Una macchina non deve potersi avviare se la temperatura è superiore a `100 °C`.

Scrivi una versione simile:

```java
public void avvia() {

    if (temperatura > 100) {
        System.out.println(
                "Avvio non consentito: temperatura troppo alta."
        );
        return;
    }

    stato = "ATTIVA";
}
```

### Test

Prova:

```java
Macchina macchina = new Macchina(
        "M001",
        "Pressa A",
        80
);

macchina.avvia();
```

Poi:

```java
macchina.setTemperatura(150);
macchina.ferma();
macchina.avvia();
```

### Obiettivo

La classe ora non rappresenta solamente dati.

Contiene anche una **regola di funzionamento**.

---

# 12. Esercizio 10 — `static`

Aggiungi alla classe:

```java
private static int numeroMacchine = 0;
```

Nel costruttore:

```java
numeroMacchine++;
```

Poi crea:

```java
public static int getNumeroMacchine() {
    return numeroMacchine;
}
```

Nel `main`:

```java
System.out.println(
        "Macchine create: " + Macchina.getNumeroMacchine()
);
```

### Domanda fondamentale

Perché si può chiamare:

```java
Macchina.getNumeroMacchine()
```

senza scrivere:

```java
macchina.getNumeroMacchine()
```

?

Spiega la differenza tra:

```text
attributo di istanza
```

e:

```text
attributo statico
```

---

# 13. Esercizio 11 — `final`

Supponiamo che il nome dell'azienda sia una costante comune a tutte le macchine.

Aggiungi:

```java
private static final String AZIENDA = "Industrial Automation S.p.A.";
```

Poi crea:

```java
public static String getAzienda() {
    return AZIENDA;
}
```

Verifica:

```java
System.out.println(Macchina.getAzienda());
```

### Domanda

Perché utilizziamo:

```java
static final
```

?

Spiega separatamente il significato di:

```text
static
```

e:

```text
final
```

---

# 14. Esercizio 12 — Caso professionale: gestione di un ordine

Ora passa a un secondo scenario.

Un sistema aziendale deve gestire gli ordini.

Un ordine contiene:

```text
numeroOrdine
cliente
importo
stato
```

Uno stato possibile può essere:

```text
CREATO
CONFERMATO
SPEDITO
ANNULLATO
```

Crea:

```java
public class Ordine {

    private String numeroOrdine;
    private String cliente;
    private double importo;
    private String stato;

}
```

Aggiungi un costruttore.

Poi aggiungi:

```java
conferma()
spedisci()
annulla()
```

### Regole

Implementa queste regole:

1. un ordine appena creato è `CREATO`;
2. un ordine può essere confermato solamente se l'importo è maggiore di zero;
3. un ordine `CREATO` può diventare `CONFERMATO`;
4. un ordine `CONFERMATO` può diventare `SPEDITO`;
5. un ordine `SPEDITO` non può essere annullato;
6. un ordine `ANNULLATO` non può essere spedito.

Non modificare direttamente `stato` dal `main`.

La logica deve essere contenuta nella classe `Ordine`.

---

# 15. Esercizio 13 — Testare il modello

Nel `main` crea:

```java
Ordine ordine = new Ordine(
        "ORD-001",
        "Cliente A",
        1250.00
);
```

Poi prova:

```java
ordine.conferma();
ordine.spedisci();
ordine.annulla();
```

Dopo ogni operazione stampa lo stato.

### Obiettivo

Verificare che l'oggetto impedisca automaticamente le operazioni non valide.

### Domande

- Cosa succede se provo a spedire un ordine appena creato?
- Cosa succede se provo ad annullare un ordine già spedito?
- Dove vengono implementate le regole?
- Perché è preferibile non mettere queste regole direttamente nel `main`?

---

# 16. Esercizio 14 — Debugger di IntelliJ

Utilizza il debugger di IntelliJ IDEA.

Inserisci un breakpoint nel metodo:

```java
public void avvia()
```

Esegui il programma in modalità **Debug**.

Osserva:

- gli attributi dell'oggetto;
- il valore di `temperatura`;
- il valore di `stato`;
- il valore di `numeroCicli`;
- il parametro ricevuto dai metodi;
- il valore di `this`.

### Attività

Durante il debug rispondi:

```text
Qual è lo stato dell'oggetto prima della chiamata?
Quale metodo sta modificando lo stato?
Quale attributo cambia?
Qual è il nuovo valore dopo l'esecuzione?
```

> **Tip del docente**
>
> Quando fai debugging di un programma a oggetti, non limitarti a guardare la riga che genera il problema.
>
> Osserva soprattutto **lo stato dell'oggetto prima e dopo l'operazione**.

---

# 17. Esercizio 15 — Fault injection

Ora introduci intenzionalmente alcuni errori.

## Fault A — temperatura non validata

Commenta temporaneamente il controllo nel setter:

```java
public void setTemperatura(double temperatura) {
    this.temperatura = temperatura;
}
```

Prova a impostare:

```java
250
```

Osserva il comportamento.

Poi ripristina la validazione.

---

## Fault B — stato modificabile direttamente

Rimuovi temporaneamente `private`:

```java
String stato;
```

Nel `main` scrivi:

```java
macchina.stato = "SPENTA";
```

Chiediti:

> Il programma può impedire che venga assegnato uno stato non previsto?

Ripristina quindi:

```java
private String stato;
```

e gestisci le modifiche attraverso metodi appropriati.

---

## Fault C — contatore errato

Modifica intenzionalmente:

```java
numeroCicli++;
```

in:

```java
numeroCicli--;
```

Esegui il programma e utilizza il debugger per individuare il problema.

---

# 18. Esercizio 16 — Troubleshooting con AI

L'obiettivo non è chiedere all'AI:

> "Correggi il mio codice."

L'obiettivo è fornire informazioni sufficienti affinché l'AI possa aiutarti nel ragionamento.

Utilizza un prompt simile:

```text
Sto sviluppando un programma Java 17 che modella una macchina industriale
attraverso una classe Macchina.

La classe utilizza attributi private e metodi per modificare lo stato.

Il problema è questo:

quando imposto una temperatura superiore a 200 °C,
l'oggetto dovrebbe rifiutare il valore, ma il valore viene comunque salvato.

Questo è il metodo coinvolto:

[incolla qui il metodo]

Questo è il codice che utilizzo per testarlo:

[incolla qui il codice]

Non limitarti a correggere il codice.
1. individua la causa;
2. spiega perché si verifica;
3. proponi una correzione;
4. indica come posso verificare la correzione;
5. segnala eventuali problemi di progettazione.
```

### Attività

Dopo aver ricevuto la risposta:

1. non accettarla automaticamente;
2. individua l'ipotesi formulata dall'AI;
3. verifica il comportamento nel debugger;
4. applica la modifica;
5. esegui nuovamente il test;
6. documenta il risultato.

### Consegna

Scrivi nel README del laboratorio:

```text
Problema:
...

Ipotesi dell'AI:
...

Verifica effettuata:
...

Soluzione applicata:
...

Risultato:
...
```

> **Tip del docente**
>
> In un contesto professionale l'AI può essere un ottimo supporto, ma non sostituisce la verifica tecnica.
>
> Una risposta plausibile non significa necessariamente una risposta corretta.

---

# 19. Esercizio finale — Mini sistema di monitoraggio

Realizza un piccolo programma per la gestione di una linea produttiva.

## Classe `Macchina`

La classe deve contenere almeno:

```text
codice
nome
temperatura
stato
numeroCicli
```

Gli attributi devono essere `private`.

Deve essere presente un costruttore.

Deve essere presente almeno un metodo getter.

Deve essere presente un metodo per aggiornare la temperatura con validazione.

Deve essere presente un metodo per avviare la macchina.

Deve essere presente un metodo per fermarla.

Deve essere presente un metodo per incrementare il numero di cicli.

Deve essere presente un metodo per stampare lo stato.

---

## Classe `Main`

Nel `main` crea almeno tre macchine:

```text
M001 → Pressa A
M002 → Robot B
M003 → Forno C
```

Simula una sequenza operativa:

1. creazione delle macchine;
2. avvio;
3. aumento della temperatura;
4. esecuzione di alcuni cicli;
5. controllo dello stato;
6. tentativo di impostare una temperatura non valida;
7. arresto di una macchina;
8. stampa dello stato finale.

---

# 20. Challenge opzionale

Aggiungi alla classe `Macchina`:

```java
public boolean necessitaManutenzione()
```

La macchina deve richiedere manutenzione quando:

```text
numeroCicli >= 1000
```

oppure quando:

```text
temperatura > 90 °C
```

Il metodo deve restituire:

```text
true
```

oppure:

```text
false
```

Nel `main` stampa:

```text
M001 → manutenzione richiesta
M002 → manutenzione non richiesta
...
```

---

# 21. Challenge avanzata — Separare responsabilità

Crea una seconda classe:

```java
GestoreProduzione
```

che possa lavorare con oggetti `Macchina`.

Ad esempio:

```java
public class GestoreProduzione {

    public void stampaMacchine(Macchina[] macchine) {
        // ...
    }
}
```

Nel `main` crea un array:

```java
Macchina[] macchine = {
        macchina1,
        macchina2,
        macchina3
};
```

e utilizza il gestore.

### Obiettivo

Iniziare a comprendere che, nei programmi reali, non tutto deve essere concentrato nella classe `Main`.

---

# 22. Errori comuni da verificare

Prima della consegna controlla di non aver commesso questi errori:

- usare attributi pubblici senza motivo;
- dimenticare `private`;
- confondere classe e oggetto;
- dimenticare `new`;
- confondere parametro e attributo;
- non utilizzare `this` quando serve;
- creare getter/setter senza comprendere il loro scopo;
- mettere tutta la logica nel `main`;
- permettere modifiche dello stato senza validazione;
- utilizzare `static` per dati che appartengono alla singola istanza;
- modificare direttamente lo stato dell'oggetto dal codice esterno;
- non testare i casi non validi;
- accettare automaticamente una risposta dell'AI senza verificarla.

---

# 23. Consegna GitHub

La repository deve avere una struttura simile:

```text
UFS04-L03-Lab/
├── README.md
└── src/
    ├── Main.java
    ├── Macchina.java
    ├── Ordine.java
    └── GestoreProduzione.java
```

Nel `README.md` inserisci:

```markdown
# UFS 04 - Lezione 03 - Classi, oggetti e incapsulamento

## Studente

Nome:
Cognome:

## Obiettivi

- classi
- oggetti
- costruttori
- this
- incapsulamento
- getter/setter
- static
- final
- validazione
- debugging

## Problema professionale

Descrizione dello scenario affrontato.

## Soluzione

Descrizione delle classi realizzate.

## Test

Descrizione dei test eseguiti.

## Fault injection

Descrizione degli errori introdotti intenzionalmente e individuati.

## Utilizzo dell'AI

Descrivere:
- problema sottoposto all'AI;
- informazioni fornite;
- risposta ricevuta;
- verifica effettuata;
- modifica applicata.

## Conclusioni

Che cosa ho imparato sulla programmazione a oggetti?
```

---

# 24. Checklist finale

Prima della consegna verifica:

- [ ] Ho creato almeno una classe Java.
- [ ] Ho creato più oggetti della stessa classe.
- [ ] Ho utilizzato `new`.
- [ ] Ho utilizzato un costruttore.
- [ ] Ho compreso `this`.
- [ ] Ho utilizzato attributi `private`.
- [ ] Ho creato getter.
- [ ] Ho utilizzato almeno una validazione.
- [ ] Ho distinto membri di istanza e `static`.
- [ ] Ho utilizzato `final`.
- [ ] Ho inserito comportamento nelle classi.
- [ ] Ho evitato di concentrare tutta la logica nel `main`.
- [ ] Ho utilizzato il debugger.
- [ ] Ho introdotto almeno un errore intenzionale.
- [ ] Ho utilizzato l'AI per una fase di troubleshooting.
- [ ] Ho verificato personalmente la risposta dell'AI.
- [ ] Il progetto compila.
- [ ] Il progetto è stato eseguito e testato.
- [ ] Il repository contiene un `README.md` chiaro.

---

# 25. Competenze osservabili

Durante il laboratorio saranno osservate soprattutto queste competenze:

| Competenza | Evidenza |
|---|---|
| Modellazione | Individua classi e responsabilità |
| Java | Scrive classi e oggetti correttamente |
| OOP | Comprende stato e comportamento |
| Incapsulamento | Protegge lo stato interno |
| Validazione | Gestisce input non validi |
| Debugging | Utilizza debugger e osserva lo stato |
| Problem solving | Formula ipotesi e le verifica |
| AI literacy | Usa l'AI come supporto e ne verifica l'output |
| Qualità del codice | Mantiene una struttura leggibile e coerente |
| Professionalità | Documenta lavoro e verifiche |

---

## Risultato atteso

Al termine del laboratorio non è sufficiente avere un programma che "funziona".

Lo studente deve essere in grado di spiegare:

> **perché il programma è stato progettato in quel modo, quali responsabilità appartengono a ciascuna classe e come viene protetto lo stato degli oggetti.**

Questo passaggio è fondamentale per evolvere dalla semplice scrittura di codice alla progettazione di software.
