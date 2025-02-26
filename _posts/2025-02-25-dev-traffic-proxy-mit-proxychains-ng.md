---
title: Development Traffic Proxy mit ProxyChains-NG
author: sfast
categories: [Quick Tips]
tags: [tools, dev, proxychains]
shortdesc: Leite den gesamten Traffic einer Anwendung mit proxychains-ng durch einen Proxy.
---

**Schneller testen, weniger deployen**. 
Anstatt auf die Pipeline zu warten, einfach `proxychains-ng` verwenden, um lokale Anwendungen direkt mit entfernten Services zu verbinden.
Mit `proxychains-ng` lässt sich der gesamte Traffic eines Programms über einen Proxy umleiten.

Anhand eines Kubernetes-Clusters wird demonstriert, wie lokale Anwendungen auf Services im Cluster zugreifen können – ohne Warten auf die Pipeline und aufwändiges Deployment.

## Einführung

Beim Entwickeln von Anwendungen, die auf externe Systeme wie APIs, Datenbanken oder Message Queues (beispielsweise Kafka) zugreifen, sind lokale Tests oft problematisch. 
`kubectl port-forward` hilft in vielen Fällen, aber nicht immer – insbesondere, wenn Dienste zusätzliche DNS-Namen nutzen, die lokal nicht auflösbar sind.

Hier kommt **`proxychains-ng`** ins Spiel. 
Es leitet den gesamten Traffic einer lokalen Anwendung durch einen Proxy im Cluster und ermöglicht so den direkten Zugriff auf entfernte Services.

Ein einfaches Beispiel zeigt, wie ein `curl`-Befehl direkt auf einen Service im Kubernetes-Cluster zugreift. 
Dieselbe Technik funktioniert auch für komplexere Szenarien: 
Eine lokal gestartete Java-Anwendung mit einem Kafka-Consumer kann sich nahtlos mit einem Kafka-Broker im Cluster verbinden und dort Nachrichten konsumieren – ohne jedes Mal ein Docker-Image bauen und deployen zu müssen.
Selbst wenn ein Broker über Service Discovery zusätzliche DNS-Namen bereitstellt, die normalerweise nur innerhalb des Clusters auflösbar sind, werden auch diese Verbindungen über `proxychains-ng` nahtlos umgeleitet.

## Setup

### Installation der Tools

