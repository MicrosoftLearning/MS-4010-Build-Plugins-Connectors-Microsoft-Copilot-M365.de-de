---
lab:
  title: Übung 3 – Verbinden der Plug-In-Definition mit dem deklarativen Agent
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Übung 3 – Verbinden der Plug-In-Definition mit dem deklarativen Agent

Nachdem Sie die API-Plug-In-Definition erstellt haben, besteht der nächste Schritt darin, sie beim deklarativen Agenten zu registrieren. Wenn Benutzende mit dem deklarativen Agenten interagieren, gleicht dieser die Eingabe der Benutzenden mit den definierten API-Plugins ab und ruft die entsprechenden Funktionen auf.

### Übungsdauer

- **Geschätzter Zeitaufwand**: 5 Minuten

## Aufgabe 1 – Verbinden der Plug-In-Definition mit dem deklarativen Agenten

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/declarativeAgent.json**.
1. Fügen Sie nach der Eigenschaft **instructions** den folgenden Codeschnipsel hinzu:

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    Mit diesem Codeschnipsel verbinden Sie den deklarativen Agent mit dem API-Plug-In. Sie geben eine eindeutige ID für das Plug-In an und weisen den Agent an, wo er die Definition des Plug-Ins finden kann.

1. Die vollständige Datei sieht wie folgt aus:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 2 – Aktualisieren von deklarativen Agenten-Informationen und -Anweisungen

Der deklarative Agent, den Sie in dieser Übung erstellen, hilft Benutzenden, die Speisekarte des lokalen italienischen Restaurants zu durchsuchen und Bestellungen aufzugeben. Um den Agent für dieses Szenario zu optimieren, aktualisieren Sie den Namen, die Beschreibung und die Anweisungen.

In Visual Studio Code:

1. Aktualisieren Sie die Informationen des deklarativen Agents:
    1. Öffnen Sie die Datei **appPackage/declarativeAgent.json**.
    1. Aktualisieren Sie den Wert der Eigenschaft **name** in **Il Ristorante**.
    1. Aktualisieren Sie den Wert der Eigenschaft **description** in **Order the most delicious Italian dishes and drinks from the comfort of your desk.**.
    1. Speichern Sie die Änderungen.
1. Aktualisieren Sie die Anweisungen des deklarativen Agents:
    1. Öffnen Sie die Datei **appPackage/instruction.txt**.
    1. Ersetzen Sie den Inhalt durch:

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        Beachten Sie, dass wir in den Anweisungen das allgemeine Verhalten des Agents definieren und ihn anweisen, was er kann. Darüber hinaus nehmen wir Anweisungen für ein bestimmtes Verhalten beim Aufgeben einer Bestellung auf, einschließlich der Form der Daten, die die API erwartet. Wir fügen diese Informationen ein, um sicherzustellen, dass der Agent wie beabsichtigt funktioniert.

    > [!NOTE]
    > Um die Formatierung beizubehalten, müssen Sie möglicherweise mehrere Kopier-/Einfügevorgänge in Editor ausführen, bevor Sie in Visual Studio Code kopieren.

    1. Speichern Sie die Änderungen.
1. Fügen Sie Gesprächseinstiege hinzu, um Benutzenden zu helfen, zu verstehen, wofür sie den Agent verwenden können:
    1. Öffnen Sie die Datei **appPackage/declarativeAgent.json**.
    1. Fügen Sie nach der Eigenschaft **instructions** eine neue Eigenschaft mit dem Namen **conversation_starters** hinzu:

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. Die vollständige Datei sieht wie folgt aus:

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. Speichern Sie die Änderungen.

## Aufgabe 3: Aktualisieren der API-URL

Bevor Sie Ihren deklarativen Agent testen können, müssen Sie die URL der API in der API-Spezifikationsdatei aktualisieren. Derzeit ist die URL auf `http://localhost:7071/api` festgelegt. Dies ist die URL, die Azure Functions bei einer lokalen Ausführung verwendet. Da Sie jedoch möchten, dass Copilot Ihre API aus der Cloud aufruft, müssen Sie Ihre API für das Internet verfügbar machen. Das Teams-Toolkit macht Ihre lokale API automatisch über das Internet verfügbar, indem ein Entwicklertunnel erstellt wird. Jedes Mal, wenn Sie mit dem Debuggen Ihres Projekts beginnen, startet das Teams-Toolkit einen neuen Entwicklertunnel und speichert seine URL in der Variablen **OPENAPI_SERVER_URL**. In der Aufgabe **Lokalen Tunnel starten** können Sie sehen, wie das Teams-Toolkit den Tunnel startet und seine URL in der Datei **.vscode/tasks.json** speichert:

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

Um diesen Tunnel zu verwenden, müssen Sie Ihre API-Spezifikation aktualisieren, sodass die Variable **OPENAPI_SERVER_URL** verwendet wird.

In Visual Studio Code:

1. Öffnen Sie die Datei **appPackage/apiSpecificationFile/ristorante.yml**.
1. Ändern Sie den Wert der Eigenschaft **servers.url** in **${{OPENAPI_SERVER_URL}}/api**.
1. Die geänderte Datei sieht wie folgt aus:

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. Speichern Sie die Änderungen.

Ihr API-Plug-In ist fertig und in einen deklarativen Agent integriert. Fahren Sie mit dem Testen des Agents in Microsoft 365 Copilot fort.