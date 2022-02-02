---
title: Promisified setTimeout
author: mkuehnel
categories: [Quick Tips]
tags: [javascript]
shortdesc: Lesbarerer Code mit Promises und Async/Await.
featured: true
---

Man benötigt `setTimeout` nicht oft. Ein durchaus valider Anwendungsfall ist aber z.B. aufgrund von Timing Problemen einen [Event Loop](https://vimeo.com/254947206) zu skippen und seinen Code erst im nächsten Event Loop ausführen zu lassen. Da `setTimeout` eine Callback Funktion erwartet, wird der Code dadurch aber verschachtelter und dadurch schwerer lesbar.

Seit [async/await](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9) in JavaScript Einzug gehalten hat und wir es dank [Babel](https://babeljs.io/) in allen Browsern verwenden können, lässt sich die sogenannte Callback Hell umgehen. Durch Verwendung von `async` Funktionen und dem `await` Schlüsselwort lässt sich asynchroner Code schreiben, der fast wie  synchroner Code aussieht. Einzige Voraussetzung: Wir brauchen eine  Funktion, die ein Promise zurückgibt, da async/await letztlich nur eine  alternative Schreibweise darstellt, um auf die Erfüllung eines Promise  zu warten.

```javascript
// Promisified setTimeout
const delay = (duration = 0) =>
  new Promise(resolve =>
    setTimeout(() => {
      resolve();
    }, duration)
  );
```

Per default setzen wir also `setTimeout` auf `0` Millisekunden. Verwenden lässt sich unser promisified `setTimeout` dann wie folgt.

```javascript
// Usage
async function myFunction() {
  console.log('Do something');
  await delay();
  console.log('Do something else');
}
```

PS: Natürlich gibt es das Ganze auch als npm Module von verschiedenen Autoren, z.B.: https://github.com/sindresorhus/delay
