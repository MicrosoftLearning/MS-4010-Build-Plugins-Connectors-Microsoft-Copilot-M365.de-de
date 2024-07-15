---
lab:
  title: Übung 4 – Erweitern und Optimieren von Nachrichtenerweiterungen für die Verwendung mit Copilot für Microsoft 365
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Übung 4 – Erweitern und Optimieren von Nachrichtenerweiterungen für die Verwendung mit Copilot für Microsoft 365

In dieser Übung erweitern und optimieren Sie Ihre Nachrichtenerweiterung für die Verwendung mit Copilot für Microsoft 365. Sie fügen einen neuen Parameter namens Zielgruppe hinzu und aktualisieren die Nachrichtenerweiterungslogik, um mehrere Parameter zu verarbeiten. Zum Schluss debuggen Sie Ihre Nachrichtenerweiterung und testen sie in Copilot in Microsoft Teams.

## Aufgabe 1 – Aktualisieren des App-Manifests

Die Angabe kurzer und präziser Beschreibungen im App-Manifest stellt sicher, dass Copilot weiß, wann und wie Ihr Plug-In aufgerufen werden soll. Aktualisieren Sie die App-, Befehls- und Parameterbeschreibungen im App-Manifest.

Öffnen Sie Visual Studio:

1. Öffnen Sie im Ordner **appPackage** die Datei mit dem Namen **manifest.json**.
1. Aktualisieren der **Beschreibung** des Objekts

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    Wenn Sie dem Befehl einen weiteren Parameter hinzufügen werden, aktualisieren Sie auch die Befehlsbeschreibung, um den neuen Parameter einzuschließen.

1. Aktualisieren Sie im **Befehle**-Array die **Beschreibung** des Befehls.

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    Fügen Sie nun einen neuen Parameter hinzu, den Copilot verwenden kann. Dieser neue Parameter hilft bei der Suche mit Copilot nach Produkten, die auf verschiedene Zielgruppen ausgerichtet sind, wie z. B. natürliche Personen und Unternehmen.

1. Fügen Sie im **Parameter**-Array den **TargetAudience**-Parameter nach dem **ProductName**-Parameter hinzu.

    ```json
    {    
        "parameters": [
            {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
            },
            {
                "name": "TargetAudience",
                "title": "Target audience",
                "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
                "inputType": "text"
            }
        ]
    }
    ```

1. Speichern Sie Ihre Änderungen.

Die Beschreibung des **TargetAudience**-Parameters beschreibt, was er ist und erläutert, dass der Parameter **Consumer**- oder **Enterprise**-Werte akzeptieren soll.

## Aufgabe 2 – Aktualisieren der Nachrichtenerweiterungslogik

Um den neuen Parameter und komplexe Eingabeaufforderungen zu unterstützen, aktualisieren Sie die OnTeamsMessagingExtensionQueryAsync-Methode im Bot-Aktivitätshandler, damit mehrere Parameter verarbeitet werden können.

Angenommen, Sie geben Eingabeaufforderung "Contoso-Produkte für Personen mit dem Namen Mark8 suchen" ein. Aufgrund der Parameterbeschreibungen wird "für Personen" in **Consumer** übersetzt und als Wert des **TargetAudience**-Parameters übergeben. "Mark8" wird als Wert des **ProductName**-Parameters übergeben.

Fortsetzen in Visual Studio:

1. **Öffnen Sie im Suche**-Ordner die Datei mit dem Namen **SearchApp.cs**.
1. Suchen Sie in der **OnTeamsMessagingExtensionQueryAsync**-Methode den folgenden Codeblock:

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. Aktualisieren Sie den Codeblock, um den Wert des **TargetAudience**-Parameter abzurufen, und erstellen Sie eine Filterabfrage, die beim Abfragen der SharePoint Online-Liste verwendet werden soll.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 3 – Bereitstellen von Ressourcen

Führen Sie den Prozess Teams App Abhängigkeiten vorbereiten aus, um Ressourcen bereitzustellen.

Fortsetzen in Visual Studio:

1. Klicken Sie im **Projektmappen-Explorer** mit der rechten Maustaste auf das Projekt **MsgExtProductSupport**
1. Erweitern Sie das Menü **Teams Toolkit**, wählen Sie **Teams App-Abhängigkeiten vorbereiten**
1. Wählen Sie im Dialog **Microsoft 365-Konto** die Option **Fortfahren**
1. Im Dialog **Bereitstellung** wählen Sie **Bereitstellung**
1. Wählen Sie im Dialog **Teams Toolkit Warnung** die Option **Bereitstellung**
1. Im **Teams Toolkit Informationen** Dialog, **Schließen** Sie die Eingabeaufforderung

## Aufgabe 4 – Ausführen und Debuggen

Starten Sie nun den Webdienst, und testen Sie die Nachrichtenerweiterung in Copilot für Microsoft 365.

1. Drücken Sie **F5**, um eine Debugging-Sitzung zu starten und ein neues Browserfenster zu öffnen, das zum Microsoft Teams-Webclient navigiert.
1. Geben Sie die Anmeldeinformationen für Ihr Microsoft 365-Konto ein und fahren Sie mit Microsoft Teams fort.
1. Wählen Sie im Installationsdialog der App **Hinzufügen**aus
1. Öffnen Sie die **Copilot** App aus Microsoft Teams
1. Öffnen Sie im Bereich zum Verfassen von Nachrichten das **Plug-In**-Flyout.
1. Aktivieren Sie in der Liste der Plug-Ins das **Contoso-Produkt**-Plug-In.
1. Geben Sie als Nachricht **Contoso-Produkte für Personen suchen** ein und senden Sie sie.
1. In der Copilot-Antwort wird eine Anmeldeschaltfläche zurückgegeben. Wählen Sie die Schaltfläche **Anmelden** aus, um sich zu authentifizieren.
1. Nachdem sie erfolgreich authentifiziert wurden, geben Sie als Nachricht **Contoso-Produkte für Personen suchen ** ein, und senden Sie sie.
1. In der Copilot-Antwort werden die in der Plug-In-Antwort zurückgegebenen Daten angezeigt, und das Plug-In, auf das in der Antwort verwiesen wird
1. Um die adaptive Karte anzuzeigen, die für das Ergebnis relevant ist, zeigen Sie mit der Maus auf die Verweise in der Copilot-Antwort.

Schließen Sie den Browser, um die Debugging-Sitzung zu beenden.

[Weiter zur Lab-Übersicht...](./6-summary.md)