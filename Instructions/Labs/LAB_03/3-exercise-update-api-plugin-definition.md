---
lab:
  title: Übung 2 – Aktualisieren der API-Plug-In-Definition
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Übung 2 – Aktualisieren der API-Plug-In-Definition

Der nächste Schritt besteht darin, die API-Plug-In-Definition mit adaptiven Karten zu aktualisieren, die Copilot zum Anzeigen von Daten aus der API für Benutzende verwenden sollte.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1 – Hinzufügen einer adaptiven Karte zum Anzeigen eines Gerichts

In Visual Studio Code:

1. Öffnen Sie die Datei **cards/dish.json** und kopieren Sie ihren Inhalt.
1. Öffnen Sie die Datei **appPackage/ai-plugin.json**.
1. Fügen Sie der Eigenschaft **functions.getDishes.capabilities.response_semantics** eine neue Eigenschaft mit dem Namen **static_template** hinzu und legen Sie den Wert für **body** auf den Inhalt von **dish.json** fest.
1. Der vollständige Codeschnipsel sieht folgendermaßen aus:

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "Image",
                "url": "${image_url}",
                "size": "large"
              },
              {
                "type": "TextBlock",
                "text": "${name}",
                "weight": "Bolder"
              },
              {
                "type": "TextBlock",
                "text": "${description}",
                "wrap": true
              },
              {
                "type": "TextBlock",
                "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
                "weight": "Lighter"
              },
              {
                "type": "TextBlock",
                "text": "**Price:** €${formatNumber(price, 2)}",
                "weight": "Lighter",
                "spacing": "None"
              }
            ]
          }
      ]
    }
    ```

1. Speichern Sie die Änderungen.

## Aufgabe 2 – Hinzufügen einer Vorlage für adaptive Karten zum Anzeigen der Bestellzusammenfassung

In Visual Studio Code:

1. Öffnen Sie die Datei **cards/order.json** und kopieren Sie ihren Inhalt.
1. Öffnen Sie die Datei **appPackage/ai-plugin.json**.
1. Fügen Sie der Eigenschaft **functions.placeOrder.capabilities.response_semantics** eine neue Eigenschaft mit dem Namen **static_template** hinzu und legen Sie deren Inhalt auf die adaptive Karte fest.
1. Die vollständige Datei sieht wie folgt aus:

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "TextBlock",
            "text": "Order Confirmation 🤌",
            "size": "Large",
            "weight": "Bolder",
            "horizontalAlignment": "Center"
          },
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "Your order has been successfully placed!",
                "weight": "Bolder",
                "spacing": "Small"
              },
              {
                "type": "FactSet",
                "facts": [
                  {
                    "title": "Order ID:",
                    "value": "${order_id} "
                  },
                  {
                    "title": "Status:",
                    "value": "${status}"
                  },
                  {
                    "title": "Total Price:",
                    "value": "€${formatNumber(total_price, 2)}"
                  }
                ]
              }
            ]
          }
        ]
    }
    ```

1. Speichern Sie die Änderungen.
