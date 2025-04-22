---
lab:
  title: 'Übung 2: Erstellen eines deklarativen Agents und Integrieren des Graph-Connectors'
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Übung 2: Erstellen eines deklarativen Agents und Integrieren des Graph-Connectors

In dieser Übung erstellen Sie einen neuen deklarativen Agent von Grund auf neu und konfigurieren ihn so, dass er die externe Verbindung als Grounding-Quelle verwendet.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1: Erstellen eines neuen deklarativen Agents

Eine Möglichkeit zum Erstellen eines deklarativen Agents ist die Verwendung des Teams-Toolkits. Das Teams-Toolkit bietet ein Vorlagenprojekt zum Erstellen deklarativer Agents, das Ihnen einen hervorragenden Ausgangspunkt für die Konfiguration der Einstellungen Ihres Agenten und einschließlich zusätzlicher Funktionen bietet.

Um einen neuen deklarativen Agent zu erstellen, öffnen Sie Visual Studio Code.

In Visual Studio Code:

1. Öffnen Sie in der **Aktivitätsleiste** (Seitenleiste) die Erweiterung **Teams-Toolkit**.
1. Wählen Sie im Teams-Toolkit-Bereich die Schaltfläche **Neue App erstellen**.
1. Im Dialog **Neues Projekt** wählen Sie die Option **Agent**.
1. Im nächsten Dialog wählen Sie die Option **Deklarativer Agent**.
1. Fügen Sie kein Plug-In hinzu, indem Sie die Option **Kein Plug-In** wählen.
1. Wählen Sie einen Ordner aus, in dem Sie das Projekt auf Ihrem Computer speichern möchten.
1. Nennen Sie das Projekt `da-it-policies`.

## Aufgabe 2: Konfigurieren von deklarativen Agent-Anweisungen

Das Teams-Toolkit erstellt ein neues deklaratives Agent-Projekt. Um ihn auf Ihr Szenario zu beschränken, aktualisieren Sie die Beschreibung und Anweisungen des Agents.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/declarativeAgent.json**.
1. Aktualisieren Sie den Wert der Eigenschaft **Name** auf `Contoso IT policies`.
1. Aktualisieren Sie den Wert der Eigenschaft **Beschreibung** auf `Assistant specialized in Contoso IT policies`.
1. Speichern Sie die Änderungen.
1. Der aktualisierte Dateiinhalt sieht wie folgt aus:

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. Öffnen Sie die Datei **appPackage/instruction.txt**.
1. Aktualisieren Sie den Inhalt auf:

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 3: Integrieren des Microsoft Graph-Connectors in einen deklarativen Agent

Nachdem Sie einen deklarativen Agent erstellt haben, müssen Sie ihn als Nächstes in einen Microsoft Graph-Connector integrieren, damit er auf externe Daten zugreifen kann.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/declarativeAgent.json**.
1. Fügen Sie nach der Eigenschaft **instructions** eine neue Eigenschaft mit dem Namen **capabilities** und dem folgenden Code hinzu:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. Geben Sie in der Eigenschaft **connection_id** `policieslocal` als ID der externen Verbindung an. `policieslocal` ist die ID der externen Verbindung, die der Graph-Connector in den vorherigen Schritten erstellt hat.
1. Der aktualisierte Dateiinhalt sieht wie folgt aus:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. Speichern Sie die Änderungen.
