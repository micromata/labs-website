---
title: Baumeister
subtitle: Baumeister bietet Entwicklern ein einheitliches und gut funktionierendes Projektsetup für das Erstellen von Webanwendungen und zwar unabhängig von Projektgröße und Komplexität
image: logo-baumeister.jpg
tags: [features]
author: mkuehnel
---


{% include alert.html style="danger" text="Das Projekt wird aktuell nicht weiterentwickelt." %}


## Baumeister – Frontend Build Workflow für alle Bedürfnisse

Das Ziel des Projektes ist es, Entwickler dabei zu unterstützen, Ihren Frontend-Code zu entwickeln, zu testen, zu optimieren und auszuliefern. Und zwar unabhängig von Projektgröße und Komplexität – von Bootstrap Themes über komplette statische Webseiten bis hin zu Single Page Applications.

Baumeister bietet unter anderem per Default:

- eine Dateistruktur mit Fokus auf Wartbarkeit und Upgradefähigkeit
- Unterstützung bei der Entwicklung
- einfache Generierung von statischen Seiten mit Hilfe von Handlebars Templates, Partials und Frontmatters

optional:

- Transpiling, Bundling und Minifizierung von Quellcode
- ES6+ (JavaScript) und Sass (CSS Preprocessor)
- Hinzufügen von CSS Vendor Prefixes
- Linting von JavaScript, Sass und HTML
- Starten eines lokalen Entwicklungsservers
- Synchronisierung zwischen mehreren Browsern und Endgeräten für simultanes Testing
- Ausführung von Unit-Tests und Generierung von Coverage Reports

und weiteres.

Unterstützung der Qualität für die Produktion:

- Release-Management: Erhöhen der Versionsnummer, Update des Changelogs etc.
- Bildoptimierung (verlustfrei)
- Entfernen von nicht verwendeten CSS (optional)
- Überprüfen nach bekannten Schwachstellen in Dependencies
- Entfernen von Console Output and Debugger Statements

Die genannten Punkte schaffen deutliche Vorteile für den Entwickler – zunächst beim Aufsetzen neuer Projekte, da alle nötigen Prozesse mit  sinnvollen Presets automatisiert sind und der Entwickler sich direkt auf das Entwickeln konzentrieren kann. Darüber hinaus werden über  Projektgrenzen hinweg Standards geschaffen und somit für Transparenz und effizienten Know-how-Transfer zwischen Entwicklerteams gesorgt. Kommt  ein neuer Entwickler ins Team, fällt die Einarbeitungszeit wesentlich  kürzer aus.

### Was macht Baumeister aus?

Das Verwalten von Dependencies lässt sich komplett über die `package.json` des Projektes lösen. Es muss keine weitere Datei angepasst werden, etwa wenn den JavaScript- und CSS-Bundles neue Libraries oder Dateien wie  Fonts hinzugefügt werden sollen. Hierfür gibt es nämlich in der `package.json`, den Abschnitt `baumeister`, über dessen Properties `bundleCSS`, `bundleExternalJS` und `includeStaticFiles` sich dies realisieren lässt.

Ein weiterer Vorteil von Baumeister gegenüber anderen Lösungen ist  die einfache Adaption von Automatisierung-Tasks durch die wartbare  Struktur. Jeder Task ist ein separates ES6-Modul und im Gulpfile werden  diese Task nur zusammengestöpselt. Wird ein Task während des gesamten  Automatisierungsprozesses nicht benötigt, lässt er sich daher sehr  schnell lokalisieren und leicht deaktivieren – auch wenn man sich mit  der Gesamtstruktur von Baumeister nicht auskennt.

### Was macht Baumeister aus?

Das Verwalten von Dependencies lässt sich komplett über die `package.json` des Projektes lösen. Es muss keine weitere Datei angepasst werden, etwa wenn den JavaScript- und CSS-Bundles neue Libraries oder Dateien wie  Fonts hinzugefügt werden sollen. Hierfür gibt es nämlich in der `package.json`, den Abschnitt `baumeister`, über dessen Properties `bundleCSS`, `bundleExternalJS` und `includeStaticFiles` sich dies realisieren lässt.

Ein weiterer Vorteil von Baumeister gegenüber anderen Lösungen ist  die einfache Adaption von Automatisierung-Tasks durch die wartbare  Struktur. Jeder Task ist ein separates ES6-Modul und im Gulpfile werden  diese Task nur zusammengestöpselt. Wird ein Task während des gesamten  Automatisierungsprozesses nicht benötigt, lässt er sich daher sehr  schnell lokalisieren und leicht deaktivieren – auch wenn man sich mit  der Gesamtstruktur von Baumeister nicht auskennt.

### Zielgruppe

Designer mit HTML-Kenntnissen, Frontend- und Full-Stack-Entwickler  für das Erstellen von Prototypen bis hin zu komplexen Webanwendungen

### Technologie-Stack

Geschrieben in ES6 JavaScript, benötigt wird Node.js und Gulp als global installiertes Modul

### Lizenz

Alle Bestandteile der von uns entwickelten Komponenten sind unter der MIT Lizenz veröffentlicht.

### Sourcecode-Ablage

[Micromata Github](https://github.com/micromata/Baumeister)
[Baumeister.io](https://baumeister.io/)

#### Weiteres

Für eine angepasste und einfache Installation bietet es sich an, den  ebenfalls von uns bereitgestellten Yeoman Generator zu verwenden. Den  Source Code und weitere Informationen zu diesem Projekt findet ihr unter Micromata Github (https://github.com/micromata/generator-baumeister) die Installation erfolgt via npm (https://www.npmjs.com/package/generator-baumeister).






