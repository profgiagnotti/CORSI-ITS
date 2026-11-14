# UFS 04 — Basi di programmazione in Java e strumenti per lo sviluppo

## Lezione 01 — Laboratorio: primi passi con Java

**Durata:** circa 2 ore e 30 minuti  
**Lezione teorica di riferimento:** Lezione 01 — Fondamenti Java e ambiente di sviluppo  
**Tecnologie:** JDK 17/21, IntelliJ IDEA, Java  
**Livello:** introduttivo

---

## 1. Obiettivo del laboratorio

In questo laboratorio trasformiamo i concetti della prima lezione in codice Java.

L'obiettivo non è semplicemente "far funzionare un programma", ma iniziare a costruire una metodologia di lavoro professionale:

1. verificare l'ambiente di sviluppo;
2. creare un progetto Java;
3. compilare ed eseguire un programma;
4. dichiarare e utilizzare variabili;
5. lavorare con tipi primitivi e `String`;
6. utilizzare `var` nei casi appropriati;
7. utilizzare operatori ed espressioni;
8. costruire decisioni con `if`, `else` e `switch`;
9. utilizzare i cicli `for` e `while`;
10. leggere gli errori del compilatore;
11. usare il debugger in modo elementare;
12. realizzare un piccolo programma completo.

> **Metodo di lavoro:** non limitarti a copiare il codice. Dopo ogni esercizio prova a modificare almeno una cosa e osserva cosa succede.

---

## 2. Prerequisiti

Assicurati di avere:

- JDK 17 oppure JDK 21;
- IntelliJ IDEA;
- un terminale;
- una cartella locale dedicata ai progetti del corso;
- Git installato, se previsto dalla configurazione del corso.

Non è necessario conoscere Java in precedenza.

---

## 3. Struttura consigliata del repository

Una possibile organizzazione è:

```text
UFS04-java/
├── lezione-01/
│   ├── esercizio-01-hello-java/
│   ├── esercizio-02-variabili/
│   ├── esercizio-03-operatori/
│   ├── esercizio-04-condizioni/
│   ├── esercizio-05-switch/
│   ├── esercizio-06-cicli/
│   └── esercizio-07-caso-reale/
└── README.md
```

La struttura può essere adattata alle modalità organizzative del corso.

---

## 4. Esercizio 1 — Verificare il JDK

Apri il terminale ed esegui:

```bash
java -version
```

Poi:

```bash
javac -version
```

### Perché due comandi?

`java` verifica la disponibilità del comando utilizzato per eseguire applicazioni Java.

`javac` verifica invece la disponibilità del compilatore Java.

Questa distinzione è importante: un ambiente di sviluppo deve permettere non solo di eseguire applicazioni, ma anche di compilarle.

### Primo troubleshooting

Se compare un errore come:

```text
'java' is not recognized...
```

oppure:

```text
java: command not found
```

non reinstallare immediatamente tutto.

Chiediti:

- Java è realmente installato?
- il JDK è configurato?
- il comando è presente nel `PATH`?
- IntelliJ sta utilizzando un JDK diverso da quello del terminale?

> **Tip del docente**
>
> Un errore di ambiente non è un errore del programma.
>
> Prima di correggere il codice, chiediti sempre: **il problema è nel codice oppure nell'ambiente?**

---

## 5. Esercizio 2 — Creare il primo progetto IntelliJ

Apri IntelliJ IDEA e crea un nuovo progetto Java.

Imposta:

- **Language:** Java
- **JDK:** 17 oppure 21
- **Nome progetto:** `lezione-01`

Se IntelliJ richiede di scegliere un JDK e non lo trova, configura il JDK installato nel sistema.

---

## 6. Esercizio 3 — Il primo programma Java

Crea una classe chiamata:

```text
Main
```

Inserisci:

```java
public class Main {

    public static void main(String[] args) {
        System.out.println("Ciao, Java!");
    }
}
```

Esegui il programma.

Output atteso:

```text
Ciao, Java!
```

