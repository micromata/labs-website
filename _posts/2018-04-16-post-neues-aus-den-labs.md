---
title: Neues aus den Labs
author: mkuehnel
categories: [Open Source]
tags: [github, labs]
shortdesc: Neuigkeiten der letzten Monate im Open Source Bereich von Micromata – Wir haben auf GitHub 4 neue Projekte und neue Major Releases unserer wichtigsten Tools.
---

### Awesome Open Source Supporting

Wir alle nutzen jeden Tag Open Source Software und profitieren somit tagtäglich von der Arbeit von Open Source Software Maintainern. Jeder Open Source Nutzer sollte sich bewußt machen wie viel Zeit hinter vielen dieser Projekte steigt und überlegen wie er etwas zurückgeben kann. Sei es Zeit, Geld oder einfach ein bisschen Liebe.

Diese Liste will dabei unterstützen:
https://github.com/micromata/awesome-open-source-supporting

### cli-error-notifier – Sends native desktop notifications if CLI apps fail

Dieser kleiner Helfer unterstützt beim ausführen langlaufender Hintergrundprozeße im Terminal.

Wenn ein Command Line Tool mit einem Fehler abbricht, öffnet sich betriebssystemübergreifend eine native Desktop Notification deren Inhalt einfach zu konfigurieren ist.

Dies ist besonders nützlich bei Verwendung von npm scripts die auf Veränderungen im File-System lauschen um beispielsweise Quellcode zu  linten.

Auf GitHub befindet sich eine genaue Anleitung und Installationshinweise:
https://github.com/micromata/cli-error-notifier

### swd – A totally simple and very easy to configure stand alone webdav server

*swd* ist ein standalone Webdav-Server, der ganz gezielt auf Einfachheit in Benutzung, Betrieb und Konfiguration ausgelegt ist.

Dabei ist *swd* in der Programmiersprache *Go* geschrieben und besteht aus genau drei Dateien.

1. Der Server *swd*.
2. Ein CLI-Tool zum Generieren von Passwort hashes *swdcli*.
3. Einer Konfigurationsdatei.

