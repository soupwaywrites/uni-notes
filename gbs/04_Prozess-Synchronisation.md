# 04 - Prozess-Synchronisation

## 4.2 Modellierung

### **Race condition**
* Mindestens 2 Prozesse greifen gleichzeitig unkoordiniert lesend/schreibend auf gemeinsame Ressource zu. 
* Da die Ausführung dann nicht-deterministisch ablaufen würde, kann das Ergebnis fehlerhaft sein.
* Race conditions können durch mutual exclusion der konkurrierenden Aktionen vermieden werden. 

### **Mutual exclusion**
* Bei Konkurrenzsituationen paralleler Prozesse um exklusiv nutzbare Ressourcen muss eine deterministische Ausführung gewährleistet sein.
* Die Modifikationsoperationen müssen also sequentialisiert werden, indem sie wechselseitig ausgeschlossen ausgeführt werden.

### **Kritischer Abschnitt**
* Ein Abschnitt, in dem Prozesse auf gemeinsame Ressourcen zugreifen und wechselseitig ausgeschlossen sind.
* Um Race conditions zu vermeiden, muss der Zugriff sequentialisiert werden.
* Phasen des kritischen Abschnitts:
    1. Ausführung d. unkritischen Abs.
    2. Betreten d. kritischen Abs.
    3. Ausführung d. Transition des kritischen Abschnitts.
    4. Verlassen d. kritischen Abschnitts.

## 4.3 Koordinierungskonzepte

### Lösungskonzepte
#### Auf HW-Ebene
* Realisierung von Wechselseitigem Ausschluss durch **Unterbrechungssperre / Deaktivieren v. Interrupts**, sodass ohne Unterbrechung seinen krit. Abs. ausführen kann.
    * Nur bei Ein-Prozessor-Systemen
    * Nur bei kurzen krit. Bereichen immerhalb von BS- / maschinennahen Programmen
    * Nicht geeignet bei Benutzerprozessen
* **Aktives Warten** (busy waiting): Atomare Maschinenbefehle 
    * *Test-Set-Lock* (`TSL RX, LOCK`) - kontinuierliches Überprüfen einer Variable, bis ein bestimmter Wert auftaucht.
    * Kann natürlich zum Deadlock führen, wenn die Prozesse auf einer 1-Prozessor CPU laufen.

#### Durch einfache BS-System-Dienste
* **Passives Warten**:
    * `sleep` und `wakeup`, um P in Zstd. wartend oder rechenbereit überzuführen
    * Dieser blockierte P kann von einem anderen P wieder aktiviert werden, wenn die Voraussetzungen für das betreten d. kritischen Abschnitts erfüllt sind.

#### BS-Konzepte auf höherem Abstraktionsniveau
* Semaphore
    * *Ganzzahlige* Kontrollvariable
    * Operationen:
        * Initialisieren
        * up `V`
        * down `P`
* Mutex (binäre Semaphore)


#### Programmiersprachen-Konzept
**Monitor-Konzept** in Java:
* Abkapselung der sich im kritischen Abschnitt befindenden Daten und die darauf definitierten Zugriffsaktionen in Einheit - dem Monitor.
* Monitor stellt exklusive Nutzung seines INhalts sicher.
* Aufruf des Monitor gleichbedeutend mit Sperrung des Monitors bis zum Verlassen des Monitors.

### Producer-consumer problem / Bounded-buffer problem

This is a dangerous implementation of a critical region. Why?

* An instance of the consumer process is created
* Assume that the buffer is empty.
* Consumer executes line `if (count == 0)`, BUT the scheduler stops the consumer process right at this very moment and schedules a producer process. Note: the consumer did not have the opportunity to go to sleep yet.
* The producer will produce an item and proceed to wake up the consumer once `count == 1` holds true.
* The wakeup signal however, will be lost, because technically the consumer is not yet asleep.
* The producer will continue to produce items until the buffer is full and goes to sleep.
* Both producer and consumer will be asleep forevermore.
* Solution: Wakeup bit?

```C
#define N 100
int count = 0;

/* Producer process */
void producer(void) {
    int item;

    while(true) {
        item = produce_item();
        if (count == N) /* buffer full, can't insert anything */
            sleep();
        insert_item(item);
        count += 1;
        if (count == 1) /* now there are items in the buffer to take from */ 
            wakeup(consumer);
    }
}

/* Consumer process */
void consumer(void) {
    int item;

    while (true) {
        if (count == 0) /* buffer empty, nothing to take */
            sleep();
        item = remove_item();
        count -= 1;
        if (count == N - 1) 
            wakeup(producer);
        consume_item(item);
    }
}
```

