---
title: Tipps und Helfer für npm run Scripts (2/2)
author: mkuehnel
categories: [Best Practices]
tags: [npm]
shortdesc: Weitere Möglichkeiten zur Verbesserung eures Frontend Build-Setups.
permalink: /blog/2019-05-28-post-tipps-helfer-npm-scripts-teil-2/
featured: true
---

Im [ersten Teil](/blog/2019-04-10-post-tipps-helfer-npm-run-scripts-teil-1/) dieses zweiteiligen Artikels zum Thema npm Scripts ging es darum was  ihr mit vielleicht weniger bekannten Bordmitteln die npm mitliefert  erreichen könnt. Im heutigen, abschließenden Teil wollen wir third party Tools aus dem npm Ökösystem unter die Lupe nehmen, die das Potential  haben euch das Leben noch weiter zu vereinfachen.

## Watch files

Es gibt mehrere npm Pakete die es ermöglichen Dinge auszuführen  sobald sich Dateien ändern. Im Kontext npm scripts bevorzuge ich  persönlich [onchange](https://github.com/Qard/onchange), da es für diesen Use Case flexibel genug und dabei sehr einfach zu verwenden ist.

```json
{
  "name": "my-package",
  "scripts": {
    "lint": "eslint src",
    "lint:watch": "onchange \"src/**/*.js\" -- npm run lint --silent"
  }
}
```

`onchange` lauscht also auf Änderungen von JavaScript Dateien innerhalb des `src` Verzeichnisses und führt automatisch ESLint aus, sobald sich eine Datei ändert.

*Wer sich über das `--silent` wundert sollte sich nochmals den [ersten Teil](https://labs.micromata.de/best-practices/tipps-und-helfer-fuer-npm-run-scripts-teil-1/) dieses Artikels zu Gemüte führen.*

## Notifications im Fehlerfall

[cli-error-notifier](https://github.com/micromata/cli-error-notifier) zeigt native Desktop Notifications wenn npm scripts einen Fehler verursachen.

```json
{
  "scripts": {
    "lint": "onerror \"eslint src\""
  }
}
```

Das ist vor allem dann nützlich, wenn man während des codens in Verbindung mit `onchange` Dateien überwacht und sein Terminal im Hintergrund hat.

```json
{
  "scripts": {
    "lint": "eslint src",
    "lint:fix": "npm run lint --silent -- --fix",
    "lint:watch": "onchange \"src/**/*.js\" -- onerror \"npm run lint --silent\""
  }
}
```

## Mehrere Scripts parallel oder sequentiell ausführen

Die Verwendung von [npm-run-all](https://github.com/mysticatea/npm-run-all) bietet die folgenden Vorteile:

- Kann die Lesbarkeit des `scripts` des Bereichs in der `package.json` mit Hilfe glob-like Patterns erhöhen. Vorher:

  ```bash
  npm run clean && npm run build:css && npm run build:js && npm run build:html
  ```

  Nachher:

  ```bash
  npm-run-all clean build:*
  ```

- Im Vergleich zur Verwendung von

  ```bash
  npm run webpack:server & npm run lint:watch
  ```

  - funktioniert die parallele Ausführung auch unter Windows
  - können mehrere Scripts die parallel laufen ihren Output ins Terminal streamen

  ```json
  {
    "scripts": {
      "start": "npm-run-all clean --parallel webpack:server lint:watch",
      "build": "npm-run-all security test clean webpack"
    }
  }
  ```

   

## npm Scripts merken

Abhängig von der Anzahl an Tasks kann es schwer sein sich jeden einzelnen Task Namen zu merken. Eine besserer Alternative zu `npm run` zum Auflisten der verfügbaren Tasks ist ein kleines Tools Namens [ntl](https://github.com/ruyadorno/ntl). Dieses zeigt eine interaktive Liste der npm Scripts durch die man mit der Tastatur scrollen und auswählen kann 💥


![ntl](/uploads/posts/ntl.png)


## Beyond npm Scripts

### Tasks ausserhalb der package.json definieren

Die beiden größten Mankos bei Verwendung von npm Scripts für komplexere Build Setups sind:

- Keine Möglichkeit die Tasks mit Infos für anwendende Entwickler zu versehen.
  - Negative Auswirkung auf die Developer Experience.
- Keine Möglichkeit die Tasks mit Kommentaren zu versehen.
  - JSON unterstützt nun mal keine Kommentare.

Als Workaround für beide Punkte kann man ein Tool wie [nps](https://github.com/kentcdodds/nps) verwenden, welches verspricht alle Vorteile von npm Scripts zu bieten ohne die `package.json` aufzublähen und die Limitierungen von JSON als Dateiformat.

#### Enter nps

`nps` ermöglicht die Scripts in eine Datei Namens `package-scripts.js` zu portieren. Da es sich hierbei um eine gewöhnliche JavaScript Datei  handelt kann man hier sehr viel mehr mit seinen Build Tasks  veranstalten.

Nachfolgende ein einfaches Beispiel Setup:

```javascript
const npsUtils = require('nps-utils'); // not required, but handy!

module.exports = {
  scripts: {
    default: 'node index.js',
    lint: 'eslint .',
    test: {
      // learn more about Jest here: https://facebook.github.io/jest
      default: 'jest',
      watch: {
        script: 'jest --watch',
        description: 'run in the amazingly intelligent Jest watch mode'
      }
    },
    build: {
      // learn more about Webpack here: https://webpack.js.org/
      default: 'webpack',
      prod: 'webpack -p',
    },
    // learn more about npsUtils here: https://npm.im/nps-utils
    validate: npsUtils.concurrent.nps('lint', 'test', 'build'),
  }
};
```

Wenn das Setup steht, kann man sich mit `nps help` Infos zu den verfügbaren Tasks anzeigen lassen oder Tasks wie folgt aufrufen.

```bash
nps lint
```

»Sub-Tasks« lassen sich folgendermaßen aufrufen:

```bash
nps test.watch
```

Eine ausführliche Dokumentaion zu nps findet ihr auf GitHub: https://github.com/kentcdodds/nps

### npx

`npx` wird als weiteres Binary mit npm ausgeliefert (seit v5.2.0).

Mit `npx` lassen sich nicht nur global Dependencies aus  dem npm Öko-System verwenden ohne sie zu installieren zu müssen sondern  es ist auch sehr nützlich für lokal im Projekt installierte  Dependencies:

*Each command called with `npx` is executed either from the local `node_modules/.bin` directory, or from a central cache, installing any packages needed in order to run a command.*

```json
{
  "name": "my-package",
  "scripts": {
    "lint": "xo"
  },
  "devDependencies": {
    "xo": "^0.20.0"
  }
}
npx xo --fix
```

Dieser Befehl führt demnach das lokal unter `node_modules/.bin`. installierte Binary von `xo` aus.


Das war es fürs Erste. Weitere Tipps und Helfer nehmen wir gerne via [Twitter](https://twitter.com/micromata) entgegen 🐦
