# BDA, Praktikumsbericht 3

Gruppe mi6xc: Alexander Kniesz, Maximilian Neudert, Oskar Rudolf

---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({ tex2jax: {inlineMath: [['$', '$']]}, messageStyle: "none" });
</script>

## Quellen

Das PySpark Notebook findet man [hier](https://141.100.62.87:7070/#/notebook/2ECVJRCPX).

## Aufgabe 1

Wir lesen das `txt` file als DataFrame ein. Anschließend kann man mit `withColumn` diverse Operationen auf den Spalten ausführen.
Das DataFrame enthält alle Zeilen des `txt` Files als Zeilen. In unserem Fall haben wir die Daten zuerst bereinigt, sprich Sonderzeichen und leere Zeilen entfernt.
Anschließend haben wir mittels `split` die Strings in den Zeilen in Wörter Arrays umgewandelt, danach die Arrays mit `explode` in weitere Zeilen erweitern und abschließend MapReduce mit `lit` und `groupBy` gemacht.

![image](res/fig1-00.png)

<div style="page-break-after: always;"></div>

## Aufgabe 2

Wir verbinden uns mittels ssh auf:

```bash
141.100.62.87
```

dort haben wir eine `tmux` session mittels

```bash
tmux -S /tmp/smux new -s amo
```

erstellt, auf die wir uns dann mittels

```bash
tmux -S /tmp/smux attach -t amo
```

gemeinsam verbinden und mit netcat arbeiten können.

### Code

![image](res/fig2-00.png)

### unterschiede in den Parameters

`outputMode` besitzt folgende Paramter:

- **append**: Nur neue Zeilen werden in den Output geschrieben. Exklusiv für de Verwendung ohne Aggregationen.
- **complete**: Alle Zeilen werden jedes mal in den Output geschrieben. Exklusiv für die Verwendung mit Aggregationen.
- **update**: Nur veränderte Zeilen werden in den Output geschrieben. Ohne Aggregation wie **append**.

und `format` besitzt unter anderem folgende Parameter:

- **console**: Schreibt den verarbeiteten Stream in die Console sprich in den Standard Output.
- **memory**: Schreibt den verarbeiteten Stream in eine in Memory Datenbank. Folglich für große Datenmengen eher ungeeignet.

### Testergebnisse

Wir haben die ersten 100 Wörter von Lorem Ipsum ein paar mal über `netcat` abgesendet und erhalten folgendes Ergebnis:

![image](res/fig2-01.png)

Wir modifizieren die query, indem wir die Splits anpassen, neue Spalten generieren und anschließend `url` auswählen und mit dieser gruppieren und zählen.

![image](res/fig2-02.png)

Wir erhalten damit folgendes exemplarisches Ergebnis:

![image](res/fig2-03.png)

<div style="page-break-after: always;"></div>

## Aufgabe 3

### a)

Sliding windows lassen sich durch die `sql` Funktion `window` im `groupBy` Befehl erstellen.

![image](res/fig3a-00.png)

### b)

Tabellarisch erhalten wir folgenden Output sortiert nach url und Startzeit des Windows:

![image](res/fig3b-00.png)

Visualisiert man zum Beispiel mit einer Barchart ohne irgendwelche Einstellungen, so erhält man die Counts abhängig von der URL.

![image](res/fig3b-01.png)

Unter settings kann man dann ähnlich wie mit Tableau Daten gruppieren und aggregieren. Ein schöner Plot ergibt sich zum Beispiel, indem man mit der Startzeit des Windows zusätzlich gruppiert:

![image](res/fig3b-02.png)

Im Prinzip macht es nur wirklich Sinn in diesem Fall `start` und `url` zwischen `keys` und `groups` zu tauschen. Ein weiterer schöner Graph ist `start` als Key und `url` als Group stacked.

![image](res/fig3b-03.png)

<div style="page-break-after: always;"></div>

## Aufgabe 4

### a)

Als Modifikation fügen wir eine weitere Spalte zur Gruppierung hinzu.

![image](res/fig4a-000.png)

Zuerst einmal aus Interesse gesichtet, wie die Verteilung der Länder ist.

![image](res/fig4a-00.png)

Dies kann man dann um die Länder erweitern.

![image](res/fig4a-01.png)

Und schlussendlich um die URLs mit den Ländern als Gruppierung.

![image](res/fig4a-02.png)

Dann erkennt man zum Beispiel, dass die meisten Aufrufe aus Österreich auf die Basisseite um 15:54 bis 15:55 waren.

### b)

Wir bauen einen Filter mit `lines.country == 'Germany'` und ersetzen `country` mit `status`.

![image](res/fig4b-00.png)

Anschließend erhalten wir folgendes Ergebnis:

![image](res/fig4b-01.png)

Man sieht, dass `order` und `root` in `Germany` deutlich öfter `404` sprich nicht gefunden werfen als `about`.

### c)

Zu den Unterschiedenlichen Ausgabemodi muss man zuerst daran denken, dass `append` generell nicht auf aggregierten Daten funktioniert und `update` nicht mit sortierung. Dies macht auch Sinn, weshalb wir die ersten beiden Aufgaben mit `complete` bearbeitet haben.

Im Falle von Late Data kann es nun passieren, dass alte Fenster von neuen Daten modifiziert werden. Dazu gibt es in Sparks `watermarks`, welche mit `.withWatermark("timestamp", "xx minutes")` definiert werden können und dann Datenpakete, die mit einer Verzögerung größer als die angegebene Zeit, direkt gedroppt werden.

Lässt man die Sortierung der Daten weg (ist für die Visualisierung eh nicht wirklich nötig), so können wir `update` als Alternative zu `complete` verwenden. Im Gegensatz zu `complete` aktualisiert `update` nur die neusten Zeilen. Dies ist mit Sliding Windows vollkommen in Ordnung, da wir keine Berechnungen auf alten Daten benötigen und wir somit die Performance verbessern können.
Eine nennenswerte Veränderung ist nicht zu erwarten, außer es gäbe viele Late Data, dann würde man besonders in `a)` unterschiede sehen, da `complete` die späten Daten mit in die Berechnung einbeziehen würde, während `update` eben dies nicht tut.