### Analizziamo il programma

```java
public class Main {
```

dichiara una classe chiamata `Main`.

```java
public static void main(String[] args) {
```

è il punto di ingresso della semplice applicazione.

```java
System.out.println("Ciao, Java!");
```

stampa una stringa nella console.

---

## 7. Esercizio 4 — Modificare il programma

Modifica il programma affinché visualizzi:

```text
Benvenuti al corso UFS 04
Sto imparando Java
Questo è il mio primo programma
```

Una possibile soluzione:

```java
public class Main {

    public static void main(String[] args) {
        System.out.println("Benvenuti al corso UFS 04");
        System.out.println("Sto imparando Java");
        System.out.println("Questo è il mio primo programma");
    }
}
```

### Variante

Prova anche:

```java
System.out.println(
        "Benvenuti al corso UFS 04\n" +
        "Sto imparando Java\n" +
        "Questo è il mio primo programma"
);
```

Confronta le due soluzioni.

---

## 8. Esercizio 5 — Variabili

Esempio:

```java
int age = 25;
```

Abbiamo:

- `int` → tipo;
- `age` → nome della variabile;
- `=` → assegnazione;
- `25` → valore.

Prova:

```java
public class Main {

    public static void main(String[] args) {

        int age = 25;

        System.out.println(age);
    }
}
```

---

## 9. Esercizio 6 — Diversi tipi di dato

Crea un programma che utilizzi:

```java
int age = 25;
double price = 19.90;
boolean available = true;
char category = 'A';
String productName = "Notebook";
```

Poi stampa tutti i valori.

Una possibile soluzione:

```java
public class Main {

    public static void main(String[] args) {

        int age = 25;
        double price = 19.90;
        boolean available = true;
        char category = 'A';
        String productName = "Notebook";

        System.out.println(age);
        System.out.println(price);
        System.out.println(available);
        System.out.println(category);
        System.out.println(productName);
    }
}
```

### Attenzione a `char` e `String`

Queste dichiarazioni sono diverse:

```java
char letter = 'A';
```

e:

```java
String text = "A";
```

`char` rappresenta un singolo carattere, mentre `String` rappresenta una sequenza di caratteri.

---

## 10. Esercizio 7 — `var`

Java permette di utilizzare anche `var`:

```java
var age = 25;
var name = "Mario";
var price = 19.90;
```

Il compilatore deduce il tipo a partire dal valore assegnato.

Prova:

```java
public class Main {

    public static void main(String[] args) {

        var name = "Computer";
        var quantity = 5;
        var price = 12.50;

        System.out.println(name);
        System.out.println(quantity);
        System.out.println(price);
    }
}
```

> **Tip del docente**
>
> `var` non significa che Java diventa un linguaggio senza tipi.
> Il tipo continua a esistere. `var` evita semplicemente di scriverlo esplicitamente in alcuni casi.

---

## 11. Esercizio 8 — Operazioni matematiche

```java
public class Main {

    public static void main(String[] args) {

        int a = 10;
        int b = 3;

        System.out.println(a + b);
        System.out.println(a - b);
        System.out.println(a * b);
        System.out.println(a / b);
        System.out.println(a % b);
    }
}
```

Osserva in particolare:

```java
a / b
```

e:

```java
a % b
```

Il simbolo `%` rappresenta il resto della divisione intera.

---

## 12. Esercizio 9 — Calcolare il prezzo totale

Immaginiamo un piccolo negozio.

```java
double price = 19.90;
int quantity = 3;
```

Vogliamo calcolare il totale:

```java
public class Main {

    public static void main(String[] args) {

        double price = 19.90;
        int quantity = 3;

        double total = price * quantity;

        System.out.println("Totale: " + total);
    }
}
```

Questa istruzione:

```java
"Totale: " + total
```

combina testo e valore.

---

## 13. Esercizio 10 — Condizioni con `if`

Supponiamo di avere:

```java
double total = 120.00;
```

