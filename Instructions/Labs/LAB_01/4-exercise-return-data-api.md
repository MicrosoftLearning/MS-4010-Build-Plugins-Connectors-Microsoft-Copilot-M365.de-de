---
lab:
  title: Übung 3 – Zurückgeben von Produktdaten aus der geschützten Microsoft Entra-API
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Übung 3 – Zurückgeben von Produktdaten aus der geschützten Microsoft Entra-API

In dieser Übung aktualisieren Sie die Nachrichtenerweiterung, um Daten aus einer benutzerdefinierten API abzurufen. Sie erhalten Daten aus der benutzerdefinierten API basierend auf der Benutzerabfrage und geben Daten in Suchergebnissen an den Benutzer bzw. die Benutzerin zurück.

![Screenshot der Suchergebnisse, die von einer suchbasierten Messaging-Erweiterung in Microsoft Teams zurückgegeben werden.](../media/3-search-results-api.png)

### Übungsdauer

  - **Geschätzter Zeitaufwand**: 50 Minuten

## Aufgabe 1 – Installieren und Konfigurieren von Dev Proxy

In dieser Übung verwenden Sie Dev Proxy, ein Befehlszeilentool, das APIs simulieren kann. Es ist nützlich, wenn Sie Ihre Anwendung testen wollen, ohne eine echte API erstellen zu müssen.

Um diese Übung abzuschließen, müssen Sie die [neueste Version von Dev Proxy](/microsoft-cloud/dev/dev-proxy/get-started) installieren und die Dev Proxy-Voreinstellung für dieses Modul herunterladen.

Die Voreinstellung simuliert eine CRUD-API (Create, Read, Update, Delete) mit einem speicherinternen Datenspeicher, der durch Microsoft Entra geschützt ist. Dies bedeutet, dass Sie Ihre App testen können, als ob sie eine echte API aufrufe, die eine Authentifizierung erfordert.

1. Um Dev Proxy zu installieren, öffnen Sie ein neues **Eingabeaufforderungsfenster als Administrator**:

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. Verwenden Sie den folgenden Befehl, um die Voreinstellungen herunterzuladen:

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. Lassen Sie das Konsolenfenster für spätern Gebrauch geöffnet.

## Aufgabe 2 – Abrufen des Benutzerabfragewerts

Erstellen Sie eine Methode, die den Benutzerabfragewert anhand des Namens des Parameters abruft.

In Visual Studio und dem Projekt **ProductsPlugin**:

1. Erstellen Sie im Ordner **Helpers** eine neue Datei namens **MessageExtensionHelpers.cs**.

1. Ersetzen Sie den Code in der Datei durch Folgendes:

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. Speichern Sie die Änderungen.

Als nächstes aktualisieren Sie die **OnTeamsMessagingExtensionQueryAsync**-Methode in der SearchApp-Klasse, um die neue Hilfsmethode zu verwenden.

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**

1. Ersetzen Sie in der **OnTeamsMessagingExtensionQueryAsync**-Methode den folgenden Code:

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   durch

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. Bewegen Sie den Cursor auf die Variable **Text**, verwenden Sie `Ctrl + R`, `Ctrl + R`, und benennen Sie die Variable in **Name** um.

1. Drücken Sie die **EINGABETASTE**, um die Variable in 3 Dateien umzubenennen.

1. Speichern Sie die Änderungen.

Die **OnTeamsMessagingExtensionQueryAsync**-Methode sollte nun wie in diesem Bild aussehen:

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## Aufgabe 3 – Abrufen von Daten aus der benutzerdefinierten API

Um Daten aus der benutzerdefinierten API abzurufen, müssen Sie das Zugriffstoken im Autorisierungsheader der Anforderung senden und die Antwort in ein Modell deserialisieren, das die Produktdaten darstellt.

Erstellen Sie zunächst ein Modell, das die Produktdaten darstellt, die von der benutzerdefinierten API zurückgegeben werden.

In Visual Studio und dem Projekt **ProductsPlugin**:

1. Erstellen Sie einen Ordner mit dem Namen **Models**.

1. Erstellen Sie im Ordner **Models** eine neue Datei namens **Product.cs**

