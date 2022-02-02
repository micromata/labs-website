---
title: CSV? Es lebe SQL!
author: cclaus
categories: [Quick Tips]
tags: [cli, sql]
shortdesc: Ein nettes, kleines Tool, um auf der Kommandozeile mit SQL-Abfragen CSV-Dateien zu durchforsten
featured: true
---

Ich habe neulich ein Werkzeug gebraucht um ein paar Informationen aus einer CSV-Datei zu ziehen. Dabei bin ich über ein nettes kleines  CLI-Tool namens [textql](https://github.com/dinedal/textql) gestolpert. Mit textql kann man auf CSV-Dateien mit SQL-Operationen arbeiten. Und so gehts:

Nachdem man textql installiert hat, kann man mit folgendem Befehl eine SQLite-Konsole bekommen.

```bash
textql -console -header heroes_information.csv
```

Nun ist es ratsam, sich über das Schema zunächst einmal ein paar Informationen einzuholen:

```sql
sqlite> .tables
heroes_information

sqlite> .schema heroes_information CREATE TABLE [heroes_information] ([id] NUMERIC, [name] NUMERIC, [gender] NUMERIC, [eye_color] NUMERIC, [race] NUMERIC, [hair_color] NUMERIC, [height] NUMERIC, [publisher] NUMERIC, [skin_color] NUMERIC, [alignment] NUMERIC, [weight] NUMERIC);
```

Und jetzt, mit diesen Informationen, kann man sich nun zum Beispiel  alle menschlichen Marvel Helden – sortiert nach ihrem Gewicht – ausgeben lassen.

```sql
sqlite> select id, name, weight from heroes_information where publisher = 'Marvel Comics' and race = 'Human' order by weight desc limit 10;
373|Juggernaut|855
119|Bloodaxe|495
0|A-Bomb|441
591|She-Hulk|315
583|Scorpion|310
412|Lizard|230
391|Kingpin|203
574|Sandman|203
668|Tiger Shark|203
345|Iron Man|191
```

Ach so… Für das Beispiel habe ich ein paar Superhelden aus einem [Kaggle Datensatz](https://www.kaggle.com/claudiodavi/superhero-set) genommen 😉
