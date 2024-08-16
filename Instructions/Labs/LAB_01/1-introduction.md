---
lab:
  title: Einführung
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Einführung

Mithilfe von Messaging-Erweiterungen können Benutzerinnen und Benutzer mit externen Systemen aus Microsoft Teams und Microsoft Outlook arbeiten. Benutzerinnen und Benutzer können Messaging-Erweiterungen verwenden, um Daten nachzuschlagen und zu ändern und die Informationen aus diesen Systemen in Nachrichten und E-Mails als umfangreich formatierte Karte freizugeben.

Angenommen, Sie haben eine SharePoint Online-Liste mit Produktinformationen, die aktuell und für Ihre Organisation relevant sind. Sie möchten diese Informationen in Microsoft 365 suchen und freigeben. Sie möchten auch, dass Copilot für Microsoft 365 diese Informationen in seinen Antworten verwendet.

![Screenshot der Startseite der SharePoint Online-Teamseite für Produktsupport. Es wird eine Liste der kürzlich veröffentlichten Produkte angezeigt.](../media/1-sharepoint-online-product-support-site.png)

In diesem Lab erstellen Sie eine Nachrichtenerweiterung. Ihre Messaging-Erweiterung verwendet einen Bot, um mit Microsoft Teams, Microsoft Outlook und Copilot für Microsoft 365 zu kommunizieren.

![Screenshot der Suchergebnisse, die von einer suchbasierten Messaging-Erweiterung in Microsoft Teams zurückgegeben werden.](../media/2-search-results-nuget.png)

Sie verwendet Microsoft Entra zum Authentifizieren von Benutzerinnen und Benutzern, die es ermöglichen, Daten aus SharePoint Online mithilfe der Microsoft Graph-API in deren Namen zurückzugeben.

![Screenshot einer Authentifizierungsaufforderung in einer suchbasierten Nachrichtenerweiterung. Es wird ein Link zur Anmeldung angezeigt.](../media/3-sign-in.png)

Nachdem sich die Benutzerinnen und Benutzer authentifiziert haben, erhält Ihre Messaging-Erweiterung Produktinformationen aus SharePoint Online mithilfe der Microsoft Graph-API. Sie gibt Suchergebnisse zurück, die als umfangreich formatierte Karte in Nachrichten und E-Mails eingebettet und dann freigegeben werden können.

![Screenshot der Suchergebnisse, die von einer suchbasierten Messaging-Erweiterung in Microsoft Teams zurückgegeben werden. Die Suchergebnisse werden von SharePoint Online zurückgegeben. Jedes Suchergebnis zeigt den Produktnamen, die Kategorie und das Produktbild an.](../media/4-search-results-sharepoint-online.png)

![Screenshot des Suchergebnisses, das in eine Nachricht in Microsoft Teams eingebettet ist. Die Suchergebnisse werden als adaptive Karte mit Produktname, Kategorie, Anrufvolumen und Veröffentlichungsdatum angezeigt. Es wird eine Aktionsschaltfläche mit dem Titel „Ansicht“ angezeigt, über die Benutzende zum Produktlistenelement in SharePoint Online navigieren können.](../media/5-adaptive-card.png)

Sie funktioniert mit Copilot für Microsoft 365 als Plug-In, sodass sie die SharePoint Online-Liste im Namen der Benutzerinnen und Benutzer abfragen und die zurückgegebenen Daten in ihren Antworten verwenden kann.

![Screenshot einer Antwort in Copilot für Microsoft 365, die Informationen enthält, die vom Plug-In für die Nachrichtenerweiterung zurückgegeben wurden. Es wird eine adaptive Karte mit Produktinformationen angezeigt.](../media/6-copilot-answer.png)

Am Ende dieses Labs sind Sie in der Lage, in C# geschriebene Nachrichtenerweiterungen (die auf .NET laufen) zu erstellen. Sie kann in Microsoft Teams, Microsoft Outlook und Copilot für Microsoft 365 verwendet werden. Sie kann Daten hinter geschützten APIs abfragen und die Ergebnisse als umfangreich formatierte Karten zurückgeben.

## Voraussetzungen

- C#-Grundkenntnisse
- Bicep-Grundkenntnisse
- Authentifizierungs-Grundkenntnisse 
- Administratorzugriff auf einen Microsoft 365-Mandanten
- Zugriff auf ein Azure-Abonnement
- Der Zugriff auf Copilot für Microsoft 365 ist optional und nur erforderlich, um eine Übung abzuschließen
- Visual Studio 2022 17.9 installiert mit [Teams Toolkit](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (Microsoft Teams-Entwicklungstools-Komponente)
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Wenn Sie bereit sind, zu beginnen, [fahren Sie mit der nächsten Übung fort...](./2-exercise-create-a-message-extension.md)