1. Ersetzen Sie in der Datei den vorhandenen Code durch den folgenden:

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. Speichern Sie die Änderungen.

Erstellen Sie als Nächstes eine Dienstklasse, die die Produktdaten aus der benutzerdefinierten API abruft.

In Visual Studio und dem Projekt **ProductsPlugin**:

1. Erstellen Sie einen Ordner mit dem Namen **Dienste**.

1. Erstellen Sie im Ordner **Services** eine neue Datei namens **ProductService.cs**.

1. Ersetzen Sie in der Datei den vorhandenen Code durch den folgenden:

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. Speichern Sie die Änderungen.

Die **ProductsService**-Klasse enthält Methoden zum Abrufen von Produktdaten aus der benutzerdefinierten API. Der Klassenkonstruktor verwendet ein Zugriffstoken als Parameter und richtet eine **HttpClient**-Instanz mit dem Zugriffstoken im Autorisierungsheader ein.

Als nächstes aktualisieren Sie die Methode **OnTeamsMessagingExtensionQueryAsync**, um die Klasse **ProductsService** zu verwenden und um Produktdaten von der benutzerdefinierten API zu erhalten.

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**

1. Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** den folgenden Code nach der Variablendeklaration **Name** hinzu, um Produktdaten von der benutzerdefinierten API abzurufen:

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. Speichern Sie die Änderungen.

## Aufgabe 4 – Erstellen von Suchergebnissen

Nachdem Sie nun über die Produktdaten verfügen, können Sie sie in die Suchergebnisse einfügen, die an den Benutzer bzw. die Benutzerin zurückgegeben werden.

Als Erstes aktualisieren wir die vorhandene Vorlage für adaptive Karten, um die Produktinformationen anzuzeigen.

Fortfahren in Visual Studio und im **ProductsPlugin**-Projekt :

1. Im Ordner **Resources** benennen Sie **card.json** in **Product.json** um.

1. Erstellen Sie im Ordner **Ressourcen** eine neue Datei namens **Product.data.json** Diese Datei enthält Beispieldaten, die Visual Studio zum Generieren einer Vorschau der Vorlage für adaptive Karten verwendet.

1. Fügen Sie der Datei den folgenden JSON-Code hinzu.

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. Speichern Sie die Änderungen.

1. Öffnen Sie im Ordner **Resources** die Datei **Product.json**.

1. Ersetzen Sie in der Datei den Inhalt durch den folgenden JSON-Code:

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. Speichern Sie die Änderungen.

Das Adaptive Card template verwendet Bindungsausdrücke, um die Produktinformationen anzuzeigen. The **\$\{name\}**, **\$\{category\}**, **\$\{imageUrl\}**, **\$\{callVolume\}**, und **\$\{releaseDate\}** werden durch die entsprechenden Werte aus den Produktdaten ersetzt. Die Vorlagenfunktionen **formatNumber** und **formatDateTime** werden verwendet, um die Werte **callVolume** und **releaseDate** in eine Zahl bzw. ein Datum zu formatieren.

Nehmen Sie sich einen Moment Zeit, um die Vorschau der adaptiven Karte in Visual Studio zu erkunden. Die Vorschau zeigt, wie die Vorlage für adaptive Karten aussieht, wenn die Produktdaten an die Vorlage gebunden sind. Sie verwendet die Beispieldaten aus der **Product.data.json**-Datei, um die Vorschau zu generieren.

Als Nächstes aktualisieren Sie die Eigenschaft **validDomains** im App-Manifest, um die Domäne **raw.githubusercontent.com** einzuschließen, damit die Bilder in der Vorlage für adaptive Karten in Microsoft Teams angezeigt werden können.

Im **TeamsApp**-Projekt:

1. Öffnen Sie im Ordner **appPackage** die Datei **manifest.json**.

1. Fügen Sie in der Datei die GitHub-Domäne zur **validDomains**-Eigenschaft hinzu:

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. Speichern Sie die Änderungen.

Als Nächstes aktualisieren Sie die Methode **OnTeamsMessagingExtensionQueryAsync**, um eine Liste von Anhängen zu erstellen, die die Produktinformationen enthalten.

Im **ProductsPlugin** Projekt:

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**