Se il totale supera 100 euro, applichiamo uno sconto:

```java
public class Main {

    public static void main(String[] args) {

        double total = 120.00;

        if (total > 100) {
            System.out.println("Sconto applicato");
        }
    }
}
```

### `if` e `else`

```java
public class Main {

    public static void main(String[] args) {

        double total = 80.00;

        if (total > 100) {
            System.out.println("Sconto applicato");
        } else {
            System.out.println("Nessuno sconto");
        }
    }
}
```

Prova a cambiare il valore di `total` e osserva il comportamento.

---

## 14. Esercizio 11 — Più condizioni

Regole:

- meno di 50 → `Economico`
- da 50 a meno di 100 → `Standard`
- 100 o più → `Premium`

Una possibile soluzione:

```java
double price = 120.00;

if (price < 50) {
    System.out.println("Economico");
} else if (price < 100) {
    System.out.println("Standard");
} else {
    System.out.println("Premium");
}
```

### Domanda

Perché nel secondo caso non è necessario scrivere anche:

```java
price >= 50
```

Ragiona sull'ordine con cui vengono valutate le condizioni.

---

## 15. Esercizio 12 — `switch`

Esempio:

```java
int day = 2;

switch (day) {
    case 1:
        System.out.println("Lunedì");
        break;

    case 2:
        System.out.println("Martedì");
        break;

    case 3:
        System.out.println("Mercoledì");
        break;

    default:
        System.out.println("Valore non valido");
}
```

### Esercizio

Crea uno `switch` che gestisca:

```text
1 → Bassa
2 → Media
3 → Alta
4 → Critica
```

Per valori diversi stampa:

```text
Priorità non valida
```

---

## 16. Esercizio 13 — Ciclo `for`

```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

Output:

```text
1
2
3
4
5
```

Il ciclo contiene:

```text
inizializzazione
condizione
aggiornamento
```

Nel nostro caso:

```java
int i = 1
```

inizializza il contatore.

```java
i <= 5
```

determina quando continuare.

```java
i++
```

incrementa il contatore.

---

## 17. Esercizio 14 — Tabellina

Utilizzando un ciclo `for`, stampa la tabellina del 7.

Output atteso:

```text
7
14
21
28
35
42
49
56
63
70
```

Una possibile soluzione:

```java
for (int i = 1; i <= 10; i++) {
    System.out.println(7 * i);
}
```

---

## 18. Esercizio 15 — Ciclo `while`

```java
int counter = 1;

while (counter <= 5) {
    System.out.println(counter);
    counter++;
}
```

Confronta questo codice con il `for`.

Il concetto fondamentale è lo stesso: ripetere un blocco finché una condizione rimane vera.

### Errore intenzionale: ciclo infinito

Prova:

```java
int counter = 1;

while (counter <= 5) {
    System.out.println(counter);
}
```

Il problema è che `counter` non viene mai modificato.

La condizione rimane sempre vera.

Correggi aggiungendo:

```java
counter++;
```

> **Tip del docente**
>
> Un programma può compilare perfettamente e avere comunque un errore.
>
> **Errore di sintassi, errore di compilazione ed errore logico non sono la stessa cosa.**

---

## 19. Esercizio 16 — Errori intenzionali

### Errore 1 — tipo incompatibile

Prova:

```java
int age = "25";
```

Perché Java non lo accetta?

### Errore 2 — variabile non definita

Prova:

```java
System.out.println(name);
```

senza dichiarare `name`.

### Errore 3 — parentesi mancante

Rimuovi una `}` dal programma e osserva come cambia l'errore.

### Errore 4 — errore logico

```java
int age = 20;

