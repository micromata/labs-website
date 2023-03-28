---
title: JSConf Budapest 2017
author: mkuehnel
categories: [Konferenzen]
tags: [konferenzen]
shortdesc: Mit der JSConf Budapest gibt es neben der  JSConf EU in Berlin (der erste JSConf auf europäischem Boden) und der  JSConf Belgium eine weitere JSConf auf dem europäischen Festland die  2017 bereits zum dritten Mal stattgefunden hat und somit schon eine  gewisse Tradition vorzuweisen hat.
---

Für diejenigen die das Konzept der JSConf nicht kennen nachfolgend eine kurze Erklärung:

Die inzwischen weltweit stattfinden JSConfs sind allesamt  community-getriebene Veranstaltungen (von der lokalen Community für die  weltweite Community). Es steht also kein großer gemeinsamer Veranstalter hinter all den Konferenzen. Allerdings haben sie durchaus  Gemeinsamkeiten was durch den Verbund (http://jsconf.com) nicht nur  möglich, sondern auch stark macht.

Alle lokalen Veranstalter setzen sich die das Ziel das die  Veranstaltungen nicht nur technologie getrieben sind, sondern auch den  menschlichen Aspekt der Community den Schwerpunkt bilden. Das wird  erreicht, indem viele Möglichkeiten der sozialen Interaktion geschaffen  werden, von Lounges, zu Diskussionsrunden und zu interaktiven Spielen  auf den Abend-Veranstaltungen. Alles unter Berücksichtigung eines Code  of conducts, der nicht nur irgendwo auf der Website veröffentlicht,  sondern strikt gelebt wird.

Auch auf die Diversity sowohl im Line-up der Speaker, als auch unter  den Besuchern wird großer Wert gelegt und es z.B. durch spezielle  Diversity support Tickets deren Aufpreis verwendet wird  unverhältnismäßig schwach vertreten Personen (Frauen, People of Color  und weiteren unterrepräsentierten Gruppen in der Branche) den Besuch zu  ermöglichen.

Ich habe 2017 zum ersten Mal die JSConf Budapest besucht, nachdem er  vor einigen Jahren schon mal die JSConf EU in Berlin. Dementsprechend  war ich sehr gespannt was mich erwartet.

### Zur Location
Veranstaltungsort war dieses Jahr ein historisches Kino im Herzen des jüdischen Viertels mitten im Stadtzentrum. Eine imposante Location.

### Zum Programm
Die Konferenz streckte sich über 2 Tage und war eine Single Track  Konferenz. Man musste also keine Angst haben im falschen Track zu sitzen und gute Vorträge zu verpassen 😉

Die Mischung an Talks war für meinen Geschmack perfekt. Die  technischen Vorträge haben überwogen und es waren durchaus sehr  interessante Vorträge bei denen die Technik nicht im Vordergrund stand.

**Meine persönlichen Highlights der nicht technischen Vorträge waren die folgenden:**

#### You Have Nothing to Lose but Your Chains

Von [Bodil Stokke](https://twitter.com/bodil).

Thema des [Talks](https://bodil.lol/join-us-now/#0) war  die Open Source Bewegung. Von ihren Anfängen, über Lizenzen bis zu ihrer Bedeutung in der heutigen Zeit. Unabhängig davon wie man Open Source  und die Free Software Bewegung interpretiert: Sozialismus, Pragmatismus  oder Kunst. Die Idee und deren Verbreitung ist sehr wichtig. Denn sie  gibt uns Kraft. Open Source sorgt durch die Offenlegung von Quellcode  nicht nur dafür, dass wir Software beeinflussen, reparieren und  verbessern können. Das kann Communities (von Entwickler:innen und Nutzer:innen)  bilden die anders gar nicht zu denken wären. Vor allem aber bietet es  die Chance zu lernen.

#### Goldilocks and the three Code Reviews

Von [Vaidehi Joshi](https://twitter.com/vaidehijoshi)

Der [Talk](http://slides.com/vaidehijoshi/better-code-reviews#/) hat – in eine unterhaltsame Geschichte verpackt – Probleme mit Code Reviews aufgezeigt. Darunter:

- Zu viele Kommentare aber wenig Konversation.
- Fokussierung auf Codestyle/Syntax.
- Ego wichtiger als Empathie.

und konkrete Handlungsmöglichkeiten zum Beheben der Probleme gegeben:

- Code von jedem Reviewen. Auch Senior-Entwickler*innen:  Für eine Kultur sorgen, die Kritik als wertvoll erachtet.
- Empathie entwickeln: Auch die positiven Sachen hervorheben
- Iterieren. Nur durch Wiederholung kann man einen Prozess verbessern
- Konservationen führen

#### Impostor syndrome – am I suffering enough to talk about it?

[Madeleine Neumann](https://twitter.com/maggysche)

Madeleine hat in diesem wichtigen psychologischen Talk die persönliche Geschichte ihres Werdegangs erzählt in dem das [Impostor Syndrome](https://medium.com/@Maggysche/impostor-syndrome-am-i-suffering-enough-to-write-about-it-5de8074f2c60) immer eine Rolle gespielt hat. Ein für mich wichtigste Punkt des Talks: Jeder hat Phasen oder Zeitpunkte in denen er unter Selbstzweifeln leidet.  Zuletzt bietet Madeleine konkrete Ideen für Handlungsanweisungen wie man sich aus der Impostor Zone befreien kann. Siehe [Slides](https://www.slideshare.net/MadeleineNeumann/jsconf-budapest-impostor-syndrome-am-i-suffering-enough-to-talk-about-it).

**Meine Favoriten unter den technischen Talks waren die folgenden:**

#### Watch your back, Browser! You’re being observed

Von [Stefan Judis](https://twitter.com/stefanjudis)

Ein [Roundtrip](https://speakerdeck.com/stefanjudis/watch-your-back-browser-youre-being-observed) über moderne JavaScript Web APIs welche die grundlegende Art der  Informationsbeschaffung auf den Kopf stellen. So baute das  Kommunikationskonzept bei älteren APIs meist auf einem Pull-Prinzip auf. Wollte man Informationen haben musste man vereinfacht gesagt danach  fragen. Dies scheint sich mit modernen APIs zu einem Push-Prinzip zu  verändern.

Kurz erklärt am Beispiel der recht neuen [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API). Sie bietet die Möglichkeit zu »lauschen« ob sich ein DOM-Objekt in  sichtbaren Bereich des Browser Viewports befindet und sich falls das  zutrifft asynchron benachrichtigen zu lassen.

Prominenteste Use Cases sind vermutlich das nachladen von Bildern  sowie die Implementierung eines »infinite scrolling« als Alternativer zu einer klassischen Pagination. Dies war natürlich auch schon mit anderen Mitteln zu realisieren, allerdings deutlich weniger performant.

#### Async patterns to scale your multicore JavaScript … elegantly.

Von [Jonathan Martin](https://twitter.com/nybblr)

Ein hervorragender [Talk](https://speakerdeck.com/nybblr/async-patterns-to-scale-your-multicore-javascript-dot-dot-dot-elegantly) zu Programmierparadigmen um die Herausforderung von parallelen Abläufen  mit sequentiellen Abhängigkeiten mit JavaScript zu lösen. JavaScript  unterstützt von Hause aus erstmal kein multithreading, bietet aber mit  der zugrundeliegenden Architektur auf Basis des Event Loops, des Call  Stacks und des Callback Queues die beste Basis für asynchrone  Programmierung. Jonathon lieferte mit den Punkten »Async IIFEs«, »Web  Worker clusters« und »SharedArrayBuffers« Rezepte für elegante Lösungen  für concurrency Herausforderungen.

#### JavaScript Metaprogramming – ES6 Proxy Use and Abuse

Von [Eirik Vullum](https://twitter.com/eiriklv)

Interessanter Talk mit Real world use cases für eines der wenigen ES6 features welches sich zwar weder transpilen, noch polyfillen lässt aber in modernen Browsern und Node.js Versionen durchaus heute nutzbar ist.

Proxies kurz erklärt: Proxies ermöglichen es JavaScript grundlegende  Sprachfeatures zu erweitern bzw. zu verändern. Z.B. Zugriff auf  Objekt-Eigenschaften, Wert-Zuweisungen, Funktionsaufruf usw.

Neben durchaus sinnvollen Mini-Beispielen wie automatischem logging  von Objekt-Keys bei Zugriff auf diese, überzeugte mich vor allen ein  Open Source Projekt von Eirik: [JSON-populate](https://github.com/eiriklv/json-populate), das auf Basis von Proxies den Zugriff auf via Referenz verknüpfte JSON-Objekte stark vereinfacht.

**Abendveranstaltungen**

Ein ganz besonderes Highlight neben den Talks war die Party am Abend des ersten Konferenztages. [{ Live : JS }](http://livejs.network/) hatte einen großartigen Auftritt. Es gab Live Musik mit Hilfe von zwei  Gameboy Advance und visuelle Unterstützung ausschließlich auf Basis von  Webtechnologien.

### Fazit

Neben den Talks hat mich vor allem den soziale Aspekt dieses  Konferenzbesuches begeistert. Ich habe viele alte Freunde getroffen die  man maximal einmal im Jahr trifft und sonst nur online sieht und einige  neue Menschen zu Twitter-Profilen kennenlernen dürfen. Vielen Dank für  die netten Gespräche und die Offenherzigkeit an alle vielen Dank an die  Organisatio der JSConf Budapest und ein ganz besonderer Dank an  Micromata für die Möglichkeit die Konferenz besuchen zu können.
