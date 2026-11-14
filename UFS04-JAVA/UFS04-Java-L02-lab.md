# UFS 04 — Basi di programmazione in Java e strumenti per lo sviluppo
## Laboratorio — Lezione 02
### Metodi, parametri, valori di ritorno, scope e array

> **Durata indicativa:** 2 ore e 30 minuti  
> **Ambiente:** JDK 17/21 + IntelliJ IDEA  
> **Obiettivo:** imparare a scomporre un problema in metodi riutilizzabili e utilizzare gli array per gestire insiemi di dati.

---

## 1. Obiettivi del laboratorio

Al termine dell'attività dovresti essere in grado di:

- creare un metodo Java;
- distinguere tra parametro e argomento;
- utilizzare `void` e `return`;
- passare dati a un metodo;
- utilizzare il valore restituito da un metodo;
- comprendere lo scope delle variabili;
- creare e inizializzare array;
- accedere agli elementi tramite indice;
- utilizzare `length`;
- attraversare un array con `for`;
- utilizzare l'enhanced `for`;
- calcolare somma, media, minimo e massimo;
- effettuare una ricerca all'interno di un array;
- contare elementi che rispettano una condizione;
- suddividere un problema in più metodi;
- riconoscere e correggere alcuni errori tipici;
- effettuare un primo refactoring di codice monolitico.

---

# 2. Prima di iniziare

Verifica che il progetto della Lezione 01 sia disponibile.

Una possibile struttura è:

```text
UFS04-Java/
└── src/
    └── Main.java
```

Se utilizzi IntelliJ IDEA:

1. apri il progetto;
2. verifica che sia configurato JDK 17 o JDK 21;
3. apri `Main.java`;
4. esegui il programma prima di iniziare;
5. verifica che non ci siano errori già presenti.

### Controllo JDK

Da terminale:

```bash
java -version
javac -version
```

> **Tip del docente**
>
> Prima di iniziare un laboratorio, non dare per scontato che l'ambiente funzioni. Una buona abitudine professionale è verificare sempre il punto di partenza. Se qualcosa non funziona già prima dell'esercizio, rischiamo di confondere un problema dell'ambiente con un problema del nostro codice.

---

# 3. Esercizio 1 — Il primo metodo

## Obiettivo

Creare un metodo che stampa un messaggio.

Partiamo da:

```java
public class Main {

    public static void main(String[] args) {

        System.out.println("Inizio programma");

    }
}
```

Ora aggiungiamo:

```java
public class Main {

    public static void main(String[] args) {

        saluta();

    }

    public static void saluta() {

        System.out.println("Benvenuto al laboratorio Java!");

    }
}
```

Il metodo:

- si chiama `saluta`;
- non riceve parametri;
- non restituisce un valore;
- esegue una stampa.

La parola `void` indica che il metodo non restituisce un valore.

### Prova tu

Crea i metodi:

```java
stampaTitolo()
stampaNomeCorso()
stampaSeparatore()
```

e chiamali dal `main`.

Output possibile:

```text
========================================
UFS 04 - Java
Laboratorio Lezione 02
========================================
```

Scrivi anche, come commento, perché può essere utile dividere il programma in più metodi.

---

# 4. Esercizio 2 — Metodi con parametri

Un metodo può ricevere dati:

```java
public static void saluta(String nome) {

    System.out.println("Ciao " + nome);

}
```

Possiamo chiamarlo più volte:

```java
saluta("Mario");
saluta("Luca");
saluta("Anna");
```

Nella definizione:

```java
public static void saluta(String nome)
```

`nome` è il **parametro**.

Nella chiamata:

```java
saluta("Mario");
```

`"Mario"` è l'**argomento**.

## Esercizio

Crea:

```java
stampaPersona(String nome, String cognome, int eta)
```

che produca:

```text
Nome: Mario
Cognome: Rossi
Età: 25
```

---

# 5. Esercizio 3 — Metodi che restituiscono un valore

Un metodo può calcolare un risultato e restituirlo:

```java
public static int somma(int a, int b) {

    return a + b;

}
```

Nel `main`:

```java
int risultato = somma(10, 20);

System.out.println("Risultato: " + risultato);
```

Il valore restituito può essere utilizzato, salvato, confrontato o passato ad altri metodi.

## Esercizio — Calcolatrice

Crea:

```java
somma(int a, int b)
sottrazione(int a, int b)
moltiplicazione(int a, int b)
divisione(double a, double b)
```

Poi testa tutti i metodi dal `main`.