- [`proxychains-ng`](https://github.com/rofl0r/proxychains-ng)
  - `brew install proxychains-ng`
  - Leitet den Traffic von lokalen Anwendungen durch einen Proxy.
  - Nach der Installation ist `proxychains4` als Befehl verfügbar.
- [`shadowsocks-rust`](https://github.com/shadowsocks/shadowsocks-rust)
  - `brew install shadowsocks-rust`
  - Ein schneller und sicherer Proxy-Server und -Client.

**Optional**:
- [`k3d`](https://k3d.io/): `brew install k3d`
  -  Zum Erstellen eines lokalen Kubernetes-Clusters, um die Beispiele auszuprobieren.

### Konfiguration von `proxychains-ng`

Nach der Installation von `proxychains-ng` muss folgende Konfigurationsdatei angelegt werden:

Dateipfad: `~/.proxychains/proxychains.conf`
```conf
dynamic_chain
proxy_dns
tcp_read_time_out 15000
tcp_connect_time_out 8000

[ProxyList]
socks5  127.0.0.1 1080
```
- **`dynamic_chain`**: Verwendet die Proxys in der angegebenen Reihenfolge.
- **`proxy_dns`**: Sorgt dafür, dass DNS-Anfragen ebenfalls durch den Proxy gehen.
- **Timeouts:** Die TCP-Timeouts sind in Millisekunden angegeben.

## Verwendung

### Deployment des Proxy-Servers im Kubernetes-Cluster

Im Kubernetes-Cluster muss ein **Shadowsocks-Proxy**-Server bereitgestellt werden.
Dafür kann die folgende Pod Definition als Vorlage verwendet werden:

**`shadowsocks.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shadowsocks-proxy
  labels:
    app: shadowsocks-proxy
spec:
  containers:
  - name: shadowsocks
    image: ghcr.io/shadowsocks/ssserver-rust:latest
    command: ["/usr/bin/ssserver"]
    args:
      - "-s"
      - "0.0.0.0:4444"
      - "-k"
      - "my-shadowsocks-password"
      - "-m"
      - "aes-256-gcm"
      - "-U"
      - "--tcp-fast-open"
    ports:
    - containerPort: 4444
```

- Der Pod startet einen **Shadowsocks-Proxy** auf Port 4444.
- Mit `-U` wird UDP (für DNS Anfragen) aktiviert.
- `my-shadowsocks-password` sollte durch ein sicheres Passwort ersetzt werden.

Der Pod wird mit folgendem Befehl im Cluster deployed:
```bash
kubectl apply -f shadowsocks.yaml
```

### Verbindung zum Proxy herstellen

1. **Port-Forwarding** vom Kubernetes-Cluster auf die lokale Maschine:
```bash
kubectl port-forward shadowsocks-proxy 4444:4444
```
2. **Shadowsocks-Client lokal starten:**
```bash
sslocal -b 127.0.0.1:1080 -s 127.0.0.1:4444 -k my-shadowsocks-password -m aes-256-gcm -U --tcp-fast-open
```
- Der Client läuft auf **Port 1080** und leitet Anfragen an den Shadowsocks-Server im Cluster weiter.
- Das Passwort muss mit dem in der Pod-Definition festgelegten Passwort übereinstimmen.

### Traffic durch den Proxy leiten

Im Folgenden werden einige Beispiele gezeigt, wie der Traffic von lokalen Anwendungen durch den Proxy geleitet werden kann.

#### Einzelne Befehle mit `proxychains-ng`

Indem man `proxychains4` vor einen Befehl setzt, wird der Traffic des Befehls abgefangen und durch den Proxy geleitet:
```bash
proxychains4 curl http://some-kubernetes-service:8080
```
- Die Anfrage an den Kubernetes-Service wird durch den Proxy im Cluster geleitet.
- Da auch DNS-Anfragen durch den Proxy gehen, können auch Hostnamen, die eigentlich nur im Cluster auflösbar sind, verwendet werden.
- **Achtung**: Siehe die weiter unten genannten [Limitierungen](#limitierungen) für macOS bei vorinstalliertem `curl`.

#### Gesamte Shell-Session mit Proxy

Wird eine Shell-Session über `proxychains4` gestartet, so wird der gesamte Traffic für alle Befehle in dieser Shell durch den Proxy geleitet:
```bash
proxychains4 zsh
```

In dieser Shell wird der Traffic nun auch dann durch den Proxy geleitet, wenn kein `proxychains4` vor den Befehlen steht:
```bash
curl http://some-kubernetes-service:8080
```

#### Eigene Anwendungen durch den Proxy starten

Eine in Java oder Kotlin geschriebene Spring Boot Anwendung mit dem Build-Tool Gradle kann beispielsweise wie folgt durch den Proxy gestartet werden:
```bash
proxychains4 ./gradlew bootRun
```
Analog für Maven:
```bash
proxychains4 mvn spring-boot:run
```

## Limitierungen
- **macOS:** Vorinstallierte System-Tools wie `curl` funktionieren nicht, weil `proxychains-ng` aufgrund von SIP (System Integrity Protection) keine Bibliotheken einhängen kann.
  - Viele vorinstallierte Tools wie `curl` lassen sich mit `brew` (`brew install curl`) installieren und sind dann nicht von SIP betroffen.
- **Statisch gelinkte Binaries** funktionieren nicht, da `proxychains-ng` die `libc`-Bibliothek nutzt.
  - Für die meisten Anwendungen ist dies kein Problem.
    - Kotlin, Java, Rust, Node, Python, Ruby und viele andere Sprachen erzeugen Anwendungen, die `libc` verwenden und dynamisch gelinkt sind.
  - In **Go** entwickelte Anwendungen sind normalerweise statisch gelinkt. 
    Mit bestimmten Einstellungen können jedoch dynamisch gelinkte Binaries erstellt werden:
    - Entweder durch Hinzufügen folgendes Imports:
      ```go
      import "C"
      ```
    - Oder durch Hinzufügen dieses Build-Flags beim Bauen:
      ```bash
      go build -ldflags='-linkmode external'
      ```

## Ausblick

Es existiert ein weiteres cooles Tool, das nicht von den [Limitierungen](#limitierungen) von `proxychains-ng` betroffen ist: [`cproxy`](https://github.com/NOBLES5E/cproxy).
Es nutzt `cgroup` und `iptables`, um den gesamten Netzwerkverkehr über einen Proxy zu leiten. 
Da beide Technologien Linux-spezifisch sind, ist `cproxy` derzeit **ausschließlich für Linux-Systeme** verfügbar.

Wenn der Hauptanwendungsfall das Leiten des Traffics einer Anwendung von der lokalen Maschine in und aus einem Kubernetes Cluster ist, dann eignet sich das Tool [`mirrord`](https://github.com/metalbear-co/mirrord).
`mirrord` automatisiert viele der bei `proxychains-ng` manuell durchzuführenden Schritte und ist speziell für den Einsatz in Kubernetes-Clustern entwickelt.
Das Tool läuft auf allen gängigen Betriebssystemen und ermöglicht unter anderem das Spiegeln des Traffics eines Kubernetes-Pods in eine lokale Anwendung.

## Fazit
Mit `proxychains-ng` lässt sich der Traffic von lokalen Anwendungen schnell und einfach über einen Proxy in entfernte Systeme leiten.
Das spart Zeit beim Entwickeln und Testen – ohne langwieriges Warten auf die Pipeline und Deployments. 

---

Wenn du weitere Use Cases oder Tipps zu `proxychains-ng` und ähnliche Tools kennst, lass es uns gerne wissen! 🚀
