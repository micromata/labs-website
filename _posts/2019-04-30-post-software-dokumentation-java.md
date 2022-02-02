---
title: Software Dokumentation (für Java Entwickler)
author: atippel
categories: [Best Practices]
tags: [java]
shortdesc: Dokumentation da wo sie hingehört.
featured: true
---

Fragt man einen Softwareentwickler nach Dokumentation, wird er  vermutlich zwei Dinge antworten. Wenn er neu in ein Projekt kommt wird   er sich beschweren, dass die Software ja nicht bzw. nicht ausreichend  dokumentiert ist um sich angemessen schnell einzuarbeiten.

Entwickelt er selbst und der Zeitdruck ist wie immer hoch, wird er sagen, dass er  die Dokumentation nach dem Release noch „geradeziehen“ wird, weil ja die Auslieferung vor der Tür steht und noch wichtige Dinge zu tun sind!  Aber nach dem Projekt ist bekanntlich ja vor dem Projekt und deshalb  wird die Dokumentation üblicherweise nicht nach der Auslieferung  geradegezogen, weil man sich ja in das neue Thema einarbeiten muss. Ein  Teufelskreis!

Ein weiteres Problem auf dem Weg zur perfekten Dokumentation ist das  Tool mit dem diese geschrieben werden muss. Dazu ein Beispiel. Häufig  erlebe ich das Dokumentation in Microsoft-Word geschrieben wird. Gar  nicht so einfach wenn man selbst ein Linux Entwicklungssystem besitzt  und wegen der Word-Makros auch Libre-Office keine Alternative darstellt. Dann heißt es die Virtuelle-Maschine anwerfen um in der Windows10-VM  Word zu starten und dort das Dokument zu bearbeiten. Auch Office365 ist  kein wirklicher Ersatz, denn hier müssen die Dokumente zuerst an  OneDrive übertragen werden. Das ist u. U. nicht ohne weiteres erlaubt.  Ebenfalls häufig wird auf Atlassian Confluence zurückgegriffen. Wieder  ein System das völlig losgelöst von der Software existiert. Hier  benötige ich wieder Zugriffsrechte und muss wissen wo die Dokumentation  eigentlich zu finden ist. All das sind Widerstände die der eher  ungeliebten Tätigkeit „dokumentieren“ im Weg stehen und sie noch  ungeliebter machen. Auch wenn jedes Tool für sich genommen ausgezeichnet funktioniert! Da programmier ich doch lieber noch ein paar Zeilen…

Ein Ansatz den ich persönlich bevorzuge ist es die Dokumentation im  gleichen Repository wie den Quellcode unterzubringen. Damit liegt die  Dokumentation schon mal dort wo man sie benötigt! Bleibt nur noch die  Dokumentation auch in einem Format zu schreiben das möglichst wenige  System-Vorraussetzungen mitbringt.

## JBake

Elegant kann man das mit JBake lösen. JBake ist ein Tool zum  generieren statischer Webseiten. Es lässt sich einfach mit maven oder  gradle in den Build-Prozess integrieren. Die Doku selbst kann dann als  Markdown oder Asciidoc Text-Datei erstellt werden. Daraus erzeugt JBake  dann die HTML Dateien. Die können dann wiederum einfach mit dem Browser  offline gelesen oder bei Bedarf auf einem Server bereitgestellt werden.  Weitere Vorteile sind, dass die Dokumente mit einfachsten Mitteln  durchsucht werden können, es ist keine spezielle Software dafür  notwendig. Ebenfalls erhält man über die Versionsverwaltung eine  ordentliche Historie bei der man die vorgenommenen Änderungen wunderbar  erkennen kann.

### Wie geht das?

Man erstellt bspw. parallel zum klassischen „src“ Ordner einen Ordner „documentation“. Dieser wird dann von JBake initialisiert oder mit  einem bestehenden JBake-Template gefüllt. Dort existiert dann ein Ordner „content“ und in diesem können wir bspw. unsere Dokumentation als  Markdown-Datei(en) ablegen. JBake wird dann bei jedem Build eine  statische HTML Seite daraus erstellen. Damit kann jeder Entwickler  direkt aus seiner IDE heraus die Dokumentation bearbeiten. Existiert ein Buildsystem wie bspw. Jenkins, erstellt es automatisch bei jedem Build  den aktuellen Stand und unser Kunde erhält eine vernünftig lesbare  Online-Dokumentation die selbst offline funktioniert!