> **Tip del docente**
>
> Quando un metodo deve produrre un dato che verrà utilizzato dal resto del programma, è generalmente più utile restituire quel dato piuttosto che stamparlo direttamente.

---

# 6. Esercizio 4 — Scope delle variabili

Osserva:

```java
public static void esempio() {

    int numero = 10;

    System.out.println(numero);

}
```

`numero` è disponibile all'interno del proprio scope.

Questo codice invece non compila:

```java
public static void esempio() {

    int numero = 10;

}

public static void main(String[] args) {

    System.out.println(numero);

}
```

Il `main` non può accedere direttamente alla variabile locale di `esempio()`.

## Esercizio

Correggi il problema senza rendere la variabile globale.

Una possibile soluzione è restituire il valore:

```java
public static int creaNumero() {

    int numero = 100;

    return numero;

}
```

e poi:

```java
int numero = creaNumero();

System.out.println(numero);
```

> **Tip del docente**
>
> Quando una variabile non è visibile dove ti serve, non pensare subito: "Devo renderla globale". Prima chiediti: **chi deve possedere questo dato e come posso passarlo al punto del programma che ne ha bisogno?**

---

# 7. Esercizio 5 — Primo array

Un array permette di memorizzare più valori dello stesso tipo:

```java
int[] voti = {24, 28, 30, 18, 27};
```

Possiamo accedere agli elementi:

```java
System.out.println(voti[0]);
System.out.println(voti[1]);
System.out.println(voti[4]);
```

Ricorda:

```text
indice:    0   1   2   3   4
valore:   24  28  30  18  27
```

### Errore intenzionale

Prova:

```java
System.out.println(voti[5]);
```

Otterrai un errore come:

```text
ArrayIndexOutOfBoundsException
```

Perché cinque elementi hanno indici da `0` a `4`.

---

# 8. Esercizio 6 — length

Per conoscere il numero di elementi:

```java
int[] voti = {24, 28, 30, 18, 27};

System.out.println(voti.length);
```

Risultato:

```text
5
```

Attenzione: `length` non è un metodo.

Corretto:

```java
voti.length
```

Non corretto:

```java
voti.length()
```

---

# 9. Esercizio 7 — Scorrere un array con for

```java
int[] voti = {24, 28, 30, 18, 27};

for (int i = 0; i < voti.length; i++) {

    System.out.println(voti[i]);

}
```

Il ciclo parte dall'indice `0` e continua finché l'indice rimane valido.

---

# 10. Esercizio 8 — Enhanced for

Quando non ci serve direttamente l'indice possiamo utilizzare:

```java
for (int voto : voti) {

    System.out.println(voto);

}
```

### Confronto

| Situazione | Scelta |
|---|---|
| Mi serve l'indice | `for` tradizionale |
| Mi serve solo il valore | enhanced `for` |
| Devo lavorare sulla posizione | `for` tradizionale |
| Devo semplicemente leggere tutti gli elementi | enhanced `for` |

---

# 11. Esercizio 9 — Calcolare la somma

Dato:

```java
int[] valori = {10, 20, 30, 40, 50};
```

Scrivi:

```java
int somma = 0;

for (int valore : valori) {

    somma += valore;

}

System.out.println("Somma: " + somma);
```

Poi trasformalo in un metodo:

```java
public static int calcolaSomma(int[] valori) {

    int somma = 0;

    for (int valore : valori) {
        somma += valore;
    }

    return somma;

}
```

Test:

```java
int somma = calcolaSomma(valori);

System.out.println("Somma: " + somma);
```

Questo è il primo esempio di **decomposizione del problema**.

---

# 12. Esercizio 10 — Calcolare la media

Scrivi:

```java
public static double calcolaMedia(int[] valori) {

    int somma = calcolaSomma(valori);

    return (double) somma / valori.length;

}
```

Poi:

```java
double media = calcolaMedia(valori);

System.out.println("Media: " + media);
```

Presta attenzione alla divisione intera e al cast verso `double`.

---

# 13. Esercizio 11 — Trovare il massimo

Implementa:

```java
public static int trovaMassimo(int[] valori) {

    int massimo = valori[0];

    for (int valore : valori) {

        if (valore > massimo) {
            massimo = valore;
        }

    }

    return massimo;

}
```

Test:

```java
int[] valori = {10, 45, 23, 89, 12};

System.out.println("Massimo: " + trovaMassimo(valori));
```

### Perché partire da `valori[0]`?

Una soluzione come:

```java
int massimo = 0;
```

funziona solo per alcuni insiemi di dati.

Con:

```java
int[] valori = {-10, -4, -25, -8};
```

il massimo corretto è `-4`.

Partire da un elemento reale dell'array rende l'algoritmo più robusto.

---

# 14. Esercizio 12 — Trovare il minimo

Completa:

```java
public static int trovaMinimo(int[] valori) {

    int minimo = valori[0];

    // completa il metodo

}
```

Testalo con valori positivi e negativi.

---

# 15. Esercizio 13 — Ricerca di un valore

Dato:

```java
int[] valori = {12, 25, 31, 18, 42};
```

crea un metodo:

```java
public static boolean contiene(int[] valori, int cercato) {

    // completa

}
```

Il metodo deve restituire `true` se trova il valore e `false` altrimenti.

Una possibile logica:

```java
boolean trovato = false;

for (int valore : valori) {

    if (valore == cercato) {

        trovato = true;
        break;

    }

}

return trovato;
```

---

# 16. Esercizio 14 — Contare sopra una soglia

Dato:

```java
int[] temperature = {
    18, 21, 24, 27, 19, 30, 22
};
```

vogliamo sapere quante temperature sono superiori a `23`.

Crea:

```java
public static int contaSopraSoglia(int[] valori, int soglia) {

    int conteggio = 0;

    for (int valore : valori) {

        if (valore > soglia) {
            conteggio++;
        }

    }

    return conteggio;

}
```

Testa il metodo con soglie diverse.

---

# 17. Esercizio 15 — Piccola libreria di metodi

Il `Main.java` dovrebbe ora contenere almeno:

```java
public class Main {

    public static int calcolaSomma(int[] valori) {
        // ...
    }

    public static double calcolaMedia(int[] valori) {
        // ...
    }

    public static int trovaMassimo(int[] valori) {
        // ...
    }

    public static int trovaMinimo(int[] valori) {
        // ...
    }

    public static boolean contiene(int[] valori, int cercato) {
        // ...
    }

    public static int contaSopraSoglia(int[] valori, int soglia) {
        // ...
    }

    public static void main(String[] args) {

        // test

    }
}
```

Il `main` dovrebbe soprattutto **coordinare** le operazioni.

---

# 18. Esercizio 16 — Caso reale: statistiche degli ordini

## Scenario

Un'azienda registra gli importi degli ordini:

```java
double[] ordini = {
    120.50,
    80.00,
    450.75,
    32.90,
    210.00,
    95.50,
    310.00
};
```

Il responsabile vuole:

1. totale;
2. media;
3. ordine massimo;
4. ordine minimo;
5. numero di ordini superiori a 100 €.

### Progettazione prima del codice

| Problema | Metodo | Parametri | Ritorno |
|---|---|---|---|
| Totale | `calcolaTotale` | `double[]` | `double` |
| Media | `calcolaMedia` | `double[]` | `double` |
| Massimo | `trovaMassimo` | `double[]` | `double` |
| Minimo | `trovaMinimo` | `double[]` | `double` |
| Sopra soglia | `contaSopraSoglia` | `double[], double` | `int` |

### Implementazione

```java
public static double calcolaTotale(double[] ordini) {

    double totale = 0;

    for (double ordine : ordini) {
        totale += ordine;
    }

    return totale;

}
```

```java
public static double calcolaMedia(double[] ordini) {

    return calcolaTotale(ordini) / ordini.length;

}
```

```java
public static double trovaMassimo(double[] ordini) {

    double massimo = ordini[0];

    for (double ordine : ordini) {

        if (ordine > massimo) {
            massimo = ordine;
        }

    }

    return massimo;

}
```

```java
public static double trovaMinimo(double[] ordini) {

    double minimo = ordini[0];

    for (double ordine : ordini) {

        if (ordine < minimo) {
            minimo = ordine;
        }

    }

    return minimo;

}
```

```java
public static int contaSopraSoglia(
        double[] ordini,
        double soglia) {

    int conteggio = 0;

    for (double ordine : ordini) {

        if (ordine > soglia) {
            conteggio++;
        }

    }

    return conteggio;

}
```

### Main