if (age < 18) {
    System.out.println("Maggiorenne");
}
```

Il programma compila, ma il comportamento è logicamente errato.

La condizione corretta è:

```java
if (age >= 18) {
    System.out.println("Maggiorenne");
}
```

---

## 20. Metodo di troubleshooting

Quando compare un errore, non modificare casualmente il codice.

Segui questa sequenza:

1. **Leggi l'errore.**
2. **Individua la riga.**
3. **Controlla il contesto.**
4. **Formula un'ipotesi.**
5. **Modifica una cosa alla volta.**
6. **Ricompila ed esegui.**
7. **Verifica il risultato.**

Esempio di ipotesi:

> "Probabilmente sto assegnando una `String` a una variabile `int`."

> **Frase del docente**
>
> "Non voglio che impariate a indovinare la soluzione. Voglio che impariate a capire perché il programma non funziona."

---

## 21. Esercizio 17 — Primo utilizzo del debugger

Usa:

```java
double price = 19.90;
int quantity = 3;

double total = price * quantity;

System.out.println("Totale: " + total);
```

Inserisci un breakpoint sulla riga:

```java
double total = price * quantity;
```

Avvia il programma in modalità **Debug**.

Osserva:

- `price`;
- `quantity`;
- `total`.

Procedi passo-passo e osserva quando `total` viene valorizzata.

### Obiettivo

Iniziare a usare il debugger come strumento di analisi, non soltanto come strumento per fermare il programma.

---

## 22. Esercizio 18 — Caso reale: gestione di un ordine

### Scenario

Un'azienda vende un prodotto.

Abbiamo:

- nome prodotto;
- prezzo unitario;
- quantità;
- soglia per ottenere lo sconto.

Regola:

- se il totale supera 100 €, viene applicato uno sconto del 10%;
- altrimenti non viene applicato alcuno sconto;
- al termine dobbiamo stampare il totale finale.

### Implementazione

```java
public class Main {

    public static void main(String[] args) {

        String productName = "Notebook";
        double unitPrice = 750.00;
        int quantity = 2;

        double total = unitPrice * quantity;

        double discount = 0;

        if (total > 100) {
            discount = total * 0.10;
        }

        double finalTotal = total - discount;

        System.out.println("Prodotto: " + productName);
        System.out.println("Prezzo unitario: " + unitPrice);
        System.out.println("Quantità: " + quantity);
        System.out.println("Totale: " + total);
        System.out.println("Sconto: " + discount);
        System.out.println("Totale finale: " + finalTotal);
    }
}
```

### Analisi

```java
String productName
```

contiene il nome del prodotto.

```java
double unitPrice
```

contiene il prezzo.

```java
int quantity
```

contiene la quantità.

Poi:

```java
double total = unitPrice * quantity;
```

calcola il totale.

Successivamente:

```java
if (total > 100) {
    discount = total * 0.10;
}
```

determina lo sconto.

Infine:

```java
double finalTotal = total - discount;
```

calcola il risultato finale.

---

## 23. Challenge — Modificare il caso reale

Aggiungi questa regola:

- se la quantità è almeno 10, applica un ulteriore 5% di sconto.

Prima di scrivere codice chiediti:

1. Mi serve una nuova variabile?
2. Devo utilizzare un altro `if`?
3. Lo sconto deve essere calcolato sul totale originale?
4. Oppure sul totale già scontato?
5. Quale comportamento voglio ottenere?

Non cercare immediatamente una soluzione online: prima formula una possibile strategia.

---

## 24. Challenge avanzata — Fasce di quantità

Modifica il programma utilizzando:

| Quantità | Sconto |
|---:|---:|
| 1–4 | 0% |
| 5–9 | 5% |
| 10–19 | 10% |
| 20 o più | 15% |

Il programma deve:

1. calcolare il totale;
2. determinare la percentuale di sconto;
3. calcolare l'importo dello sconto;
4. calcolare il totale finale;
5. stampare un riepilogo leggibile.

La fase di correzione fa parte dell'esercizio.

---

## 25. Uso dell'AI come assistente tecnico

L'AI può essere utilizzata come supporto, ma non deve sostituire il ragionamento.

Prompt poco utile:

```text
Il mio programma non funziona. Sistemalo.
```

Prompt migliore:

```text
Sto sviluppando un programma Java che calcola il totale di un ordine
e applica uno sconto in base alla quantità.

