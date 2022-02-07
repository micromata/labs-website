---
title: Docker Image ohne Registry verteilen
author: cclaus
categories: [Quick Tips]
tags: [docker]
shortdesc: Docker bietet die Möglichkeit ein oder mehrere Images in ein tar-Archiv zu speichern und wieder zu laden. Hier siehst du wie. ere sinnvoll wenn npm  Scripts mit npm Scripts kombiniert werden.
---

Es kann Situationen geben, in denen man ein Docker Image von einem Rechner zum anderen bewegen möchte. Zum Beispiel dann, wenn der  Zielrechner keine Anbindung an eine private Registry hat und die  öffentlichen Registries keine Alternative sind.

Docker bietet die Möglichkeit ein oder mehrere Images in ein  tar-Archiv zu speichern und wieder zu laden. Die Befehle dazu lauten:

```bash
docker save [OPTIONS] IMAGE [IMAGE...]
docker load [OPTIONS]
```

Kombiniert man die beiden Befehle jetzt und sorgt noch für Kompression, kann man bequem und performant seine lokalen Images auf ein Zielsystem schieben.

```bash
docker save "hello-world" | bzip2 | ssh user@myserver.com 'bunzip2 | docker load'
```

Cheers!