```java
public static void main(String[] args) {

    double[] ordini = {
        120.50,
        80.00,
        450.75,
        32.90,
        210.00,
        95.50,
        310.00
    };

    double totale = calcolaTotale(ordini);
    double media = calcolaMedia(ordini);
    double massimo = trovaMassimo(ordini);
    double minimo = trovaMinimo(ordini);
    int sopraSoglia = contaSopraSoglia(ordini, 100);

    System.out.println("Totale: " + totale);
    System.out.println("Media: " + media);
    System.out.println("Massimo: " + massimo);
    System.out.println("Minimo: " + minimo);
    System.out.println("Ordini sopra 100 €: " + sopraSoglia);
}
```

---

# 19. Esercizio 17 — Refactoring

Considera questo programma:

```java
public static void main(String[] args) {

    int[] voti = {24, 28, 30, 18, 27};

    int somma = 0;

    for (int voto : voti) {
        somma += voto;
    }

    double media = (double) somma / voti.length;

    int massimo = voti[0];

    for (int voto : voti) {

        if (voto > massimo) {
            massimo = voto;
        }

    }

    int minimo = voti[0];

    for (int voto : voti) {

        if (voto < minimo) {
            minimo = voto;
        }

    }

    System.out.println("Somma: " + somma);
    System.out.println("Media: " + media);
    System.out.println("Massimo: " + massimo);
    System.out.println("Minimo: " + minimo);
}
```

Il programma funziona, ma il `main` contiene responsabilità diverse.

### Obiettivo

Trasformalo in:

```java
public static int calcolaSomma(int[] valori) {
    // ...
}

public static double calcolaMedia(int[] valori) {
    // ...
}

public static int trovaMassimo(int[] valori) {
    // ...
}

public static int trovaMinimo(int[] valori) {
    // ...
}

public static void main(String[] args) {

    // dati
    // chiamate
    // stampa

}
```

### Regola

Il refactoring non deve cambiare il risultato del programma.

> **Tip del docente**
>
> Refactoring non significa "riscrivere il codice perché non mi piace". Significa migliorare la struttura mantenendo invariato il comportamento previsto.

---

# 20. Esercizio 18 — Debugging guidato

Il seguente programma contiene due problemi:

```java
public class Main {

    public static double calcolaMedia(int[] valori) {

        int somma = 0;

        for (int i = 0; i <= valori.length; i++) {
            somma += valori[i];
        }

        return somma / valori.length;
    }

    public static void main(String[] args) {

        int[] valori = {10, 20, 30};

        double media = calcolaMedia(valori);

        System.out.println("Media: " + media);
    }
}
```

### Problema 1

Individua il problema in:

```java
i <= valori.length
```

### Problema 2

Individua il problema in:

```java
return somma / valori.length;
```

La media deve essere decimale.

### Metodo di troubleshooting

```text
1. Osservo l'errore
       ↓
2. Individuo la riga coinvolta
       ↓
3. Capisco quale regola Java sto violando
       ↓
4. Formulo un'ipotesi
       ↓
5. Modifico il minimo necessario
       ↓
6. Rieseguo
       ↓
7. Verifico il risultato
```

---

# 21. Esercizio 19 — Debugging con IntelliJ

Inserisci un breakpoint nella riga:

```java
somma += valore;
```

Avvia il programma in modalità **Debug**.

Osserva:

- il valore di `somma`;
- il valore corrente di `valore`;
- l'indice, se usi un `for` tradizionale;
- quante volte viene eseguito il ciclo.

Rispondi:

1. Qual è il valore iniziale di `somma`?
2. Qual è il valore dopo la prima iterazione?
3. Qual è il valore dopo la seconda?
4. Qual è l'ultima iterazione?
5. Perché il ciclo termina?

---

# 22. Esercizio 20 — Challenge finale: gestione delle temperature

Un sistema industriale registra:

```java
double[] temperature = {
    18.5,
    19.2,
    21.0,
    23.4,
    24.1,
    22.8,
    20.5,
    25.2,
    26.0,
    21.7
};
```

Realizza metodi separati per:

1. calcolare la media;
2. trovare il minimo;
3. trovare il massimo;
4. contare le temperature superiori a 22 °C;
5. verificare se esiste una temperatura superiore a 25 °C.

Devi creare almeno:

```java
calcolaMedia(...)
trovaMinimo(...)
trovaMassimo(...)
contaSopraSoglia(...)
esisteSopraSoglia(...)
```

Il `main` deve limitarsi principalmente a definire i dati, chiamare i metodi e visualizzare i risultati.

Output possibile:

```text
Temperatura media: ...
Temperatura minima: ...
Temperatura massima: ...
Temperature sopra 22 °C: ...
Esiste una temperatura sopra 25 °C: true
```

---

# 23. Challenge extra — Qualità del codice