What is a better approach? This one with semaphores.
The critical region that is just to be accessed by one process at a time is each encapsulated within a `down`- and `up`-call of our binary semaphore to maintain the mutual exclusion of the producer and consumer processes.

```C
#define N 100
typedef int semaphore;

semaphore mutex = 1;
semaphore empty = N;
semaphore full = 0;

/* Producer process */
void producer(void) {
    int item;

    while(true) {
        item = produce_item();
        down(&empty);
            down(&mutex);
                insert_item(item);
            up(&mutex);
        up(&full);
    }
}

/* Consumer process */
void consumer(void) {
    int item;

    while (true) {
        down(&full);
            down(&mutex);
                item = remove_item();
            up(&mutex);
        up(&empty);
        consume_item(item);
    }
}
```

## 4.4 Deadlocks
Ein Zustand, der eine Menge von Prozessen beschreibt, in der jeder Prozess auf ein Ereignis wartet, das nur ein anderer Prozess aus dieser Menge auslösen kann.

### Deadlock-Bedingungen (Coffman, 1971)
1. Mutual Exclusion
    * Exklusive Nutzbarkeit einer Ressource
    * Entweder einem P zugeordnet, oder verfügbar
2. Hold-and-wait
    * P mit zugeteilten Ressourcen belegen diese, auch wenn sie noch weitere Ressourcen anfordern.
3. No-preemption
    * Zugeteilte R können einem P nicht entzogen werden.
4. Circular wait
    * Zyklische Kette von P, die auf R warten, die vom nächsten P in der Kette belegt sind.
    * Z.B. im Belegungsanforderungsgraphen `(P1)--->[R1]--->(P2)--->[R2]--->(P1)`

### Deadlock-Modellierung
Belegungsanforderungsgraph (Holt, 1972):
* Prozesse (Kreise)
* Ressourcen (Quadrate)
* Prozess P belegt Ressource R `(P)<----[R]`
* Prozess P fordert Ressource R `(P)---->[R]`

### Deadlock-Strategien
Was tut das BS?
1. **Ignorieren**
2. **Detection**
    * Erkennung, dann:
    * Beseitigung:
        * P-Abbruch, anschließend Neustart
        * R-Enzug
        * Zurückführung verklemmter Prozesse auf Checkpoint
        * *BUT*: Verlust durch Abbruch
3. **Avoidance**
    * BS führt keine Ressourcenzuteilung durch, die überhaupt in einem Deadlock enden könnte
    * **Bankier-Algorithmus** (Dijkstra, 1965)
    * *BUT*: Statisch, inflexibel, ineffizient
4. **Prevention**
    * Außerkraftsetzen einer der Deadlock-Bedingungen
    * **Deadlock-frei by Design**: Sicherstellen, dass mind. eine der vier Dealock-Bedingungen nicht erfüllt ist.
        1. Mutual Exclusion:
            * **Spooling**: Nur ein Spooler-Prozess hat als einziger die Ressource zugeteilt.
            * Verwaltung d. Aufträge über Queue.
            * Indirekte Nutzung d. R über den Spooler.
        2. Hold-and-wait
            * Vorheriges Anforderung d. benötigten Ressourcen.
            * Keine Ressourcenbelegung während des Wartens auf eine nicht-freie Ressource.
            * Problem: Ressourcenbedarf nicht vorhersehbar.
        3. No-preemption:
            * Virtualisierung von Ressourcen, Entzug bei Bedarf.
        4. Circular wait
            * Geordnete Ressourcen Anforderung z.B. über festgelegte lineare Ordnung auf den Ressourcen.
    * *BUT*: Ressourcenbedarf ist i.d.R. nicht vorhersehbar, lange Blockaden.

### Weitere Verklemmungsarten
**Livelock**:
* 2 parallele Prozesse setzen Locks, um R zu belegen, geben sie aber wieder frei, wenn sie merken, dass R von anderem P belegt ist.
* Fortsetzung d. Ausführung, erneutes Setzen d. Locks für R, die vom anderen P belegt sind.
* Kein Fortschritt, also befinden sich beide P in einem Livelock.

**Starvation**:
* Verhungern eines Prozesses, wenn eine Ressourcenanfrage nie erfüllt wird
* Ursache: Unfaire Schedulingstrategie