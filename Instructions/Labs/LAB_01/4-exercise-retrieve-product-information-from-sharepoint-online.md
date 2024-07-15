---
lab:
  title: Übung 3 - Abrufen von Produktinformationen aus SharePoint Online
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Übung 3 - Abrufen von Produktinformationen aus SharePoint Online

In dieser Übung stellen Sie eine SharePoint Online-Website bereit und konfigurieren sie, die Produktinformationen als Elemente in einer Liste speichert. Sie aktualisieren den Code der Messaging-Erweiterung, um die Listenelemente aus SharePoint Online mit dem Microsoft Graph SDK abzurufen und die Daten der Listenelemente in den Suchergebnissen zurückzugeben. Schließlich führen Sie Ihre Messaging-Erweiterung aus und testen sie in Microsoft Teams.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Screenshot der Suchergebnisse, die von einer suchbasierten Messaging-Erweiterung in Microsoft Teams zurückgegeben werden. Die Suchergebnisse werden von SharePoint Online zurückgegeben. Jedes Suchergebnis zeigt den Produktnamen, die Kategorie und das Produktbild an." lightbox="../media/4-search-results-sharepoint-online.png":::

## Aufgabe 1 - Bereitstellung und Konfiguration der SharePoint-Website für das Produktmarketing

Beginnen Sie mit der Erstellung einer SharePoint Online-Site unter Verwendung des SharePoint Look Book Service.

In einem Webbrowser:

1. Gehen Sie zum **SharePoint Look Book** unter [https://lookbook.microsoft.com](https://lookbook.microsoft.com)
1. Erweitern Sie in der oberen Navigation **Designs anzeigen**
1. Erweitern Sie im Menü **Designs anzeigen** das Menü **Team** und wählen Sie **Produktunterstützung**.
1. Wählen Sie **Ihren Mandanten hinzufügen**
1. Wenn Sie dazu aufgefordert werden, melden Sie sich bei Ihrem Mandanten an.
1. Überprüfen Sie auf dem Zustimmungsbildschirm die erforderlichen Berechtigungen und wählen Sie **Akzeptieren**, um zum SharePoint Look Book Service zurückzukehren.
1. Übernehmen Sie im Formular die Standardeinstellungen und wählen Sie **Bereitstellung**

Eine E-Mail wird an Ihre E-Mail-Adresse gesendet, um Sie zu benachrichtigen, wenn die Bereitstellung der Website abgeschlossen ist.. Dieser Vorgang kann einige Minuten dauern.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="Screenshot der Homepage der SharePoint Online-Teamseite für Produktsupport. Es wird eine Liste der kürzlich veröffentlichten Produkte angezeigt." lightbox="../media/1-sharepoint-online-product-support-site.png":::

Um die Filterung der Spalten Titel und Einzelhandelskategorie bei der Abfrage der Liste mit Microsoft Graph API zu aktivieren, erstellen Sie Indizes für die Liste.

Fortsetzen im Webbrowser:

1. Gehen Sie zur **Produktsupport** Seite unter **<https://tenant.sharepoint.com/sites/productmarketing>**, ersetzen Sie **Mandant** mit dem Namen Ihrer SharePoint Online Instanz
1. Wählen Sie in der Leiste **Microsoft 365 Suite** das Zahnrad **Einstellungen**, um das Seitenfenster Einstellungen zu öffnen.
1. Wählen Sie unter der Überschrift **SharePoint** die Option **Websiteinhalte**.
1. Bewegen Sie in der Liste der Listen und Bibliotheken den Mauszeiger über die Liste **Produkte**, wählen Sie das Symbol **vertikale drei Punkte**, um das Menü **Aktionen anzeigen** zu erweitern, und wählen Sie dann **Einstellungen**.
1. Im Abschnitt **Spalten** wählen Sie unter der Liste der Spalten **Indexierte Spalten**.
1. Wählen Sie **Einen neuen Index erstellen**

## Aufgabe 2 - Hinzufügen von SharePoint-Hostnamen und Site-URL-Umgebungsvariablen

Als Nächstes legen wir den Hostnamen Ihrer SharePoint Online-Instanz und die URL der Produkt-Support-Site als Umgebungsvariablen fest. Sie geben dann die Werte als Umgebungsvariable zur Laufzeit frei und aktualisieren den Code, um den Wert zu lesen.

Öffnen Sie Visual Studio:

1. Öffnen Sie im Ordner **env** die Datei mit dem Namen **.env.local**.
1. Fügen Sie in der Datei die Umgebungsvariablen **SPO_HOSTNAME** und **SPO_SITE_URL** hinzu und ersetzen Sie **Mieter** durch den Namen Ihrer SharePoint Online-Instanz:

    ```text
    SPO_HOSTNAME=tenant.sharepoint.com
    SPO_SITE_URL=sites/productmarketing
    ```

1. Speichern Sie Ihre Änderungen.

Aktualisieren Sie dann die Aktion, um die Umgebungsvariablen in die Einstellungsdatei der Anwendung zu schreiben.

1. Öffnen Sie im Stammordner des Projekts die Datei **teamsapp.local.yml**
1. Suchen Sie den Schritt, der die Aktion **file/createOrUpdateJsonFile** verwendet, die auf die Datei **./appsettings.Development.json** abzielt
1. Aktualisieren Sie in der Datei das Array **Inhalt** und fügen Sie die Variablen **SPO_HOSTNAME** und **SPO_SITE_URL** hinzu:

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
            SPO_HOSTNAME: ${{SPO_HOSTNAME}}
            SPO_SITE_URL: ${{SPO_SITE_URL}}
    ```

1. Speichern Sie Ihre Änderungen.

Aktualisieren Sie nun die Klasse ConfigOptions, um die neuen Umgebungsvariablen aufzunehmen

1. Öffnen Sie im Projektstammordner Config.cs
1. Fügen Sie in der ConfigOptions-Klasse neue Zeichenfolgeneigenschaften mit den Namen SPO_HOSTNAME und SPO_SITE_URL hinzu

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
      public string SPO_HOSTNAME { get; set; }
      public string SPO_SITE_URL { get; set; }
    }
    ```

1. Speichern Sie Ihre Änderungen.

Aktualisieren Sie als Nächstes die App-Konfiguration mit den beiden Umgebungsvariablen.

1. Öffnen Sie im Stammordner des Projekts die Datei Program.cs
1. Fügen Sie neue Zeilen hinzu, um die Umgebungsvariablen SPO_HOSTNAME und SPO_SITE_URL als App-Konfigurationseinstellungen hinzuzufügen.

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    builder.Configuration["SPO_HOSTNAME"] = config.SPO_HOSTNAME;
    builder.Configuration["SPO_SITE_URL"] = config.SPO_SITE_URL;
    ```

1. Speichern Sie Ihre Änderungen.

Der letzte Schritt besteht darin, den Bot Activity Handler zu aktualisieren, um die Werte aus der App-Konfiguration zu lesen und den Wert in schreibgeschützten Eigenschaften zu speichern.

1. Öffnen Sie im Suchordner die Datei mit dem Namen SearchApp.cs
1. Erstellen Sie in der SearchApp-Klasse schreibgeschützte Zeichenfolgeneigenschaften namens spoHostname und spoSiteUrl.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
      private readonly string spoHostname;
      private readonly string spoSiteUrl;
    }
    ```

1. Aktualisieren Sie den Konstruktor, um die Eigenschaftswerte mithilfe der eingefügten App-Konfiguration festzulegen:

    ```csharp
    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    } 
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 3 – Suchbefehl aktualisieren

Wenn die Nachrichtenerweiterung Produktinformationen zurückgibt, aktualisieren Sie den Suchbefehlstitel und die Beschreibung, aktualisieren Sie auch den Parameternamen und seine Beschreibung.

Fortsetzen in Visual Studio:

1. Öffnen Sie im Ordner **appPackage** die Datei mit dem Namen **manifest.json**.
1. Aktualisieren Sie im Array **composeExtensions** das Befehlsobjekt mit:

    ```json
    "composeExtensions": [
      {
        "botId": "${{BOT_ID}}",
        "commands": [
          {
            "id": "Search",
            "type": "query",
            "title": "Products",
            "description": "Find products by name",
            "initialRun": false,
            "fetchTask": false,
            "context": [
              "commandBox",
              "compose",
              "message"
            ],
            "parameters": [
              {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
              }
            ]
          }
        ]
      }
    ],
    ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 4 – Abrufen des Benutzerabfragewerts

Wenn die Methode OnTeamsMessagingExtensionQueryAsync ausgeführt wird, wollen wir als erstes verstehen, was in das Suchfeld eingegeben wurde.

Zunächst entfernen wir den vorhandenen Code.

Fortsetzen in Visual Studio:

1. Öffnen Sie im **Suchordner** die Datei mit dem Namen **SearchApp.cs**.
1. Entfernen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** den gesamten Code **nach** der **If**-Anweisung, die auf ein Zugriffstoken überprüft.
1. Entfernen Sie in der **SearchApp**-Klasse die **FindPackages**-Methode und die **adaptiveCardFilePath**-Eigenschaft.

Nachdem Sie den vorhandenen Code entfernt haben, sollte Ihre **SearchApp**-Klasse dem folgenden Codeschnipsel entsprechen:

```csharp
public class SearchApp : TeamsActivityHandler
{
    private readonly string connectionName;
    private readonly string spoHostname;
    private readonly string spoSiteUrl;

    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    }

    protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
    {
        var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
        var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);

        if (!HasToken(tokenResponse))
        {
            return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
        }
    }
}
```

Als Nächstes erstellen wir den Code, um den Wert des **ProductName**-Parameters abzurufen.

1. Fügen Sie in der **OnTeamsMessagingExtensionQueryAsync**-Methode Code hinzu, um den Wert des **ProductName**-Parameters aus dem **Parameters**-Array im **MessagingExtensionQuery**-Objekt abzurufen.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    ```

1. Implementieren Sie in der **SearchApp**-Klasse die **GetQueryData**-Methode.

    ```csharp
    private static string GetQueryData(IList<MessagingExtensionParameter> parameters, string key)
    {
      if (parameters.Any() != true)
      {
        return string.Empty;
      }
    
      var foundPair = parameters.FirstOrDefault(pair => pair.Name == key);
      return foundPair?.Value?.ToString() ?? string.Empty;
    }
    ```

1. Speichern Sie Ihre Änderungen.

Die **GetQueryData-Methode** wird verwendet, um den einem bestimmten Schlüssel zugeordneten Wert aus einer Liste von **MessagingExtensionParameter**-Objekten abzurufen. Sie bietet eine bequeme Möglichkeit, Daten aus dem Parameters-Array im **MessagingExtensionQuery**-Objekt zu extrahieren.

## Aufgabe 5 – OData-Filter für SharePoint-Listenabfragen erstellen

Nachdem der übergebene Wert nun bekannt ist, verwenden Sie diesen, um einen OData-Abfragefilter zu erstellen. Der Filter wird verwendet, um die SharePoint Online-Liste nach der Spalte „Titel“ abzufragen, die den Produktnamen enthält.

Fortsetzen in Visual Studio:

1. Fügen Sie in der **OnTeamsMessagingExtensionQueryAsync**-Methode Code zum Erstellen der **FilterQuery**-Variable hinzu.

    ```csharp
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Speichern Sie Ihre Änderungen.

Der Code erstellt eine Filterabfrage basierend auf dem **name**-Parameter. Bei Angabe des „name“-Parameters wird, wird ein Filterausdruck erstellt, der nach Elementen mit einem **Title**-Feld sucht, beginnend mit dem angegebenen Namen. Wenn der „name“-Parameter nicht angegeben wird, weist er eine leere Zeichenfolge als Filterabfrage zu. Die resultierende Filterabfrage wird später im Code verwendet, um gefilterte Elemente von einer SharePoint-Website abzurufen.

## Aufgabe 6 – Installieren und Konfigurieren des Microsoft Graph SDK

Um authentifizierte Anforderungen an Microsoft Graph auszuführen, verwenden Sie das **Microsoft Graph SDK**.

Installieren Sie das Microsoft Graph SDK-Paket von NuGet und erstellen Sie dann eine **TokenProvider**-Klasse , mit der Sie das vom Tokendienst abgerufene Zugriffstoken verwenden und dann einen neuen **GraphServiceClient** initialisieren können.

Fortsetzen in Visual Studio:

1. Klicken Sie in Projektmappen-Explorer mit der rechten Maustaste auf das **MsgExtProductSupport**-Projekt.
1. Wählen Sie **NuGet-Pakete verwalten...** aus.
1. Wählen Sie die Registerkarte **Durchsuchen** aus, und suchen Sie nach **Microsoft.Graph**.
1. Wählen Sie in der Ergebnisliste **Microsoft Graph** aus.
1. Wählen Sie in der Dropdownliste **Version** die Option **5.42.0** aus.
1. Wählen Sie **Installieren** aus.
1. Klicken im Dialogfeld **Zustimmung zur Lizenz** auf **Ich stimme zu** und installieren Sie das SDK.

Erstellen Sie nach der Installation des Pakets einen Tokenanbieter für das Microsoft Graph SDK.

1. Erstellen Sie im **Suchordner** eine neue Datei mit dem Namen **TokenProvider.cs**.
1. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```csharp
    using Microsoft.Kiota.Abstractions.Authentication;
    
    namespace MsgExtProductSupport.Search
    {
       public class TokenProvider : IAccessTokenProvider
        {
            public string Token { get; set; }
            public AllowedHostsValidator AllowedHostsValidator => throw new NotImplementedException();
    
            public Task<string> GetAuthorizationTokenAsync(Uri uri, Dictionary<string, object>? additionalAuthenticationContext = null, CancellationToken cancellationToken = default)
            {
                return Task.FromResult(Token);
            }
        }
    }
    ```

1. Speichern Sie Ihre Änderungen.

Erstellen Sie nun eine Methode zum Erstellen einer neuen **GraphServiceClient**-Instanz.

1. Öffnen Sie im **Suchordner** die Datei mit dem Namen **SearchApp.cs**.
1. Importieren Sie in der Datei die benötigten Namespaces:

    ```csharp
    using Microsoft.Graph;
    using Microsoft.Kiota.Abstractions.Authentication;
    ```

1. Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** Code hinzu, um einen neuen Graph-Client zu erstellen, mit dem Sie Anfragen an Microsoft Graph senden können

    ```csharp
    var graphClient = CreateGraphClient(tokenResponse);
    ```

1. In der Klasse **SearchApp** implementieren Sie die Methode **CreateGraphClient**

    ```csharp
    private static GraphServiceClient CreateGraphClient(TokenResponse tokenResponse)
    {
      TokenProvider provider = new() { Token = tokenResponse.Token };
      var authenticationProvider = new BaseBearerTokenAuthenticationProvider(provider);
      var graphClient = new GraphServiceClient(authenticationProvider);
      return graphClient;
    }
    ```

1. Speichern Sie Ihre Änderungen.

Dieser Code richtet den Authentifizierungsanbieter ein und erstellt ein Clientobjekt, das zur Interaktion mit der Microsoft Graph-API unter Verwendung des bereitgestellten Zugriffstokens verwendet werden kann.

## Aufgabe 7 – Liste der Abfrageprodukte

Um die Elemente in der Liste Produkte abzufragen und später die Suchergebnisse zu erstellen, verwenden Sie den GraphServiceClient, um Anfragen zum Abrufen von Produktdaten aus SharePoint Online zu senden.

Fortsetzen in Visual Studio:

Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** Code hinzu, um die SharePoint-Daten zu erhalten:

  ```csharp
  var site = await GetSharePointSite(graphClient, spoHostname, spoSiteUrl, cancellationToken);
  var drive = await GetSharePointDrive(graphClient, site.SharepointIds.SiteId, "Product Imagery", cancellationToken);
  var items = await GetProducts(graphClient, site.SharepointIds.SiteId, filterQuery, cancellationToken);
  ```

Dieser Code leistet Folgendes:

- **Abfrage der Produktmarketing-Site**, das Site-Objekt enthält die ID der SharePoint-Site, die zur Rückgabe und Abfrage von Objekten auf der Site verwendet wird
- **Laufwerk für die Produktbilder abrufen**, das Laufwerk steht für die Dokumentenbibliothek, die die Produktbilder enthält. Sie verwenden das Laufwerk später, um die Produktbilder zu erhalten, die Sie in den Suchergebnissen anzeigen
- **Produkte abrufen**, mit denen Sie die Produktliste auf der Grundlage der Benutzerabfrage abfragen können

Implementieren Sie die drei Methoden in der **SearchApp**-Klasse.

- Implementieren Sie die **GetSharePointSite**-Methode

    ```csharp
    private static async Task<Site> GetSharePointSite(GraphServiceClient graphClient, string hostName, string siteUrl, CancellationToken cancellationToken)
    {
        return await graphClient.Sites[$"{hostName}:/{siteUrl}"].GetAsync(r => r.QueryParameters.Select = new string[] { "sharePointIds" }, cancellationToken);
    }
    ```

Diese Methode verwendet den GraphServiceClient, um eine Anfrage an Microsoft Graph zu senden und das Site-Objekt unter Verwendung eines Pfades zurückzugeben. Der Pfad wird aus der Kombination des SharePoint Online-Hostnamens und der Website-URL erstellt Da Sie nur die SharePointIds-Eigenschaftswerte benötigen, ist der Abfrageparameter Select so konfiguriert, dass nur diese Eigenschaft in der Antwort zurückgegeben wird.

- Implementierung der Methode **GetSharePointDrive**

    ```csharp
    private static async Task<Drive> GetSharePointDrive(GraphServiceClient graphClient, string siteId, string name, CancellationToken cancellationToken)
    {
        var drives = await graphClient.Sites[siteId].Drives.GetAsync(r => r.QueryParameters.Select = new string[] { "id", "name" }, cancellationToken);
        var drive = drives.Value.Find(d => d.Name == name);
        return drive;
    }
    ```

Diese Methode verwendet den GraphServiceClient und die Site-ID, um eine Sammlung von Dokumentbibliotheken von der Site zurückzugeben, wobei die Eigenschaften ID und Name jeder Dokumentbibliothek zurückgegeben werden. Die Sammlung der Bibliotheken wird dann gefiltert, um das Laufwerk zurückzugeben, das denselben Namen wie der Parameter der Methode name hat.

- Implementierung der Methode **GetProducts**

    ```csharp
    private static async Task<SiteCollectionResponse> GetProducts(GraphServiceClient graphClient, string siteId, string filterQuery, CancellationToken cancellationToken)
    {
        var fields = new string[]
        {
            "fields/Id",
            "fields/Title",
            "fields/RetailCategory",
            "fields/PhotoSubmission",
            "fields/CustomerRating",
            "fields/ReleaseDate"
        };
    
        var request = graphClient.Sites.WithUrl($"https://graph.microsoft.com/v1.0/sites/{siteId}/lists/Products/items?expand={string.Join(",", fields)}&$filter={filterQuery}");
        return await request.GetAsync(null, cancellationToken);
    }
    ```

Diese Methode verwendet den GraphServiceClient, um gefilterte Listenelemente aus der Liste Products unter Verwendung der übergebenen Filterabfrage zurückzugeben, und gibt die im Fields-Array definierten Listenelementdaten zurück.

## Aufgabe 8 – Erstellen von Suchergebnissen

Nachdem Sie die Produkte aus SharePoint abgerufen haben, erstellen Sie die Suchergebnisse, die an den Benutzenden zurückgegeben werden.

Die Erstellung der Suchergebnisse besteht aus der Iteration über das Array-Element, für jedes item erstellen Sie ein MessagingExtensionAttachment, das die Vorschau- und Inhaltskarten enthält.

Bevor Sie über die Elemente iterieren, erstellen Sie eine adaptive Kartenvorlage, die Sie in Ihrer Schleife verwenden.

Fortsetzen in Visual Studio:

1. Erstellen Sie im Ordner **Ressourcen** eine neue Datei namens **Product.json**

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.6",
      "body": [
        {
          "type": "TextBlock",
          "text": "${Product.Title}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${Product.RetailCategory}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${ProductImage}",
              "altText": "${Product.Title}"
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
              "value": "${formatNumber(Product.CustomerRating,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(Product.ReleaseDate,'dd/MM/yyyy')}"
            }
          ]
        },
        {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View",
              "url": "https://${SPOHostname}/${SPOSiteUrl}/Lists/Products/DispForm.aspx?ID=${Product.Id}"
            }
          ]
        }
      ]
    }
    ```

Die Vorlage verwendet Datenbindungsausdrücke, die beim Rendern der adaptiven Karte durch tatsächliche Werte ersetzt werden. Wenn die Karte gerendert wird, enthält sie einige Produktinformationen, ein Produktbild und eine Interaktive Schaltfläche. Die Interaktive Schaltfläche öffnet einen Browser, der zum Anzeigeformular der SharePoint-Liste für das Produktelement navigiert.

Fügen Sie als Nächstes den Code hinzu, um die JSON-Datei in eine Vorlage für adaptive Karten zu transformieren.

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**
1. Importieren Sie in der Datei die benötigten Namespaces:

    ```csharp
    using AdaptiveCards.Templating;
    ```

1. In der Methode **OnTeamsMessagingExtensionQueryAsync** fügen Sie den Code hinzu, um den Inhalt der JSON-Datei zu lesen und ein neues **AdaptiveCardTemplate**-Objekt zu erstellen.

    ```csharp
    var card = File.ReadAllText(@"Resources\Product.json");
    var template = new AdaptiveCardTemplate(card);
    ```

1. Speichern Sie Ihre Änderungen.

Als Nächstes erstellen Sie eine Schleife, um die Listenelemente zu durchlaufen. Jede Iteration führt folgendes aus:

- Deserialisieren der aktuellen Elementdaten in ein Produktobjekt
- Abrufen der Miniaturansicht des Produktbilds
- Erstellen einer Inhaltskarte
- Erstellen einer Vorschaukarte
- Erstellen einer MessagingExtensionAttachment-Kombination der Inhalts- und Vorschaukarten
- Hinzufügen des MessagingExtensionAttachment zur Liste

Sobald die Schleife beendet ist, haben Sie eine Liste von Anhängen, die an den Benutzenden zurückgegeben werden können.

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**
1. Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** Code hinzu, um eine neue Liste zu erstellen, um **MessagingExtensionAttachment**-Objekte darin zu speichern

    ```csharp
    var attachments = new List<MessagingExtensionAttachment>();
    ```

1. Erstellen Sie eine Foreachschleife, um die Listenelemente zu durchlaufen.

    ```csharp
    foreach (var item in items.Value) { 
            
    }
    ```

1. Fügen Sie den folgenden Code in die Foreachschleife ein.

    ```csharp
    var product = JsonConvert.DeserializeObject<Product>(item.AdditionalData["fields"].ToString());
    product.Id = item.Id;
    
    var thumbnails = await GetThumbnails(graphClient, drive.Id, product.PhotoSubmission, cancellationToken);
    
    var resultCard = template.Expand(new
    {
      Product = product,
      ProductImage = thumbnails.Large.Url,
      SPOHostname = spoHostname,
      SPOSiteUrl = spoSiteUrl,
    });
    
    var previewcard = new ThumbnailCard
    {
      Title = product.Title,
      Subtitle = product.RetailCategory,
      Images = new List<CardImage> { new() { Url = thumbnails.Small.Url } }
    }.ToAttachment();
    
    var attachment = new MessagingExtensionAttachment
    {
      Content = JsonConvert.DeserializeObject(resultCard),
      ContentType = AdaptiveCard.ContentType,
      Preview = previewcard
    };
    
    attachments.Add(attachment);
    ```

1. Speichern Sie Ihre Änderungen.

Um die Daten der Listenelemente stark zu typisieren, erstellen Sie ein Modell, das das **Produkt** darstellt.

1. Erstellen Sie im Stammordner des Projekts einen neuen Ordner namens **Modelle**
1. Erstellen Sie im Ordner **Modelle** eine neue Datei namens **Product.cs**

    ```csharp
    namespace MsgExtProductSupport.Models
    {
        public class Product
        {
            public string Title { get; set; }
            public string RetailCategory { get; set; }
            public Link Specguide { get; set; }
            public string PhotoSubmission { get; set; }
            public double CustomerRating { get; set; }
            public DateTime ReleaseDate { get; set; }
            public string Id { get; set; }
            public string ContentType { get; set; }
            public DateTime Modified { get; set; }
            public DateTime Created { get; set; }
        }
    
        public class Link
        {
            public string Description { get; set; }
            public string Url { get; set; }
        }
    }
    ```

1. Speichern Sie Ihre Änderungen.

Als nächstes implementieren Sie die Methode **GetThumbnails**, um Miniaturbilder aus Microsoft Graph für ein Produkt abzurufen.

1. Öffnen Sie im **Suchordner** die Datei mit dem Namen **SearchApp.cs**.
1. Erstellen Sie in der Klasse **SearchApp** die Methode **GetThumbnails**

    ```csharp
    private static async Task<ThumbnailSet> GetThumbnails(GraphServiceClient graphClient, string driveId, string photoUrl, CancellationToken cancellationToken)
    {
        var fileName = photoUrl.Split('/').Last();
        var driveItem = await graphClient.Drives[driveId].Root.ItemWithPath(fileName).GetAsync(null, cancellationToken);
        var thumbnails = await graphClient.Drives[driveId].Items[driveItem.Id].Thumbnails["0"].GetAsync(r => r.QueryParameters.Select = new string[] { "small", "large" }, cancellationToken);
        return thumbnails;
    }
    ```

1. Speichern Sie Ihre Änderungen.

Die Methode **GetThumbnails** verwendet den Thumbnails-Endpunkt in der Microsoft Graph API, um kleine und große Miniaturbilder des in SharePoint gespeicherten Produktbildes zurückzugeben.

## Aufgabe 9 – Suchergebnisse zurückgeben

Da wir nun über eine Sammlung von MessagingExtensionResult-Objekten verfügen, können wir sie als Suchergebnisse zurück an den Benutzenden zurückgeben.

- Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** Code hinzu, um die Suchergebnisse als Antwort der Messaging-Erweiterung zurückzugeben.

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

## Aufgabe 3: Bereitstellung der Ressourcen

Führen Sie den Prozess Teams App Abhängigkeiten vorbereiten aus, um Ressourcen bereitzustellen.

Fortsetzen in Visual Studio:

1. Klicken Sie im **Projektmappen-Explorer** mit der rechten Maustaste auf das Projekt **MsgExtProductSupport**
1. Erweitern Sie das Menü **Teams Toolkit**, wählen Sie **Teams App-Abhängigkeiten vorbereiten**
1. Wählen Sie im Dialog **Microsoft 365-Konto** die Option **Fortfahren**
1. Im Dialog **Bereitstellung** wählen Sie **Bereitstellung**
1. Wählen Sie im Dialog **Teams Toolkit Warnung** die Option **Bereitstellung**
1. Im **Teams Toolkit Informationen** Dialog, **Schließen** Sie die Eingabeaufforderung

## Aufgabe 11 – Ausführen und Debuggen

Starten Sie nun den Webdienst und testen Sie die Messaging-Erweiterung in Microsoft Teams.

Fortsetzen in Visual Studio:

1. Drücken Sie **F5**, um eine Debugging-Sitzung zu starten und ein neues Browserfenster zu öffnen, das zum Microsoft Teams-Webclient navigiert.
1. Geben Sie die Anmeldeinformationen für Ihr Microsoft 365-Konto ein und fahren Sie mit Microsoft Teams fort.
1. Wählen Sie im Installationsdialog der App **Hinzufügen**aus
1. Öffnen Sie einen neuen oder bestehenden Microsoft Teams-Chat
1. Wählen Sie im Bereich Nachrichten verfassen **...** aus sum das App-Flyout zu öffnen
1. Wählen Sie in der Liste der Apps **Contoso-Produkte**, um die Messaging-Erweiterung zu öffnen
1. Geben Sie **Mark8** in das Textfeld ein. Es werden zwei Ergebnisse angezeigt, **Mark8** und **Mark8 Controller**
1. Wählen Sie **Mark8**, um eine Karte in das Feld zum Verfassen einer Nachricht einzubetten
1. **Senden Sie die Nachricht**, die die Karte enthält
1. Wählen Sie in der übermittelten Karte die Schaltfläche **Ansehen**, um das SharePoint-Listenelement für das Produkt in der Liste Produkte in einer neuen Registerkarte anzuzeigen

Schließen Sie den Browser, um die Debugging-Sitzung zu beenden.

[Fahren Sie mit der nächsten Übung fort...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)