Fai una seconda revisione del programma.

### Nomi

Evita nomi poco significativi quando puoi utilizzare:

```java
temperature
soglia
media
massimo
```

### Metodi

Ogni metodo fa una cosa comprensibile?

### Parametri

Ogni metodo riceve soltanto i dati di cui ha bisogno?

### Return

I metodi restituiscono i risultati invece di stamparli direttamente?

### Main

Il `main` è leggibile?

---

# 24. Utilizzare l'AI per il debugging

L'obiettivo non è chiedere all'AI di svolgere l'esercizio al posto nostro.

L'obiettivo è imparare a formulare una richiesta tecnica corretta.

## Prompt debole

```text
Questo codice non funziona, sistemalo.
```

## Prompt migliore

```text
Sto sviluppando un programma Java con JDK 17.

Il programma deve calcolare la media dei valori
contenuti in un array.

Ricevo una ArrayIndexOutOfBoundsException.

Questo è il codice:

[incolla qui il codice]

Spiegami:
1. quale riga causa il problema;
2. perché l'indice non è valido;
3. quale regola sugli array Java sto violando;
4. proponimi una correzione minima;
5. spiegami come verificare che la correzione sia corretta.

Non riscrivere tutto il programma.
Voglio capire il problema.
```

### Attività

Prendi uno degli errori introdotti nel laboratorio e chiedi supporto all'AI.

Poi verifica personalmente la risposta.

Annota:

```text
Errore riscontrato:
...

Ipotesi iniziale:
...

Prompt utilizzato:
...

La risposta dell'AI era corretta?
...

Come l'ho verificata:
...

Cosa ho imparato:
...
```

> **Tip del docente**
>
> Una risposta dell'AI non è una prova che il codice sia corretto. L'AI può formulare una spiegazione plausibile ma tecnicamente errata. La verifica deve rimanere nostra responsabilità.

---

# 25. Consegna del laboratorio

Alla fine dell'attività il progetto deve dimostrare:

- metodi senza parametri;
- metodi con parametri;
- metodi con valore di ritorno;
- utilizzo di `void`;
- array;
- `length`;
- `for`;
- enhanced `for`;
- somma;
- media;
- minimo;
- massimo;
- ricerca;
- conteggio con soglia;
- decomposizione del problema;
- debugging;
- almeno un refactoring.

Una possibile struttura:

```text
UFS04-Java/
└── src/
    └── Main.java
```

---

# 26. Checklist finale

- [ ] So spiegare la differenza tra parametro e argomento.
- [ ] So creare un metodo.
- [ ] So passare un valore a un metodo.
- [ ] So restituire un valore con `return`.
- [ ] So spiegare `void`.
- [ ] So spiegare lo scope di una variabile.
- [ ] So creare un array.
- [ ] So accedere a un elemento tramite indice.
- [ ] So spiegare perché il primo indice è `0`.
- [ ] So utilizzare `length`.
- [ ] So attraversare un array con `for`.
- [ ] So utilizzare enhanced `for`.
- [ ] So calcolare una somma.
- [ ] So calcolare una media.
- [ ] So trovare minimo e massimo.
- [ ] So cercare un valore.
- [ ] So contare valori che rispettano una condizione.
- [ ] So dividere un problema in più metodi.
- [ ] So leggere un `ArrayIndexOutOfBoundsException`.
- [ ] So usare un breakpoint.
- [ ] So verificare criticamente una risposta dell'AI.

---

# 27. Conclusione

Il punto principale di questo laboratorio non è aver scritto alcuni metodi o aver utilizzato un array.

Il vero obiettivo è iniziare a cambiare il modo in cui affrontiamo un problema.

Prima:

```text
PROBLEMA
   ↓
SCRIVO TUTTO NEL MAIN
```

Ora:

```text
PROBLEMA
   ↓
ANALIZZO IL PROBLEMA
   ↓
LO DIVIDO IN RESPONSABILITÀ
   ↓
DEFINISCO I METODI
   ↓
PASSO I DATI NECESSARI
   ↓
RICEVO I RISULTATI
   ↓
VERIFICO IL COMPORTAMENTO
```

Questo modo di ragionare sarà la base delle prossime lezioni.

## Domanda finale

> Se ti viene consegnato un programma di 150 righe con un unico `main`, quali domande ti fai per capire come poterlo scomporre in metodi?

Scrivi la risposta in fondo al file oppure nel README del progetto.

Questa domanda è più importante di qualsiasi esercizio singolo svolto oggi.