Dabei kann der *swd* Server sowohl unter Linux-, als auch Windows- und Mac-OS-Systemen betrieben werden. Vorkompilierte Binaries stehen auf [Github](https://github.com/micromata/swd/releases) bereit.

Die folgenden Features werden von der aktuellen Release-Version 0.2.0 abgedeckt:

- Authentifizierung via HTTP-Basic.
- TLS Support zur Bereitstellung eines sicheren Kommunikationskanals (optional, aber empfohlen).
- Einfache Benutzerverwaltung. Benutzer sehen entweder ein für sie definiertes Verzeichnis oder das Stammverzeichnis des Servers.
- Live-Config-Reload um Benutzer während des laufenden Serverbetriebs zu ändern.
- Support um hinter einem Proxy unter Angabe eines definierten Pfades zu arbeiten.

*swd* ist genau das richtige Werkzeug um einfach, schnell und sicher eine Plattform zur Dateifreigabe herzustellen, die mit Boardmitteln wie dem OSX Finder, Windows Explorer oder Nautilus unter Linux genutzt werden können.

Weitere Informationen zur Konfiguration und Mitarbeit befinden sich in der Dokumentation auf [Github](https://github.com/micromata/swd/).

### Hungry Fetch – Nomnomnom… eats and remembers your fetch calls

Hungry Fetch ist ein leichtgewichtiges Tool um das Testen von Web Apps die die [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) verwenden zu vereinfachen.

Hungry Fetch monkeypatcht the globale `fetch(…)` Funktion und speichert alle Aufrufe der Funktion mit den dazugehörigen Parametern (ähnlich der `jest.fn()` Funktion von Jest). Im Unterschied zu normalen Mock Funktionen löst Hungry Fetch den Fetch Aufruf mit einer gemocketen Antwort auf oder verschluckt die  Anfrage (gibt `undefined` als Antwort zurück). Der  Grundgedankte war es die Schnittstelle zwischen Frontend und Backend zu  testen anstatt jede Netzwerkbezogene Funktion zu mocken.

Hungry Fetch wurde ursprünglich für [Redux Thunks](https://github.com/gaearon/redux-thunk) *action creators* designt die Fetch verwenden. Mit Hungry Fetch kann man die *action creators* einfach ausführen und prüfen welche Daten gesendet würden. In Kombination mit [Jests Snapshot Tests](https://facebook.github.io/jest/docs/en/snapshot-testing.html) kann man diese *action creators* super einfach testen. Es hat sich allerdings gezeigt, dass Hungry Fetch sehr  gut geeignet ist beliebige Funktionen zu testen die Netwerkaufrufe  auslösen.

Hungry Fetch wird weiter entwickelt und bietet bislang die  grundlegenden Funktionen. In Zukunft sollen mehr Hilfsfunktionen zum  Auswerten der Fetch Aufrufe dazu kommen. Vorschläge sowie Unterstützung  sind immer willkommen.

Beispiele und weitere Informationen gibt’s auf GitHub: https://github.com/micromata/hungry-fetch

## Neue Releases und Änderungen

### Awesome JavaScript Learning

Unsere Awesome List steuert langsam aber sicher auf die ersten 1.000 GitHub Stars zu. Neu hinzugekommen sind folgende Links:

- [Robust Client-Side JavaScript](https://molily.de/robust-javascript/) – Guide focused on writing robust code by describing possible failures and explaining how to prevent them.
- [Understanding Hoisting](https://scotch.io/tutorials/understanding-hoisting-in-javascript) – Detailed explanation of the concept of hoisting in JavaScript.
- [Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) – Describes thoroughly how to use the Fetch API to receive and send data.
- [30 seconds of code](https://github.com/Chalarangelo/30-seconds-of-code) – Useful ES6 snippets that you can understand in 30 seconds or less.

Den Artikel zum Thema Async Functions haben wir durch ersetzt durch » [Async/Await](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9) – Tutorial showing the advantages of consuming Promises via async functions.«

Zur kompletten Liste:
https://github.com/micromata/awesome-javascript-learning

### Baumeister v3

> Baumeister is here to help you to build your things. From Bootstrap themes over static websites to single page applications.

Nach 2. Beta Versionen wurde die finale Version 3.0.0 von Baumeister am 9. April released.

Recap der wichtigsten Changes seit dem letzten 2.x Release:

- Gulp durch webpack und npm scripts ersetzt.
- Upgrade auf Bootstrap 4.
- Dynamic import der Polyfills.
  - Polyfills werden nur geladen wenn ein Browser sie tatsächlich  benötigt. Dadurch bleibt das Vendor Bundle in modernen Browsern klein.
- Der Yeoman Generator erstellt bei Bedarf eine React App – inklusive benötigten Tooling.

Der Baumeister Generator stellt nun also die folgenden zwei Projekt-Typen bereit:

- Einen Static Site Generator (mit Handlebars und Frontmatters)
- Eine React App (mit entsprechenden ESLint Settings/Plugins und Babel Plugins)

Die ausführliche Release Notes inklusive Migration Guide finden sich hier:

- https://github.com/micromata/Baumeister/releases/tag/3.0.0
- https://github.com/micromata/generator-baumeister/releases/tag/3.0.0

Für Baumeister Neulinge empfiehlt sich zum Einstieg die [Dokumenation](https://github.com/micromata/Baumeister#baumeister--the-frontend-build-workflow-for-your-needs) zu lesen.

### Fake Backend v4

Bereits im Dezember 2017 gab es ein Major Release des fake Backends. Im Februar folgte mit v4.1 ein weiteres Minor release.

Nachfolgend eine kurze Zusammenstallung der wichtigsten Änderungen seit Version 4:

- Unterstützung für andere Content Types als JSON
- Neben Inhalt von Dateien, können jetzt Dateien selbst als Response gesendet werden
- Möglichkeit eigene custom Response Header zu setzen

Release Notes finden sich hier:
https://github.com/micromata/http-fake-backend/releases

Um über die Entwicklungen im Labs Bereich der Micromata auf dem laufendem zu bleiben empfiehlt es sich Micromata auf [Twitter](https://twitter.com/miocromata) zu folgen.