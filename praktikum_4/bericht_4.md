# BDA, Praktikumsbericht 4

Gruppe mi6xc: Alexander Kniesz, Maximilian Neudert, Oskar Rudolf

---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({ tex2jax: {inlineMath: [['$', '$']]}, messageStyle: "none" });
</script>

## Quellen

Das PySpark Notebook findet man [hier](https://141.100.62.89:7070/#/notebook/2EFFKHFN5).

## Aufgabe 1

### a)

Zuerst verbinden wir uns auf eine Node, die als producer dienen soll, zum Beispiel `saltshore`:

```bash
mosh istuser@saltshore.fbi.h-da.de
```

anschließend wechseln wir in das Verzeichnis mit den vorbereiteten Scripts und starten den producer:

```bash
cd /opt/kafka/bin
./kafka-console-producer.sh --broker-list saltshore.fbi.h-da.de:9092 --topic bda-gruppe3-topic
```

Analog gehen wir vor, verbinden uns auf eine andere Node, wechseln in den Ordner und starten den consumer, der sich zum Producer zunächst ohne `--form-beginning` verbindet:

```bash
mosh istuser@sunspear.fbi.h-da.de
cd /opt/kafka/bin
./kafka-console-consumer.sh --bootstrap-server saltshore.fbi.h-da.de:9092 --topic bda-gruppe3-topic
```

![image](res/fig1-00.png)

Wenn wir beim Producer nun Nachrichten schreiben, dann werden diese mit kurzer Verzögerung vom Consumer empfangen und dort auf der Console ausgegeben.
Fügen wir nun zusätzlich den Parameter `--form-beginning` hinzu, so erhalten wir erwartungsgemäß alle Nachrichten, die bisher auf dem angegebenen Topic gesendet wurden.
Da wohl ein paar Spaßvögel mit Scripts das Topic geflutet haben dauert das sogar eine Weile auszugeben.

![image](res/fig1-01.png)

### b)

Schauen wir uns die Spalten der Kafka Ausgabe per SQL an

![image](res/fig1-03.png)

so sehen wir, dass wir neben `key` und `value` auch eine Reihe an Metadaten wie `timestamp`, `topic` und `partition` erhalten.

<div style="page-break-after: always;"></div>

Starten wir die WordCount Query

```python
%pyspark
#aufgabe 1b): Deklaration des Kafka-Consumer-Streams und Start der query

from pyspark.sql.functions import explode
from pyspark.sql.functions import split

# read text file
lines = spark \
.readStream \
.format("kafka") \
.option("kafka.bootstrap.servers", "141.100.62.88:9092") \
.option("subscribe", "bda-gruppe3-topic") \
.option("startingOffsets", "earliest") \
.option("kafkaConsumer.pollTimeoutMs", "8192") \
.load()

# cast value object to string
lines = lines\
    .withColumn("value", lines.value.cast('string'))\
    .select('value')

# split lines into words
words = lines\
    .select(explode(split(lines.value, " "))\
    .alias("word"))

# count words
count_words  = words\
    .groupBy("word").count()\
    .orderBy("count", ascending=False)

# Start running the query that prints the running counts to memory sink
writer = count_words \
    .writeStream \
    .queryName("mi6xc_kafkawords") \
    .outputMode("complete") \
    .format("memory")

query = writer.start()
```

<div style="page-break-after: always;"></div>

so erhalten wir folgendes Ergebnis:

![image](res/fig1-02.png)
