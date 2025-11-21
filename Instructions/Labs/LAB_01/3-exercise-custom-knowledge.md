---
lab:
  title: Übung 2 – Konfigurieren von benutzerdefiniertem Wissen
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Übung 2 – Konfigurieren von benutzerdefiniertem Wissen

In dieser Übung lernen Sie Microsoft Learn als Grundlage für Ihren Agent. Ihr Agent wird zum Experten für Microsoft 365.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1: Konfigurieren von Groundingdaten

Konfigurieren Sie Microsoft Learn als Quelle für die Grundlage von Grounding-Daten im Manifest für den deklarativen Agent.

In Visual Studio Code:

1. Öffnen Sie im Ordner **appPackage** die Datei **declarativeAgent.json**.
1. Fügen Sie den folgenden Codeausschnitt nach der Definition **"instructions"** in die Datei ein und ersetzen Sie **{URL}** durch die direkte URL zur Microsoft 365-Lernseite auf Microsoft Learn:

    ```json
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "{URL}"
                }
            ]
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
    ]
}
```

## Aufgabe 3 - Aktualisieren von benutzerdefinierten Anweisungen

Aktualisieren Sie die Anweisungen im Manifest für den deklarativen Agent, um unserem Agent zusätzlichen Kontext zu geben und ihn beim Beantworten von Kundschaftsanfragen zu unterstützen.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/instruction.txt**, und aktualisieren Sie den Inhalt mit:

    ```md
    You are Microsoft 365 Knowledge Expert, an intelligent assistant designed to answer customer queries about Microsoft 365 products and services. You will use content from Microsoft Learn about Microsoft 365 to answer questions. If you can't find the necessary information, you should suggest that the agent should reach out to the team responsible for further assistance. Your responses should be concise and always include a cited source.
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 4 – Laden Sie den deklarativen Agenten nach Microsoft 365 hoch

Laden Sie Ihre Änderungen in Microsoft 365 hoch.

In Visual Studio Code:

1. Öffnen Sie in der **Aktivitätsleiste** die Erweiterung **Teams Toolkit**.
1. Im Abschnitt **Lebenszyklus** wählen Sie **Bereitstellung** und dann **Veröffentlichen**.
1. **Bestätigen** Sie, dass Sie eine Aktualisierung an den App-Katalog senden möchten.
1. Warten Sie, bis die Veröffentlichungsaufgaben abgeschlossen sind.

## Aufgabe 5 – Testen des deklarativen Agenten in Microsoft 365 Copilot

Laden Sie Ihren deklarativen Agent in Microsoft 365 hoch, und überprüfen Sie die Ergebnisse. Fahren Sie im Browser aus der vorherigen Übung fort und aktualisieren Sie das Fenster (**F5**).

Zunächst testen wir die Anweisungen:

1. Wählen Sie in **Microsoft 365 Copilot** das Symbol oben rechts aus, um den **Copilot-Seitenbereich zu erweitern**.
1. Suchen Sie **Microsoft 365 Knowledge Expert** in der Liste der Agenten und wählen Sie ihn aus, um das immersive Erlebnis zu betreten und direkt mit dem Agenten zu chatten.
1. Fragen Sie den Produkt-Support-Agent **Was bist du fähig zu tun?** und übermitteln Sie den Prompt.
1. Warten Sie auf die Antwort. Beachten Sie, wie sich die Antwort von den vorherigen Anweisungen unterscheidet und die neuen Anweisungen widerspiegelt.

    ![Screenshot von Microsoft Edge mit Microsoft 365 Copilot Es wird eine Antwort des Microsoft 365 Knowledge Expert-Agents angezeigt, der seine Funktionen erläutert.](../media/LAB_01/test-m365-knowledge-expert.png)

Als Nächstes testen wir die Grounding-Daten.

1. Geben Sie in das Nachrichtenfeld **Erzähl mir etwas über Informationsschutz** ein und senden Sie die Nachricht.
1. Warten Sie auf die Antwort. Bitte beachten Sie, dass die Antwort Informationen zum Datenschutz enthält. Die Antwort enthält Zitate und Verweise auf die spezifische Website, die zum Generieren der Antwort verwendet wurde.

    ![Screenshot von Microsoft Edge mit Microsoft 365 Copilot Eine Antwort des Microsoft 365 Knowledge Expert-Agents wird mit Informationen zum Informationsschutz in Microsoft 365 angezeigt.](../media/LAB_01/test-m365-knowledge-expert-1.png)

Lassen Sie uns einige weitere Eingabeaufforderungen ausprobieren:

1. Geben Sie in das Nachrichtenfeld **Empfehle ein Produkt, das für eine Echtzeitkommunikation geeignet ist**.
1. Geben Sie im Meldungsfeld **Erkläre mir die Support-Optionen für Microsoft 365** ein.

Schließen Sie den Browser, um die Debugsitzung in Visual Studio Code zu beenden.
