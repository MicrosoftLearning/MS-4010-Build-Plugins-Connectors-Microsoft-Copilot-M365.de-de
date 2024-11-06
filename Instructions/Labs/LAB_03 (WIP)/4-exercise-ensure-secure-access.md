---
lab:
  title: Übung 3 – Sicherstellen des sicheren Zugriffs mit Zugriffssteuerungsliste
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Übung 3 – Sicherstellen des sicheren Zugriffs mit Zugriffssteuerungsliste

In dieser Übung aktualisieren Sie den Code, der für den Import lokaler Abschriften-Dateien verantwortlich ist, und konfigurieren ACLs für ausgewählte Artikel.

## Vor der Installation

Diese Übung dauert etwa **XX Minuten**.

## Aufgabe 1 - Importieren von Inhalten, die für alle Mitarbeiter des Unternehmens verfügbar sind

Als Sie in der vorigen Übung den Code für den Import externer Inhalte implementiert haben, haben Sie ihn so konfiguriert, dass er für alle Mitarbeitenden des Unternehmens verfügbar ist. Hier sehen Sie den Code, den Sie verwendet haben:

```csharp
static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
{
  var baseUrl = new Uri("https://learn.microsoft.com/graph/");

  return content.Select(a =>
  {
    var docId = GetDocId(a.RelativePath ?? "");

    return new ExternalItem
    {
      Id = docId,
      Properties = new()
      {
        AdditionalData = new Dictionary<string, object> {
            { "title", a.Title ?? "" },
            { "description", a.Description ?? "" },
            { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).ToString() }
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

Sie haben die ACL so konfiguriert, dass jeder Zugriff gewährt wird. Lassen Sie uns sie für ausgewählte Markdownseiten anpassen, die Sie importieren.

## Aufgabe 2 – Importieren von Inhalten, die für ausgewählte Benutzende verfügbar sind

Konfigurieren Sie zunächst eine der Seiten, die Sie importieren, damit nur für einen bestimmten Benutzenden zugegriffen werden kann.

Im Webbrowser:

1. Navigieren Sie zum Azure-Portal unter [https://portal.azure.com](https://portal.azure.com) und melden Sie sich mit Ihrem Geschäfts-, Schul- oder Unikonto an.
1. Wählen Sie in der Seitenleiste **Ansicht** unter **Microsoft Entra ID**.
1. Wählen Sie in der Navigation **Verwalten** > **Benutzende**.
1. Öffnen Sie in der Benutzerliste einen der Benutzenden, indem Sie seinen Namen auswählen.
1. Kopieren Sie den Wert der Eigenschaft **Objekt-ID**.

  ![Screenshot des Azure-Portals mit einem geöffneten Benutzerprofil.](../media/8-user.png)

Verwenden Sie diesen Wert, um eine neue ACL für eine bestimmte Markdownseite zu definieren.

Im Code-Editor:

1. Öffnen Sie die Datei **ContentService.cs**, und suchen Sie die Methode `Transform`.
1. Definieren Sie innerhalb des `Select`-Delegaten die Standard-ACL, die für alle importierten Elemente gilt:

   ```csharp
   var acl = new Acl
   {
     Type = AclType.Everyone,
     Value = "everyone",
     AccessType = AccessType.Grant
   };
   ```

1. Als nächstes überschreiben Sie die Standard-ACL für die Markdowndatei, deren Name auf `use-the-api.md` endet:

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   ```

1. Aktualisieren Sie schließlich den Code, der das externe Element zurückgibt, um die definierte ACL zu verwenden:

   ```csharp
   return new ExternalItem
   {
     Id = docId,
     Properties = new()
     {
       AdditionalData = new Dictionary<string, object> {
         { "title", a.Title ?? "" },
         { "description", a.Description ?? "" },
         { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
       }
     },
     Content = new()
     {
       Value = a.Content ?? "",
       Type = ExternalItemContentType.Html
     },
     Acl = new()
     {
       acl
     }
   };
   ```

1. Die aktualisierte Methode `Transform` sieht wie folgt aus:

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
             { "title", a.Title ?? "" },
             { "description", a.Description ?? "" },
             { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
           acl
         }
       };
     });
   }
   ```

1. Speichern Sie die Änderungen.

## Aufgabe 3 - Importieren von Inhalten, die für eine ausgewählte Gruppe verfügbar sind

Erweitern wir nun den Code so, dass eine andere Seite nur für eine ausgewählte Gruppe von Benutzenden zugänglich ist.

![Screenshot des Azure-Portals mit einer geöffneten Gruppenseite.](../media/8-group.png)

Verwenden Sie diesen Wert, um eine neue ACL für eine bestimmte Markdownseite zu definieren.

Im Code-Editor:

1. Öffnen Sie die Datei **ContentService.cs**, und suchen Sie die Methode `Transform`
1. Erweitern Sie die zuvor definierte `if`-Klausel um eine zusätzliche Bedingung, um die ACL für die Markdown-Datei zu definieren, deren Name auf `traverse-the-graph.md` endet:

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
   {
     acl = new()
     {
       Type = AclType.Group,
       // Sales and marketing
       Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
       AccessType = AccessType.Grant
     };
   }
   ```

1. Die aktualisierte Methode `Transform` sieht wie folgt aus:

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
       else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
       {
         acl = new()
         {
           Type = AclType.Group,
           // Sales and marketing
           Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
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
             acl
         }
       };
     });
   }
   ```

1. Speichern Sie die Änderungen.

## Aufgabe 4 – Anwenden der neuen ACLs

Der letzte Schritt besteht darin, die neu konfigurierten ACLs anzuwenden.

1. Öffnen Sie ein Terminal, und wechseln Sie in den Projektordner.
1. Erstellen Sie das Projekt, indem Sie den Befehl `dotnet build` ausführen.
1. Starten Sie das Laden des Inhalts, indem Sie den `dotnet run -- load-content` Befehl ausführen.

[Fahren Sie mit der nächsten Übung fort...](./5-exercise-enable-inline-results.md)