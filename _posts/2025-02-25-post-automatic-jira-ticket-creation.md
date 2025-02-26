---
title: Automatische Jira-Ticket-Erstellung via REST API
author: sginter
categories: [Quick Tips]
tags: [jira]
shortdesc: Erstelle und bearbeite automatisch Jira Tickets mit der Jira REST-API.
---

Jira bietet eine leistungsstarke REST-API, mit der sich Aufgaben wie das Erstellen, Bearbeiten und Aktualisieren von Tickets automatisieren lassen.  
In diesem Beitrag zeigen wir, wie das Python-Skript **"Jira Ticket Updater"** verwendet werden kann, um automatisch die Priorität aus einer Excel-Tabelle in Jira-Tickets zu übertragen.  

## Vorteile der Automatisierung
- **Zeitersparnis:** Manuelle Eingaben werden minimiert.  
- **Fehlerreduktion:** Automatisierung reduziert das Risiko von Tipp- und Übertragungsfehlern.  
- **Effizienz:** Mehrere Tickets können in einem Durchlauf aktualisiert werden.  

## Was kann man damit machen?
Die Jira REST Api ist sehr groß und im folgenden werden nur Beispiele gezeigt die für einen leichten einstieg geeignet sind
**1.Tickets abrufen**
Mit einer GET-Anfrage lassen sich die Informationen zu Tickets einfach von der Jira REST API abrufen. Die JSON-Antwort enthält Ticketdetails wie ID, Status und Beschreibung.  

```GET /rest/api/3/issue/{issueIdOrKey} ```
Quelle: https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-issueidorkey-get

**2. Tickets erstellen**
Bei einer POST-Anfrage können die Jira-Ticket-Properties als JSON-Payload mitgegeben werden. Dabei ist es beispielsweise möglich, in der Ticket-Beschreibung eigene Jira-Befehle zu verwenden, um Elemente wie Panels und andere Formatierungen zu erstellen. Die Antwort liefert die ID des erstellten Tickets.  
```POST /rest/api/3/issue```
```json
  {
    "fields": {
      "project": {"key": "Porject"},
      "summary": "Automatisch erstelltes Ticket",
      "description": "Dieses Ticket wurde via REST API erstellt.",
      "issuetype": {"name": "Task"}
    }
  }
  ```
Quelle: https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/#api-rest-api-3-issue-post

**3. Tickets aktualisieren**  
- Statusänderungen, Kommentare oder Feldanpassungen erfolgen per PUT-Anfrage.  

- Panel hinzufügen:  
  ```json
  {
    "description": "{panel:title=Erfolgskriterium:}Kommentar hinzugefügt via API{panel}"
  }
  ```

**4. Jira-Tickets zwischen Instanzen synchronisieren**  
- Tickets können zwischen zwei Jira-Instanzen übertragen werden, indem die REST API der Quell- und Zielinstanz kombiniert genutzt wird.  
- Workflow:  
  - Ticketdetails mit GET von der Quellinstanz abrufen.  
  - Ticket mit POST in der Zielinstanz neu erstellen.  

**5. Informationen zu benutzerdefinierten Feldern abrufen**
- Um die verfügbaren benutzerdefinierten Felder und deren Feld-IDs zu ermitteln, kann die folgende API-Anfrage genutzt werden:  
  ```bash
  curl -X GET -H "Authorization: Bearer <bearer_token>" \
       -H "Content-Type: application/json" \
       "https://<jira-domain>/rest/api/3/field"
  ```
- Die JSON-Antwort listet alle Felder mit deren Namen und IDs. Beispielausgabe:  
  ```json
  [
    {
      "id": "customfield_10789",
      "name": "Projektphase",
      "schema": {
        "type": "number"
      }
    },
    {
      "id": "customfield_10987",
      "name": "Priorität",
      "schema": {
        "type": "string"
      }
    }
  ]
  ```

  ---
##### **Beispiel: Python-Skript zur Aktualisierung der Ticket-Priorität und Einfügen der Beschreibung in ein Panel**  
Das folgende Python-Skript liest Ticketdaten aus einer Excel-Datei, verbindet sich mit Jira und aktualisiert die Tickets. Es setzt die Priorität der Tickets und fügt die Beschreibung in ein Panel im Markdown-Format ein.  

Für die Nutzung des Skripts sind folgende Anpassungen erforderlich:  
- **API-Token:** Ein API-Token muss im Jira-Konto generiert und im Skript hinterlegt werden.  
- **Benutzername:** Die zu verwendende E-Mail-Adresse des Jira-Kontos ist anzugeben.  
- **Feldnummern:** Die IDs der anzupassenden Felder müssen ermittelt werden. Siehe Schritt 5.

```python
import pandas as pd
from jira import JIRA
from jira.exceptions import JIRAError

# Excel-Datei einlesen
excel_file_path = '<fill_file_path>/TicketPrioritaetUpdate.xlsx'
df = pd.read_excel(excel_file_path, sheet_name='Tabelle1')

# Jira-Verbindung konfigurieren
jira_url = 'https://jira.com/'
jira_username = '<fill email address>'
jira_bearer_token = '<fill bearer_token>'

# Jira-Verbindung herstellen
jira = JIRA(server=jira_url, token_auth=jira_bearer_token)

# Durch die Tabelle iterieren und Tickets aktualisieren
for index, row in df.iterrows():
    ticket_id = row['Ticket']
    priority = row['Priority']
    description = row['Description']

    try:
        issue = jira.issue(ticket_id)

        # Ticket aktualisieren: Priorität setzen und Beschreibung in Panel einfügen
        issue.update(fields={
            'customfield_10987': priority,
            'description': f"{{panel:bgColor=#deebff}}\n{description}\n{{panel}}"
        })

        print(f"Successfully updated {ticket_id} with priority: {priority}")

    except JIRAError as e:
        print(f"Failed to update the issue {ticket_id}: {str(e)}")

print("Tickets wurden erfolgreich aktualisiert.")
```

Beispiel TicketPrioritaetUpdate Excel Tabelle:
```markdown
| Ticket   | Priority | Description                      |
|----------|----------|----------------------------------|
| PROJ-123 | High     | Fehler in der Benutzeroberfläche |
| PROJ-124 | Medium   | Performance-Optimierung          |
```
---

##### **Fazit**  
Die Automatisierung der Jira-Ticket-Erstellung und -Aktualisierung über die REST API ist besonders hilfreich, wenn große Mengen an Tickets angepasst werden müssen – etwa zum Setzen von Prioritäten, zur Markdown-Formatierung der Beschreibung oder für die Integration mit Build- und Test-Pipelines. Das gezeigte Python-Skript spart Zeit und minimiert Fehler.
