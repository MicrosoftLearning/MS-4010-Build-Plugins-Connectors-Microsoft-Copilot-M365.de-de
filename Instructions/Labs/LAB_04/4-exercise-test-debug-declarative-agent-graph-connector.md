---
lab:
  title: 'Übung 3: Testen und Debuggen'
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Übung 3: Testen und Debuggen

In dieser Übung testen und implementieren Sie Ihren deklarativen Agents in Microsoft 365 und testen ihn mit Microsoft 365 Copilot Chat.

### Übungsdauer

- **Geschätzter Zeitaufwand**: 5 Minuten

## Aufgabe 1 – Testen des deklarativen Agenten in Microsoft 365 Copilot

Um Ihren deklarativen Agent zu testen, stellen Sie ihn als App für Ihren Microsoft 365-Mandanten bereit. Überprüfen Sie nach dem Öffnen in Microsoft 365 Copilot, ob er wie vorgesehen funktioniert.

In Visual Studio Code:

1. Öffnen Sie in der **Aktivitätsleiste** (Seitenleiste) die Erweiterung **Teams-Toolkit**.
1. Wählen Sie im Bereich **Lebenszyklus** die Option **Bereitstellung**. Das Teams-Toolkit packt das deklarative Agent-Projekt als App und lädt es in Microsoft 365 hoch.
1. Öffnen Sie einen Webbrowser.

In einem Webbrowser:

1. Navigieren Sie zu [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat).
1. Melden Sie sich mit Ihrem Arbeitskonto an, das zu Ihrem Microsoft 365-Mandanten gehört.
1. Wählen Sie im Microsoft 365 Copilot im Bedienfeld den Agenten **Contoso IT-Richtlinien**, um ihn zu aktivieren.
1. Fragen Sie in dem Chat-Textfeld: `What's the acceptable use policy at Contoso?`.
1. Warten Sie, bis der Agent reagiert. Beachten Sie, dass die Antwort Verweise auf externe Inhalte enthält, die der Graph-Connector aufgenommen hat. Die URL in jedem Verweis verweist auf den Ort im externen System, an dem der Inhalt gespeichert ist.

    ![Screenshot des Microsoft 365 Copilot, der auf den Prompt einer Person reagiert.](../media/LAB_04/3-copilot-response.png)