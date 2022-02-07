---
title: Schlanke Docker-Layer in Spring-Boot
author:
categories: [Best Practices]
tags: [docker]
shortdesc: Damit mein docker push nicht mehr so lange dauert…
---

Immer mehr Anwendungen werden in Docker-Containern deployed und  ausgeführt. Zur Speicheroptimierung besitzt Docker dabei ein  Layer-Konzept, d.h. nach jedem Befehl in einem Dockerfile wird ein  separates Layer erzeugt. Zusätzlich vermindert dieses Konzept die Menge  an zu übertragenen Layern an einen anderen Host, auf dem das Image als  Container laufen soll.

Spring Boot bietet neben viele anderen  Annehmlichkeiten als vollumfassendes Application Framework auch die  Möglichkeit, ein sog. Fat-Jar zu erstellen, d.h. sowohl mein  Applikationscode als auch alle benötigten Java-Bibliotheken werden als  einzelnes JAR gepackt. Auf der einen Seite ermöglicht dieses komfortable Paketieren einfache Deployments, auf der anderen Seite führt dies dazu, dass selbst einfache Änderung stets die Übertragung des gesamten  JAR-Layers nach sich ziehen.

In diesem Artikel stelle ich den in der [Spring-Dokumentation](https://spring.io/guides/gs/spring-boot-docker) vorgestellten Ansatz anhand eines Beispiels vor und belege mit  Messungen den spürbaren Geschwindigkeits- und Platzvorteil, den wir  durch ein geschickteres Layering erreichen.

Eine minimale aber komplette Beispielapplikation findet sich auf [GitHub](https://github.com/micromata/spring-slim-docker-images).

# Was habe ich wirklich davon?

Bevor ich mit der Erklärung zur Implementierung loslege, zeige ich  zunächst anhand eines kleinen Beispiels, dass sich dieser — in den  meisten Fällen — kleine Umbau wirklich lohnt. In unserem Demo-Szenario  wollen wir eine minimale Spring-Applikation deployen, d.h. keine  Controller, Views, etc. Entsprechend schlank ist der dependency-Baum in  der pom.xml:

```bash
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
  </dependency>
</dependencies>
```

Ein vollständiger Maven-Build führt zu einem paketieren JAR von ca. 8 MB. Dabei haben wir essentielle Spring-Bibliotheken wie z. B. web oder  jpa noch nicht einmal mit inkludiert! In einem klassischen Docker-Build  wird das JAR über eine ADD Anweisung der Form

```bash
ADD target/demo-0.0.1-SNAPSHOT.jar /app/demo-0.0.1-SNAPSHOT.jar
```

hinzugefügt. Bei jeder Codeänderung wird dieses 8MB+ Layer entsprechend auch vollständig neu übertragen:

```bash
# Der Push ging so schnell, dass ich davon kein Copy&Paste machen konnte ;-)
The push refers to repository [docker.io/mlesniak/spring]
d6d52e4ca19c: Pushing [==========================================> ] 6.784MB/7.538MB
714e1dd7c2e4: Layer already exists
344fb4b275b7: Layer already exists
bcf2f368fe23: Layer already exists
```

In dem hier vorgestellten Ansatz separieren wir Bibliotheken und unseren eigenen Code in unterschiedlichen Layer. Während der **initiale** Upload also genauso lange dauert, d.h.

```bash
2e4b6d736fb0: Pushed
9fcb602b6691: Pushed
09a69bdbb3a5: Pushing [=========> ] 1.46MB/7.44MB
714e1dd7c2e4: Layer already exists
344fb4b275b7: Layer already exists
bcf2f368fe23: Layer already exists
```

werden bei **zukünftigen** Änderungen jeweils nur unsere eigentlichen (kleinen) Änderungen übertragen:

```bash
The push refers to repository [docker.io/mlesniak/spring]
826bed412cb0: Pushed
9fcb602b6691: Layer already exists
09a69bdbb3a5: Layer already exists
714e1dd7c2e4: Layer already exists
344fb4b275b7: Layer already exists
bcf2f368fe23: Layer already exists
```

Statt **8 MB** werden hier also nur ca. **5 KB** pro Upload hochgeladen!

# Implementierung

Der vorgestellte Ansatz hat den Charme, dass nur kleine Umbauten im  Dockerfile und in der pom.xml notwendig sind. Beim Maven-Build muss das  paketierte JAR über das maven-dependency-plugin wieder passend  ausgepackt werden:

```markup
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <executions>
    <execution>
      <id>unpack</id>
      <phase>package</phase>
      <goals>
        <goal>unpack</goal>
      </goals>
      <configuration>
        <artifactItems>
          <artifactItem>
            <groupId>${project.groupId}</groupId>
            <artifactId>${project.artifactId}</artifactId>
            <version>${project.version}</version>
          </artifactItem>
        </artifactItems>
      </configuration>
     </execution>
   </executions>
</plugin>
```

Im Dockerfile werden die ausgepackten einzelnen Bestandteile dem  Image durch separate ADD Befehle (und damit in separaten Layern)  hinzugefügt:

```bash
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.mlesniak.demo.DemoApplication"]
```

Das war’s auch schon. Viel Spaß mit euren schnelleren Deployments :-)!
