---
lab:
  title: Übung 4 – Erkunden des Plug-In-Quellcodes
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Übung 4 – Erkunden des Plug-In-Quellcodes

In dieser Übung überprüfen Sie den Anwendungscode, damit Sie verstehen können, wie eine **Messaging-Erweiterung** funktioniert.

## Aufgabe 1 – Untersuchen des Manifests

Der Kern jeder Microsoft 365-Anwendung ist das Anwendungsmanifest. Hier geben Sie die Informationen an, die Microsoft 365 für den Zugriff auf Ihre Anwendung benötigt.

Öffnen Sie in Ihrem **Arbeitsverzeichnis** die Datei **appPackackage/manifest.json**. Diese JSON-Datei wird in einem Zip-Archiv mit zwei Icon-Dateien abgelegt, um das Anwendungspaket zu erstellen. Die Eigenschaft **Symbole** enthält Pfade zu diesen Symbolen.

```json
"icons": {
    "color": "Northwind-Logo3-192-${{TEAMSFX_ENV}}.png",
    "outline": "Northwind-Logo3-32.png"
},
```

Beachten Sie das Token `${{TEAMSFX_ENV}}` in einem der Symbolnamen. Teams Toolkit ersetzt dieses Token durch den Namen Ihrer Umgebung, z. B. **local** oder **dev** (für eine Azure-Bereitstellung in der Entwicklung). Daher ändert sich die Symbolfarbe je nach Umgebung.

### Application description (Anwendungsbeschreibung)

Schauen Sie sich nun den **Namen** und die **Beschreibung** an. Beachten Sie, dass die **Beschreibung** ziemlich lang ist! Dies ist wichtig, damit sowohl die Benutzenden als auch Copilot lernen können, was Ihre Anwendung tut und wann sie zu verwenden ist.

```json
    "name": {
        "short": "Northwind Inventory",
        "full": "Northwind Inventory App"
    },
    "description": {
        "short": "App allows you to find and update product inventory information",
        "full": "Northwind Inventory is the ultimate tool for managing your product inventory. With its intuitive interface and powerful features, you'll be able to easily find your products by name, category, inventory status, and supplier city. You can also update inventory information with the app. \n\n **Why Choose Northwind Inventory:** \n\n Northwind Inventory is the perfect solution for businesses of all sizes that need to keep track of their inventory. Whether you're a small business owner or a large corporation, Northwind Inventory can help you stay on top of your inventory management needs. \n\n **Features and Benefits:** \n\n - Easy Product Search through Microsoft Copilot. Simply start by saying, 'Find northwind dairy products that are low on stock' \r - Real-Time Inventory Updates: Keep track of inventory levels in real-time and update them as needed \r  - User-Friendly Interface: Northwind Inventory's intuitive interface makes it easy to navigate and use \n\n **Availability:** \n\n To use Northwind Inventory, you'll need an active Microsoft 365 account . Ensure that your administrator enables the app for your Microsoft 365 account."
    },
```

### Bot-Definition

Blättern Sie ein wenig nach unten zu **composeExtensions**. Compose-Erweiterung ist der ehemalige Begriff für Messaging-Erweiterung; hier werden die Messaging-Erweiterungen der App definiert. Messaging-Erweiterungen kommunizieren mit dem Azure Bot Framework; Dies bietet einen schnellen und sicheren Kommunikationskanal zwischen Microsoft 365 und Ihrer Anwendung. Als Sie Ihr Projekt zum ersten Mal gestartet haben, hat Teams Toolkit einen Bot registriert und seine **botID** hier abgelegt.

```json
    "composeExtensions": [
        {
            "botId": "${{BOT_ID}}",
            "commands": [
                {
                    ...
```

### Befehlsdefinitionen

Diese Messaging-Erweiterung verfügt über zwei Befehle, die im `commands` Array definiert sind. Wenn Sie die vorhergehende Übung abgeschlossen haben, gibt es noch einen dritten Befehl für die Suche nach Unternehmensnamen. Überspringen wir den ersten Befehl für einen Moment, da er der komplexeste ist. Mit dem folgenden Befehl kann Copilot (oder ein Benutzender) nach vergünstigten Produkten innerhalb einer Northwind-Kategorie suchen. Dieser Befehl akzeptiert einen einzigen Parameter,**categoryName**.

```json
{
    "id": "discountSearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search for discounted products by category",
    "title": "Discounts",
    "type": "query",
    "parameters": [
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category to find discounted products",
            "inputType": "text"
        }
    ]
},
```