1. Aktualisieren Sie **card.json** auf **Product.json**, um die Änderung des Dateinamens widerzuspiegeln. Ersetzen Sie in Zeile 34 den folgenden Code:

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   durch

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. Fügen Sie den folgenden Code nach der **Vorlage**-Variablendeklaration ein, um eine Liste von Anhängen zu erstellen:

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. Aktualisieren Sie die Anweisung **return**, um die Variable **attachments** einzubeziehen:

   ```csharp
   return new MessagingExtensionResponse
   {
       ComposeExtension = new MessagingExtensionResult
       {
           Type = "result",
           AttachmentLayout = "list",
           Attachments = attachments
       }
   };
   ```

1. Änderungen speichern

## Aufgabe 5 – Erstellen und Aktualisieren von Ressourcen

Wenn nun alles vorhanden ist, führen Sie den Prozess **Prepare Teams App Dependencies** aus, um neue Ressourcen zu erstellen und bestehende zu aktualisieren.

Fortsetzen in Visual Studio:

1. Klicken Sie im **Solution Explorer** mit der rechten Maustaste auf das Projekt **TeamsApp**.

1. Erweitern Sie das Menü **Teams Toolkit** und wählen Sie **Teams App-Abhängigkeiten vorbereiten**.

1. Wählen Sie im Dialog **Microsoft 365-Konto** die Option **Fortfahren**

1. Im Dialog **Bereitstellung** wählen Sie **Bereitstellung**

1. Wählen Sie im Dialog **Teams Toolkit Warnung** die Option **Bereitstellung**

1. Wählen Sie im Dialog **Teams Toolkit Informationen** das Kreuzsymbol aus, um den Dialog zu schließen.

## Aufgabe 6 – Ausführen und Debuggen

Starten Sie mit den bereitgestellten Ressourcen eine Debugsitzung, um die Nachrichtenerweiterung zu testen.

Starten Sie zunächst Dev Proxy, um die benutzerdefinierte API zu simulieren.

1. Führen Sie in dem **Eingabeaufforderungsfenster**, das Sie noch geöffnet haben, den folgenden Befehl aus, um Dev Proxy zu starten:

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. Wenn Sie dazu aufgefordert werden, akzeptieren Sie alle Zertifikatswarnungen.

> [!NOTE]
> Wenn Dev Proxy ausgeführt wird, fungiert er als systemweiter Proxy.

Starten Sie anschließend eine Debug-Sitzung in Visual Studio:

1. Um eine neue Debug-Sitzung zu starten, drücken Sie <kbd>F5</kbd> oder wählen Sie **Start** in der Symbolleiste.

1. Warten Sie, bis sich ein Browser-Fenster öffnet und der Dialog zur Installation der App im Microsoft Teams-Webclient angezeigt wird. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen für Ihr Microsoft 365-Konto an.

1. Wählen Sie im Installationsdialog der App **Hinzufügen** aus.

1. Öffnen Sie einen neuen oder bestehenden Microsoft Teams-Chat.

1. Geben Sie im Bereich zum Verfassen von Nachrichten **/apps** ein, um die App-Auswahl zu öffnen.

1. Wählen Sie in der Liste der Apps **Contoso-Produkte** aus, um die Messaging-Erweiterung zu öffnen.

1. Geben Sie **mark8** in das Textfeld ein. Möglicherweise müssen Sie Ihre Suchabfrage mehrmals eingeben.

1. Warten Sie, bis die Suche abgeschlossen ist, und die Ergebnisse angezeigt werden.

    ![Screenshot der Suchergebnisse, die von einer suchbasierten Messaging-Erweiterung in Microsoft Teams zurückgegeben werden.](../media/3-search-results-api.png)

1. Wählen Sie in der Ergebnisliste ein Suchergebnis aus, um eine Karte in das Feld „Nachricht verfassen“ einzubetten.

Kehren Sie zu Visual Studio zurück und wählen Sie **Anhalten** aus der Symbolleiste aus oder drücken Sie <kbd>Umschalt</kbd> + <kbd>F5</kbd>, um die Debug-Sitzung zu beenden. Schalten Sie außerdem Dev Proxy mit <kbd>Strg</kbd> + <kbd>C</kbd> aus.

[Fahren Sie mit der nächsten Übung fort...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)