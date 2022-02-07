---
layout: page
width: small
hero:
    title: Kapitel 5 - HTTPS mit Spring Boot
    subtitle: Im fünften Kapitel der Tutorial-Beitragsreihe zu Spring Security geht es darum, den integrierten Tomcat von Spring Boot so zu konfigurieren, dass die Applikation nur noch mit dem HTTPS-Protokoll arbeitet und alle HTTP-Anfragen auf HTTPS umgeleitet werden.
    image:
permalink: /publikationen/tutorial-spring-security/https-mit-spring-boot/
author: jfast
scope: [Tutorial Spring Security]
---

Dies ist das fünfte Kapitel der **Tutorial-Beitragsreihe** zu **Spring Security**. In diesem Beitrag geht es darum, den integrierten Tomcat von Spring  Boot so zu konfigurieren, dass die Applikation nur noch mit dem [HTTPS](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol_Secure)-Protokoll arbeitet und alle [HTTP](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol)-Anfragen auf HTTPS umgeleitet werden. Grundsätzlich kann hierfür ein neues  Spring-Boot-Projekt verwendet werden, jedoch wurde für diesen Beitrag  das Projekt aus [Default Schutz durch Spring Security](/publikationen/tutorial-spring-security/default-schutz-durch-spring-security/) verwendet. Den Source Code zu diesem Tutorial findet ihr auf dem [Micromata Github Bereich](https://github.com/micromata/labs-spring-security/tree/master/chapter5/https-with-spring-boot).

## **Das Zertifikat**

Zunächst benötigen wir ein SSL-Zertifikat. In diesem Beitrag wird  dafür ein selbstsigniertes Zertifikat benutzt. Dieses lässt sich  beispielsweise folgendermaßen erstellen:

```java
keytool -genkey -keyalg RSA -alias myselfsigned -storetype PKCS12 -keysize 2048 -keystore keystore.p12 -validity 360
```

Es folgt eine Anfrage nach einem Passwort für das Zertifikat und  anschließend eine Reihe ausfüllbarer Felder, die jedoch alle optional  sind und übersprungen werden können.

Das Zertifikat `keystore.p12` wird anschließend unter `/resources/ssl/keystore.p12` abgelegt.

## Die Konfiguration

In der Konfiguration wird nun Folgendes eingetragen:

```java
server:
  port: 8443
  ssl:
    key-store: src/main/resources/ssl/keystore.p12
    key-store-password: mypassword
    keyStoreType: PKCS12
    keyAlias: myselfsigned
```

Nun ist unsere Applikation ausschließlich über HTTPS unter dem Port 8443 erreichbar.

## HTTP Redirect

Da wir jedoch wollen, dass unsere Applikation auch über HTTP erreichbar ist, damit ein Aufruf wie `meineApplikation.de` möglich ist, benötigen wir einen Redirect von HTTP auf HTTPS. Jedoch ermöglicht Spring Boot nur die Konfiguration eines Tomcat Connectors über die  Konfigurationsdatei. Der andere Connector muss programmatisch angelegt  werden. Da es einfacher ist den HTTP Connector programmatisch zu  erstellen, passen wir die Klasse `SimpleSpringSecurityApplication` im Folgenden entsprechend an. Zunächst fügen wir zwei Membervariablen hinzu, die mit der Konfigurationsdatei `application.yml` verknüpft sind. Diese Verknüpfung wird über die Annotation `@Value` hergestellt.

```java
@Value("${server.port}")
private int serverPort;
 
@Value("${server.http.port}")
private int serverHttpPort;
```

`server.port` haben wir bereits mit dem Wert 8443 konfiguriert, jedoch müssen wir nun noch `server.http.port` nachtragen.

```java
server:
  port: 8443
  http.port: 8080
  ssl:
    key-store: src/main/resources/ssl/keystore.p12
    key-store-password: mypassword
    keyStoreType: PKCS12
    keyAlias: myselfsigned
```

Anschließend fügen wir den neuen Connector in `SimpleSpringSecurityApplication` hinzu.

```java
@Bean
public EmbeddedServletContainerFactory createAdditionalTomcatConnector() {
    TomcatEmbeddedServletContainerFactory embeddedTomcat = new TomcatEmbeddedServletContainerFactory() {
        @Override
        protected void postProcessContext(Context context) {
            SecurityConstraint securityConstraint = new SecurityConstraint();
            securityConstraint.setUserConstraint("CONFIDENTIAL");
            SecurityCollection collection = new SecurityCollection();
            collection.addPattern("/*");
            securityConstraint.addCollection(collection);
            context.addConstraint(securityConstraint);
        }
    };
    embeddedTomcat.addAdditionalTomcatConnectors(createHttpConnector());
    return embeddedTomcat;
}
 
private Connector createHttpConnector() {
    Connector httpConnector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    httpConnector.setScheme("http");
    httpConnector.setPort(serverHttpPort);
    httpConnector.setRedirectPort(serverPort);
    return httpConnector;
}
```

Um den zweiten Tomcat Connector anzulegen, benötigen wir zunächst eine `EmbeddedServletContainerFactory-`Bean mit `TomcatEmbeddedServletContainerFactory` als Implementation. Hier überschreiben wir die Methode `postProcessContext`, welche in der Superklasse leer ist. Diese Methode wird genutzt, um den  Kontext anzupassen, bevor er den Tomcat Server erreicht. In der Methode  definieren wir das User Constraint als `CONFIDENTIAL` und das Pattern auf `/*`, damit der Redirect später auf alle Anfragen angewendet wird. Würden wir das User Constraint beispielsweise auf `NONE` setzen, so würde keinerlei Weiterleitung stattfinden. An dieser Stelle kann man die Weiterleitung auch auf bestimmte URLs begrenzen, falls dies  gewünscht ist. Anschließend wird dem integrierten Tomcat der neue HTTP  Connector, welcher in der überschaubaren Methode `createHttpConnector` erstellt wird, hinzugefügt. Die endgültige Klasse `SimpleSpringSecurityApplication` sieht dann folgendermaßen aus:

```java
package de.micromata.spring.security.example;
 
import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.EmbeddedServletContainerFactory;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;
 
@SpringBootApplication
public class SimpleSpringSecurityApplication {
 
    @Value("${server.port}")
    private int serverPort;
 
    @Value("${server.http.port}")
    private int serverHttpPort;
 
    public static void main(String[] args) {
        SpringApplication.run(SimpleSpringSecurityApplication.class, args);
    }
 
    @Bean
    public EmbeddedServletContainerFactory createAdditionalTomcatConnector() {
        TomcatEmbeddedServletContainerFactory embeddedTomcat = new TomcatEmbeddedServletContainerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        embeddedTomcat.addAdditionalTomcatConnectors(createHttpConnector());
        return embeddedTomcat;
    }
 
    private Connector createHttpConnector() {
        Connector httpConnector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        httpConnector.setScheme("http");
        httpConnector.setPort(serverHttpPort);
        httpConnector.setRedirectPort(serverPort);
        return httpConnector;
    }
}
```