Jetzt kehren wir zum ersten Befehl zurück, **inventorySearch**, der fünf Parameter enthält, was viel komplexere Abfragen ermöglicht.

```json
{
    "id": "inventorySearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search products by name, category, inventory status, supplier location, stock level",
    "title": "Product inventory",
    "type": "query",
    "parameters": [
        {
            "name": "productName",
            "title": "Product name",
            "description": "Enter a product name here",
            "inputType": "text"
        },
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category of the product",
            "inputType": "text"
        },
        {
            "name": "inventoryStatus",
            "title": "Inventory status",
            "description": "Enter what status of the product inventory. Possible values are 'in stock', 'low stock', 'on order', or 'out of stock'",
            "inputType": "text"
        },
        {
            "name": "supplierCity",
            "title": "Supplier city",
            "description": "Enter the supplier city of product",
            "inputType": "text"
        },
        {
            "name": "stockQuery",
            "title": "Stock level",
            "description": "Enter a range of integers such as 0-42 or 100- (for >100 items). Only use if you need an exact numeric range.",
            "inputType": "text"
        }
    ]
},
```

Copilot kann diese basierend auf den Beschreibungen ausfüllen, und die Nachrichtenerweiterung gibt eine Liste der Produkte zurück, die nach allen nicht leeren Parametern gefiltert werden.

## Aufgabe 2 – Untersuchen des Bot-Codes

