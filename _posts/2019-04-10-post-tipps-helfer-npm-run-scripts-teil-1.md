---
title: Tipps und Helfer für npm run Scripts (1/2)
author: mkuehnel
categories: [Best Practices]
tags: [npm]
shortdesc: Möglichkeiten zur Verbesserung eures Frontend Build-Setups. 
permalink: /blog/2019-04-10-post-tipps-helfer-npm-run-scripts-teil-1/
featured: false
---

Der Zweck dieses zweiteiligen Artikels ist nicht, npm als  Build-Tool anzupreisen. Zu diesem Thema gibt bereits viele gute Artikel, in denen sowohl gute Gründe für die Verwendung von npm scripts als auch exemplarische Umsetzungen von bestimmten, gängigen Build-Schritten  beschrieben sind. Es geht vielmehr darum, einige vielleicht nicht so  offensichtliche Möglichkeiten aufzuzeigen und dadurch potentiell euer  Build-Setup zu verbessern.

Wenn ihr jedoch erstmal eine Einführung in die Vorteile der  Verwendung von CLI Tools via npm Scripts benötigt und erfahren wollt,  welche Vorteile euch das im Gegensatz zu Taskrunnern wie Gulp und deren  Plugin Öko-System bietet, empfehle ich den Artikel [Why npm Scripts?](https://css-tricks.com/why-npm-scripts/).

Im heutigen, ersten Teil des Artikels geht es darum, was ihr mit Bordmitteln die euch npm bietet erreichen könnt.

## Kurze Einführung

Ihr könnt einfach npm Scripts definieren, in dem ihr sie in die `scripts` Property in eurer `package.json` Datei eintragt und sie dann via `npm run script-name` [ausführen](https://docs.npmjs.com/cli/run-script). Mit Hilfe des Befehls `npm run` könnt ihr alle verfügbaren Scripts auflisten.

Binaries von lokal installierten Packages sind automatisch  »verlinkt«, so dass sie mit ihrem Namen aufgerufen werden können anstatt über den Pfad `node_modules/.bin/package-name`.

```json
{
  "name": "my-package",
  "scripts": {
    "lint": "eslint ."
  },
  "devDependencies": {
    "eslint": "^5.16.0"
  }
}
npm run lint
```

## Exit Codes

Wenn ein Command Line Tool, welches man via `npm run`  ausführt mit einem non-zero Exit Code beendet wird, dann bricht auch der npm Prozess mit einen non-zero Exit Code ab. Dies ist zum einen  elementar in CI/CD Umgebungen und man kann es sich auch bei der  Verwendungen von Lifecycle-Scripts zu nutze machen.

## Lifecycle Scripts

npm hat vordefinierte [Lifecyle Scripts](https://docs.npmjs.com/misc/scripts), die unter bestimmten Voraussetzungen ausgeführt werden sofern sie in eurer `package.json` definiert sind.

```json
{
  "name": "my-package",
  "scripts": {
    "prepublishOnly": "snyk test",
    "postinstall": "echo 'Installed. Enter `npm run` to see available scripts.'"
  },
  "devDependencies": {
    "snyk": "^1.147.4"
  }
}
```

`prepublishOnly` wird automatisch ausgeführt, bevor euer Paket via `npm publish` in der npm Registry veröffentlicht bzw. aktualisiert wird. In dem  konkreten Beispiel werden eure Production Dependencies nach bekannten  Schwachstellen durchsucht und im Fall, dass etwas gefunden wird, der  Prozeß vor dem Veröffentlichen abgebrochen.

`postinstall` wird automatisch nach erfolgreicher Installation via `npm install` ausgeführt und kann bspw. genutzt werden, um Informationen auszugeben,  Shell Scripts ausführbar zu machen und Node.js Scripts auszuführen.  Eurer Kreativität sind hierbei keine Grenzen gesetzt.

In den [npm Docs](https://docs.npmjs.com/misc/scripts#description) könnt ihr eine komplette Liste der verfügbaren Lifecycle Scripts einsehen.

## »npm start« und »npm test«

Diese gehören ebenfalls in die Kategorie der Lifecycle Scripts, werden jedoch nicht automatisch ausgeführt.

```json
{
  "name": "my-package",
  "scripts": {
    "start": "node server.js",
    "test": "jest"
  },
  "devDependencies": {
    "jest-cli": "^24.7.1"
  }
}
```

Da es sich hierbei um Lifecycle Scripts handelt können sie ohne `run` wie folgt aufgerufen werden:

```bash
npm test
npm start
```

## »pre« und »post« Scripts

»pre« und »post« Scripts können genutzt werden um Scripts automatisch in einer bestimmten Reihenfolge auszuführen bzw. voneinander abhängig  zu machen.

```json
{
  "name": "my-package",
  "scripts": {
    "pretest": "eslint .",
    "test": "jest"
  },
  "devDependencies": {
    "eslint": "^5.16.0",
    "jest-cli": "^24.7.1"
  }
}
npm test
```

Dies Script linted eure Dateien via ESLint, bevor eure Tests  ausgeführt werden. Die Tests werden nicht ausgeführt, wenn  Linting-Fehler bestehen – oder allgemeiner ausgedrückt: das nachfolgende Script wird nicht ausgeführt, wenn eines der Scripts in einer Sequenz  mit »post« oder »pre« Scripts mit einem non-zero Exit Code beendet wird.

*Wichtig: »pre« und »post« Scripts können auch für eigene Custom Scripts genutzt werden. So wird z.B. mit Eingabe von `npm run build` auch `prebuild` und `postbuild` ausgeführt, wenn sie denn definiert sind.*

## Optionen für npm Scripts

### Optionen an verwendete Tools übergeben

Ihr könnt beim Aufruf von npm Scripts dem im jeweiligen Script verwendeten Tool via `-- --flag` Optionen mitgeben.

```json
{
  "name": "my-package",
  "scripts": {
    "lint": "eslint",
    "lint:fix": "npm run lint -- --fix",
  }
}
```

Der Aufruf von `npm run lint:fix` verhält sich also wie wenn man `eslint --fix` aufrufen würde.

### Unnötige Ausgaben im Terminal verringern

`npm run` verfügt über eine `--silent` Option,  die die Ausgabe im Terminal auf den Output der in npm Scripts  verwendeten Tools beschränkt. Dies ist insbesondere sinnvoll wenn npm  Scripts mit npm Scripts kombiniert werden.

Z.B. bei folgendem Setup in Sachen JavaScript Linting:

```json
{
  "name": "my-package",
  "scripts": {
    "lint": "xo",
    "lint:fix": "npm run lint --silent -- --fix"
  }
}
```

Aufruf ohne `--silent`:

```bash
$ npm run lint:fix

> my-package@1.0.0 lint:fix /Users/mkuehnel/temp/my-package
> npm run lint -- --fix

> my-package@1.0.0 lint /Users/mkuehnel/temp/my-package
> xo "--fix"
```

Aufruf mit `--silent`:

```bash
$ npm run lint:fix

> my-package@1.0.0 lint:fix /Users/mkuehnel/temp/my-package
> npm run lint --silent -- --fix
```

Auf Twitter findet ihr ein weiteres Beispiel dafür, was die Verwendung von `--silent` bei komplexeren Build-Setups für einen Unterschied machen kann:

<iframe id="twitter-widget-0" scrolling="no" allowtransparency="true" allowfullscreen="true" class="" style="position: static; visibility: visible; width: 550px; height: 707px; display: block; flex-grow: 1;" title="Twitter Tweet" src="https://platform.twitter.com/embed/Tweet.html?creatorScreenName=mkuehnel&amp;dnt=false&amp;embedId=twitter-widget-0&amp;features=eyJ0ZndfZXhwZXJpbWVudHNfY29va2llX2V4cGlyYXRpb24iOnsiYnVja2V0IjoxMjA5NjAwLCJ2ZXJzaW9uIjpudWxsfSwidGZ3X2hvcml6b25fdHdlZXRfZW1iZWRfOTU1NSI6eyJidWNrZXQiOiJodGUiLCJ2ZXJzaW9uIjpudWxsfSwidGZ3X3NwYWNlX2NhcmQiOnsiYnVja2V0Ijoib2ZmIiwidmVyc2lvbiI6bnVsbH19&amp;frame=false&amp;hideCard=false&amp;hideThread=false&amp;id=957965749473210369&amp;lang=de&amp;origin=https%3A%2F%2Flabs.micromata.de%2Fbest-practices%2Ftipps-und-helfer-fuer-npm-run-scripts-teil-1%2F&amp;sessionId=e6b18e67798f4db385f9486641b935546565ece1&amp;siteScreenName=micromata&amp;theme=light&amp;widgetsVersion=0a8eea3%3A1643743420422&amp;width=550px" data-tweet-id="957965749473210369" frameborder="0"></iframe>



## package.json Vars

Alle Properties der `package.json` Datei werden in Variablen mit dem Präfix `npm_package_` gespeichert. Die `name` Property ist somit als Variable `npm_package_name` verfügbar.

### Verwendung innerhalb der package.json Datei

Diese Variablen lassen sich via `$Variablenname` referenzieren.
 Somit kann man sich das z.B das wiederholte Schreiben der gleichen Pfade sparen. So ließe sich hieraus:

```json
{
  "name": "my-package",
  "scripts": {
    "lint": "eslint ./src/*",
    "test": "jest ./src/*"
  }
}
```

Folgendes machen:

```json
{
  "name": "my-package",
  "config": {
    "src": "./src/*"
  },
  "scripts": {
    "lint": "eslint $npm_package_config_src",
    "test": "jest $npm_package_config_src"
  }
}
```

### Verwendung innerhalb eigenen Codes

Wenn man Node.js Dateien via npm Scripts ausführt, lassen sich diese  Variablen sogar hier nutzen, da sie als Umgebungsvariablen in dem  Node.js Prozess verfügbar gemacht werden. Siehe folgendes kleines  Beispiel:

*package.json:*

```json
{
  "name": "my-package",
  "config": {
    "port": "8080"
  },
  "scripts": {
    "start": "node server.js"
  }
}
```

*server.js:*

```javascript
console.log(`Running on port ${process.env.npm_package_config_port}`);
```

*Terminal:*

```bash
$ npm start
Running on port 8080
```

Eine weitere Besonderheit bei Verwendung des config Key in der `package.json` ist, dass sich die Werte der Properties hierunter via `npm config` bedarfsweise überschreiben lassen:

```bash
$ npm config set my-package:port 80
$ npm start
Running on port 80
```

Das war der erste Teil des Blog Posts »Tipps und Helfer für npm run Scripts«. Im [zweiten Teil](/blog/2019-05-28-post-tipps-helfer-npm-scripts-teil-2/) wird es darum gehen, wie euch weitere Node.js Packages unterstützen können, um noch effektiver mit npm Scripts zu arbeiten.
