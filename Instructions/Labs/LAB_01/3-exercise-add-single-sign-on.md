---
lab:
  title: Übung 2 – Einmaliges Anmelden hinzufügen
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Übung 2 – Einmaliges Anmelden hinzufügen

In dieser Übung fügen Sie der Nachrichtenerweiterung einmaliges Anmelden (Single Sign-On) hinzu, um Benutzerabfragen zu authentifizieren.

![Screenshot einer Authentifizierungsaufforderung in einer suchbasierten Nachrichtenerweiterung. Es wird ein Link zur Anmeldung angezeigt.](../media/2-sign-in.png)

### Übungsdauer

  - **Geschätzter Zeitaufwand**: 40 Minuten

## Aufgabe 1 – Konfigurieren der Registrierung von Back-End-API-Apps

Erstellen Sie zunächst eine Microsoft Entra-App-Registrierung für die Back-End-API. Für die Zwecke dieser Übung erstellen Sie eine neue Registrierung. In einer Produktionsumgebung würden Sie jedoch eine bestehende Registrierung verwenden.

In einem Browserfenster:

1. Navigieren Sie zum [Azure-Portal](https://portal.azure.com).

1. Öffnen Sie das Portalmenü und wählen Sie dann **Microsoft Entra ID** aus.

1. Wählen Sie **App-Registrierungen verwalten** aus, und wählen Sie dann **Neue Registrierung** aus.

1. Geben Sie im Formular Registrierung eines Antrags die folgenden Werte ein:

    1. **Name**: Produkt-API

    1. **Unterstützte Kontotypen**: Konten in jedem Organisationsverzeichnis (Jeder Microsoft Entra ID-Mandant - Multi-Mandant)

1. Wählen Sie **Registrieren** aus, um die App-Registrierung zu erstellen.

1. Wählen Sie im linken Menü der App-Registrierung die Option **Verwalten > Expose an API** aus.

1. Wählen Sie neben **Application ID URI** die Option **Hinzufügen** und **Speichern**, um eine neue Application ID URI zu erstellen.

1. Wählen Sie im Abschnitt „Von dieser API definierte Bereiche“ die Option **Einen Bereich hinzufügen**.

1. Geben Sie im Formular „Einen Bereich hinzufügen" die folgenden Werte an:

    1. **Bereichsname**: Product.Read

    1. **Wer kann zustimmen?**: Admins und Benutzer

    1. **Admin Zustimmung Anzeigename**: Produkte lesen

    1. **Beschreibung der Administratorzustimmung**: Ermöglicht der App das Lesen von Produktdaten

    1. **Benutzereinwilligung Anzeigename**: Produkte lesen

    1. **Benutzerzustimmung Beschreibung**: Erlaubt der App, Produktdaten zu lesen

    1. **Status**: Aktiviert

1. Wählen Sie die Schaltfläche **Bereich hinzufügen** aus, um den Bereich zu erstellen.

Notieren Sie sich als Nächstes die App-Registrierungs-ID und die Bereichs-ID. Sie benötigen diese Werte, um die App-Registrierung zu konfigurieren, die verwendet wird, um ein Zugriffs-Token für die Backend-API zu erhalten.

1. Wählen Sie im linken Menü der App-Registrierung **Manifest** aus.

1. Kopieren Sie den Wert der Eigenschaft **appId** und speichern Sie ihn zur späteren Verwendung.

1. Kopieren Sie den Eigenschaftswert der **oauth2Permissions.id** und speichern Sie ihn zur späteren Verwendung.

Da wir diese Werte im Projekt benötigen, fügen Sie sie der Umgebungsdatei hinzu.

In Visual Studio und dem **TeamsApp**-Projekt:

1. Öffnen Sie im Ordner **env** die Datei **.env.local**.

1. Fügen Sie in der Datei die folgenden Umgebungsvariablen hinzu und setzen Sie die Werte auf die **App-Registrierungs-ID** und **Scope-ID**, die Sie zuvor gespeichert haben:

    ```text
    BACKEND_API_ENTRA_APP_ID=<app-registration-id>
    BACKEND_API_ENTRA_APP_SCOPE_ID=<scope-id>
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 2 – Erstellen einer App-Registrierungsmanifestdatei für die Authentifizierung mit der Back-End-API

Um sich bei der Backend-API zu authentifizieren, benötigen Sie eine App-Registrierung, um ein Zugriffstoken zu erhalten, mit dem Sie die API aufrufen können.

Erstellen Sie als Nächstes eine App-Registrierungsmanifestdatei. Das Manifest definiert die API-Berechtigungsbereiche und die Umleitungs-URI für die App-Registrierung.

In Visual Studio und dem **TeamsApp**-Projekt:

1. Erstellen Sie im Ordner **infra\entra** eine neue Datei (<kbd>Strg+Shift+A</kbd>) mit dem Namen **entra.products.api.manifest.json**.

1. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```json
    {
      "id": "${{PRODUCTS_API_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{PRODUCTS_API_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-product-api-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [
        {
          "resourceAppId": "${{BACKEND_API_ENTRA_APP_ID}}",
          "resourceAccess": [
            {
              "id": "${{BACKEND_API_ENTRA_APP_SCOPE_ID}}",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. Speichern Sie die Änderungen.

Die Eigenschaft **requiredResourceAccess** spezifiziert die App-Registrierungs-ID und die Scope-ID der Back-End-API.

Die Eigenschaft **replyUrlsWithType** gibt den Redirect-URI an, der vom Bot Framework Token Service verwendet wird, um das Zugriffstoken nach der Benutzerauthentifizierung an den Token Service zurückzugeben.

Als Nächstes aktualisieren Sie den automatisierten Workflow zur Erstellung und Aktualisierung der App-Registrierung.

Im **TeamsApp**-Projekt:

1. Öffnen Sie **teamsapp.local.yml**.

1. Suchen Sie in der Datei den Schritt, der die Aktion **aadApp/update** verwendet.

1. Fügen Sie nach der Aktion die Aktionen **aadApp/create** und **aadApp/update** hinzu, um die App-Registrierung zu erstellen und zu aktualisieren (ab **Zeile 31**):

    ```yml
      - uses: aadApp/create
        with:
            name: ${{APP_INTERNAL_NAME}}-products-api-${{TEAMSFX_ENV}}
            generateClientSecret: true
            signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
            clientId: PRODUCTS_API_ENTRA_APP_ID
            clientSecret: SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET
            objectId: PRODUCTS_API_ENTRA_APP_OBJECT_ID
            tenantId: PRODUCTS_API_ENTRA_APP_TENANT_ID
            authority: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY
            authorityHost: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
            manifestPath: "./infra/entra/entra.products.api.manifest.json"
            outputFilePath : "./infra/entra/build/entra.products.api.${{TEAMSFX_ENV}}.json"
    ```

1. Speichern Sie Ihre Änderungen.

Die **aadApp/create** Aktion erstellt eine neue App-Registrierung mit dem angegebenen Namen, der Zielgruppe und generiert ein Client-Geheimnis. Die Eigenschaft **writeToEnvironmentFile** schreibt die Registrierungs-ID der App, das Client-Geheimnis, die Objekt-ID, die Tenant-ID, die Autorität und den Autoritäts-Host in die Umgebungsdateien. Das Client-Geheimnis wird verschlüsselt und sicher in der Datei **env.local.user** gespeichert. Dem Namen der Umgebungsvariablen für das Client-Geheimnis wird **SECRET_** vorangestellt. Damit wird Teams Toolkit angewiesen, den Wert nicht in die Protokolle zu schreiben.

Die **aadApp/update** Aktion aktualisiert die App-Registrierung mit der angegebenen Manifestdatei.

## Aufgabe 3 – Zentralisieren des Namens der Verbindungseinstellung

Als nächstes zentralisieren Sie den Namen der Verbindungseinstellung in der Umgebungsdatei und aktualisieren die App-Konfiguration, um zur Laufzeit auf den Wert der Umgebungsvariablen zuzugreifen.

Fortfahren in Visual Studio und im **TeamsApp**-Projekt:

1. Öffnen Sie im Ordner **env** die Datei **.env.local**.

1. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```text
    CONNECTION_NAME=ProductsAPI
    ```

1. Öffnen Sie **teamsapp.local.yml**.

1. Suchen Sie in der Datei den Schritt, der die Aktion **Datei/createOrUpdateJsonFile** verwendet, die auf die Datei **./appsettings.Development.json** zielt. Aktualisieren Sie das Inhaltsarray, um die Umgebungsvariable **CONNECTION_NAME** aufzunehmen und schreiben Sie den Wert in die Datei **appsettings.Development.json**:

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../ProductsPlugin/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Speichern Sie die Änderungen.

Als nächstes aktualisieren Sie die App-Konfiguration, um auf die Umgebungsvariable **CONNECTION_NAME** zuzugreifen.

Im **ProductsPlugin** Projekt:

1. Öffnen Sie **Config.cs**.

1. Fügen Sie in der Klasse **ConfigOptions** eine neue Eigenschaft namens **CONNECTION_NAME** hinzu:

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Speichern Sie die Änderungen.

1. Öffnen Sie **Program.cs**.

1. Aktualisieren Sie in der Datei den Code, der die App-Konfiguration liest, um die Eigenschaft **CONNECTION_NAME** aufzunehmen:

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["ConnectionName"] = config.CONNECTION_NAME;
    ```

1. Speichern Sie die Änderungen.

Als nächstes aktualisieren Sie den Bot-Code, um den Namen der Verbindungseinstellung zur Laufzeit zu verwenden.

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**

1. Erstellen Sie am Anfang der Klasse **SearchApp** einen Konstruktor, der ein **IConfiguration** Objekt akzeptiert und den Wert der Eigenschaft **CONNECTION_NAME** einem privaten Feld namens **Verbindungsname** zuweist:

    ```csharp
    private readonly string connectionName;
    public SearchApp(IConfiguration configuration)
    {
      connectionName = configuration["CONNECTION_NAME"];
    }  
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 4 – Konfigurieren der Verbindungseinstellung für die Produkte-API

Um sich mit der Back-End-API zu authentifizieren, müssen Sie eine Verbindungseinstellung in der Azure Bot-Ressource konfigurieren.

Fortfahren in Visual Studio und dem **TeamsApp**-Projekt:

1. Öffnen Sie im Ordner **infra** die Datei **azure.parameters.local.json**.

1. Fügen Sie in der Datei die Parameter **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** und **connectionName** hinzu:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        },
        "backendApiEntraAppClientId": {
          "value": "${{BACKEND_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientId": {
          "value": "${{PRODUCTS_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientSecret": {
          "value": "${{SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Speichern Sie die Änderungen.

Als Nächstes aktualisieren Sie die Bicep-Datei, um die neuen Parameter aufzunehmen, und übergeben sie an die Azure Bot-Ressource.

1. Öffnen Sie im Ordner **infra** die Datei namens **azure.local.bicep**.

1. Fügen Sie in der Datei nach der **botAppDomain** Parameterdeklaration die **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** und **connectionName** Parameterdeklarationen hinzu:

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. Fügen Sie in der **azureBotRegistration** Moduldeklaration die neuen Parameter hinzu:

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botEntraAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        backendApiEntraAppClientId: backendApiEntraAppClientId
        productsApiEntraAppClientId: productsApiEntraAppClientId
        productsApiEntraAppClientSecret: productsApiEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Speichern Sie die Änderungen.

Aktualisieren Sie schließlich die Bicep-Datei für die Bot-Registrierung, um die neue Verbindungseinstellung aufzunehmen.

1. Öffnen Sie im Ordner **infra/botRegistration** die Datei **azurebot.bicep**.

1. Fügen Sie in der Datei nach der **botAppDomain** Parameterdeklaration die **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** und **connectionName** Parameterdeklarationen hinzu:

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. Fügen Sie in der Datei eine neue Ressource namens **botServicesProductsApiConnection** am Ende der Datei hinzu:

    ```bicep
    resource botServicesProductsApiConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: productsApiEntraAppClientId
        clientSecret: productsApiEntraAppClientSecret
        scopes: 'api://${backendApiEntraAppClientId}/Product.Read'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${botEntraAppClientId}'
          }
        ]
      }
    }
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 5 – Konfigurieren der Authentifizierung in der Nachrichtenerweiterung

Um Benutzeranfragen in der Nachrichtenerweiterung zu authentifizieren, verwenden Sie das Bot Framework SDK, um ein Zugriffstoken für den Benutzenden vom Bot Framework Token Service zu erhalten. Das Zugriffstoken kann dann für den Zugriff auf Daten von einem externen Dienst verwendet werden.

Um den Code zu vereinfachen, erstellen Sie eine Hilfsklasse, die die Benutzerauthentifizierung behandelt.

Fortfahren in Visual Studio und dem **ProductsPlugin**-Projekt:

1. Erstellen Sie einen neuen Ordner mit dem Namen **Hilfsmethoden**.

1. Erstellen Sie im Ordner **Hilfsmethoden** eine neue Klassendatei namens **AuthHelpers.cs**.

1. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    using Microsoft.Bot.Schema.Teams;
    internal static class AuthHelpers
    {
        internal static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
        {
            var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);
            return new MessagingExtensionResponse
            {
                ComposeExtension = new MessagingExtensionResult
                {
                    Type = "auth",
                    SuggestedActions = new MessagingExtensionSuggestedAction
                    {
                        Actions = [
                            new() {
                                Type = ActionTypes.OpenUrl,
                                Value = resource.SignInLink,
                                Title = "Sign In",
                            },
                        ],
                    },
                },
            };
        }
        internal static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
        {
            var magicCode = string.Empty;
            if (!string.IsNullOrEmpty(state))
            {
                if (int.TryParse(state, out var parsed))
                {
                    magicCode = parsed.ToString();
                }
            }
            return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
        }
        internal static bool HasToken(TokenResponse tokenResponse) => tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
    }
    ```

1. Speichern Sie die Änderungen.

Die drei Hilfsmethoden in der Klasse **AuthHelpers** behandeln die Benutzerauthentifizierung in der Nachrichtenerweiterung.

- **CreateAuthResponse** Methode konstruiert eine Antwort, die einen Anmeldelink in der Benutzeroberfläche wiedergibt. Der Anmeldelink wird mit der Methode **GetSignInResourceAsync** vom Token-Dienst abgerufen.

- **GetToken** Methode verwendet den Token Service Client, um ein Zugriffstoken für den aktuellen Benutzer zu erhalten. Die Methode verwendet einen magischen Code, um die Authentizität der Anfrage zu überprüfen.

- **HasToken** Methode prüft, ob die Antwort des Token-Dienstes ein Zugriffstoken enthält. Wenn das Token nicht null oder leer ist, gibt die Methode true zurück.

Als nächstes aktualisieren Sie den Code der Nachrichtenerweiterung, um die Hilfsmethoden zur Authentifizierung von Benutzeranfragen zu verwenden.

1. Öffnen Sie im Ordner **Suchen** die Datei **SearchApp.cs**

1. Fügen Sie am Anfang der Datei die folgende using-Anweisung hinzu:

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** den folgenden Code am Anfang der Methode hinzu:

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

1. Speichern Sie die Änderungen.

Als Nächstes fügen Sie die Tokendienst-Domäne zur App-Manifestdatei hinzu, um sicherzustellen, dass der Client der Domäne vertrauen kann, wenn er einen Single Sign-On-Flow initiiert.

Im **TeamsApp**-Projekt:

1. Öffnen Sie im Ordner **appPackage** die Datei **manifest.json**.

1. Aktualisieren Sie in der Datei das Array ** validDomains**, und fügen Sie die Domain des Tokendiensts hinzu:

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 6 – Erstellen und Aktualisieren von Ressourcen

Wenn nun alles vorhanden ist, führen Sie den Prozess **Prepare Teams App Dependencies** aus, um neue Ressourcen zu erstellen und bestehende zu aktualisieren.

> [!NOTE]
> Wenn Ihre Bereitstellung die Abhängigkeiten nicht vorbereiten kann, stellen Sie sicher, dass Sie die richtigen Werte für **BACKEND_API_ENTRA_APP_ID** und **BACKEND_API_ENTRA_APP_SCOPE_ID** in **env.local** haben.

Fortsetzen in Visual Studio:

1. Klicken Sie im **Solution Explorer** mit der rechten Maustaste auf das Projekt **TeamsApp**.

1. Erweitern Sie das Menü **Teams Toolkit** und wählen Sie **Teams App-Abhängigkeiten vorbereiten**.

1. Wählen Sie im Dialog **Microsoft 365-Konto** die Option **Fortfahren**

1. Im Dialog **Bereitstellung** wählen Sie **Bereitstellung**

1. Wählen Sie im Dialog **Teams Toolkit Warnung** die Option **Bereitstellung**

1. Wählen Sie im Dialog **Teams Toolkit Informationen** das Kreuzsymbol aus, um den Dialog zu schließen.

## Aufgabe 7 - Ausführen und Debuggen

Starten Sie mit den bereitgestellten Ressourcen eine Debugsitzung, um die Nachrichtenerweiterung zu testen.

1. Um eine neue Debug-Sitzung zu starten, drücken Sie <kbd>F5</kbd> oder wählen Sie **Start** in der Symbolleiste.

1. Warten Sie, bis sich ein Browser-Fenster öffnet und der Dialog zur Installation der App im Microsoft Teams-Webclient angezeigt wird. Wenn Sie dazu aufgefordert werden, geben Sie die Anmeldeinformationen für Ihr Microsoft 365-Konto an.

1. Wählen Sie im Installationsdialog der App **Hinzufügen** aus.

1. Öffnen Sie einen neuen oder bestehenden Microsoft Teams-Chat.

1. Beginnen Sie im Bereich zum Verfassen von Nachrichten mit der Eingabe von **/apps**, um die App-Auswahl zu öffnen.

1. Wählen Sie in der Liste der Apps **Contoso-Produkte** aus, um die Messaging-Erweiterung zu öffnen.

1. Geben Sie in das Textfeld **Hallo** ein. Möglicherweise müssen Sie Ihre Suche mehrmals eingeben.

1. Es wird die Meldung **Sie müssen sich anmelden, um diese App zu verwenden** angezeigt:

    ![Screenshot einer Authentifizierungsaufforderung in einer suchbasierten Nachrichtenerweiterung. Es wird ein Link zur Anmeldung angezeigt.](../media/2-sign-in.png)

1. Folgen Sie dem Link **Anmelden**, um den Authentifizierungsvorgang zu starten.

1. Stimmen Sie den angeforderten Berechtigungen zu und kehren Sie zu Microsoft Teams zurück:

    ![Screenshot des Microsoft Entra API-Zustimmungsdialogs.](../media/18-api-permission-consent.png)

1. Warten Sie, bis die Suche abgeschlossen ist, und die Ergebnisse angezeigt werden.

1. Wählen Sie in der Ergebnisliste **Hallo**, um eine Karte in das Feld zum Verfassen einer Nachricht einzubetten.

Kehren Sie zu Visual Studio zurück und wählen Sie **Anhalten** in der Symbolleiste aus oder drücken Sie <kbd>Umschalt</kbd> + <kbd>F5</kbd>, um die Debug-Sitzung zu beenden.

[Fahren Sie mit der nächsten Übung fort...](./4-exercise-return-data-api.md)