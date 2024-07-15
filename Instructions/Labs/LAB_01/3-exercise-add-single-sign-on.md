---
lab:
  title: Übung 2 – Einmaliges Anmelden hinzufügen
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Übung 2 – Einmaliges Anmelden hinzufügen

In dieser Übung aktualisieren Sie die Messaging-Erweiterung so, dass Benutzerinnen und Benutzer aufgefordert werden, sich anzumelden und sich zu authentifizieren. Sie konfigurieren die Bot Microsoft Entra-App-Registrierung und das App-Manifest, um einmaliges Anmelden zu aktivieren. Sie konfigurieren eine Microsoft Entra-App-Registrierung für die Authentifizierung bei Microsoft Graph und aktualisieren die Messaging-Erweiterungslogik, um ein Zugriffstoken mithilfe des Bot Framework-Tokendiensts abzurufen. Anschließend führen Sie Ihre Massaging-Erweiterung aus, um sie in Microsoft Teams zu testen.

## Aufgabe 1 – Konfigurieren des einmaligen Anmeldens

Konfigurieren Sie zunächst eine Bot Microsoft Entra-App-Registrierung.

In Visual Studio:

1. Öffnen Sie im Ordner **infra\entra** die Datei mit dem Namen **entra.bot.manifest.json**
1. Aktualisieren Sie in der Datei das Array **identifierUris**, um den Anwendungs-ID-URI festzulegen.

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ]
    ```

1. Aktualisieren Sie in der Datei das Array **oauth2Permissions**, um einen Bereich zu erstellen, der es Teams erlaubt, Web-APIs als Administratorin oder Administrator oder Benutzerin oder Benutzer aufzurufen:

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. Aktualisieren Sie in der Datei das Array ** preAuthorizedApplications**, um Microsoft Teams, Microsoft Outlook und Copilot für Microsoft 365-Clients zur Liste der autorisierten Clients hinzuzufügen:

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ]
    ```

1. Speichern Sie Ihre Änderungen.

Aktualisieren Sie als Nächstes die App-Manifestdatei, um die Ressource zu definieren, die der Client beim Initiieren eines Single Sign-On-Flows in der App verwenden soll.

Fortsetzen in Visual Studio:

1. Öffnen Sie im Ordner **appPackage** die Datei mit dem Namen **manifest.json**.
1. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    }
    ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 2 – Erstellen der Microsoft Entra-App-Registrierungsmanifestdatei für Microsoft Graph

Um sich bei Microsoft Graph zu authentifizieren, erstellen Sie eine neue App-Registrierungsmanifestdatei.

Fortsetzen in Visual Studio:

1. Erstellen Sie im Ordner **infra\entra** eine Datei mit dem Namen **entra.graph.manifest.json**
2. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
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
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
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

1. Speichern Sie Ihre Änderungen.

Das Array **requiredResourceAccess-** definiert die API-Berechtigungsbereiche, und das Array **replyUrlsWithType** definiert die Umleitungs-URIs für die App-Registrierung.

Aktualisieren Sie nun die Projektdatei mit Aktionen, um die App-Registrierung zu erstellen.

1. Öffnen Sie im Projektstammordner **teamsapp.local.yml**
1. Suchen Sie in der Datei den Schritt, der die Aktion **arm/deploy** verwendet.
1. Fügen Sie vor dem Schritt den folgenden Code hinzu:

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 3 – Erstellen einer OAuth-Verbindungseinstellung

Azure KI Bot Service-Verbindungseinstellungen werden zum Verwalten der Benutzerauthentifizierung in Bots und Messaging-Erweiterungen verwendet.

Zentralisieren Sie zunächst den Namen der OAuth-Verbindungseinstellung, die zum Erstellen der Verbindungseinstellung als Umgebungsvariable verwendet wird, und fügen Sie dann Code hinzu, um den Umgebungsvariablenwert zur Laufzeit zu verwenden.

Fortsetzen in Visual Studio:  

1. Öffnen Sie im Ordner **env** die Datei **.env.local**
1. Fügen Sie in der Datei  den folgenden Code hinzu:

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. Öffnen Sie im Stammordner des Projekts die Datei **teamsapp.local.yml**
1. Suchen Sie in der Datei den Schritt, in dem die Aktion**file/createOrUpdateJsonFile** für die Datei **./appsettings.Development.json** verwendet wird.
1. Aktualisieren Sie das Inhaltsarray, indem Sie die Variable **CONNECTION_NAME** hinzufügen

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Speichern Sie Ihre Änderungen.
1. Öffnen Sie im Projektstammordner **Config.cs**
1. Fügen Sie in der Klasse **ConfigOptions** eine neue Zeichenfolgeneigenschaft mit dem Namen **CONNECTION_NAME** hinzu

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Speichern Sie Ihre Änderungen.
1. Öffnen Sie im Projektstammordner die Datei mit dem Namen **Program.cs**
1. Fügen Sie in der Datei eine neue Zeile hinzu, um die Umgebungsvariable ** CONNECTION_NAME** als App Configuration-Einstellung hinzuzufügen:

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

Aktualisieren Sie als Nächstes den Bot-Aktivitätshandler, um auf die App Configuration zuzugreifen.

1. Öffnen Sie im **Suchordner** die Datei mit dem Namen **SearchApp.cs**.
1. Erstellen Sie in der Klasse **SearchApp** eine schreibgeschützte Zeichenfolgeneigenschaft mit dem Namen **connectionName**.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. Erstellen Sie einen Konstruktor, der die App Configuration eingibt, und den Wert der connectionName-Eigenschaft festlegt.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. Speichern Sie Ihre Änderungen.

Als nächstes aktualisieren Sie die Bicep-Dateien, um die OAuth-Verbindungseinstellung bereitzustellen.

Aktualisieren Sie zunächst die Parameterdatei, um die Anmeldeinformationen für die Registrierung der Microsoft Graph Microsoft Entra-App und den Namen der Verbindungseinstellung zu übergeben.

1. Öffnen Sie im infra **Ordner** die Datei mit dem Namen **azure.parameters.local.json**
1. In den Parametern **Objekt** fügen Sie die Parameter **graphEntraAppClientId**, **graphEntraAppClientSecret** und **Verbindungsname** hinzu

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
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Speichern Sie Ihre Änderungen.

Update der Bicep-Datei.

1. Öffnen Sie im Ordner **infra** die Datei mit dem Namen **azure.local.bicep**
1. Fügen Sie in der Datei nach der Parameterdeklaration **botAppDomain** die Parameterdeklarationen **graphEntraAppClientId**, **graphEntraAppClientSecret** und **connectionName** hinzu

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. Fügen Sie im Modul **azureBotRegistration** die Parameter **graphEntraAppClientId**, **graphEntraAppClientSecret** und **connectionName** hinzu

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Speichern Sie die Änderungen.

Aktualisieren Sie schließlich die Bicep-Datei für die Bot-Registrierung.

1. Öffnen Sie im Ordner **infra/botRegistration** die Datein amens **azurebot.bicep**
1. Fügen Sie in der Datei nach der Parameterdeklaration **botAppDomain** die Parameterdeklarationen **graphEntraAppClientId**, **graphEntraAppClientSecret** und **connectionName** hinzu

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. Fügen Sie nach der Ressource **botServiceM365ExtensionsChannel** eine neue Ressource für die Microsoft Graph-Verbindung hinzu

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 4 – Authentifizieren von Benutzerabfragen

Als Nächstes fügen Sie Code hinzu, um Benutzer zu authentifizieren, wenn sie eine Suche über die Messaging-Erweiterung starten.

Fortsetzen in Visual Studio:

1. Öffnen Sie im **Suchordner** die Datei mit dem Namen **SearchApp.cs**.
1. Beginnen Sie in der Datei mit dem Hinzufügen des **Authentifizierungs**-Namespaces aus dem Bot Framework SDK.

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. Fügen Sie in der Methode **OnTeamsMessagingExtensionQueryAsync** den folgenden Code am Anfang der Methode hinzu:

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

Der obige Codeblock verwendet drei Methoden für die Benutzerauthentifizierung.

- **GetToken** verwendet den Tokendienst-Client, um ein Zugriffstoken für den aktuellen Benutzenden zu erhalten
- **HasToken** prüft, ob die Antwort des Tokendienstes ein Zugriffstoken enthält
- **CreateAuthResponse** wird aufgerufen, wenn kein Token zurückgegeben wird und liefert eine Antwort, die einen Anmeldelink in der Benutzeroberfläche anzeigt

Erstellen Sie nun die Methoden in der Klasse **SearchApp**.

- Erstellen Sie die Methode **GetToken** mit folgendem Code:

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
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
```

Zunächst wird geprüft, ob der Parameter state nicht null oder leer ist. Der Code versucht dann, ihn mit der Methode int.TryParse als Ganzzahl zu analysieren. Wenn die Analyse erfolgreich ist, wird der analysierte Wert der magicCode-Variablen als Zeichenfolge zugewiesen. Der magicCode wird dann zusammen mit anderen Parametern als Argument an die GetUserTokenAsync-Methode übergeben. Die GetUserTokenAsync-Methode verwendet den magicCode, um die Authentizität der Anforderung zu überprüfen. Sie stellt sicher, dass das Benutzer-Token von derselben Stelle angefordert wird, die den Authentifizierungsprozess eingeleitet hat.

- Erstellen Sie die HasToken-Methode mithilfe des folgenden Codes:

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

Sie überprüfen, ob ein gültiges Token vom Tokendienst erhalten wurde, indem Sie prüfen, ob die Token-Antwort und die Token-Eigenschaft der Antwort nicht leer oder null ist.

- Erstellen Sie die **CreateAuthResponse-Methode** mit dem folgenden Code:

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

Zunächst verwenden Sie die Methode **GetSignInResourceAsync**, um den Anmeldelink vom Tokendienst abzurufen. Der Anmeldelink wird verwendet, um ein **MessagingExtensionResponse** Objekt zu erstellen. Sie erstellen ein neues Objekt und setzen die **ComposeExtension** Eigenschaft der Antwort auf ein neues **MessagingExtensionResult** Objekt. Die Typ-Eigenschaft des Ergebnisses wird auf „auth“ gesetzt, was bedeutet, dass das Ergebnis eine Authentifizierungsantwort ist. Die **SuggestedActions**-Eigenschaft des Ergebnisses wird auf ein neues **MessagingExtensionSuggestedAction** Objekt gesetzt. Die Eigenschaft Actions der vorgeschlagenen Aktionen wird auf eine Liste gesetzt, die ein einzelnes **CardAction**-Objekt enthält. Dieses **CardAction**-Objekt stellt eine Aktion dar, die vom Benutzer ausgeführt werden kann. Die Eigenschaft Type der **CardAction** ist auf **ActionTypes.OpenUrl** gesetzt, was bedeutet, dass es sich um eine Aktion handelt, die eine URL öffnet. Die Eigenschaft Wert wird auf den von der Ressource abgerufenen Anmeldelink gesetzt. Die Eigenschaft Titel wird auf „Anmelden“ gesetzt, was den Titel der Aktion angibt. Schließlich wird die konstruierte Antwort von der Methode zurückgegeben.

Wenn ein Benutzender dem Anmeldelink folgt, wird er zu einer Ressource weitergeleitet, die auf einer externen Domain gehostet wird. Die Domain muss in der Manifestdatei der Anwendung enthalten sein. Fügen Sie die Bot Framework Token Service-Domain zum App-Manifest hinzu.

Fortsetzen in Visual Studio:

1. Öffnen Sie im Ordner **appPackage**die Option **manifest.json**
1. Aktualisieren Sie in der Datei das Array ** validDomains**, und fügen Sie die Domain des Tokendiensts hinzu:

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 5: Bereitstellen der Ressourcen

Führen Sie jetzt den Prozess „Teams-App-Abhängigkeiten vorbereiten“ aus, um die erforderlichen Ressourcen bereitzustellen.

Fortsetzen in Visual Studio:

1. Klicken Sie im **Projektmappen-Explorer** mit der rechten Maustaste auf das Projekt **MsgExtProductSupport**
1. Erweitern Sie das Menü **Teams Toolkit**, wählen Sie **Teams App-Abhängigkeiten vorbereiten**
1. Wählen Sie im Dialog **Microsoft 365-Konto** die Option **Fortfahren**
1. Im Dialog **Bereitstellung** wählen Sie **Bereitstellung**
1. Wählen Sie im Dialog **Teams Toolkit Warnung** die Option **Bereitstellung**
1. Wählen Sie im Dialogfeld **Informationen zum Teams-Toolkit** die Option **Bereitgestellte Ressourcen anzeigen**  aus, um ein neues Browserfenster zu öffnen.

Nehmen Sie sich einen Moment Zeit, um die Ressourcen zu erkunden, die in Azure erstellt und aktualisiert werden.

## Aufgabe 6 – Ausführen und Debuggen

Starten Sie nun den Webdienst und testen Sie die Messaging-Erweiterung in Microsoft Teams.

Fortsetzen in Visual Studio:

1. Drücken Sie **F5**, um eine Debugging-Sitzung zu starten und ein neues Browserfenster zu öffnen, das zum Microsoft Teams-Webclient navigiert.
1. Geben Sie im Browser und bei Bedarf Ihre Anmeldeinformationen für Ihr Microsoft 365-Konto ein, und fahren Sie mit Microsoft Teams fort.
1. Wählen Sie im Installationsdialog der App **Hinzufügen**aus
1. Öffnen Sie einen neuen oder bestehenden Microsoft Teams-Chat
1. Wählen Sie im Bereich Nachrichten verfassen **...** aus sum das App-Flyout zu öffnen
1. Wählen Sie in der Liste der Apps **Contoso-Produkte**, um die Messaging-Erweiterung zu öffnen
1. Geben Sie in das Textfeld **Bot Builder** ein, um eine Suche zu starten
1. Wählen Sie in der Ergebnisliste die Option **Ein Ergebnis auswählen** aus, um eine Karte in das Feld zum Verfassen einer Nachricht einzubetten
1. Eine Meldung, **Sie müssen sich anmelden, um diese App zu verwenden**, wird angezeigt.
1. Wählen Sie den **Anmeldelink** aus, um eine neue Registerkarte zu öffnen und den Authentifizierungs-Flow zu starten.
1. Überprüfen Sie auf der Seite „Berechtigungsgenehmigung“ die angeforderten Berechtigungen.
1. Wählen Sie *Annehmen* aus, um die Registerkarte zu schließen und zu Microsoft Teams zurückzukehren.
1. Wählen Sie im Bereich Nachrichten verfassen **...** aus sum das App-Flyout zu öffnen
1. Wählen Sie in der Liste der Apps **Contoso-Produkte**, um die Messaging-Erweiterung zu öffnen
1. Geben Sie in das Textfeld **Bot Builder** ein, um eine Suche zu starten
1. Sie werden zur Anmeldung aufgefordert. Folgen Sie erneut dem **Anmeldelink**, um die Suche zu starten.
1. Wählen Sie in der Ergebnisliste die Option **Ein Ergebnis auswählen** aus, um eine Karte in das Feld zum Verfassen einer Nachricht einzubetten

Schließen Sie den Browser, um die Debugging-Sitzung zu beenden.

[Fahren Sie der nächsten Übung fort...](./4-exercise-retrieve-product-information-from-sharepoint-online.md)