Mi aspetto:
- 1-4 pezzi: 0%
- 5-9 pezzi: 5%
- 10-19 pezzi: 10%
- almeno 20 pezzi: 15%

Con quantità 10 ottengo un risultato che non mi aspettavo.

Questo è il codice:

[incolla qui il codice]

Non riscrivere subito tutto il programma.
Prima:
1. individua il problema;
2. spiega perché si verifica;
3. indica quale condizione viene valutata;
4. proponi una correzione minima;
5. spiegami come verificare che la correzione sia corretta.
```

> **Tip del docente**
>
> Una risposta dell'AI non è automaticamente corretta.
>
> **La verifica finale spetta sempre allo sviluppatore.**

---

## 26. Verifica finale

### Ambiente

- Che differenza c'è tra `java` e `javac`?
- Che cos'è il JDK?
- Che ruolo ha IntelliJ?

### Java

- Che cos'è una classe?
- A cosa serve `main`?
- Che cos'è una variabile?
- Qual è la differenza tra `int`, `double`, `boolean`, `char` e `String`?
- Che cosa significa usare `var`?

### Logica

- Come funziona `if`?
- Quando può essere utile `switch`?
- Qual è la differenza tra `for` e `while`?
- Come si può creare accidentalmente un ciclo infinito?

### Debugging

- Che differenza c'è tra errore di compilazione ed errore logico?
- Che cos'è un breakpoint?
- Perché non è una buona strategia modificare casualmente il codice?

---

## 27. Consegna

Al termine del laboratorio il progetto deve contenere almeno:

```text
Main.java
```

con un programma funzionante relativo al caso reale dell'ordine.

È consigliabile mantenere anche gli esercizi precedenti in classi separate oppure in progetti/cartelle separate, secondo l'organizzazione scelta dal docente.

---

## 28. Checklist dello studente

- [ ] JDK installato e funzionante
- [ ] `java -version` verificato
- [ ] `javac -version` verificato
- [ ] progetto IntelliJ creato
- [ ] primo programma eseguito
- [ ] variabili utilizzate
- [ ] tipi primitivi utilizzati
- [ ] `String` utilizzata
- [ ] `var` provato
- [ ] operatori utilizzati
- [ ] `if/else` utilizzato
- [ ] `switch` utilizzato
- [ ] `for` utilizzato
- [ ] `while` utilizzato
- [ ] almeno un errore di compilazione analizzato
- [ ] almeno un errore logico analizzato
- [ ] breakpoint utilizzato
- [ ] caso reale dell'ordine completato
- [ ] challenge sulla quantità completata

---

## 29. Risultato atteso

Al termine della prima attività pratica lo studente non deve ancora essere in grado di sviluppare una grande applicazione.

Deve però aver acquisito un primo modello mentale del lavoro con Java:

```text
Problema
   ↓
Dati
   ↓
Variabili
   ↓
Elaborazione
   ↓
Decisioni
   ↓
Ripetizioni
   ↓
Risultato
   ↓
Verifica
   ↓
Debug
```

Questo schema sarà ripreso e ampliato nelle lezioni successive.

---

## 30. Collegamento con la prossima lezione

Nella prossima lezione inizieremo a superare la logica dei semplici programmi procedurali.

Entreremo nel cuore della programmazione a oggetti:

- classi;
- oggetti;
- attributi;
- metodi;
- costruttori;
- `this`;
- incapsulamento.

L'obiettivo sarà iniziare a modellare oggetti e concetti del mondo reale attraverso il codice Java.

---

## Nota per il docente

Durante questa prima attività è importante non trasformare il laboratorio in una semplice sequenza di esercizi da copiare.

L'obiettivo didattico principale è costruire una prima abitudine professionale:

> **scrivere → eseguire → osservare → formulare un'ipotesi → correggere → verificare**

Questa metodologia verrà utilizzata durante tutto il corso, anche negli argomenti più avanzati.