Wir benötigen dazu unsere klassische Ordnerstruktur für ein maven  Projekt in der wir zusätzlich einen Ordner „documentation“ erstellen.

```
projekt-ordner
|
|- pom.xml
|
|- src
| |
| |- main
| | |- java
| | |-...
| |
| |- documentation
```

In unserer pom.xml integrieren wir nun das jbake-maven-plugin in der build-Sektion und definieren die zwei Properties `my.jbake.in` und `my.jbake.out`. Dort definieren wir den Quell- und den Zielordner für unseren „Bäcker“.

```xml
    <properties>
        <my.jbake.in>${project.basedir}/src/documentation</my.jbake.in>
        <my.jbake.out>${project.basedir}/target/documentation</my.jbake.out>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.jbake</groupId>
                <artifactId>jbake-maven-plugin</artifactId>
                <version>0.3.1</version>
                <executions>
                    <execution>
                        <id>default-generate</id>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <inputDirectory>${my.jbake.in}</inputDirectory>
                    <outputDirectory>${my.jbake.out}</outputDirectory>
                </configuration>
                <dependencies>
                    <!-- for freemarker templates (.ftl) -->
                    <dependency>
                        <groupId>org.freemarker</groupId>
                        <artifactId>freemarker</artifactId>
                        <version>2.3.28</version>
                    </dependency>
                    <dependency>
                        <groupId>com.vladsch.flexmark</groupId>
                        <artifactId>flexmark-all</artifactId>
                        <version>0.34.48</version>
                    </dependency>
                    <dependency>
                        <groupId>org.asciidoctor</groupId>
                        <artifactId>asciidoctorj</artifactId>
                        <version>1.5.7</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```

Um ein vorgefertigtes JBake Template zu verwenden und unseren  „documentation“ Ordner damit zu initialisieren wechseln wir auf der  Kommandozeile in den „projekt-ordner“ und führen folgendes Kommando aus.

```bash
mvn jbake:seed -Dmy.jbake.out=./src/documentation
```

Anschließend sehen wir folgende Ordnerstruktur im Ordner „documentation“:

```
.
|- assets
|   |- css
|   |   |- asciidoctor.css
|   |   |- base.css
|   |   |- bootstrap.min.css
|   |   └- prettify.css
|   |- favicon.ico
|   |- fonts
|   |   |- glyphicons-halflings-regular.eot
|   |   |- glyphicons-halflings-regular.svg
|   |   |- glyphicons-halflings-regular.ttf
|   |   └- glyphicons-halflings-regular.woff
|   └- js
|       |- bootstrap.min.js
|       |- html5shiv.min.js
|       |- jquery-1.11.1.min.js
|       └- prettify.js
|--- content
|   |- about.html
|   └- blog
|       └- 2013
|           |- first-post.html
|           |- fourth-post.adoc
|           |- second-post.md
|           └- third-post.adoc
|- jbake.properties
|- LICENSE
|- README.md
└- templates
    |- archive.ftl
    |- feed.ftl
    |- footer.ftl
    |- header.ftl
    |- index.ftl
    |- menu.ftl
    |- page.ftl
    |- post.ftl
    |- sitemap.ftl
    └- tags.ftl
```

Nun können wir beliebige Inhalte unterhalb vom Ordner „content“  erstellen. Uns stehen dafür unterschiedliche Formate zur Verfügung.  Unter anderem markdown, asciidoc und html. Ich bevorzuge hier markdown  Dateien mit der Endung .md.

Nun können wir die Seite wie folgt generieren.

```bash
mvn jbake:generate
```

Alternativ ist es möglich einen Webserver auf Port 80 zu starten und  dort immer die aktuelle Version der Dokumentation zu sehen. Dafür  folgendes aufrufen.

```bash
mvn jbake:inline
```

Damit kann man an der Dokumentation arbeiten, JBake erkennt die  Änderungen an den Dateien und generiert die Dokumentation neu. Lesen  kann man die Doku in diesem Fall wenn man im Browser folgende URL  eingibt `http://localhost:8080`.

Für das eigene Unternehmen bietet es sich an ein eigenes  JBake-Template zu erstellen anstatt „seed“ zu verwenden. Hier kann dann  das Corporate-Design einfließen und auch von der JBake Intention eines  Blogs abgewichen werden. Weitere Informationen bzgl. Templates gibt es  auf der [JBake](https://jbake.org/docs/2.6.4/) Webseite.

Viel Spaß beim dokumentieren!
