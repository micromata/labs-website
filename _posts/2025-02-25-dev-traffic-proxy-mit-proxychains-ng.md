---
title: Development Traffic Proxy mit ProxyChains-NG
author: sfast
categories: [Quick Tips]
tags: [tools, dev, proxychains]
shortdesc: TBD
---

Dieser Blogpost zeigt, wie mit `proxychains-ng` der gesamte Traffic eines Programms durch einen Proxy geleitet werden kann. 
Anhand eines Kubernetes-Clusters wird demonstriert, wie lokale Anwendungen auf Services im Cluster zugreifen können – ohne Warten auf die Pipeline und aufwändiges Deployment.

## Einführung

Beim Entwickeln von Anwendungen, die auf externe Systeme wie APIs, Datenbanken und Message Queues wie Kafka zugreifen, sind lokale Tests und Debugging gegen diese externen Systeme oft umständlich. 
Ein `kubectl port-forward` reicht nicht immer aus, da viele Systeme zusätzliche DNS-Namen verwenden, die lokal nicht aufgelöst werden können. 
Hier hilft **`proxychains-ng`**, indem es den Traffic der lokalen Anwendung durch einen Proxy im Cluster leitet.

Dieser Post zeigt ein Beispiel mit einem **Kafka Consumer**, der lokal entwickelt wird, aber Nachrichten aus einem Kubernetes-Cluster konsumiert. 
Dank `proxychains-ng` kann die Anwendung direkt auf den Kafka-Service im Cluster zugreifen – ohne jedes Mal ein Docker-Image bauen und deployen zu müssen.

## Setup

### Installation der Tools

- [`proxychains-ng`](https://github.com/rofl0r/proxychains-ng) 
  - Leitet den Traffic von lokalen Anwendungen durch einen Proxy.
  - Der nach der Installation verfügbare Befehl heißt `proxychains4`.
- [`shadowsocks-rust`](https://github.com/shadowsocks/shadowsocks-rust) 
  - Ein schneller und sicherer Proxy-Server und -Client.

**Optional**:
- [`k3d`](https://k3d.io/) 
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
- **Timeouts:** TCP-Timeouts in Millisekunden.

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
- Anstelle von `my-shadowsocks-password` sollte ein sicheres Passwort verwendet werden.

Der Pod kann dann wie folgt im Cluster deployed werden:
```bash
kubectl apply -f shadowsocks.yaml
```

### Verbindung zum Proxy herstellen

1. **Port-Forwarding** von Kubernetes auf die lokale Maschine:
```bash
kubectl port-forward shadowsocks-proxy 4444:4444
```
2. **Shadowsocks-Client lokal starten:**
```bash
sslocal -b 127.0.0.1:1080 -s 127.0.0.1:4444 -k my-shadowsocks-password -m aes-256-gcm -U --tcp-fast-open
```
- Der Client lauscht lokal auf **Port 1080** und leitet Anfragen an den Shadowsocks-Server im Cluster weiter.
- Das Passwort muss mit dem im Pod definierten Passwort übereinstimmen.

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
```bash
proxychains4 ./my-kafka-consumer
```
- Der Kafka-Consumer verbindet sich jetzt direkt mit dem Kafka-Cluster im Kubernetes-Cluster – ohne zusätzliches Deployment.

## Limitierungen
- **macOS:** Vorinstallierte System-Tools wie `curl` funktionieren nicht, da `proxychains-ng` aufgrund von SIP (System Integrity Protection) die Bibliotheken nicht einhängen kann.
  - Viele vorinstallierte Tools wie `curl` können jedoch auch beispielsweise mit `brew` (also zum Beispiel `brew install curl`) installiert werden und werden dann nicht durch SIP beeinträchtigt. 
- **Statisch gelinkte Binaries** funktionieren nicht, da `proxychains-ng` die `libc`-Bibliothek nutzt.
  - Für die meisten Anwendungen ist dies kein Problem.
    - Kotlin, Java, Rust, Node, Python, Ruby und viele andere Sprachen erzeugen Anwendungen, die `libc` verwenden und dynamisch gelinkt sind.
  - In **Go** entwickelte Anwendungen sind standardmäßig statisch gelinkt. Es gibt jedoch Möglichkeiten so zu bauen, dass dynamisch gelinkt Binaries erzeugt werden:
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
`cproxy` verwendet `cgroup` und `iptables`, um den Traffic durch einen Proxy zu leiten.
`cgroup` und `iptables` sind Linux Features und daher ist `cproxy` aktuell **nur für Linux-Systeme** verfügbar.

Wenn der Hauptanwendungsfall das Leiten des Traffics einer Anwendung von der lokalen Maschine in und aus einem Kubernetes Cluster ist, dann eignet sich das Tool [`mirrord`](https://github.com/metalbear-co/mirrord).
`mirrord` automatisiert viele der bei `proxychains-ng` manuell durchzuführenden Schritte und ist speziell für den Einsatz in Kubernetes-Clustern entwickelt.
Das Tool ist für alle gängigen Betriebssysteme verfügbar und bietet viele nützliche Features, wie zum Beispiel das Spiegeln von Traffic eines Pods im Kubernetes Cluster in eine Anwendung auf der lokalen Maschine.

## Fazit
Mit `proxychains-ng` lässt sich der Traffic von lokalen Anwendungen schnell und einfach über einen Proxy in entfernte Systeme leiten.
Das spart Zeit beim Entwickeln und Testen – ohne langwierige Warten auf die Pipeline und Deployments. 

---

Wenn du weitere Use Cases oder Tipps zu `proxychains-ng` kennst, lass es uns gerne wissen! 🚀
