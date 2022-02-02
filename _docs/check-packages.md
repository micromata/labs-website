---
title: Check Packages
subtitle: Check packages ermöglicht die Überprüfung von npm  Dependencies eines Projektes via whitelisting (oder blacklisting).
tags: [npm, node, cli]
author: ckuehl
---

### Nutzung von Open Source Software

Bei der kommerziellen Nutzung von Open Source Software ist das Thema Lizenzen von hoher Bedeutung. Je nach Lizenz der verwendeten Software  ergeben sich unterschiedliche Rechte und Pflichten, die gesetzlich  bindend zu beachten sind.

Projekte und deren Stakeholder haben verschiedene Anforderung an den  Umgang mit Open Source Software. Meist kann man diesen Anforderungen mit White- bzw. Blacklisten gerecht werden.

#### White/Blacklisting von Lizenzen

Im einfachen Fall gibt es eine Liste mit freigebenen (oder verbotenen) Lizenzen. Diese Überprüfung lässt für Abhängigkeiten aus dem npm Öko-System zum Beispiel mit Hilfe des Tools [license-checker](https://www.npmjs.com/package/license-checker) automatisieren.

#### White/Blacklisting von konkreten Abhängigkeiten

Es gibt jedoch auch die Anforderung Abhängigkeiten nicht nach  verwendeten Lizenzen freizugeben, sondern nur eine Liste bestimmter  Pakete zu erlauben bzw. zu verbieten.

Zur Automatisierung dieses Use Case wurde die CLI Applikation [`check-packages`](https://github.com/micromata/check-packages) entwickelt.

### Zielgruppe

Entwickler, DevOps, Legal

### Technologie-Stack

CLI via JavaScript und Node.js

### Installation

via [npm](https://www.npmjs.com/package/check-packages#install)

### Lizenz

[MIT](https://github.com/micromata/check-packages/blob/master/LICENSE)

### Sourcecode und weitere Infos

[Micromata Github](https://github.com/micromata/check-packages/blob/master/LICENSE)
