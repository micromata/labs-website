---
layout: page
width: small
hero:
    title: Kapitel 1 – Einstieg in Spring Security
    subtitle: Dies ist das erste Kapitel der Tutorial-Beitragsreihe zu Spring Security. Es soll einen Einstieg in Spring Security veranschaulichen. 
    image:
permalink: /publikationen/tutorial-spring-security/einstieg-in-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Dazu wird ein simpler Login-Prozess vorgestellt und gezeigt, wie  Seiten ohne Login aufrufbar gemacht werden können. Den Source Code zu  diesem Tutorial findet ihr auf dem Micromata Github Bereich.

Einen Überblick über das gesamte Tutorial, bereits veröffentlichte Kapitel sowie den Ausblick auf kommende Kapitel ist hier zu  finden. Falls ihr Fragen zum Tutorial oder Source Code habt, meldet euch einfach über das LABS Kontaktformular oder wendet euch direkt über den  Micromata Github Bereich an mich, Jürgen Fast (Micromata).

## Vorbereitung

Zunächst wird ein simples Spring-Boot-Projekt angelegt. Dieses sollte nach der automatischen Anlage durch die IDE (z. B. IntelliJ) folgendes  enthalten: eine Java-Klasse zum Starten der Applikation, eine pom.xml  und eine leere application.properties. Nun müssen die Dependencies zur  Pom hinzugefügt werden, um einen Controller und eine erste Security-Konfiguration anlegen zu können.

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>de.micromata.spring.security.example</groupId>
    <artifactId>introduction-to-spring-security</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
 
    <name>introduction-to-spring-security</name>
    <description>A short introduction to spring security</description>
 
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    </dependencies>
     
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
     
</project>
```

## Hello World

Der Controller erhält für den Anfang nur ein Request Mapping für den Kontextpfad. Dieser Text soll nur angezeigt werden, falls der Benutzer  eingeloggt ist.

```java
package de.micromata.spring.security.example;
 
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
public class DefaultController {
 
    @RequestMapping("/")
    public String index() {
        return "You can only see this, if you are logged in!";
    }
 
}
```

Dazu erstellen wir die folgende Spring-Security-Konfiguration:

```java
package de.micromata.spring.security.example.security.conf;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
 
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Autowired
    public void globalSecurityConfiguration(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser("user").password("password").roles("USER");
        auth.inMemoryAuthentication().withUser("admin").password("password").roles("USER","ADMIN");
    }
 
}
```

Durch die Annotation`EnableWebSecurity`wird diese Security-Konfiguration nun bei jedem Request angezogen. In der Funktion `globalSecurityConfiguration` wird der Login mit den Benutzern `user` und `admin` konfiguriert. Die Funktion wird über die Annotation `@Autowired` eingebunden, um Zugriff auf den `AuthenticationManagerBuilder` der Applikation zu erhalten. Der Name **globalSecurityConfiguration** spielt dabei keine Rolle und kann nach belieben angepasst werden. Wird  die Applikation nun gestartet, dann wird beim Aufruf von http://localhost:8080/ eine Login-Seite angezeigt. Dies geschieht aufgrund der Methode `configure(HttpSecurity http)` der Elternklasse von `WebSecurityConfig`.

```java
protected void configure(HttpSecurity http) throws Exception {
    logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");
 
    http
        .authorizeRequests()
            .anyRequest().authenticated()
            .and()
        .formLogin().and()
        .httpBasic();
}
```

Diese Konfiguration sorgt dafür, dass bei jedem Request eine Authentifizierung stattfindet und zwar über ein Form-Login. Die Form  selbst wird dabei in diesem Fall von Spring selbst zur Verfügung  gestellt.

## Seiten ohne Authentifizierung

Damit Seiten auch ohne Login aufgerufen werden können, kann diese Methode in der Klasse WebSecurityConfig überschrieben werden.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/noSecurity").permitAll()
        .anyRequest().authenticated()
        .and().formLogin().permitAll();
}
```

Hierdurch ist die Seitehttp://localhost:8080/noSecurity auch ohne Login erreichbar. Zu beachten ist, dass `antMatchers()` vor `anyRequest()`aufgerufen wird. Würde die Reihenfolge getauscht werden, so würde für jeden Request ein Login verlangt werden, da `anyRequest` für alle Requests angesprochen wird und `antMatchers` somit nicht mehr erreicht werden kann.

Damit diese Seite aufrufbar ist, muss entsprechend noch ein Request Mapping in unserem Controller hinzugefügt werden:

```java
@RequestMapping("/noSecurity")
public String noSecurity() {
    return "Everybody can see this!";
}
```
