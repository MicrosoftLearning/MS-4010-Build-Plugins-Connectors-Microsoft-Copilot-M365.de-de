---
lab:
  title: Übung 4 – Testen des deklarativen Agenten mit API-Plug-In in Microsoft 365 Copilot
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Übung 4 – Testen des deklarativen Agenten mit API-Plug-In in Microsoft 365 Copilot

Durch das Erweitern eines deklarativen Agenten mit Aktionen können Daten abgerufen und aktualisiert werden, die in externen Systemen in Echtzeit gespeichert sind. Mithilfe von API-Plug-Ins können Sie über ihre APIs eine Verbindung mit externen Systemen herstellen, um Informationen abzurufen und zu aktualisieren.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1 – Testen des deklarativen Agents

Der letzte Schritt besteht darin, den deklarativen Agenten mit API-Plug-In in Microsoft 365 Copilot zu testen.

In Visual Studio Code:

1. Wählen Sie in der **Aktivitätsleiste** **Teams Toolkit** aus.
1. Stellen Sie im Abschnitt **Konten** sicher, dass Sie bei Ihrem Microsoft 365 Mandanten mit Microsoft 365 Copilot angemeldet sind.

  ![Screenshot des Bereichs der Teams Toolkit-Konten in Visual Studio Code.](../media/LAB_02/3-teams-toolkit-accounts.png)

1. Wählen Sie in der **Aktivitätsleiste** die Option **Ausführen und Debuggen**.
1. Wählen Sie die Konfiguration **In Copilot debuggen** und starten Sie das Debugging über die Schaltfläche **Debugging starten**.  

  ![Screenshot der Konfiguration des Debuggens in Copilot in Visual Studio Code.](../media/LAB_02/3-visual-studio-code-start-debugging.png)

1. Visual Studio Code erstellt und stellt Ihr Projekt in Ihrem Microsoft 365-Mandanten bereit und öffnet ein neues Webbrowserfenster.

Im Webbrowser:

1. Melden Sie sich bei Aufforderung mit dem Konto an, das zu Ihrem Microsoft 365-Mandanten mit Microsoft 365 Copilot gehört.
1. Wählen Sie in der Seitenleiste **Il Ristorante**.

  ![Screenshot der Microsoft 365 Copilot-Oberfläche mit dem ausgewählten Agenten „Il Ristorante“.](../media/LAB_02/3-copilot-select-agent.png)

1. Wählen Sie den Gesprächsanlass **Was gibt es heute zu Mittag?** und übermitteln Sie den Prompt.

  ![Screenshot der Microsoft 365 Copilot-Oberfläche mit dem Prompt für das Mittagessen.](../media/LAB_02/3-copilot-lunch-prompt.png)

1. Wenn Sie dazu aufgefordert werden, prüfen Sie die Daten, die der Agent an die API sendet, und bestätigen Sie mit der Schaltfläche **Einmal zulassen**.

  ![Screenshot der Microsoft 365 Copilot-Oberfläche mit der Bestätigung des Mittagessens.](../media/LAB_02/3-copilot-lunch-confirm.png)

1. Warten Sie, bis der Agent reagiert. Beachten Sie, dass zwar Zitate für die von der API abgerufenen Informationen angezeigt werden, das Popup aber nur den Titel des Gerichts anzeigt. Es werden keine zusätzlichen Informationen angezeigt, da das API-Plug-In keine Vorlage für adaptive Karten definiert.

  ![Screenshot der Microsoft 365 Copilot-Oberfläche mit der Antwort auf das Mittagessen.](../media/LAB_02/3-copilot-lunch-response.png)

1. Geben Sie eine Bestellung auf, indem Sie in das Prompt-Textfeld tippen: **1 x Spaghetti, 1 x Eistee** und senden Sie den Prompt ab.
1. Prüfen Sie die Daten, die der Agent an die API sendet und fahren Sie mit der Schaltfläche **Bestätigen** fort.

  ![Screenshot der Microsoft 365 Copilot-Schnittstelle mit der Bestellbestätigung.](../media/LAB_02/3-copilot-order-confirm.png)

1. Warten Sie, bis der Agent die Bestellung abgibt, und geben Sie die Bestellzusammenfassung zurück. Beachten Sie erneut, dass der Agent die Bestellzusammenfassung im Nur-Text anzeigt, da er keine Vorlage für adaptive Karten enthält.

  ![Screenshot der Microsoft 365 Copilot-Schnittstelle mit der Bestellantwort.](../media/LAB_02/3-copilot-order-response.png)

1. Wechseln Sie zurück zu Visual Studio Code und beenden Sie das Debuggen.
1. Wechseln Sie auf die Registerkarte **Terminal** und schließen Sie alle aktiven Terminals.

  ![Screenshot der Visual Studio Code-Terminalregisterkarte mit der Option zum Schließen aller Terminals.](../media/LAB_02/3-visual-studio-code-close-terminal.png)
