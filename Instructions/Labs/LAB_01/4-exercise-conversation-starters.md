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
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
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
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]",
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "https://learn.microsoft.com/microsoft-365/"
                }
            ]
        }
    ],
  "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
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
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
  ]
}
```

## Aufgabe 2: Testen des deklarativen Agents in Microsoft 365 Copilot Chat

Laden Sie als Nächstes Ihre Änderungen hoch, und starten Sie eine Debugsitzung.

In Visual Studio Code:

1. Öffnen Sie in der **Aktivitätsleiste** die Erweiterung **Teams Toolkit**.
1. Im Abschnitt **Lebenszyklus** wählen Sie **Bereitstellung** und dann **Veröffentlichen**.
1. **Bestätigen** Sie, dass Sie eine Aktualisierung an den App-Katalog senden möchten.
1. Warten Sie, bis der Upload abgeschlossen ist.

Fortsetzen im Webbrowser:

1. Wählen Sie in **Microsoft 365 Copilot** das Symbol oben rechts aus, um den **Copilot-Seitenbereich zu erweitern**.
1. Suchen Sie die Option **Produktsupport** in der Liste der Agents und wählen Sie sie aus, um das immersive Erlebnis zu betreten und direkt mit dem Agent zu chatten. Beachten Sie, dass die von Ihnen im Manifest definierten Gesprächseinstiegen in der Benutzeroberfläche angezeigt werden.

![Screenshot von Microsoft Edge mit dem deklarativen Agenten „Microsoft 365 Knowledge Expert“ in der immersiven Erfahrung mit benutzerdefinierten Gesprächsstartern.](../media/LAB_01/test-conversation-starters.png)

Schließen Sie den Browser, um die Debugsitzung in Visual Studio Code zu beenden.