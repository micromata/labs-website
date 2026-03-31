---
title: Vor und Nachteile einer linearen Git-Historie
author: mkuehnel
categories: [Best Practices]
tags: [Git, version control, linear history]
shortdesc: Was spricht für eine Linerare History also »rebasen« statt »mergen«? Was gewinne ich dadurch, welche Einschränkungen ergeben sich?
---

Saubere Git-Historien erleichtern das Verständnis von Code-Änderungen. Dieser Beitrag betrachtet lineare und nicht-lineare History-Strategien, vergleicht Git Flow und GitHub Flow und zeigt, wann Rebase statt Merge sinnvoll ist.

## Lineare vs. nicht-lineare History
Eine Git-Historie kann linear bleiben oder durch Merge-Commits eine Verzweigung aufweisen. Lineare Verläufe zeigen eine klar definierte Reihenfolge von Commits, während nicht-lineare Verläufe Merge-Commits beinhalten, die mehrere Entwicklungsstränge zusammenführen.

##Kurzer Rückblick auf Branching-Modelle

**Git Flow vs. GitHub Flow**  
Git Flow teilt die Entwicklung in mehrere Branches wie `develop`, `release` und `feature`. Bei komplexen Projekten kann das hilfreich sein, wirkt aber in kleineren Teams oft wie Over-Engineering. GitHub Flow konzentriert sich auf `main` und kurze Feature-Branches, die via Pull Request in `main` gemergt werden.

**Warum einfach, wenn es kompliziert geht?**  
Git Flow erfordert mehrere Schritte pro Release und mehr Koordination. GitHub Flow bleibt übersichtlicher, da frühzeitiges Integrieren und Kurze-Branch-Phasen das Risiko divergierender Histories verringern.

##Non-lineare History
Nicht-lineare Verläufe entstehen durch klassische Merge-Commits. Je größer das Team und je mehr Release-Zweige es gibt, desto schwieriger wird das Nachvollziehen einzelner Änderungen. Überschneidende Branches führen zu einem unübersichtlichen Verlauf, was die Fehlersuche erschwert.

## Lineare History
Eine lineare Historie entsteht, wenn Pull Requests oder lokale Updates über Rebase integriert werden. Statt Merge-Commits wird der Feature-Branch auf den Zielbranch neu aufgebaut. Das setzt gelegentliches Force-Push voraus, was sorgfältige Absprachen erfordert. Alternativ kann ein Fast-Forward-Merge genutzt werden, um den Verlauf ohne zusätzlichen Merge-Commit fortzuführen.  
- **Vorteile**: Bessere Lesbarkeit und klare Reihenfolge aller Änderungen.  
- **Nachteile**: Aufwendigeres Release-Management, besonders wenn Releases nur von `main` ausgehen und `main` immer den letzten Release-Commit repräsentiert. Cherry-Picking für einzelne Features wird schwieriger, da Branches rebasebar bleiben müssen.

## Umgang mit Releases
Ein simples Setup umfasst `develop` für neue Features und `main` für Releases:
- **Reguläre Releases**: Fast-Forward-Merge von `develop` nach `main` und anschließend ein Release-Tag.  
- **Hotfix-Releases**: Direkt von `main` aus starten, um kritische Fehler schnell zu beheben. Anschließend einen Fast-Forward-Merge zurück nach `develop` durchführen, damit beide Branches synchron bleiben.

---

Weitere Hinweise finden sich in den [offiziellen Git-Docs](https://git-scm.com/docs). Eine saubere Git-Historie erfordert klare Absprachen und sorgfältige Workflows, um langfristig mehr Transparenz und Wartbarkeit zu schaffen.