Öffnen Sie nun die Datei **src/searchApp.ts**, die den Code für den Bot enthält, der das [Bot Builder SDK](https://learn.microsoft.com/azure/bot-service/index-bf-sdk) zur Kommunikation mit dem Azure Bot Framework verwendet. Beachten Sie, dass der Bot eine SDK-Klasse **TeamsActivityHandler** erweitert.

```typescript
export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  ...
```

### Abfrage der Messaging-Erweiterung

Die Anwendung ist in der Lage, Nachrichten (**Aktivitäten** genannt) zu verarbeiten, die von Microsoft 365 kommen, indem die Methoden von **TeamsActivityHandler** überschrieben werden.

Die erste davon ist eine **Messaging-Erweiterungsabfrage**-Aktivität. Diese Funktion wird aufgerufen, wenn ein Benutzer eine Messaging-Erweiterung eingibt oder wenn Copilot sie aufruft. Der Handler verteilt die Abfrage basierend auf der **commandID**. Hierbei handelt es sich um die gleiche Befehls-ID, die im App-Manifest verwendet wird.

```typescript
  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }
  }
  ...
```

### Aktionen für adaptive Karten

Die andere Art von Aktivitäten, die unsere App handhaben muss, sind die Aktionen der adaptiven Karte, z. B. wenn ein Benutzer **Bestand aktualisieren** oder **Nachbestellen** auf einer adaptiven Karte auswählt. Da es keine spezifische Methode für eine adaptive Kartenaktion gibt, setzt der Code `onInvokeActivity()` außer Kraft, was eine viel breitere Klasse von Aktivitäten ist, die Abfragen zur Messaging-Erweiterung umfasst. Aus diesem Grund prüft der Code manuell den Aktivitätsnamen und leitet ihn an den entsprechenden Handler weiter. Wenn der Aktivitätsname nicht für eine adaptive Kartenaktion steht, führt die `else`-Klausel die Basisimplementierung von `onInvokeActivity()` aus, die unter anderem unsere `handleTeamsMessagingExtensionQuery()`-Methode aufruft, wenn die **Invoke**-Aktivität eine Abfrage ist.

```typescript
import {
  TeamsActivityHandler,
  TurnContext,
  MessagingExtensionQuery,
  MessagingExtensionResponse,
  InvokeResponse
} from "botbuilder";
import productSearchCommand from "./messageExtensions/productSearchCommand";
import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
import revenueSearchCommand from "./messageExtensions/revenueSearchCommand";
import actionHandler from "./adaptiveCards/cardHandler";

export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }

  }

  // Handle adaptive card actions
  public async onInvokeActivity(context: TurnContext): Promise<InvokeResponse> {
    let runEvents = true;
    // console.log (`🎬 Invoke activity received: ${context.activity.name}`);
    try {
      if(context.activity.name==='adaptiveCard/action'){
        switch (context.activity.value.action.verb) {
          case 'ok': {
            return actionHandler.handleTeamsCardActionUpdateStock(context);
          }
          case 'restock': {
            return actionHandler.handleTeamsCardActionRestock(context);
          }
          case 'cancel': {
            return actionHandler.handleTeamsCardActionCancelRestock(context);
          }
          default:
            runEvents = false;
            return super.onInvokeActivity(context);
        }
      } else {
          runEvents = false;
          return super.onInvokeActivity(context);
      }
    } 
```

## Aufgabe 3 – Überprüfen des Befehlscodes für die Messaging-Erweiterung

In dem Bemühen, den Code modularer, lesbarer und wiederverwendbar zu machen, ist jeder Befehl der Messaging-Erweiterung in einem eigenen TypeScript-Modul untergebracht. Schauen Sie sich **src/messageExtensions/discountSearchCommand.ts** für ein Beispiel an.

Zunächst ist zu beachten, dass das Modul eine Konstante `COMMAND_ID` exportiert, die dieselbe **commandID** enthält, die im App-Manifest zu finden ist, und es ermöglicht, dass die switch-Anweisung in **searchApp.ts** ordnungsgemäß funktioniert.

Anschließend stellt sie eine Funktion bereit, `handleTeamsMessagingExtensionQuery()`um eingehende Abfragen für **ermäßigte Produkte nach Kategorie** zu verarbeiten.

```typescript
async function handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
): Promise<MessagingExtensionResponse> {

    // Seek the parameter by name, don't assume it's in element 0 of the array
    let categoryName = cleanupParam(query.parameters.find((element) => element.name === "categoryName")?.value);
    console.log(`💰 Discount query #${++queryCount}: Discounted products with categoryName=${categoryName}`);

    const products = await getDiscountedProductsByCategory(categoryName);

    console.log(`Found ${products.length} products in the Northwind database`)
    const attachments = [];
    products.forEach((product) => {
        const preview = CardFactory.heroCard(product.ProductName,
            `Avg discount ${product.AverageDiscount}%<br />Supplied by ${product.SupplierName} of ${product.SupplierCity}`,
            [product.ImageUrl]);

        const resultCard = cardHandler.getEditCard(product);
        const attachment = { ...resultCard, preview };
        attachments.push(attachment);
    });
    return {
        composeExtension: {
            type: "result",
            attachmentLayout: "list",
            attachments: attachments,
        },
    };
}
```

Beachten Sie, dass der Index im Array `query.parameters` möglicherweise nicht mit der Position des Parameters im Manifest übereinstimmt. Obwohl dies im Allgemeinen nur bei einem Befehl mit mehreren Parametern ein Problem darstellt, erhält der Code den Wert dennoch auf der Grundlage des Parameternamens, anstatt einen Index hart zu kodieren.

Nach dem Bereinigen des Parameters (Trimmen und Umgang mit der Tatsache, dass Copilot manchmal davon ausgeht, dass „**\***“ eine Wildcard ist, die mit allem übereinstimmt), ruft der Code die Northwind-Datenzugriffsebene zu `getDiscountedProductsByCategory()`auf.

Anschließend durchläuft er die Produkte und erstellt jeweils zwei Karten:

- eine _Vorschaukarte_, die als **Hero-Karte** implementiert wird. Dies wird in den Suchergebnissen auf der Benutzeroberfläche und in einigen Zitaten in Copilot angezeigt.

- eine _Ergebniskarte_, die als **adaptive** Karte implementiert wird, die alle Details enthält.

In der nächsten Aufgabe überprüfen wir den Code für adaptive Karten und sehen uns den Designer für adaptive Karten an.

## Aufgabe 4 – Untersuchen der adaptiven Karten und des zugehörigen Codes

Die adaptiven Karten des Projekts befinden sich im Ordner ** src/adaptiveCards/**. Es gibt drei Karten, die jeweils als JSON-Datei implementiert sind.

- **editCard.json** – Dies ist die anfängliche Karte, die von der Messaging-Erweiterung oder einem Copilot-Verweis angezeigt wird.

- **successCard.json** – Wenn Benutzerinnen und Benutzer eine Aktion ausführen, wird diese Karte angezeigt, um den Erfolg anzuzeigen. Diese ist meist identisch mit der Bearbeitungskarte, mit der Ausnahme, dass sie eine Nachricht an die Benutzerinnen und Benutzer enthält.

- **errorCard.json** – Wenn eine Aktion fehlschlägt, wird diese Karte angezeigt.

Sehen wir uns die Bearbeitungskarte im **Designer für adaptive Karten** an. Öffnen Sie Ihren Webbrowser auf [https://adaptivecards.io](https://adaptivecards.io) und wählen Sie oben die Option **Designer** aus.

![Screenshot des Designers für adaptive Karten.](../media/5-01-adaptive-card-designer-01.png)

Beachten Sie die Datenbindungsausdrücke, z. B. `"text": "📦 ${productName}",`. Dadurch wird die `productName`-Eigenschaft in den Daten an den Text auf der Karte gebunden.

Wählen Sie jetzt **Microsoft Teams** als Hostanwendung 1️⃣ aus. Fügen Sie den gesamten Inhalt von **editCard.json** in den Kartennutzlast-Editor 2️⃣ und den Inhalt von **sampleData.json** in den Beispieldaten-Editor 3️⃣ ein. Die Beispieldaten sind mit einem Produkt identisch, wie im Code angegeben. Die Karte sollte als gerendert angezeigt werden, mit Ausnahme eines kleinen Fehlers, der dadurch entsteht, dass der Designer nicht in der Lage ist, eines der adaptiven Kartenformate anzuzeigen.

![Screenshot von Copilot der adaptiven Karte, die basierend auf der JSON gerendert wird.](../media/5-01-adaptive-card-designer-02.png)

Versuchen Sie oben auf der Seite, das **Design** und das **emulierte Gerät** zu ändern, um zu sehen, wie die Karte im dunklen Design oder auf einem mobilen Gerät aussehen würde. Dies ist das Tool, das zum Erstellen adaptiver Karten für die Beispielanwendung verwendet wurde.

Zurück in in Visual Studio Code, öffnen Sie jetzt **cardHandler.ts**. Die Funktion `getEditCard()` wird von jedem der Messaging-Erweiterungsbefehle aufgerufen, um eine **Ergebniskarte** zu erhalten. Der Code liest das JSON der adaptiven Karte – das als Vorlage betrachtet wird – und bindet es dann an die Produktdaten. Das Ergebnis ist mehr JSON – die gleiche Karte wie die Vorlage, wobei alle Datenbindungsausdrücke ausgefüllt sind. Schließlich wird das `CardFactory`-Modul verwendet, um den endgültigen JSON in ein adaptives Kartenobjekt zum Rendern zu konvertieren.

```typescript
function getEditCard(product: ProductEx): any {

    var template = new ACData.Template(editCard);
    var card = template.expand({
        $root: {
            productName: product.ProductName,
            unitsInStock: product.UnitsInStock,
            productId: product.ProductID,
            categoryId: product.CategoryID,
            imageUrl: product.ImageUrl,
            supplierName: product.SupplierName,
            supplierCity: product.SupplierCity,
            categoryName: product.CategoryName,
            inventoryStatus: product.InventoryStatus,
            unitPrice: product.UnitPrice,
            quantityPerUnit: product.QuantityPerUnit,
            unitsOnOrder: product.UnitsOnOrder,
            reorderLevel: product.ReorderLevel,
            unitSales: product.UnitSales,
            inventoryValue: product.InventoryValue,
            revenue: product.Revenue,
            averageDiscount: product.AverageDiscount
        }
    });
    return CardFactory.adaptiveCard(card);
}
```

Wenn Sie nach unten scrollen, wird der Handler für jede der interaktiven Schaltflächen auf der Karte angezeigt. Die Karte sendet Daten, wenn auf eine interaktive Schaltfläche geklickt wird – insbesondere `data.txtStock`, das das Eingabefeld ** Menge** auf der Karte ist und `data.productId`, das in jeder Kartenaktion gesendet wird, um den Code darüber zu informieren, welches Produkt aktualisiert werden soll.

```typescript
async function handleTeamsCardActionUpdateStock(context: TurnContext) {

    const request = context.activity.value;
    const data = request.action.data;
    console.log(`🎬 Handling update stock action, quantity=${data.txtStock}`);

    if (data.txtStock && data.productId) {

        const product = await getProductEx(data.productId);
        product.UnitsInStock = Number(data.txtStock);
        await updateProduct(product);

        var template = new ACData.Template(successCard);
        var card = template.expand({
            $root: {
                productName: product.ProductName,
                unitsInStock: product.UnitsInStock,
                productId: product.ProductID,
                categoryId: product.CategoryID,
                imageUrl: product.ImageUrl,
                ...
```

Wie Sie sehen können, ruft der Code diese beiden Werte ab, aktualisiert die Datenbank und sendet dann eine neue Karte, die eine Nachricht und die aktualisierten Daten enthält.

## Herzlichen Glückwunsch!

Sie haben Übung 5 und das Plug-In-Lab für Microsoft 365-Massaging-Erweiterungen abgeschlossen. Vielen Dank, dass Sie diese Labs gemacht haben!

[Weiter zur Lab-Zusammenfassung...](./7-summary.md)
