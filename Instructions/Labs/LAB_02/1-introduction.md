---
lab:
  title: Einführung
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Einführung

Mit Microsoft 365 Copilot-Agents können Sie KI-gestützte Assistenten erstellen, die für bestimmte Szenarien optimiert sind. Mithilfe von Anweisungen definieren Sie den Kontext für den Agent und geben Einstellungen an, z. B. Tonfall oder Reaktionsweise. Indem Sie die Fähigkeiten des Agents konfigurieren, können Sie mit externen Systemen interagieren, bestimmte Verhaltensweisen unter Systembedingungen auslösen oder benutzerdefinierte Workflowlogik verwenden. Eine Art von Fähigkeiten sind Aktionen, die es einem deklarativen Agent ermöglichen, mit APIs zu kommunizieren, um Daten abzurufen und zu ändern.

![Diagramm, das die Anatomie eines deklarativen Agents für Microsoft 365 Copilot zeigt.](../media/LAB_02/1-anatomy-declarative-agent.png)

## Beispielszenario

Angenommen, Sie arbeiten in einer Organisation, in der Sie regelmäßig Lebensmittel aus einem lokalen Restaurant bestellen. Das Restaurant arbeitet mit einem täglichen Menü, das sie im Internet veröffentlichen. Sie möchten schnell sehen können, welche Kurse verfügbar sind, aber auch Allergene für den Fall, dass Sie Gäste haben. Das Restaurant macht ihr Menü über eine API verfügbar. Anstatt eine separate App zu erstellen, möchten Sie die Informationen in Microsoft 365 Copilot integrieren, damit Sie die verfügbaren Gerichte, die Sie bestellen können, leicht finden und ihre Zutaten herausfinden können. Sie möchten natürliche Sprache verwenden, um durch das Menü und die Bestellung zu navigieren.

## Wie werden wir vorgehen?

In diesem Modul erstellen Sie eine Aktion für einen deklarativen Agent mit einem API-Plug-In. Die Aktion ermöglicht es dem Agent, mithilfe seiner anonymen API mit einem externen System zu interagieren. Folgendes wird beschrieben:

- **Erstellen**: Erstellen Sie ein API-Plug-In, das eine Verbindung zu einer anonymen API herstellt.
- **Konfigurieren**: Konfigurieren Sie das API-Plug-In, um die Daten aus der API anzuzeigen.
- **Erweitern**: Erweitern Sie einen deklarativen Agent mit einer Aktion unter Verwendung eines API-Plug-Ins.
- **Bereitstellung**: Laden Sie Ihren deklarativen Agent in Microsoft 365 Copilot hoch und validieren Sie die Ergebnisse.

![Screenshot eines deklarativen Agents, der auf einen Benutzer bzw. auf eine Benutzerin mit Informationen aus einer externen API reagiert.](../media/LAB_02/1-agent-response-api-plugin.png)

## Übungsdauer

- **Geschätzter Zeitaufwand**: 35 Minuten

## Lernziele

Am Ende dieses Moduls wissen Sie, wie Sie deklarative Agents mit API-Plug-Ins integrieren, die mit anonymen APIs verbunden sind, damit sie in Echtzeit mit externen Systemen interagieren können.