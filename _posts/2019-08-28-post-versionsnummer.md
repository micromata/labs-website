---
title: Versionsnummer bitte…
author: atippel
categories: [Best Practices]
tags: [cli]
shortdesc: Versionskennzeichen automatisch und zentral vergeben.
featured: true
---

Wer kennt das nicht, unser Kunde meldet sich und hat ein Problem. Jetzt gilt es natürlich erst mal herauszufinden um was es genau geht und vor allem welche Softwareversion im Einsatz ist. Gegenbenenfalls  handelt es sich um ein Problem das bereits in einer neueren Version  unser Software behoben ist oder um ein bekanntes Problem.

Nun gibt es unterschiedliche Möglichkeiten eine Versionsnummer in unserer Software unterzubringen:

- wir können eine Konstante definieren á la `public static final String Version = "1.0.0"` und ihren Inhalt anzeigen

- wir können die Version aus einer properties Datei einlesen `InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("META-INF/system.properties")`

Wenn es sich um ein Java-Maven Projekt handelt haben wir aber bereits eine Versionsnummer in unserer `pom.xml` Datei. Deswegen und weil wir die Versionummer nicht immer wieder an  eine andere Stelle im Projekt kopieren wollen, wäre es doch schick die  Versionummer aus der Datei direkt zu verwenden. Damit vermeiden wir auch das das anpassen der Versionummer mal vergessen wird.

Nichts leichter als das! Dankenswerter Weise gibt es das `templating-maven-plugin`, das uns dabei hilft unsere Maven-Properties in den Java-Code zu  übernehmen bevor der Compiler läuft. Also eine Art Pre-Prozessor wie  beim guten alten C.

In unserer `pom.xml` Datei fügen wir folgendes ein:

```xml
 <build>
     <plugins>
         ...
         <plugin>
             <groupId>org.codehaus.mojo</groupId>
             <artifactId>templating-maven-plugin</artifactId>
             <version>1.0.0</version>
             <executions>
                 <execution>
                     <goals>
                         <goal>filter-sources</goal>
                     </goals>
                 </execution>
             </executions>
         </plugin>
         ...
     </plugins>
 </build>
```

Damit wird maven nun Properties wie z.B. `${project.version}` in unserem Java Code suchen und durch die entsprechenden Werte ersetzen.

Steht nun in unserer pom.xml eine Versionsnummer `1.0.0-SNAPSHOT` wie in folgendem Beispiel, können wir sie in einem Java Template verwenden.

```xml
 <?xml version="1.0" encoding="UTF-8"?>
 <project xmlns="http://maven.apache.org/POM/4.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <modelVersion>4.0.0</modelVersion>
 

     <groupId>org.datenmuehle</groupId>
     <artifactId>org.datenmuehle.version.template</artifactId>
     <version>1.0-SNAPSHOT</version>
     ...
```

Unser Java-Template platzieren wir nun in einem Verzeichnis `src/main/java-templates/[package Verzeichnis(se)]/Version.java`

Unsere Template-Klasse kann dann wie folgt aussehen:

```java
 package de.micromata.sysinfo;
 
 public final class Version {
    public static final String VERSION = "${project.version}";
 }
```

Um die Versionsnummer zu verwenden, können wir uns noch eine `SysInfo` Klasse erstellen. Die Klasse platzieren wir aber nicht im Java-Template Verzeichnis, sondern im eigentlichen Java Verzeichnis `src/main/java/[package Verzeichnis(se)]/SystemInfo.java`.

```java
package de.micromata.sysinfo;
 
 public class SystemInfo {
     public static String getVersion() {
         return Version.VERSION;
     }
 }
```

Bauen wir nun unsere Anwendung mit `mvn clean install` und untersuchen dann das `target` Verzeichnis, entdecken wir ein Verzeichnis `java-templates/[package Verzeichnis(se)]` welches unsere Pre-Kompilierte Template-Klasse `Version` enthält.

```java
 package de.micromata.sysinfo;
 
 public final class Version {
     public static final String VERSION = "1.0-SNAPSHOT";
 }
```

Damit haben wir nun also unsere Versionsnummer aus der pom.xml immer  als Konstante im Quellcode vorliegen und können sie verwenden.

Nun kann es aber vorkommen, das wir vergessen unsere Versionsnummer  in der pom.xml anzupassen. Das wäre fatal, da wir dann zwei  unterschiedliche Versionsstände unserer Software mit der gleichen  Versionsnummer ausliefern!

Deswegen müssen wir noch etwas nachlegen und eine sogenannte  Build-Nummer einführen. Und diese Nummer soll bitteschön automatisch bei jedem Build angepasst werden und dann in unserer Klasse `Version` landen!

Dafür greifen wir auf das `buildnumber-maven-plugin` zurück und konfigurieren es so, dass es uns einen Zeitstempel als Build-Nummer erzeugt:

```xml
  <build>
         <plugins>
             ...
             <plugin>
                 <groupId>org.codehaus.mojo</groupId>
                 <artifactId>buildnumber-maven-plugin</artifactId>
                 <version>1.4</version>
                 <configuration>
                     <revisionOnScmFailure>no.scm.config.in.pom</revisionOnScmFailure>
                 </configuration>
                 <executions>
                     <execution>
                         <phase>validate</phase>
                         <goals>
                             <goal>create-timestamp</goal>
                         </goals>
                         <configuration>
                             <timestampFormat>yyyy-MM-dd HH:mm:ss.S</timestampFormat>
                             <timestampPropertyName>buildnumber</timestampPropertyName>
                         </configuration>
                     </execution>
                 </executions>
             </plugin>
             ...
         </plugins>
     </build>
     ...
```

Der Name der Property ist wie oben definiert `buildnumber`. Nun können wir unser Java-Template einfach erweitern:

```java
 package de.micromata.sysinfo;
 
 public final class Version {
 
     public static final String VERSION = "${org.datenmuehle.sysinfo.version}";
     public static final String BUILD   = "${buildnumber}";
 }
```

Anschließend fügen wir in unserer SysInfo-Klasse einen neuen getter für die Build-Nummer hinzu:

```java
 package de.micromata.sysinfo;
 

 public class SystemInfo {
     public static String getVersion() {
         return Version.VERSION;
     }
 
     public static String getBuild()
     {
         return Version.BUILD;
     }
 }
```

Bauen wir nun unser Projekt wird `Version.BUILD` einen Zeitstempel ähnlich wie diesen `2019-08-25 19:12:40.992` enthalten.

Damit haben wir jetzt ein eindutiges Versionsmerkmal für unsere Software geschaffen.

Ein komplettes Beispiel findet Ihr unter https://github.com/datenmuehle/maventemplate

Viel Spaß beim Versionieren!
