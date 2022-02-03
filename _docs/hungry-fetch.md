---
title: Hungry Fetch
subtitle: GWiki is a wiki-based multi purpose content management system.
image: logo-wwwwwwiki.png
tags: [api]
author: pmandler
---

Hungry Fetch monkey patcht the globale fetch(…) Funktion und speichert alle Aufrufe der Funktion mit den dazugehörigen Parametern (ähnlich der jest.fn() Funktion von Jest). Im Unterschied zu normalen Mock Funktionen löst Hungry Fetch den Fetch Aufruf mit einer gemocketen Antwort auf oder verschluckt die Anfrage (gibt undefined als Antwort zurück). Der Grundgedankte war es die Schnittstelle zwischen Frontend und Backend zu testen anstatt jede Netzwerkbezogene Funktion zu mocken.

Hungry Fetch wurde ursprünglich für Redux Thunks action creators designt die Fetch verwenden. Mit Hungry Fetch kann man die action creators einfach ausführen und prüfen welche Daten gesendet würden. In Kombination mit Jests Snapshot Tests kann man diese action creators super einfach testen. Es hat sich allerdings gezeigt, dass Hungry Fetch sehr gut geeignet ist beliebige Funktionen zu testen die Netwerkaufrufe auslösen.

Hungry Fetch wird weiter entwickelt und bietet bislang die grundlegenden Funktionen. In Zukunft sollen mehr Hilfsfunktionen zum Auswerten der Fetch Aufrufe dazu kommen. Vorschläge sowie Unterstützung sind immer willkommen.

Beispiele und weitere Informationen gibt’s auf [GitHub](https://github.com/micromata/hungry-fetch).

