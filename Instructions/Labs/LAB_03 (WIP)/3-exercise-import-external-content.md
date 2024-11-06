---
lab:
  title: Übung 2 – Importieren externer Inhalte
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Übung – Importieren externer Inhalte

In dieser Übung erweitern Sie den benutzerdefinierten Microsoft Graph-Connector mit einem Code zum Importieren lokaler Markdowndateien in Microsoft 365.

## Vor der Installation

Diese Übung dauert etwa **XX Minuten**.

## Aufgabe 1 – Herunterladen externer Inhalte

In dieser Übung kopieren Sie die Beispielinhaltsdateien der Übung von [GitHub](https://pnp.github.io/download-partial/?url=https://github.com/pnp/graph-connectors-samples/tree/main/samples/dotnet-csharp-graphdocs/content) und speichern sie in Ihrem Projekt in einem Ordner mit dem Namen **Inhalt**.

![Screenshot eines Code-Editors mit den Inhaltsdateien dieser Übung.](../media/6-content-files.png)

Damit der Code ordnungsgemäß funktioniert, muss der **Inhaltsordner** und dessen Inhalt in den Build-Ausgabeordner kopiert werden.

Im Code-Editor:

1. Öffnen Sie die Datei **.csproj** und fügen Sie den folgenden Code vor dem schließenden `</Project>` -Tag hinzu.

   ```xml
   <ItemGroup>
     <ContentFiles Include="content\**"    CopyToOutputDirectory="PreserveNewest" />
   </ItemGroup>
   
   <Target Name="CopyContentFolder" AfterTargets="Build">
     <Copy SourceFiles="@(ContentFiles)" DestinationFiles="@   (ContentFiles->'$(OutputPath)\content\%(RecursiveDir)%(Filename)%   (Extension)')" />
   </Target>
   ```

1. Speichern Sie die Änderungen.

## Aufgabe 2 – Hinzufügen von Bibliotheken zum Analysieren von Markdown und YAML

Der Microsoft Graph-Connector, den Sie erstellen, importiert lokale Markdowndateien in Microsoft 365. Jede dieser Dateien enthält einen Header mit Metadaten im YAML-Format, auch als Frontmatter bezeichnet. Darüber hinaus werden die Inhalte jeder Datei in Markdown geschrieben. Um Metadaten zu extrahieren und den Textkörper in HTML zu konvertieren, verwenden Sie benutzerdefinierte Bibliotheken:

1. Öffnen Sie ein Terminal, und wechseln Sie in den Projektordner.
1. Führen Sie den folgenden Befehl aus, um die Markdown-Verarbeitungsbibliothek hinzuzufügen: `dotnet add package Markdig`
1. Führen Sie den folgenden Befehl aus, um die  YAML-Verarbeitungsbibliothek hinzuzufügen: `dotnet add package YamlDotNet`.

## Aufgabe 3 – Definieren der Klasse, die die importierte Datei darstellt

Um das Arbeiten mit importierten Markdowndateien und deren Inhalten zu vereinfachen, definieren wir eine Klasse mit den erforderlichen Eigenschaften.

Im Code-Editor:

1. Erstellen Sie eine neue Datei mit dem Namen **ContentService.cs**.
1. Fügen Sie den folgenden Code hinzu:

   ```csharp
   using YamlDotNet.Serialization;
   
   public interface IMarkdown
   {
     string? Markdown { get; set; }
   }
   
   class DocsArticle : IMarkdown
   {
     [YamlMember(Alias = "title")]
     public string? Title { get; set; }
     [YamlMember(Alias = "description")]
     public string? Description { get; set; }
     public string? Markdown { get; set; }
     public string? Content { get; set; }
     public string? RelativePath { get; set; }
   }
   ```

   Die `IMarkdown`-Schnittstelle stellt den Inhalt der lokalen Markdowndatei dar. Sie muss separat definiert werden, um die Deserialisierung von Dateiinhalten zu unterstützen. Die `DocsArticle`-Klasse stellt das endgültige Dokument mit analysierten YAML-Eigenschaften und HTML-Inhalten dar. `YamlMember` Attribute ordnen Metadaten in der Kopfzeile jedes Dokuments Eigenschaften zu.

1. Speichern Sie die Änderungen.

## Aufgabe 4 – Definieren der `ContentService`-Klasse

Erstellen Sie als Nächstes eine Klasse, die den Code zum Laden lokaler Markdowndateien, zum Umwandeln  in externe Elemente und zum Laden in Microsoft 365 enthält.

Im Code-Editor:

1. Stellen Sie sicher, dass Sie die **ContentService.cs**-Datei bearbeiten.
1. Fügen Sie die folgenden using-Anweisungen am Anfang der Datei  ein:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Fügen Sie am Ende der Datei den folgenden Code hinzu:

   ```csharp
   static class ContentService
   {
     static IEnumerable<DocsArticle> Extract()
     {}
   
     static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
     {}
   
     static async Task Load(IEnumerable<ExternalItem> items)
     {}
   
     public static async Task LoadContent()
     {
       var content = Extract();
       var transformed = Transform(content);
       await Load(transformed);
     }
   }
   ```

   Die `ContentService`-Klasse definiert drei Methoden, die den Prozess der Inhaltsverarbeitung darstellen:

   1. `Extract`, wodurch lokale Markdowndateien geladen zur einfachen Handhabung in Instanzen der `DocsArticle` -Klasse analysiert werden.
   1. `Transform`, das `DocsArticle`-Objekte in Instanzen der `ExternalItems` -Klasse konvertiert, die Teil des Microsoft Graph .NET-SDK ist und die externen Elemente darstellt, die in Microsoft 365 geladen werden sollen.
   1. `Load`, das externe Elemente mithilfe der Microsoft Graph-API in Microsoft 365 lädt.

   Diese Methoden werden in dieser spezifischen Reihenfolge aus der `LoadContent`-Methode aufgerufen.

1. Speichern Sie die Änderungen.

## Aufgabe 5 – Konfigurieren der Markdownverarbeitung

Beginnen wir mit dem Extrahieren des Inhalts aus den lokalen Markdowndateien.

Fügen Sie zunächst Hilfsmethoden hinzu, um die `Markdig` - und die `YamlDotNet`-Bibliothek einfach zu verwenden.

Im Code-Editor:

1. Erstellen Sie eine neue Datei mit dem Namen **MarkdownExtensions.cs**.
1. Fügen Sie in der Datei  den folgenden Code hinzu:

   ```csharp
   // from: https://khalidabuhakmeh.com/parse-markdown-front-matter-with-csharp
   using Markdig;
   using Markdig.Extensions.Yaml;
   using Markdig.Syntax;
   using YamlDotNet.Serialization;
   
   public static class MarkdownExtensions
   {
     private static readonly IDeserializer YamlDeserializer =
         new DeserializerBuilder()
         .IgnoreUnmatchedProperties()
         .Build();
         
     private static readonly MarkdownPipeline Pipeline
         = new MarkdownPipelineBuilder()
         .UseYamlFrontMatter()
         .Build();
   }
   ```

   Die `YamlDeserializer`-Eigenschaft definiert einen neuen Deserializer für den YAML-Block in den einzelnen Markdowndateien, die Sie extrahieren. Der von Ihnen konfigurierte Deserializer ignoriert Eigenschaften, die nicht Teil der Klasse sind, für die die Datei deserialisiert wird.

   Die `Pipeline`-Eigenschaft definiert eine Verarbeitungspipeline für den Markdown-Parser. Sie wird konfiguriert, um den YAML-Header zu analysieren. Ohne diese Konfiguration werden die Informationen aus dem Header verworfen.

1. Als Nächstes erweitern Sie die `MarkdownExtensions`-Klasse mit dem folgenden Code:

   ```csharp
   public static T GetContents<T>(this string markdown) where T :    IMarkdown, new()
   {
     var document = Markdown.Parse(markdown, Pipeline);
     var block = document
         .Descendants<YamlFrontMatterBlock>()
         .FirstOrDefault();
   
     if (block == null)
       return new T { Markdown = markdown };
   
     var yaml =
         block
         // this is not a mistake
         // we have to call .Lines 2x
         .Lines // StringLineGroup[]
         .Lines // StringLine[]
         .OrderByDescending(x => x.Line)
         .Select(x => $"{x}\n")
         .ToList()
         .Select(x => x.Replace("---", string.Empty))
         .Where(x => !string.IsNullOrWhiteSpace(x))
         .Aggregate((s, agg) => agg + s);
   
     var t = YamlDeserializer.Deserialize<T>(yaml);
     t.Markdown = markdown.Substring(block.Span.End + 1);
     return t;
   }
   ```

   Die `GetContents` Methode konvertiert eine Markdownzeichenfolge mit YAML-Metadaten im Header in den angegebenen Typ, der die `IMarkdown`-Schnittstelle implementiert. Aus der Markdownzeichenfolge extrahiert sie den YAML-Header und deserialisiert ihn in den angegebenen Typ. Anschließend extrahiert sie den Haupttext des Artikels und legt ihn zur weiteren Verarbeitung auf die `Markdown`-Eigenschaft fest.

1. Speichern Sie die Änderungen.

## Aufgabe 6 – Extrahieren von Markdown- und YAML-Inhalten

Nachdem Sie die Hilfsmethoden eingerichtet haben, implementieren Sie die Extract-Methode, um die lokalen Markdowndateien zu laden und die Informationen daraus zu extrahieren.

Im Code-Editor:

1. Öffnen Sie die Datei **ContentService.cs**.
1. Fügen Sie am Anfang der Datei die folgende using-Anweisung hinzu:

   ```csharp
   using Markdig;
   ```

1. Implementieren Sie als Nächstes in der `ContentService`-Klasse die `Extract`-Methode mit dem folgenden Code:

   ```csharp
   static IEnumerable<DocsArticle> Extract()
   {
     var docs = new List<DocsArticle>();
   
     var contentFolder = "content";
     var contentFolderPath = Path.Combine(Directory.GetCurrentDirectory(),    contentFolder);
     var files = Directory.GetFiles(contentFolder, "*.md", SearchOption.   AllDirectories);
   
     foreach (var file in files)
     {
       try
       {
         var contents = File.ReadAllText(file);
         var doc = contents.GetContents<DocsArticle>();
         doc.Content = Markdown.ToHtml(doc.Markdown ?? "");
         doc.RelativePath = Path.GetRelativePath(contentFolderPath, file);
         docs.Add(doc);
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   
     return docs;
   }
   ```

   Die Methode beginnt mit dem Laden von Markdowndateien aus dem Ordner **content** . Für jede Datei wird der Inhalt als Zeichenfolge geladen. Sie wandelt die Zeichenfolge in ein Objekt um, wobei die Metadaten und der Inhalt in separaten Eigenschaften gespeichert werden. Dazu verwendet sie die `GetContents`-Erweiterungsmethode, die zuvor in der `MarkdownExtensions` Klasse definiert wurde. Als Nächstes konvertiert sie die Markdownzeichenfolge in HTML. Schließlich speichert sie den relativen Pfad zur Datei und fügt das Objekt einer Auflistung zur weiteren Verarbeitung hinzu.

1. Speichern Sie die Änderungen.

## Aufgabe 7 – Transformieren von Inhalten in externe Elemente

Nachdem Sie den externen Inhalt gelesen haben, besteht der nächste Schritt darin, ihn in externe Elemente zu transformieren, die in Microsoft 365 geladen werden.

Beginnen Sie mit dem Hinzufügen einer Hilfsmethode, die eine eindeutige ID für jedes externe Element basierend auf seinem relativen Dateipfad generiert.

Im Code-Editor:

1. Vergewissern Sie sich, dass Sie die Datei **ContentService.cs** bearbeiten.
1. Fügen Sie in der `ContentService`-Klasse die folgende Methode hinzu:

   ```csharp
   static string GetDocId(string relativePath)
   {
     var id = relativePath.Replace(Path.DirectorySeparatorChar.ToString(),    "__").Replace(".md", "");
     return id;
   }
   ```

   Die `GetDocId`-Methode verwendet den relativen Dateipfad und ersetzt alle Verzeichnistrennzeichen durch einen doppelten Unterstrich. Dies ist erforderlich, da Pfadtrennzeichen nicht in einer externen Element-ID verwendet werden können.

1. Speichern Sie die Änderungen.

Implementieren Sie nun die `Transform`-Methode, die Objekte, die lokale Markdowndateien darstellen, in externe Elemente aus Microsoft Graph umwandelt.

Im Code-Editor:

1. Vergewissern Sie sich, dass Sie sich in der Datei **ContentService.cs** befinden.
1. Implementieren Sie die `Transform`-Methode mithilfe des folgenden Codes:

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var docId = GetDocId(a.RelativePath ?? '');
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             new()
             {
               Type = AclType.Everyone,
               Value = "everyone",
               AccessType = AccessType.Grant
             }
         }
       };
     });
   }
   ```

   Zunächst definieren Sie eine Basis-URL. Sie verwenden diese URL, um eine vollständige URL für jedes Element zu erstellen. Wenn das Element sichtbar ist, kann zum ursprünglichen Element navigiert werden. Als Nächstes transformieren Sie jedes Element aus einem `DocsArticle` in ein `ExternalItem`. Sie beginnen, indem Sie eine eindeutige ID für jedes Element basierend auf seinem relativen Dateipfad abrufen. Anschließend erstellen Sie eine neue Instanz von `ExternalItem` und füllen dessen Eigenschaften mit Informationen aus dem `DocsArticle`. Anschließend legen Sie den Inhalt des Elements auf den HTML-Inhalt fest, der aus der lokalen Datei extrahiert wurde, und legen den Elementinhaltstyp auf HTML fest. Schließlich konfigurieren Sie die Berechtigung des Elements so, dass es für alle Personen in der Organisation sichtbar ist.

1. Speichern Sie die Änderungen.

## Aufgabe 8 – Laden externer Elemente in Microsoft 365

Der letzte Schritt der Verarbeitung der Inhalte besteht darin, die transformierten externen Elemente in Microsoft 365 zu laden.

Im Code-Editor:

1. Stellen Sie sicher, dass Sie die **ContentService.cs**-Datei bearbeiten.
1. Implementieren Sie in der `ContentService` -Klasse die `Load`-Methode mit folgendem Code:

   ```csharp
   static async Task Load(IEnumerable<ExternalItem> items)
   {
     foreach (var item in items)
     {
       Console.Write(string.Format("Loading item {0}...", item.Id));
   
       try
       {
         await GraphService.Client.External
           .Connections[Uri.EscapeDataString(ConnectionConfiguration.   ExternalConnection.Id!)]
           .Items[item.Id]
           .PutAsync(item);
   
         Console.WriteLine("DONE");
       }
       catch (Exception ex)
       {
         Console.WriteLine("ERROR");
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

   Sie verwenden für jedes externe Element das Microsoft Graph .NET SDK, um die Microsoft Graph-API aufzurufen und das Element hochzuladen. In der Anforderung geben Sie die ID der zuvor erstellten externen Verbindung, die ID des hochzuladenden Elements und den vollständigen Inhalt des Elements an.

1. Speichern Sie die Änderungen.

## Aufgabe 9 – Hinzufügen des Inhaltsladebefehls

Bevor Sie den Code testen können, müssen Sie die Konsolenanwendung mit einem Befehl erweitern, der die Logik zum Laden von Inhalten aufruft.

Im Code-Editor:

1. Öffnen Sie die Datei **Program.cs**.
1. Fügen Sie mithilfe des folgenden Codes einen neuen Befehl zum Laden des Inhalts hinzu:

    ```csharp
    var loadContentCommand = new Command("load-content", "Loads content   into the external connection");
    loadContentCommand.SetHandler(ContentService.LoadContent);
    ```

1. Registrieren Sie den neu definierten Befehl mit dem Stammbefehl, damit er mithilfe des folgenden Codes aufgerufen werden kann:

     ```csharp
     rootCommand.AddCommand(loadContentCommand);
     ```

1. Speichern Sie die Änderungen.

## Aufgabe 10 - Testen Sie den Code

Zum Schluss muss getestet werden, ob der Microsoft Graph-Connector externe Inhalte ordnungsgemäß importiert.

1. Öffnen Sie ein Terminal.
1. Ändern Sie Ihr Arbeitsverzeichnis in den Projektordner:
1. Erstellen Sie das Projekt, indem Sie den Befehl `dotnet build` ausführen.
1. Starten Sie das Laden des Inhalts, indem Sie den `dotnet run -- load-content` Befehl ausführen.
1. Warten Sie, bis der Befehl abgeschlossen ist, und laden Sie den Inhalt.

[Fahren Sie mit der nächsten Übung fort...](./4-exercise-ensure-secure-access.md)