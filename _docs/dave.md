---
title: Dave
subtitle: Dave ist ein simpler und einfach zu konfigurierender standalone Webdav-Server. Er ist ganz gezielt auf Einfachheit in Benutzung, Betrieb und Konfiguration ausgelegt.
image:
tags: [cli, webdav, go]
author: cclaus
---

*Dave* in der Programmiersprache *Go* geschrieben und besteht aus genau drei Dateien.

1. Der Server *dave*.
2. Ein CLI-Tool zum Generieren von Passwort hashes *davecli*.
3. Einer Konfigurationsdatei.

Dabei kann der *dave* Server sowohl unter Linux-, als auch Windows- und Mac-OS-Systemen betrieben werden. Vorkompilierte Binaries stehen auf [Github](https://github.com/micromata/dave/releases) bereit.

Die folgenden Features werden von der aktuellen Release-Version 0.2.0 abgedeckt:

- Authentifizierung via HTTP-Basic.
- TLS Support zur Bereitstellung eines sicheren Kommunikationskanals (optional, aber empfohlen).
- Einfache Benutzerverwaltung. Benutzer sehen entweder ein für sie definiertes Verzeichnis oder das Stammverzeichnis des Servers.
- Live-Config-Reload um Benutzer während des laufenden Serverbetriebs zu ändern.
- Support um hinter einem Proxy unter Angabe eines definierten Pfades zu arbeiten.

*Dave* ist genau das richtige Werkzeug um einfach, schnell und sicher eine Plattform zur Dateifreigabe herzustellen, die mit Boardmitteln wie dem OSX Finder, Windows Explorer oder Nautilus unter Linux genutzt werden können.

Weitere Informationen zur Konfiguration und Mitarbeit befinden sich in der Dokumentation auf [Github](https://github.com/micromata/dave).







