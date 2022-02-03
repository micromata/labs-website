---
title: CLI Error Notifier
subtitle: Dieser kleiner Helfer unterstützt beim Ausführen langlaufender Hintergrundprozesse im Terminal.
tags: [npm, node, cli]
author: mkuehnel
---

### Desktop Notifications im Fehlerfall

Bei im Hintergrund laufenden Prozessen möchte man in manchen Fällen gerne sofort mitbekommen wenn etwas schief läuft. Genau das ermöglicht unser `onerror `CLI: Wenn ein Command Line Tool mit einem Fehler abbricht, öffnet sich betriebssystemübergreifend eine native Desktop Notification deren Inhalt einfach zu konfigurieren ist.

#### Wie das?

Man gibt `onerror` den auszuführenden Befehl mit. Wird dieser mit einem Exit Code `!= 0` beendet wird eine Notification angezeigt.

Einfachstes Beispiel:

```bash
onerror "wget unknown-host.xyz"
```

Dies wird besonders nützlich bei Verwendung von npm scripts die auf Veränderungen im File-System lauschen um beispielsweise Quellcode zu analysieren. Ein konkretes Anwendungsbeispiel findet ihr in der [Readme Datei](https://github.com/micromata/cli-error-notifier#usage-with-npm-scripts) auf GitHub.

Ebenfalls dort findet ihr eine genaue Anleitung mit allen verfügbaren Optionen und Installationshinweise.

### Zielgruppe

Software-Entwickler

### Technologie-Stack

CLI via JavaScript und Node.js

### Installation

via [npm](https://www.npmjs.com/package/check-packages#install)

### Lizenz

[MIT Lizenz](https://github.com/micromata/cli-error-notifier/blob/master/license)

### Sourcecode und weitere Infos

[Micromata Github](https://github.com/micromata/cli-error-notifier)
