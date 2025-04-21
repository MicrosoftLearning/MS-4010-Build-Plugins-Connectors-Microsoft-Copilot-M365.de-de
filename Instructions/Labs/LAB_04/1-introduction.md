---
lab:
  title: Einführung
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Einführung

Mit Microsoft 365 Copilot-Agents können Sie KI-gestützte Assistenten erstellen, die für bestimmte Szenarien optimiert sind. Mithilfe von Anweisungen definieren Sie den Kontext für den Copiloten und konfigurieren dessen Tonfall oder wie er reagieren soll. Durch die Konfiguration des Wissens des Agenten gewähren Sie ihm Zugriff auf externe Daten, die nicht Teil des Large Language Model (LLM) sind, sodass er präzisere Antworten geben kann. 

## Beispielszenario

Angenommen, Sie arbeiten in einer IT-Abteilung in einer großen Organisation. Ihre Organisation standardisiert die IT über verschiedene Richtlinien, die sie in einem spezialisierten System speichert. Sie und Ihre Teammitglieder in der IT-Abteilung erhalten regelmäßig Anfragen, die in Richtlinien behandelt werden. Das Nachschlagen von Antworten im Richtlinienverwaltungssystem ist zeitaufwändig. Sie möchten Ihrer Organisation einen KI-gestützten Assistenten zur Verfügung stellen, der die Fragen Ihrer Teammitglieder anhand von zuverlässigen Informationen aus den Richtlinien beantworten kann.

## Lernziele

Am Ende dieses Moduls sind Sie in der Lage, deklarative Agents für Microsoft 365 Copilot zu erstellen. Sie werden verstehen, wie Sie ihre Anweisungen konfigurieren, um sie für ein bestimmtes Szenario zu optimieren. Sie erfahren außerdem, wie Sie diese mit Microsoft Graph-Connectors integrieren können, um ihnen Zugriff auf externe Daten zu gewähren, die nicht Teil des LLM von Microsoft 365 Copilot sind.

## Voraussetzungen

- Kenntnisse darüber, was Microsoft 365 Copilot ist und wie es auf Anfängerniveau funktioniert
- Kenntnisse zum Erstellen eines deklarativen Microsoft 365 Copilot-Agents
- Kenntnisse über den Aufbau eines Graph-Connectors
- Microsoft 365-Mandant mit Microsoft 365 Copilot und Mandantenadminrechten
- [Visual Studio Code](https://code.visualstudio.com/) mit der installierten Erweiterung [Teams-Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
