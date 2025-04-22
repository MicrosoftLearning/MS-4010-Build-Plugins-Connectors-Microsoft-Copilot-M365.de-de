---
lab:
  title: 'Übung 1: Integrieren eines API-Plug-Ins mit einer durch einen Schlüssel gesicherten API'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Übung 1: Integrieren eines API-Plug-Ins mit einer durch einen Schlüssel gesicherten API

Mit API-Plug-Ins für Microsoft 365 Copilot können Sie eine Integration mit APIs durchführen, die mit einem Schlüssel gesichert sind. Sie bewahren den API-Schlüssel sicher auf, indem Sie ihn im Teams-Tresor registrieren. Zur Runtime führt Microsoft 365 Copilot Ihr Plug-In aus, ruft den API-Schlüssel aus dem Tresor ab und verwendet ihn zum Aufrufen der API. Wenn Sie diesem Prozess folgen, bleibt der API-Schlüssel sicher und wird nie für den Client verfügbar gemacht.

In dieser Übung erstellen Sie einen neuen deklarativen Agent mit einem API-Plug-In, das sich mit einem von Ihnen generierten API-Schlüssel authentifiziert.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1: Erstellen eines neuen Projekts

Erstellen Sie zunächst ein neues API-Plug-In für Microsoft 365 Copilot. Öffnen Sie Visual Studio Code.

In Visual Studio Code:

1. Aktivieren Sie in der **Aktivitätsleiste** (Seitenleiste) die Erweiterung Teams-Toolkit.
1. Wählen Sie im Bedienfeld **Teams-Toolkit** der Erweiterung **Neue App erstellen**.
1. Wählen Sie aus der Liste der Projektvorlagen **Copilot-Agent**.
1. Wählen Sie in der Liste der App-Features den **deklarativen Agent** aus.
1. Wählen Sie die Option **Plug-In hinzufügen**.
1. Wählen Sie die Option **Mit einer neuen API beginnen**.
1. Wählen Sie aus der Liste der Authentifizierungsarten **API-Schlüssel (Bearertoken Auth)**.
1. Wählen Sie die Programmiersprache **TypeScript** aus.
1. Wählen Sie einen Ordner aus, in dem Ihr Projekt gespeichert werden soll.
1. Nennen Sie Ihr Projekt **da-repairs-key**.

Das Teams-Toolkit erstellt ein neues Projekt, das einen deklarativen Agent, ein API-Plug-In und eine mit einem Schlüssel gesicherte API enthält.

## Aufgabe 2: Überprüfen der Konfiguration der API-Schlüsselauthentifizierung

Bevor Sie fortfahren, überprüfen Sie bitte die API-Schlüssel-Authentifizierungskonfiguration im generierten Projekt.

### Untersuchen der API-Definition

Sehen Sie sich zunächst an, wie die API-Schlüsselauthentifizierung in der API-Definition definiert ist.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/apiSpecificationFile/repair.yml**. Diese Datei enthält die OpenAPI-Definition für die API.
1. Beachten Sie im Abschnitt **components.securitySchemes** die Eigenschaft **apiKey**:

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  Die Eigenschaft definiert ein Sicherheitsschema, das den API-Schlüssel als Bearertoken im Autorisierungsanforderungsheader verwendet.

1. Suchen Sie die Eigenschaft **paths./repairs.get.security**. Beachten Sie, dass auf das Sicherheitsschema **apiKey** verwiesen wird.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### Untersuchen der API-Implementierung

Als Nächstes erfahren Sie, wie die API den API-Schlüssel für jede Anforderung überprüft.

In Visual Studio Code:

1. Öffnen Sie die Datei **src/functions/repairs.ts**.
1. Suchen Sie in der Handler-Funktion **repairs** die folgende Zeile, die alle nicht autorisierten Anfragen ablehnt:

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. Die Funktion **isApiKeyValid** wird in der Datei „repairs.ts“ weiter implementiert:

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  Die Funktion überprüft, ob der Autorisierungsheader ein Bearertoken enthält, und vergleicht es mit dem API-Schlüssel, der in der Umgebungsvariablen **API_KEY** definiert ist.

Dieser Code zeigt eine vereinfachte Implementierung der API-Schlüsselsicherheit, vermittelt jedoch einen Eindruck davon, wie die API-Schlüsselsicherheit in der Praxis funktioniert.

### Überprüfen der Konfiguration der Tresor-Aufgabe

In diesem Projekt verwenden Sie das Teams-Toolkit, um den API-Schlüssel zum Tresor hinzuzufügen. Das Teams-Toolkit registriert den API-Schlüssel im Tresor mithilfe einer speziellen Aufgabe in der Konfiguration des Projekts.

In Visual Studio Code:

1. Öffnen Sie die Datei **./teampsapp.local.yml**.
1. Suchen Sie im Abschnitt **Bereitstellung** die Aufgabe **apiKey/register**.

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  Die Aufgabe nimmt den Wert der Projektvariablen **SECRET_API_KEY**, die in der Datei **env/.env.local.user** gespeichert ist, und registriert ihn im Tresor. Dann nimmt er die ID des Tresoreintrags und schreibt sie in die Umgebungsdatei **env/.env.local**. Das Ergebnis dieser Aufgabe ist eine Umgebungsvariable namens **APIKEY_REGISTRATION_ID**. Das Teams-Toolkit schreibt den Wert dieser Variable in die Datei **appPackages/ai-plugin.json**, die die Plug-In Definition enthält. Zur Runtime verwendet der deklarative Agent, der das API-Plug-In lädt, diese ID, um den API-Schlüssel aus dem Tresor abzurufen und die API sicher aufzurufen.

## Aufgabe 3: Konfigurieren des API-Schlüssels für die lokale Entwicklung

Bevor Sie das Projekt testen können, müssen Sie einen API-Schlüssel für Ihre API definieren. Speichern Sie dann den API-Schlüssel im Tresor, und notieren Sie die Tresoreintrags-ID in Ihrem API-Plug-In. Für die lokale Entwicklung speichern Sie den API-Schlüssel in Ihrem Projekt und verwenden Sie Teams-Toolkit, um ihn für Sie im Tresor zu registrieren.

In Visual Studio Code:

1. Öffnen Sie den **Terminalbereich** (STRG + `).
1. In einer Befehlszeile:
  1. Stellen Sie die Abhängigkeiten des Projekts wieder her, indem Sie `npm install` ausführen.
  1. Generieren Sie einen neuen API-Schlüssel, indem Sie Folgendes ausführen: `npm run keygen`.
  1. Kopieren Sie den generierten Schlüssel in die Zwischenablage.
1. Öffnen Sie die Datei **env/.env.local.user**.
1. Aktualisieren Sie die Eigenschaft **SECRET_API_KEY** mit dem neu generierten API-Schlüssel. Die aktualisierte Eigenschaft sieht wie folgt aus:

  ```text
  SECRET_API_KEY='your_key'
  ```

1. Speichern Sie die Änderungen.

Bei jedem Erstellen des Projekts aktualisiert Teams-Toolkit automatisch den API-Schlüssel im Tresor und aktualisiert Ihr Projekt mit der Tresoreintrags-ID.