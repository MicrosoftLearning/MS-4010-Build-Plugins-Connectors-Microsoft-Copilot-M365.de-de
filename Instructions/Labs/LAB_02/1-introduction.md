---
lab:
  title: Einführung
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Einführung

Angenommen, Sie haben ein externes System, in dem Sie Wissensdatenbank-Artikel speichern. Diese Artikel enthalten Informationen über verschiedene Prozesse in Ihrem Unternehmen. Sie möchten in der Lage sein, relevante Informationen aus Microsoft 365 leicht zu finden und zu entdecken. Sie möchten auch, dass Copilot für Microsoft 365 Informationen aus diesen Wissendsdatenbank-Artikeln in seine Antworten einbezieht.

Um diese externen Informationen innerhalb von Microsoft 365 verfügbar zu machen, erstellen Sie einen benutzerdefinierten Microsoft Graph-Connector. Microsoft Graph-Connectors stellen eine Verbindung zu Ihrem externen System her (1), um Inhalte abzurufen, verwenden die Informationen von Microsoft Entra ID zur Authentifizierung bei Microsoft 365 (2) und importieren die Inhalte über die Microsoft Graph-API (3) in Microsoft 365.

:::image type="content" source="../media/1-graph-connector-concept.png" alt-text="Diagramm, das die konzeptionelle Arbeit eines Microsoft Graph-Connectors zeigt.":::

In diesem Modul erfahren Sie, was Microsoft Graph-Connectors sind und warum Sie sie in Ihrer Organisation verwenden sollten. Sie erstellen einen Microsoft Graph-Connector, der lokale Markdowndateien in Microsoft 365 importiert. Sie erfahren auch, wie Sie sicherstellen können, dass die von Ihnen importierten externen Inhalte nur für Personen mit den entsprechenden Berechtigungen zugänglich sind. Schließlich optimieren Sie Ihren Microsoft Graph Connector für die Verwendung mit Copilot für Microsoft 365.

## Voraussetzungen

- C#-Grundkenntnisse
- Authentifizierungs-Grundkenntnisse 
- Zugriff auf einen [Microsoft 365-Mandanten](https://developer.microsoft.com/microsoft-365/dev-program?ocid=MSlearn)
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Wenn Sie bereit sind, zu beginnen, [fahren Sie mit der nächsten Übung fort...](./2-exercise-configure-connection-schema.md)