---
lab:
  title: Übung 3 – Hinzufügen von Unterhaltungseinstiegsmöglichkeiten zu Ihrem deklarativen Agenten
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Übung 3 – Hinzufügen von Unterhaltungseinstiegsmöglichkeiten zu Ihrem deklarativen Agenten

In dieser Übung werden Sie den deklarativen Agent aktualisieren, um Aufhänger für Gespräche hinzuzufügen, die den Benutzenden Beispielansagen bieten, um ihnen zu helfen, die Arten von Fragen zu verstehen, die sie stellen können.

### Übungsdauer

- **Geschätzter Zeitaufwand**: 5 Minuten

## Aufgabe 1 – Hinzufügen von Konversationseinstiegsmöglichkeiten

In Visual Studio Code:

1. Öffnen Sie im Ordner **appPackage** die Datei **declarativeAgent.json**.
1. Fügen Sie den folgenden Codeschnipsel in die Datei ein:

   ```json
   "conversation_starters": [
       {
           "title": "Product information",
           "text": "Tell me about Eagle Air"
       },
       {
           "title": "Returns policy",
           "text": "What is the returns policy?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Repair information",
           "text": "Can you provide information on how to get a product repaired?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
   ]
   ```

1. Speichern Sie die Änderungen.

Die Datei **declarativeAgent.json** sollte wie folgt aussehen:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
  "version": "v1.0",
  "name": "Product support",
  "description": "Product support agent that can help answer customer queries about Contoso Electronics products",
  "instructions": "$[file('instruction.txt')]",
  "capabilities": [
    {
      "name": "OneDriveAndSharePoint",
      "items_by_url": [
        {
          "url": "https://{tenant}-my.sharepoint.com/personal/{user}/Documents/Products"
        }
      ]
    }
  ],
  "conversation_starters": [
    {
      "title": "Product information",
      "text": "Tell me about Eagle Air"
    },
    {
      "title": "Returns policy",
      "text": "What is the returns policy?"
    },
    {
      "title": "Product information",
      "text": "Can you provide information on a specific product?"
    },
    {
      "title": "Product troubleshooting",
      "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
    },
    {
      "title": "Repair information",
      "text": "Can you provide information on how to get a product repaired?"
    },
    {
      "title": "Contact support",
      "text": "How can I contact support for help?"
    }
  ]
}
```

## Aufgabe 2 – Testen des deklarativen Agents in Microsoft 365 Copilot

Laden Sie als Nächstes Ihre Änderungen hoch, und starten Sie eine Debugsitzung.

In Visual Studio Code:

1. Öffnen Sie in der **Aktivitätsleiste** die Erweiterung **Teams Toolkit**.
1. Im Abschnitt **Lebenszyklus** wählen Sie **Bereitstellung**.
1. Warten Sie, bis der Upload abgeschlossen ist.
1. Wechseln Sie in der **Aktivitätsleiste** zur Ansicht **Ausführen und Debuggen**.
1. Wählen Sie die Schaltfläche **Debugging starten** neben dem Dropdown-Menü der Konfiguration, oder drücken Sie <kbd>F5</kbd>. Ein neues Browserfenster wird gestartet und navigiert zu Microsoft 365 Copilot.

Als Nächstes testen Sie Ihren deklarativen Agent in Microsoft 365 und validieren die Ergebnisse.

Fortsetzen im Webbrowser:

1. Wählen Sie in **Microsoft 365 Copilot** das Symbol oben rechts aus, um den **Copilot-Seitenbereich zu erweitern**.
1. Suchen Sie die Option **Produktsupport** in der Liste der Agents und wählen Sie sie aus, um das immersive Erlebnis zu betreten und direkt mit dem Agent zu chatten. Beachten Sie, dass die von Ihnen im Manifest definierten Gesprächseinstiegen in der Benutzeroberfläche angezeigt werden.

![Screenshot von Microsoft Edge, der den deklarativen Agent für Produktsupport in der immersiven Erfahrung mit benutzerdefinierten Gesprächseinstiegen zeigt.](../media/LAB_01/test-conversation-starters.png)

Schließen Sie den Browser, um die Debugsitzung in Visual Studio Code zu beenden.