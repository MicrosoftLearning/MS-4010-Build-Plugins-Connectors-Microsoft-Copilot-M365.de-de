---
lab:
  title: 'Übung 2: Testen des deklarativen Agents mit API-Plug-In in Microsoft 365 Copilot Chat'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Übung 2: Testen des deklarativen Agents mit API-Plug-In in Microsoft 365 Copilot Chat

In dieser Übung testen und implementieren Sie Ihren deklarativen Agents in Microsoft 365 und testen ihn mit Microsoft 365 Copilot Chat.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1: Testen des deklarativen Agents mit dem API-Plug-In in Microsoft 365 Copilot Chat

Der letzte Schritt besteht darin, den deklarativen Agent mit API-Plug-In in Microsoft 365 Copilot zu testen.

In Visual Studio Code:

1. Aktivieren Sie in der Aktivitätsleiste die Erweiterung **Teams Toolkit**.
1. Vergewissern Sie sich im Bedienfeld **Teams Toolkit** im Abschnitt **Konten**, dass Sie bei Ihrem Microsoft 365 Mandanten angemeldet sind.

  ![Screenshot des Teams-Toolkits mit dem Status der Verbindung mit Microsoft 365.](../media/LAB_05/3-teams-toolkit-account.png)

1. Wechseln Sie in der Aktivitätsleiste zur Ansicht Ausführen und Debuggen.
1. Wählen Sie aus der Liste der Konfigurationen **Debuggen in Copilot (Edge)** und drücken Sie die Wiedergabetaste, um das Debuggen zu starten.

  ![Screenshot der Debug-Option in Visual Studio Code.](../media/LAB_05/3-vs-code-debug.png)

  Visual Studio Code öffnet einen neuen Webbrowser mit Microsoft 365 Copilot Chat. Wenn Sie dazu aufgefordert werden, melden Sie sich mit Ihrem Microsoft 365-Konto an.

Im Webbrowser:

1. Wählen Sie im Bedienfeld den Agent **da-repairs-keylocal** aus.

  ![Screenshot des benutzerdefinierten Agents, der in Microsoft 365 Copilot angezeigt wird.](../media/LAB_05/3-copilot-agent-sidebar.png)

1. Geben Sie in das Prompt-Textfeld `What repairs are assigned to Karin?` ein und übermitteln Sie den Prompt.
1. Bestätigen Sie, dass Sie Daten an das API-Plug-In senden möchten, indem Sie auf die Schaltfläche **Immer zulassen** klicken.

  ![Screenshot des Prompts, um das Senden von Daten an die API zuzulassen.](../media/LAB_05/3-allow-data.png)

1. Warten Sie, bis der Agent reagiert.

  ![Screenshot der Antwort des benutzerdefinierten Agents auf die Aufforderung des Benutzers/der Benutzerin.](../media/LAB_05/3-copilot-response.png)

Beenden Sie die Debugsitzung in Visual Studio Code, wenn Sie mit dem Testen fertig sind.