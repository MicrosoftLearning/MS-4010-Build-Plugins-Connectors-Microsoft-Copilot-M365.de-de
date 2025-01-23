---
lab:
  title: Übung 2 – Erstellen der API-Plug-In-Definition
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Übung 2 – Erstellen der API-Plug-In-Definition

Der nächste Schritt besteht darin, dem Projekt die Plug-In-Definition hinzuzufügen. Die Plug-In-Definition enthält folgende Informationen:

- Welche Aktionen das Plug-In ausführen kann.
- Was die Form der Daten ist, die erwartet und zurückgegeben werden.
- Wie der deklarative Agent die zugrunde liegende API aufrufen muss.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1 – Fügen Sie die grundlegende Plugin-Definitionsstruktur hinzu

In Visual Studio Code:

1. Fügen Sie im Ordner **appPackage** eine neue Datei namens **ai-plugin.json** hinzu.
1. Fügen Sie den folgenden Inhalt ein:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    Die Datei enthält eine grundlegende Struktur für ein API-Plug-In mit einer Beschreibung für den Menschen und das Modell. Die **description_for_model** enthält detaillierte Informationen darüber, was das Plug-In kann, damit der Agent versteht, wann er es aufrufen sollte.
1. Speichern Sie die Änderungen.

## Aufgabe 2 – Definieren von Funktionen

Ein API-Plug-In definiert eine oder mehrere Funktionen, die API-Vorgängen zugeordnet sind, die in der API-Spezifikation definiert sind. Jede Funktion besteht aus einem Namen und einer Beschreibung und einer Antwortdefinition, die den Agent anweist, wie die Daten den Benutzenden angezeigt werden.

### Definieren einer Funktion zum Abrufen des Menüs

Beginnen Sie mit der Definition einer Funktion, um die Informationen zum heutigen Menü abzurufen.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/ai-plugin.json**.
1. Fügen Sie in das Array **Funktionen** den folgenden Schnipsel ein:

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    Sie beginnen mit der Definition einer Funktion, die den Vorgang **getDishes** aus der API-Spezifikation aufruft. Als Nächstes geben Sie eine Funktionsbeschreibung an. Diese Beschreibung ist wichtig, weil Copilot sie verwendet, um zu entscheiden, welche Funktion für den Prompt von Benutzenden aufgerufen werden soll.

    In der Eigenschaft „response_semantics“ geben Sie an, wie Copilot die von der API empfangenen Daten anzeigen soll. Da die API die Informationen über die Gerichte auf der Speisekarte in der Eigenschaft **Gerichte** zurückgibt, legen Sie die Eigenschaft **data_path** auf den JSONPath-Ausdruck `$.dishes` fest.

    Als Nächstes ordnen Sie im Abschnitt **Eigenschaften** zu, welche Eigenschaften aus der API-Antwort den Titel, die Beschreibung und die URL darstellen. Da die Gerichte in diesem Fall keine URL haben, ordnen Sie nur den **Titel** und die **Beschreibung** zu.

1. Der vollständige Codeschnipsel sieht folgendermaßen aus:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Speichern Sie die Änderungen.

### Definieren einer Funktion zum Platzieren der Bestellung

Definieren Sie als Nächstes eine Funktion, um die Bestellung zu platzieren.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/ai-plugin.json**.
1. Fügen Sie am Ende des Arrays **Funktionen** den folgenden Schnipsel ein:

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    Sie beginnen mit dem Verweis auf den API-Vorgang mit der ID **placeOrder**. Dann geben Sie eine Beschreibung an, die Copilot verwendet, um diese Funktion mit dem Prompt der Benutzenden abzugleichen. Als Nächstes weisen Sie Copilot an, wie die Daten zurückgegeben werden. Im Folgenden sind die Daten aufgeführt, die die API nach dem Aufgeben einer Bestellung zurückgibt:

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    Da sich die Daten, die Sie anzeigen möchten, direkt im Stammverzeichnis des Antwortobjekts befinden, legen Sie den **data_path** auf **$** fest, was den obersten Knoten des JSON-Objekts angibt. Sie definieren den Titel, um die Bestellnummer und den Untertitel, um den Preis anzuzeigen.

1. Die vollständige Datei sieht wie folgt aus:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 3 – Definieren von Laufzeiten

Nachdem Sie die Funktionen definiert haben, die Copilot aufrufen soll, müssen Sie ihm als Nächstes Anweisungen geben, wie er sie aufrufen soll. Das tun Sie im Abschnitt **Laufzeiten** der Plug-In-Definition.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/ai-plugin.json**.
1. Fügen Sie im Laufzeitarray den folgenden Code hinzu:

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    Sie beginnen damit, Copilot anzuweisen, dass Sie ihm OpenAPI-Informationen über die aufzurufende API (**type: OpenApi**) zur Verfügung stellen und dass diese anonym ist (**auth.type: None**). Als Nächstes geben Sie im **Spezifikationsabschnitt** den relativen Pfad zur API-Spezifikation an, die sich in Ihrem Projekt befindet. Schließlich werden in der Eigenschaft **run_for_functions** alle Funktionen aufgelistet, die zu dieser API gehören.

1. Die vollständige Datei sieht wie folgt aus:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Speichern Sie die Änderungen.

