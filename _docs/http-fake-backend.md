---
title: HTTP Fake Backend
subtitle: HTTP Fake Backend ermöglicht es Frontendentwicklern mit Hilfe eines eigenen Servers komfortabel und flexibel ein simuliertes Backend sowie gemockte Daten für die Anwendungsentwicklung bereitzustellen.
image: logo-fake-backend.png
tags: [api, json, javascript, node]
author: mkuehnel
---

### Open-Source Software für die Simulation von Backend-Daten

Um das Frontend einer Single Page Applikation unabhängig vom echten Backend entwicklen zu können, muss man die Endpunkte einer API mocken, um JSON geliefert zu bekommen.

Das HTTP Fake Backend bietet die Möglichkeit auf einfachen Wege Endpunkte zu erstellen, die statisches JSON liefern. Das Fake Backend basiert auf einem Node.js Server der Dummy-Daten über HTTP bereitstellt. Die Daten lässt sich über JSON-Dateien im Filesystem oder alternativ über JavaScript Object Literals erzeugen. Es funktioniert durch CORS-Domain übergreifend und bietet die folgenden Funktionalitäten:

- Konfiguration beliebiger Ports und API prefixes (z.B. http://localhost:8081/myApi)
- Unterstützung alle HTTP-Verben
- Multiple HTTP Verben pro Endpunkt
- URL Pfad Parameter mit variablen Segmenten (z.B. /update/{id})
- Optionale URL Pfad Parameter (z.B. /update/{id?})
- Definition beliebiger HTTP Status codes für den Response – Bedarfsweise vordefinierte Fehlermeldung für das Faken von Fehlern

Die Endpunkte lassen sich per JavaScript sehr einfach konfigurativ erstellen. Hierzu sind keine tiefen JavaScript Kenntnisse notwendig.

### Zielgruppe

Frontend-Entwickler

### Technologie-Stack

JavaScript und Node.js

### Lizenz

Alle Bestandteile der von uns entwickelten Komponenten sind unter der [MIT Lizenz ](https://opensource.org/licenses/MIT)veröffentlicht.

### Sourcecode-Ablage

[Micromata Github](https://github.com/micromata/http-fake-backend)

### Weiteres

Um die Installation des HTTP Fake Backends zu vereinfachen, gibt es den Generator HTTP Fake Backend. Dieser basiert auf  Yeoman. Den Source Code und weitere Informationen zu diesem Projekt findet ihr unter [Micromata Github](https://github.com/micromata/generator-http-fake-backend) und ein npm-Installationspaket liegt ebenfalls unter [nmp](https://www.npmjs.com/package/generator-http-fake-backend).
