---
lab:
  title: Übung 1 – Konfigurieren einer externen Verbindung und Bereitstellen des Schemas
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Übung – Konfigurieren einer externen Verbindung und Bereitstellen des Schemas

In dieser Übung erstellen Sie einen benutzerdefinierten Microsoft Graph-Connector als Konsolenanwendung. Sie registrieren eine neue Microsoft Entra-App-Registrierung und fügen den Code hinzu, um eine externe Verbindung zu erstellen und ihr Schema bereitzustellen.

## Vor der Installation

Diese Übung dauert etwa **XX Minuten**.

## Aufgabe 1 – Registrieren einer neuen Microsoft Entra-App-Registrierung

Beginnen Sie, indem Sie eine neue Entra-App-Registrierung registrieren, die der benutzerdefinierte Graph-Connector für die Authentifizierung bei Microsoft 365 verwendet.

In einem Webbrowser:

1. Wechseln Sie zum **Azure-Portal** unter [https://portal.azure.com](https://portal.azure.com).
1. Wählen Sie in der Navigation **Ansicht** unter **Microsoft Entra ID**.
1. Erweitern Sie in der Seitennavigation die Option **Verwalten** und wählen Sie **App-Registrierungen**.
1. Wählen Sie im oberen Navigationsbereich **Neue Registrierung** aus.
1. Füllen Sie die folgenden Werte aus:
   1. **Name:** MSGraph docs Graph-Connector
   1. **Unterstützte Kontotypen:** Nur Konten in diesem Organisationsverzeichnis (einzelner Mandant)
1. Wählen Sie **Registrieren** aus, um Ihren Eintrag zu bestätigen.
1. Kopieren Sie auf dem Übersichtsbildschirm die Werte der Eigenschaften ** Anwendungs-ID** und **Verzeichnis-ID (Mandaten-ID)**. Sie benötigen diese Angaben später.

## Aufgabe 2 – Erstellen einer Anmeldeinformation

Da dieser benutzerdefinierte Graph-Connector ohne Benutzerinteraktion ausgeführt wird, müssen Sie ihn so konfigurieren, dass er automatisch authentifiziert wird. Erstellen Sie aus Gründen der Einfachheit einen geheimen Schlüssel.

Fortsetzen im Webbrowser:

1. Erweitern Sie in der seitlichen Navigation **Verwalten** und wählen Sie **Zertifikate und Geheimnisse**.
1. Wählen Sie die Registerkarte **Client-Geheimnisse** und wählen Sie dann **Neuer geheimer Clientschlüssel**.
1. Geben Sie eine **Beschreibung** des geheimen Schlüssels des **MSGraph-Dokuments für den Graph-Connector ein**.
1. Erstellen Sie den geheimen Schlüssel, indem Sie **Hinzufügen** auswählen.
1. Kopieren Sie den **Wert** des neu erstellten geheimen Schlüssels. Sie benötigen die Information später.

## Schritt 3: Gewähren Sie API-Berechtigungen

Der letzte Schritt beim Konfigurieren der Entra-App-Registrierung besteht darin, ihr API-Berechtigungen zu erteilen, damit sie die externe Verbindung und das Schema erstellen kann.

Fortsetzen im Webbrowser:

1. Wählen Sie im seitlichen Navigationsbereich die Option **API-Berechtigungen** aus.
1. Wählen Sie **Berechtigung hinzufügen** aus.
1. Wählen Sie in der Liste der APIs die Option **Microsoft Graph** aus.
1. Wählen Sie als nächstes **Anwendungsberechtigungen** aus.
1. Geben Sie im Filtertextfeld **extern** ein.
1. Erweitern Sie den Abschnitt ** ExternalConnection**, und wählen Sie die Berechtigung ** ExternalConnection.ReadWrite.OwnedBy** aus.
1. Erweitern Sie den Abschnitt **ExternalItem**, und wählen Sie die Berechtigung ** ExternalItem.ReadWrite.OwnedBy** aus.
1. Um Ihre Auswahl zu bestätigen, wählen Sie die Schaltfläche **Berechtigungen hinzufügen** aus.
1. Um die Konfiguration abzuschließen, erteilen Sie Administratoreinwilligung, indem Sie die Schaltfläche **Administratoreinwilligung für (Mandant) erteilen** auswählen.
1. Bestätigen Sie den Dialog, indem Sie **Ja** auswählen.

## Aufgabe 4 – Erstellen einer neuen Konsolen-App und Installieren von Abhängigkeiten

Nachdem Sie die Entra-App-Registrierung konfiguriert haben, besteht der nächste Schritt darin, eine Konsolen-App zu erstellen, in der Sie den Code des Graph-Connectors implementieren.

Öffnen Sie ein Windows-Terminal, um eine neue Konsolenanwendung zu erstellen:

1. Erstellen Sie einen neuen Ordner, indem Sie `mkdir documents\console_app` eingeben und navigieren Sie dann zu dem neuen Ordner, indem Sie `cd .\documents\console_app` eingeben.
1. Erstellen Sie eine neue Konsolenanwendung, indem Sie `dotnet new console` ausführen.
1. Fügen Sie Abhängigkeiten hinzu, die Sie zum Erstellen des Connectors benötigen:
   1. Fügen Sie Nuget.org als Paketquelle hinzu und führen Sie `dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org` aus.
   1. Um die Bibliothek hinzuzufügen, die für die Authentifizierung mit Microsoft 365 erforderlich ist, führen Sie `dotnet add package Azure.Identity` aus.
   1. Um die Clientbibliothek für die Kommunikation mit Graph-APIs hinzuzufügen, führen Sie `dotnet add package Microsoft.Graph` aus.
   1. Führen Sie `dotnet add package Microsoft.Extensions.Configuration.UserSecrets` zum Hinzufügen der Bibliothek aus, die zum Arbeiten mit geheimen Benutzerschlüsseln erforderlich ist, die Sie im nächsten Schritt konfigurieren.
   1. Sie implementieren den Graph Connector als App in der Konsole mit zwei Befehlen: einem zum Erstellen der externen Verbindung und zum Bereitstellen des Schemas und einem weiteren zum Importieren des Inhalts. Um die Definition von Befehlen in Ihrer App zu unterstützen, führen Sie `dotnet add package System.CommandLine --prerelease` aus.

## Aufgabe 5 - Registrierungsdaten für die Entra-App sicher speichern

Nachdem Sie die Entra App-Registrierung erstellt haben, notieren Sie deren Informationen wie die Anwendungs- und Mandanten-ID und das Geheimnis. Sie verwenden diese Informationen in Ihrem Connector, um sich bei Microsoft 365 zu authentifizieren. Um sie in Ihrem Code verfügbar zu machen, speichern Sie sie sicher mit Ihrem Projekt als Benutzergeheimnis.

In einem Terminal:

1. Stellen Sie sicher, dass Ihr Arbeitsverzeichnis auf Ihre neu erstellte Konsolen-App festgelegt ist.
1. Um Benutzergeheimnisse einzugeben, führen Sie `dotnet user-secrets init` aus.
1. Um die Informationen über die App-Registrierung sicher zu speichern, ersetzen Sie die Token durch die tatsächlichen Werte, die Sie zuvor kopiert haben, und führen Sie sie aus:

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## Aufgabe 6 - Microsoft Graph-Client erstellen

Benutzerdefinierte Graph-Connectors verwenden die Microsoft Graph-API zur Verwaltung ihrer externen Verbindungen und Elemente. Beginnen Sie, indem Sie eine Instanz der Klasse `GraphServiceClient` aus dem NuGet-Paket **Microsoft.Graph** erstellen, das Sie im Projekt installiert haben.

1. Öffnen Sie Ihr Projekt in Visual Studio 2022.
1. Fügen Sie in Ihrem Projekt eine neue Codedatei mit dem Namen **GraphService.cs** hinzu.
1. Beginnen Sie in der Datei mit dem Hinzufügen von Verweisen auf die zu verwendenden Namespaces, indem Sie hinzufügen:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. Definieren Sie als Nächstes eine neue Klasse mit dem Namen GraphService:

   ```csharp
   class GraphService
   {
   }
   ```

1. Definieren Sie in der Klasse `GraphService` ein Singleton zum Speichern einer Instanz von `GraphServiceClient` für die Kommunikation mit Microsoft Graph APIs:

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. Im `Client` Singleton implementieren Sie den Getter, so dass er eine neue Instanz von `GraphServiceClient` erzeugt, wenn sie noch nicht existiert.

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. Erstellen Sie innerhalb des Abrufs eine neue Instanz von `GraphServiceClient` und verwenden Sie dabei einen Berechtigungsnachweis mit den Informationen zur Entra-App-Registrierung, die Sie zuvor gespeichert haben:

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   Sie beginnen mit der Erstellung eines Konfigurationsgenerators, um auf die in den Benutzergeheimnissen gespeicherten Informationen über die Registrierung der Entra-App zuzugreifen. Als Nächstes verwenden Sie den Generator, um Informationen zur App-Registrierung abzurufen. Anschließend erstellen Sie Anmeldeinformationen mit einem neuen geheimen Clientschlüssel, der die Mandant- und Client-ID sowie den geheimen Clientschlüssel enthält. Schließlich erstellen Sie eine Instanz von `GraphServiceClient` und übergeben die neu erstellten Anmeldeinformationen.

1. Der vollständige Code sieht wie folgt aus:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 7 - Definition der externen Verbindung und der Schemakonfiguration

Der nächste Schritt besteht darin, die externe Verbindung und das Schema zu definieren, die der Graph-ConneCtor verwenden soll. Da der Code des ConneCtors an mehreren Stellen Zugriff auf die ID der externen Verbindung benötigt, speichern Sie sie an einer zentralen Stelle in Ihrem Code.

Im Code-Editor:

1. Erstellen Sie eine neue Datei mit dem Namen **ConnectionConfiguration.cs**.
1. Fügen Sie einen Verweis auf den Namespace mit Microsoft Graph-Modellen hinzu:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Definieren Sie anschließend in derselben Datei eine neue statische Klasse namens `ConnectionConfiguration`:

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1. Fügen Sie in der Klasse `ConnectionConfiguration` eine neue Eigenschaft namens `ExternalConnection` hinzu. Implementieren Sie es, um eine Instanz des `ExternalConnection` Microsoft Graph Modells zurückzugeben:

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. Als nächstes fügen Sie eine weitere Eigenschaft namens `Schema` hinzu. Implementieren Sie sie, um eine Instanz des `Schema` Graphenmodells zurückzugeben:

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. Definieren Sie im Schema Eigenschaften, die Sie für jedes externe Element verfolgen, das Sie über den Connector einlesen:

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   Sie beginnen mit der Definition der Eigenschaft **Titel**, die den Titel des in Microsoft 365 importierten externen Elements speichert. Der Titel des Elements ist Teil des Volltextindexes (`IsSearchable = true`). Die Benutzenden können den Inhalt auch explizit in Schlüsselwortabfragen (`IsQueryable = true`) abfragen. Der Titel kann auch abgerufen und in den Suchergebnissen angezeigt werden (`IsRetrievable = true`). Die Eigenschaft **Titel** steht für den Titel des Elements, den Sie mit der semantischen Bezeichnung `Title` angeben.

   Als nächstes definieren Sie die Eigenschaft **Beschreibung**, in der die Zusammenfassung des Inhalts des externen Elements gespeichert wird. Seine Definition ähnelt dem Titel. Es gibt jedoch keine semantische Bezeichnung für die Beschreibung, weshalb Sie sie nicht definieren.

   Als Nächstes definieren Sie eine Eigenschaft zum Speichern der URL des Symbols für jedes Element. Copilot für Microsoft 365 benötigt diese Eigenschaft, die mit dem semantischen Label `IconUrl` abgebildet werden muss.

   Schließlich definieren Sie die Eigenschaft **url**, die die ursprüngliche URL des externen Elements speichert. Benutzende verwenden diese URL, um über Suchergebnisse oder Copilot von Microsoft 365 zu dem externen Element zu navigieren. Die URL ist eine der Eigenschaften, die Copilot für Microsoft 365 benötigt, weshalb Sie sie mit der semantischen Beschreibung `Url` zuordnen.

1. Der vollständige Code sieht wie folgt aus:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. Speichern Sie Ihre Änderungen.

## Aufgabe 8 - Erstellen einer externen Verbindung

Fahren Sie mit dem Hinzufügen von Code fort, der die Informationen über die externe Verbindung verwendet, die Sie im vorherigen Abschnitt definiert haben, um die externe Verbindung in Microsoft 365 zu erstellen.

Im Code-Editor:

1. Erstellen Sie eine neue Datei namens **ConnectionService.cs**.
1. Beginnen Sie in der Datei mit dem Hinzufügen von Verweisen auf die Namespaces:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Definieren Sie anschließend in derselben Datei eine neue statische Klasse namens `ConnectionService`:

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. Fügen Sie in der Klasse `ConnectionService` eine neue Methode namens `CreateConnection` hinzu:

   ```csharp
   async static Task CreateConnection()
   {

   }
   ```

1. In der `CreateConnection`-Methode verwenden Sie die Microsoft Graph-Clientinstanz, um Microsoft Graph-APIs aufzurufen und die externe Verbindung mit den zuvor definierten Verbindungsinformationen zu erstellen:

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. Der vollständige Code sieht wie folgt aus:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. Speichern Sie die Änderungen.

Bevor Sie diesen Code testen, fügen Sie den Code zur Erstellung des Schemas hinzu.Code zum Erstellen des Schemas hinzu. Auf diese Weise können Sie den gesamten Flow der Erstellung und Konfiguration der externen Verbindung testen.

## Aufgabe 9 - Erstellen eines externen Verbindungsschemas

Der letzte Teil der Erstellung einer externen Verbindung ist die Erstellung ihres Schemas.

Im Code-Editor:

1. Öffnen Sie die Datei **ConnectionService.cs**.
1. Fügen Sie in der Klasse ConnectionService eine neue Methode namens `CreateSchema` hinzu:

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. Verwenden Sie in der Methode `CreateSchema` die Microsoft Graph-Clientinstanz, um die Microsoft Graph-API aufzurufen und das Schema zu erstellen. Warten Sie dann auf seine Erstellung.

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. Fügen Sie in derselben Datei eine neue Methode mit dem Namen `ProvisionConnection` hinzu. Rufen Sie in seinem Code die Methoden `CreateConnection` und `CreateSchema` auf, die Sie zuvor definiert haben:

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. Der vollständige Code sieht wie folgt aus:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. Speichern Sie die Änderungen.

## Aufgabe 10 - Testen Sie den Code

Der letzte verbleibende Schritt besteht darin, einen Einstiegspunkt in der Anwendung zu erstellen, der die Verbindung und ihr Schema erzeugt. Dazu erstellen Sie einen Befehl, den Sie durch Starten der Anwendung über die Befehlszeile aufrufen.

Im Code-Editor:

1. Öffnen Sie die Datei **Program.cs**.
1. Ersetzen Sie ihren Inhalt durch den folgenden Code:

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   Sie beginnen mit der Definition eines Befehls namens `provision-connection.`. Dieser Befehl ruft die Methode `ConnectionService.ProvisionConnection` auf, die Sie zuvor definiert haben. Schließlich registrieren Sie den Befehl beim Befehlszeilenprozessor und starten die Anwendung, indem Sie die in der Befehlszeile übergebenen Argumente überwachen.

1. Speichern Sie Ihre Änderungen.

So testen Sie die Anwendung:

1. Öffnen Sie ein Terminal.
1. Ändern Sie das Arbeitsverzeichnis in den Projektordner.
1. Führen Sie `dotnet build` aus, um das Projekt zu erstellen.
1. Führen Sie `dotnet run -- provision-connection` aus, um die App zu starten.
1. Warten Sie einige Minuten, bis die Verbindung und das Schema erstellt sind.

[Weiter mit der nächsten Übung...](./3-exercise-import-external-content.md)