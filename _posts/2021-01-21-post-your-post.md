---
title: Deine Java App mit jib in einen Container gesteckt
author: cclaus
categories: [Quick Tips]
tags: [maven]
shortdesc: Mit GoogleContainerTools zum Docker Image 
featured: true
---

Wer kennt es nicht, ein neues Projekt steht an. Voller Tatendrang  legen wir los mit dem Projekt-Setup. Welche Sprache nehmen wir? Ein  kurzes Quiz im Chat und das Team ist sich einig: Es wird Kotlin. Schön!  Jetzt noch die neusten Version von unserem Lieblingsframeworks mit in  die Maven POM aufgenommen. Frontend Build Prozess skizziert und nicht zu vergessen unsere Test-Suite. Last but not least denken wir daran, dass  die App am Ende in Form eines Docker-Images aus der CI/CD Pipeline  purzeln soll. Aber wie gehen wir das am besten an?

- Eigene Dockerfile schreiben?
- Das [Spring-Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#goals-build-image) zum Bauen von Docker Images nutzen?
- Oder gibt es da draußen noch etwas anderes?

Ja, es gibt etwas anderes da draußen. Google hat im Rahmen seiner GoogleContainerTools [jib](https://github.com/GoogleContainerTools/jib) in petto und ermöglicht es Entwickler:innen – selbst ohne tiefe Docker oder Container Kenntnisse – via Maven oder Gradle ein Docker oder OCI  Image einer Java Anwendung zu erstellen. Klingt gut. Was muss man dafür  tun?

Als erstes nimmt man das Build-Plugin in das Projektmodell auf. Hier am Beispiel von Maven:

```xml
...
<build>
    <plugins>
        <plugin>
            <groupId>com.google.cloud.tools</groupId>
            <artifactId>jib-maven-plugin</artifactId>
            <configuration>
                <from>
                    <image>adoptopenjdk/openjdk11:x86_64-debianslim-jdk-11.0.10_9-slim</image>
                </from>
                <to>
                    <image>the-great-application:latest</image>
                    <tags>
                        <tag>${project.version}</tag>
                        <tag>latest</tag>
                    </tags>
                </to>
                <container>
                    <ports>
                        <port>8080</port>
                    </ports>
                    <format>Docker</format>
                </container>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>dockerBuild</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
...
```

Wir binden damit die Erzeugung des Docker Images an die Ausführung  eines Maven package Lifecycle Abschnitts. Führen wir nun unser Maven  Build aus, bekommen wir folgenden Output:

```bash
...
[INFO] --- jib-maven-plugin:2.7.1:dockerBuild (default) @ vls-spa-backend ---
[INFO]
[INFO] Containerizing application to Docker daemon as the-great-application, the-great-application:1.0.0
[INFO] The base image requires auth. Trying again for adoptopenjdk/openjdk11:x86_64-debianslim-jdk-11.0.10_9-slim...
[INFO] Using credentials from Docker config (${HOME}/.docker/config.json) for adoptopenjdk/openjdk11:x86_64-debianslim-jdk-11.0.10_9-slim
[INFO] Using base image with digest: sha256:95ebbda303df52fe7e9bbf404378e10e1260a57976ac3b6e16b56af965b81395
[INFO]
[INFO] Built image to Docker daemon as the-great-application, the-great-application:1.0.0
[INFO] Executing tasks:
[INFO] [==============================] 100,0% complete
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.814 s
[INFO] Finished at: 2021-12-23T09:01:06+01:00
[INFO] ------------------------------------------------------------------------
```

Das Plugin bietet einige Möglichkeiten wie private Container  Registry-Nutzung, Entrypoints, JVM-Start parameter und ähnliches. Am  besten ihr schaut euch das [Projekt jib](https://github.com/GoogleContainerTools/jib) mal an.

Es lohnt sich!
