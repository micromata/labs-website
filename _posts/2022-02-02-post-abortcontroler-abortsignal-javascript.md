---
title: AbortController und AbortSignal in JavaScript
author: nmollenhauer
categories: [Quick Tips]
tags: [javascript]
shortdesc: Generalisiertes und vereinfachtes Abbrechen von Aufgaben 
featured: true
---

Um in JavaScript auf Events zu reagieren, kommt man man selten um die Funktion `addEventListener` herum. Hier gibt es seit einiger Zeit Abhilfe durch den [`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) und das [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal).

Möchte man auf mehreren Elementen einen `EventListener` hinzufügen, gestaltet sich der Code in etwa so:

```javascript
element1.addEventListener("click", () => console.log("element1 clicked"));
element2.addEventListener("click", () => console.log("element2 clicked"));
element3.addEventListener("click", () => console.log("element3 clicked"));
```

Möchte man die `EventListener` auch wieder entfernen, sind dafür einige Umstrukturierungen notwendig. Um einen Listener wieder zu  entfernen, muss die Referenz auf die Handler-Funktion vorher in eine  Variable ausgelagert werden:

```javascript
const element1Handler = () => console.log("element1 clicked");
const element2Handler = () => console.log("element2 clicked");
const element3Handler = () => console.log("element3 clicked");

element1.addEventListener("click", element1Handler);
element2.addEventListener("click", element2Handler);
element3.addEventListener("click", element3Handler);

// ...

element1.removeEventListener("click", element1Handler);
element2.removeEventListener("click", element2Handler);
element3.removeEventListener("click", element3Handler);
```

Diese Vorgehensweise kann bei vielen Elementen schnell unübersichtlich und wartungsunfreundlich werden. Hier hilft der `AbortController`. Mit ihm können gleich `EventListener` von beliebig vielen Elementen entfernt werden. Über eine Instanz des  AbortControllers kann dabei signallisiert werden, dass etwas  ,,abgebrochen“ werden soll. Ein `AbortController` kommt immer mit einem `AbortSignal`. Das `AbortSignal` wird dabei an die Funktion oder API übergeben, die abgebrochen werden können soll.

„Abbrechen“ bedeutet hier allerdings auch, dass damit `EventListener` entfernt werden können. [`addEventListener` hat einen dritten Parameter für weitere Optionen](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#parameters), worüber das Signal zum Abbrechen übergeben werden kann. Das obige Beispiel könnte so umgeschrieben werden:

```javascript
const controller = new AbortController();

const signal = controller.signal;

element1.addEventListener("click", () => console.log("element1 clicked"), {
    signal,
});
element2.addEventListener("click", () => console.log("element2 clicked"), {
    signal,
});
element3.addEventListener("click", () => console.log("element2 clicked"), {
    signal,
});

// Remove all event listeners at once
controller.abort();
```

Dadurch, dass jedes Element kein `removeEventListener` mehr benötigt, entfällt zusätzlich die Notwendigkeit, die Handler-Funktion in eine Variable auszulagern.

*Hierzu noch ein weiterer Tipp:* Soll nur ein einziges Mal auf ein Event reagiert werden, kann auch die `once`-Option bei `addEventListener` gesetzt werden. Dadurch wird der Handler nach einmaligem Aufruf  entfernt und ein manuelles Entfernen des Listeners entfällt komplett:

```javascript
element4.addEventListener("click", () => console.log("element4 clicked"), {
    once: true,
});
```

Mit dem `AbortController` lassen sich aber nicht nur `EventListener` entfernen. Auch das Abbrechen eines [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) ist damit möglich. Dabei wird das Signal einfach als Option im `fetch`-Aufruf übergeben. Auf diese Art lässt sich ein Timeout-Mechanismus für  fetch-Requests umsetzen, der die Anfrage tatsächlich abbricht. Wird der  Request abgebrochen, wird das Promise des Requests mit einer `DOMException` rejected:

```javascript
const controller = new AbortController();

setTimeout(() => controller.abort(), 10_000);

try {
    await fetch("https://developer.mozilla.org", {
        signal: controller.signal,
    });
} catch(e) {
    console.error(e);
}
```

Möchte man selbst einen Abbruch-Mechanismus für eine eigene Funktion implementieren, kann dies über das `AbortSignal` getan werden: [Das Signal eines AbortControllers](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) besitzt nicht nur die Eigenschaft `aborted`, die angibt, ob über den entsprechenden Controller ein Abbruch angefragt wurde. Zusätzlich besitzt das AbortSignal ein Event, das im Falle eines gewünschten Abbruchs ausgelöst wird. Auf diese Weise kann das [promisified `setTimeout` aus einem vorherigen Quick-Tip](https://labs.micromata.de/quick-tips/promisified-settimeout/) um einen generaliserten Abbruch-Mechanismus erweitert werden:

```javascript
function delay(duration = 0, signal = undefined) {
    return new Promise((resolve, reject) => {
        const timeoutId = setTimeout(resolve, duration);
        signal?.addEventListener(
            "abort",
            () => {
                clearTimeout(timeoutId);
                reject(new Error("delay aborted"));
            },
            { once: true },
        );
    });
}

// ...

async function myFunction() {
    console.log("Do something");
    const controller = new AbortController();
    
    // Simulate click on "abort" button
    setTimeout(() => controller.abort(), 500);

    await delay(10_000, controller.signal);
    console.log("Do something else");
}
```

Die Klassen `AbortController` und `AbortSignal` existieren schon länger und können [in folgenden Browsern](https://caniuse.com/abortcontroller) verwendet werden:

![AbortController & AbortSignal auf caniuse.com](/uploads/posts/caniuse-abortcontroller.png)
*AbortController & AbortSignal auf <a href="https://caniuse.com/abortcontroller">canisuse.com</a>*



