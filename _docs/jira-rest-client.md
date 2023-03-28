---
title: JIRARestClient
subtitle: Mit Hilfe des JiraRestClient kann über die REST-API V2 auf die Plattform Jira von Atlassian direkt zugegriffen werden.
tags: [jira, rest, api]
author: cschulze
---

Dabei werden die Responses in Java-Objekten gekapselt, so dass ein Zugriff über die REST-API innerhalb einer Java-Programmierung  vereinfacht wird.

### Open-Source Software für den java- und objektorientierten Zugriff auf Jira über die REST API

Motivation des Projektes ist, eine Bibliothek zur Verfügung zu haben, mit der über die REST-API V2 auf die Plattform Jira von Atlassian  direkt und nicht nur über die Webseite zugegriffen werden kann.
 Der JiraRestClient kapselt die Responses der Funktionen  der Jira-REST-API in Java-Objekte. Dadurch kann eine Java-Anwendung auf  Jira zugreifen und die Response der REST-API direkt verwenden.

**An welche Personen/ Entwickler:innen evtl. Umfeld richtet sich das Projekt**
 Entwickelnde, die über ihr eignes Java Tool auf die Jira-REST-API einfach zugreifen wollen.

**Technologiestack**
 Der Client ist in Java geschrieben und verwendet Future, kann also sehr gut asyncron genutzt werden.
 Es gibt einen Builder um JQL Anfragen zusammen zu bauen.
 Verwendete Libraries  sind *gson* und *Apache HttpClient* *Version 4*.
 Mit welcher Version von Jira der JiraRestClient aktuell kompatibel ist, kann im Micromata Github eingesehen werden.

**Lizenz**:
 [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)

**Sourcecode-Ablage**
 [Micromata Github](https://github.com/micromata/JiraRestClient)
 Implementierungen für [.NET Core](https://github.com/cschulc/JiraRestClient.NETCore) sind im privaten [Github Bereich](https://github.com/cschulc/JiraRestClient.NETCore) von unserem Christian zu finden.

**Weiteres**
 Es existiert im Micromata GitHub Space ebenfalls einen Client um auf die [REST-Api eines Confluence](https://github.com/micromata/ConfluenceRestClient) zuzugreifen. Dieser Client funktioniert ähnlich wie der JiraRestClient.

