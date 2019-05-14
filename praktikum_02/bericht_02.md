# BDA, Praktikumsbericht 2

Gruppe mi6xc: Alexander Kniesz, Maximilian Neudert, Oskar Rudolf

---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({ tex2jax: {inlineMath: [['$', '$']]}, messageStyle: "none" });
</script>

## Aufgabe 1

### a)

Zuerst sollen wir ein Clustering auf der 10k-Stichprobe machen.
Die KMeans Methode von `pyspark.ml` erwartet dazu ein DataFrame mit genau einer Spalte oder ein DataFrame mit einer Spalte `features`. Um dies zu ermöglichen transformieren wir die `bin_x` Spalten mit einem `VectorAssembler` zu einer feature Spalte, in der die vorherhigen Spalten die Dimensionen der neuen Vektoren sind.
Dabei ist zu beachten, dass `VectorAssembler` je nach Speicherauslastung automatisch sparse oder voll wählt und in unserem Fall werden es sparse Vectoren, Das heißt, dass wir sehr viele 0 Werte haben.
Anschließend erstellen wir ein KMeans object `KMeans().setK(2).setSeed(1)` durch Angabe der gewünschten Clusteranzahl (k=2) und des Start-Seeds und fitten damit dann das Modell anhand des neuen DataFrames.

![image](res/1-00.png)

### b)

Als quadratischen Fehler erhalten wir für die Wahl an Centroiden:

![image](res/1-01.png)

### c)

Plotten wir die quadratischen Fehler im Bereich 2 bis 16 (Anzahl an Centroiden), so erhalten wir folgenden Plot:

![image](res/1-02.png)

Bei dem dabei entstandenen Plot wenden wir die Elbow-Methode an, bei dir wir abschätzen, ab welcher Anzahl von Clustern sich die Steigung kaum noch ändert. Wir entscheiden uns für **5** Cluster, da der Unterschied von 4 zu 5 nach unserer Meinung noch nennenswert ist, zusätzliche Cluster ab diesem Punkt aber kaum noch Mehrwert bringen:

![image](res/1-03.png)

### d)

Anwendung unseres Cut-Off-Kriteriums k=5 auf den k-means-Algorithmus und abspeichern des resultierenden Modells:

![k_5_modell](res/k_5_modell.png)

### e) - f )

Um mit den Daten in Tableau arbeiten zu können erstellen wir eine neue Tabelle mit den Profilvektoren einerseits und den aus dem Modell berechneten Predictions andererseits. Die so erstellte Tabelle zeigt uns nun für jeden Track eine zugehörige Clusternummer:

![image](res/1-04.png)

Anschließend haben wir die Tabelle mit den restlichen Daten verbunden:

![connection](res/connect_to_k_means.png)

### g)

Um sich unter den Clusten etwas vorstellen zu können, kann man diese einmal als Verlauf plotten und eine gewichtete Summe über die Bins berechnen:

![image](res/1-05.png)

Als Resultat sieht man eine Art Loudness Profil pro Cluster.

In Tableau kann man nun diverse Visualisierungen bilden. Ein paar Beispiele:

![image](res/tab-00.png)

In derser Grafik sieht man die Songs mit und deren zugeordneter Cluster.

### Hörprobe

Als Hörproben haben wir verschiedene Songs von einer ausgewählten Künstlerin (Britney Spears) angehört, die nach unserer Analyse in verschiedenen Clustern liegen und daher unterschiedliche Lautstärkeprofile haben sollten.

<div style="page-break-after: always;"></div>

## Aufgabe 2

Wir übertragen nun die Arbeit aus Aufabe 1 a)-c).
Wichtig ist dabei nochmal mittels Elbow Methode zu prüfen, ob eventuell ein andere Anzahl an Clustern notwendig ist.

![image](res/1-06.png)

![image](res/1-07.png)

Man sieht einen kleinen Unterschied und hier würde es sich anbieten 6 Cluster zu verwenden.

Die genauen Arbeitsschritte sind im Notebook unter [Zeppelin Notebook_GruppeMi6xc](https://141.100.62.87:7070/#/notebook/2EB72FHYP") dokumentiert.

Unterschiede und Gemeinsamkeiten zwischen 10k und 1M:
