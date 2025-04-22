---
lab:
  title: 'Übung 3: Integrieren eines API-Plug-Ins mit einer mit OAuth gesicherten API'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Übung 3: Integrieren eines API-Plug-Ins mit einer mit OAuth gesicherten API

API-Plug-Ins für Microsoft 365 Copilot ermöglichen ihnen die Integration mit APIs, die mit OAuth gesichert sind. Sie behalten die Client-ID und den geheimen Schlüssel der App bei, die Ihre API schützen, indem Sie sie im Teams-Tresor registrieren. Zur Runtime führt Microsoft 365 Copilot Ihr Plug-In aus, ruft die Informationen aus dem Tresor ab und verwendet sie, um ein Zugriffstoken zu erhalten und die API aufzurufen. Wenn Sie diesem Prozess folgen, bleiben die Client-ID und der geheime Schlüssel sicher und werden niemals dem Client offengelegt.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1: Öffnen des Beispielprojekts und Überprüfen der Authentifizierungskonfiguration

Laden Sie zunächst das Beispielprojekt herunter:

1. Navigieren Sie hierzu in einem Webbrowser zu [https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs). Sie erhalten eine Aufforderung zum Herunterladen einer ZIP-Datei mit dem Beispielprojekt.
1. Speichern Sie die ZIP-Datei auf Ihrem Computer.
1. Extrahieren Sie den Inhalt der ZIP-Datei.
1. Öffnen Sie in Visual Studio Code den Ordner .

Das Beispielprojekt ist ein Teams-Toolkit-Projekt, das einen deklarativen Agent, ein API-Plug-In und eine mit Microsoft Entra ID gesicherte API enthält. Die API wird auf Azure Functions ausgeführt und implementiert Sicherheit mithilfe der integrierten Authentifizierungs- und Autorisierungsfunktionen von Azure Functions, die manchmal als Easy Auth bezeichnet werden.

## Aufgabe 2: Überprüfen der OAuth2-Autorisierungskonfiguration

Bevor Sie fortfahren, überprüfen Sie die OAuth2-Autorisierungskonfiguration im Beispielprojekt.

### Untersuchen der API-Definition

Sehen Sie sich zunächst die Sicherheitskonfiguration der API-Definition an, die im Projekt enthalten ist.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/apiSpecificationFile/repair.yml**.
1. Beachten Sie im Abschnitt **components.securitySchemes** die Eigenschaft **oAuth2AuthCode**:

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  Die Eigenschaft definiert ein OAuth2-Sicherheitsschema und enthält Informationen zu den URLs, die aufgerufen werden sollen, um ein Zugriffstoken abzurufen und welche Bereiche die API verwendet.

  > **WICHTIG** Beachten Sie, dass der Geltungsbereich mit der Anwendungs-ID-URI (**api://...**) vollständig qualifiziert ist. Bei der Arbeit mit Microsoft Entra müssen Sie benutzerdefinierte Geltungsbereiche vollständig qualifizieren. Wenn Microsoft Entra einen nicht qualifizierten Bereich sieht, wird davon ausgegangen, dass er zu Microsoft Graph gehört, was zu Autorisierungsflowfehlern führt.

1. Suchen Sie die Eigenschaft **paths./repairs.get.security**. Beachten Sie, dass auf das Sicherheitsschema **oAuth2AuthCode** verwiesen wird, das der Client zur Ausführung des Vorgangs benötigt.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **WICHTIG** Die Auflistung der erforderlichen Bereiche in der API-Spezifikation dient lediglich zu Informationszwecken. Bei der Implementierung der API sind Sie dafür verantwortlich, das Token zu validieren und zu überprüfen, ob es die erforderlichen Berechtigungen enthält.

### Untersuchen der API-Implementierung

Sehen Sie sich als Nächstes die API-Implementierung an.

In Visual Studio Code:

1. Öffnen Sie die Datei **src/functions/repairs.ts**.
1. Suchen Sie in der Handler-Funktion **repairs** die folgende Zeile, die überprüft, ob die Anfrage ein Zugriffstoken mit den erforderlichen Berechtigungen enthält:

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. Die Funktion **hasRequiredScopes** wird in der Datei **repairs.ts** weiter implementiert:

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  Die Funktion beginnt mit dem Extrahieren des Bearertokens aus dem Autorisierungsanforderungsheader. Anschließend wird das Paket **jwt-decode** verwendet, um das Token zu entschlüsseln und die Liste der Bereiche aus dem Anspruch **scp** abzurufen. Abschließend wird überprüft, ob der Anspruch **scp** alle erforderlichen Bereiche enthält.

  Beachten Sie, dass die Funktion das Zugriffstoken nicht überprüft. Stattdessen wird nur überprüft, ob das Zugriffstoken die erforderlichen Bereiche enthält. In dieser Vorlage wird die API auf Azure Functions ausgeführt und die Sicherheit mithilfe von Easy Auth implementiert, das für die Validierung des Zugriffstokens zuständig ist. Wenn die Anforderung kein gültiges Zugriffstoken enthält, lehnt sie die Azure Functions-Runtime ab, bevor sie Ihren Code erreicht. Während Easy Auth das Token validiert, überprüft es nicht die erforderlichen Bereiche. Dies müssen Sie selbst tun.

### Überprüfen der Konfiguration der Tresor-Aufgabe

In diesem Projekt verwenden Sie das Teams-Toolkit, um die OAuth-Informationen zum Tresor hinzuzufügen. Das Teams-Toolkit registriert die OAuth-Informationen im Tresor mithilfe einer speziellen Aufgabe in der Konfiguration des Projekts.

In Visual Studio Code:

1. Öffnen Sie die Datei **./teampsapp.local.yml**.
1. Suchen Sie im Abschnitt **Bereitstellung** die Aufgabe **oauth/register**.

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  Die Aufgabe übernimmt die Werte der Projektvariablen **TEAMS_APP_ID**, **AAD_APP_CLIENT_ID** und **SECRET_AAD_APP_CLIENT_SECRET**, die in den Dateien **env/.env.local** und **env/.env.local.user** gespeichert sind, und registriert sie im Tresor. Als zusätzliche Sicherheitsmaßnahme ermöglicht es auch Proof Key for Code Exchange (PKCE). Dann nimmt er die ID des Tresoreintrags und schreibt sie in die Umgebungsdatei **env/.env.local**. Das Ergebnis dieser Aufgabe ist eine Umgebungsvariable namens **OAUTH2AUTHCODE_CONFIGURATION_ID**. Das Teams-Toolkit schreibt den Wert dieser Variable in die Datei **appPackages/ai-plugin.json**, die die Plug-In Definition enthält. Zur Runtime verwendet der deklarative Agent, der das API-Plug-In lädt, diese ID, um die OAuth-Informationen aus dem Tresor abzurufen und den Auth-Flow zu starten, um ein Zugriffstoken zu erhalten.

  > **WICHTIG** Die Aufgabe **oauth/register** ist nur für die Registrierung der OAuth-Informationen im Tresorraum zuständig, wenn diese noch nicht vorhanden sind. Wenn die Informationen bereits vorhanden sind, überspringt das Teams-Toolkit die Ausführung dieser Aufgabe.

1. Als nächstes suchen Sie die Aufgabe **oauth/update**.

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  Die Aufgabe speichert die OAuth-Informationen im Tresor, die mit Ihrem Projekt synchronisiert werden. Es ist erforderlich, dass Ihr Projekt ordnungsgemäß funktioniert. Eine der wichtigsten Eigenschaften ist die URL, unter der Ihr API-Plug-In verfügbar ist. Jedes Mal, wenn Sie Ihr Projekt starten, öffnet Teams Toolkit einen Dev-Tunnel unter einer neuen URL. Die OAuth-Informationen im Tresor müssen auf diese URL verweisen, damit Copilot Ihre API erreichen kann.

### Überprüfen der Authentifizierungs- und Autorisierungskonfiguration

Der nächste Teil, der untersucht werden soll, ist die Authentifizierungs- und Autorisierungseinstellungen von Azure Functions. Die API in dieser Übung verwendet die integrierten Authentifizierungs- und Autorisierungsfunktionen von Azure Functions. Das Teams-Toolkit konfiguriert diese Funktionen beim Bereitstellen von Azure-Funktionen in Azure.

In Visual Studio Code:

1. Öffnen Sie die Datei **infra/azure.bicep**.
1. Suchen Sie die Ressource **authSettings**:

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  Diese Ressource ermöglicht die integrierten Authentifizierungs- und Autorisierungsfunktionen in der Azure Functions-App. Zunächst wird im Abschnitt **globalValidation** festgelegt, dass die App nur authentifizierte Anfragen zulässt. Wenn die App eine nicht authentifizierte Anfrage empfängt, wird sie mit einem 401-HTTP-Fehler abgelehnt. Im Abschnitt **identityProviders** definiert die Konfiguration, dass sie Microsoft Entra ID (früher bekannt als Azure Active Directory) verwendet, um Anfragen zu autorisieren. Sie gibt an, welche Microsoft Entra-Anwendungsregistrierung sie zur Sicherung der API verwendet und welche Zielgruppen die API aufrufen dürfen.

### Überprüfen der Microsoft Entra-Anwendungsregistrierung

Der letzte Teil, der untersucht werden soll, ist die Registrierung der Microsoft Entra-Anwendung, die das Projekt zum Sichern der API verwendet. Wenn Sie OAuth verwenden, sichern Sie den Zugriff auf Ressourcen mithilfe einer Anwendung. Die Anwendung definiert in der Regel Anmeldeinformationen, die zum Abrufen eines Zugriffstokens erforderlich sind, z. B. einen geheimen Clientschlüssel oder ein Zertifikat. Außerdem werden die verschiedenen Berechtigungen (auch als Bereiche bezeichnet) angegeben, die der Client beim Aufrufen der API anfordern kann. Die Registrierung der Microsoft Entra-Anwendung stellt eine Anwendung in der Microsoft-Cloud dar und definiert eine Anwendung für die Verwendung mit OAuth-Autorisierungsflows.

In Visual Studio Code:

1. Öffnen Sie die Datei **./aad.manifest.json**.
1. Suchen Sie die Eigenschaft **oauth2Permissions**.

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  Die Eigenschaft definiert einen benutzerdefinierten Bereich namens **repairs_read**, der dem Client die Berechtigung erteilt, Reparaturen aus der Reparatur-API zu lesen.

1. Suchen Sie die Eigenschaft **identifierUris**.

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  Die **identifierUris-Eigenschaft** definiert einen Bezeichner, der verwendet wird, um den Bereich vollständig zu qualifizieren.
