---
title: Node Modules im Projekt updaten
author: mkuehnel
categories: [Best Practices]
tags: [node]
shortdesc: Möglichkeiten seine Node.js Module im Projekt zu liften.
---

## npm Bordmittel

Bei Verwendung von [`npm update`](https://docs.npmjs.com/cli/update.html) wird [semver](https://semver.npmjs.com/) berücksichtigt. Das heißt, je nach Definition in eurer package.json  Datei, erhalten eure Module Patch bzw. Minor releases. Das ergibt Sinn,  da man sich bei Major Releases die potentiellen Breaking changes schon  genauer ansehen sollte.

`npm outdated` dagegen zeigt einem auch Updates ausserhalb der definierten semver ranges an:

```bash
$ npm outdated
Package         Current  Wanted  Latest  Location
chalk             2.4.1   2.4.1   2.4.2  check-packages
coveralls         3.0.2   3.0.2   3.0.3  check-packages
eslint            5.6.0   5.6.0  5.14.1  check-packages
jest             23.6.0  23.6.0  24.1.0  check-packages
load-json-file    5.1.0   5.1.0   5.2.0  check-packages
ora               3.0.0   3.0.0   3.1.0  check-packages
```

Allerdings überlässt es euch dann die entsprechenden Pakete via `npm install paketName@latest` upzudaten.

## Dedizierte CLI Apps

Will man es ein bisschen bequemer haben bietet es sich an eine kleine CLI App aus dem npm Öko-System zu installieren.

### npm-check-updates

Ein sehr schönes einfaches Tool ist `npm-check-updates`. Zunächst ist es global zu installieren:

```bash
npm install -g npm-check-updates
```

Danach kann das Modul in jedem Projektverzeichnis genutzt werden um Updates anzuzeigen:

```bash
ncu
```

Die Ausgabe zeigt potentielle Updates an ohne etwas zu verändern:

```bash
 chalk            2.4.1  →   2.4.2
 load-json-file   5.1.0  →   5.2.0
 ora              3.0.0  →   3.1.0
 coveralls        3.0.2  →   3.0.3
 eslint           5.6.0  →  5.14.1
 jest            23.6.0  →  24.1.0

Run ncu with -u to upgrade package.json
```

Also sind hiernach zwei Schritte nötig um alle Dependencies auf den neuesten Stand zu bringen und zu testen.

```bash
ncu -u
npm install
```

Weitere Infos und CLI Optionen finden sich auf [GitHub](https://github.com/tjunnone/npm-check-updates).

### updtr

Noch mehr Automatisierung bietet `updtr`. Es installiert zunächst modulweise alle latest Updates und führt im Anschluss via `npm test` die Tests aus. Bricht ein Update die Tests wird das Update des entsprechend Modules verworfen.

Ebenso wie bei `ncu` lassen sich Module ausschließen. Weitere Infos ebenfalls auf [GitHub](https://github.com/peerigon/updtr).

### npm-check

Eine weitere komfortable Möglichkeit ist die interaktive Auswahl der zu aktualisierenden Module via `npm-check -u`. Weitere Infos auf [GitHub](https://github.com/dylang/npm-check).

**Hinweis:** Nutzer des alternativen npm Clients [Yarn](https://yarnpkg.com) benötigen hierfür kein weiteres zu installierendes Tool. Hier führt [`yarn upgrade-interactive --latest`](https://yarnpkg.com/en/docs/cli/upgrade-interactive) zu einem vergleichbaren Erlebnis.
