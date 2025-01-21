---
lab:
  title: Übung 1 – Projekt herunterladen und Dateien untersuchen
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Übung 1 – Projekt herunterladen und Dateien untersuchen

Durch das Erweitern eines deklarativen Agenten mit Aktionen können Daten abgerufen und aktualisiert werden, die in externen Systemen in Echtzeit gespeichert sind. Mithilfe von API-Plug-Ins können Sie über ihre APIs eine Verbindung mit externen Systemen herstellen, um Informationen abzurufen und zu aktualisieren.

### Übungsdauer

- **Geschätzte Zeit bis zur Fertigstellung**: 10 Minuten

## Aufgabe 1 -- Herunterladen des Startprojekts

Laden Sie zunächst das Beispielprojekt herunter. In einem Webbrowser:

1. Navigieren Sie zu [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript).
    1. Führen Sie die Schritte aus, um [den Quellcode des Repositorys](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) auf Ihren Computer herunterzuladen.
    1. Extrahieren Sie den Inhalt der heruntergeladenen ZIP-Datei, und erweitern Sie ihn in Ihren **Dokumentenordner**.
    1. Öffnen Sie in Visual Studio Code den Ordner .

Das Beispielprojekt ist ein Teams-Toolkit-Projekt, das einen deklarativen Agent und eine anonyme API enthält, die auf Azure Functions ausgeführt werden. Der deklarative Agent ist identisch mit einem neu erstellten deklarativen Agent, der das Teams-Toolkit verwendet. Die API gehört zu einem fiktiven italienischen Restaurant und ermöglicht es Ihnen, das heutige Menü und die Bestellung zu durchsuchen.

## Aufgabe 2 – Untersuchen der API-Definition

Sehen Sie sich zunächst die API-Definition der API des italienischen Restaurants an.

In Visual Studio Code:

1. Öffnen Sie in der Ansicht **Explorer** die Datei **appPackage/apiSpecificationFile/ristorante.yml**. Die Datei ist eine OpenAPI-Spezifikation, die die API des italienischen Restaurants beschreibt.
1. Suchen Sie die Eigenschaft **servers.url**

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    Beachten Sie, dass sie auf eine lokale URL verweist, die mit der Standard-URL übereinstimmt, wenn Azure Functions lokal ausgeführt wird.

1. Suchen Sie die Eigenschaft **paths**, die zwei Vorgänge enthält: **/dishes** zum Abrufen des heutigen Menüs und **/orders** zum Aufgeben einer Bestellung.

    > [!IMPORTANT]
    > Beachten Sie, dass jeder Vorgang die Eigenschaft **operationId** enthält, die den Vorgang in der API-Spezifikation eindeutig identifiziert. Copilot erfordert, dass jede Operation eine eindeutige ID hat, damit es weiß, welche API für bestimmte Eingabeaufforderungen von Benutzenden aufgerufen werden soll.

## Aufgabe 3 – Untersuchen der API-Implementierung

Sehen Sie sich als Nächstes die Beispiel-API an, die Sie in dieser Übung verwenden.

In Visual Studio Code:

1. Öffnen Sie in der Ansicht **Explorer** die Datei **src/data.json**. Die Datei enthält fiktives Menüelement für unser italienisches Restaurant. Jedes Gericht besteht aus:

    - Name,
    - Beschreibung,
    - Link zu einem Bild,
    - Preis,
    - als Teil welchen Gangs es serviert wird,
    - Typ (Gericht oder Getränk),
    - optional eine Liste von Allergenen

    In dieser Übung verwenden APIs diese Datei als Datenquelle.
1. Erweitern Sie als Nächstes den Ordner **src/functions**. Beachten Sie zwei Dateien mit den Namen **dishes.ts** und **placeOrder.ts**. Diese Dateien enthalten die Implementierung der beiden Vorgänge, die in der API-Spezifikation festgelegt sind.
1. Öffnen Sie die Datei **src/functions/dishes.ts**. Nehmen Sie sich einen Moment Zeit, um zu überprüfen, wie die API funktioniert. Es beginnt mit dem Laden der Beispieldaten aus der Datei **src/functions/data.json**.

    ```typescript
    import data from "../data.json";
    ```

    Als Nächstes sucht es in den verschiedenen Abfragezeichenfolge-Parametern nach möglichen Filtern, die der Client, der die API aufruft, möglicherweise weitergibt.

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    Basierend auf den in der Anforderung angegebenen Filtern filtert die API das Dataset und gibt eine Antwort zurück.

1. Überprüfen Sie als Nächstes die API zum Platzieren von Aufträgen, definiert in der Datei **src/functions/placeOrder.ts**. Die API beginnt mit dem Verweis auf die Beispieldaten. Anschließend wird die Form der Reihenfolge definiert, die der Client im Anforderungstext sendet.

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    Wenn die API die Anforderung verarbeitet, überprüft sie zunächst, ob die Anforderung einen Text enthält und ob sie über die richtige Form verfügt. Wenn nicht, wird die Anforderung mit einem Fehler „400: Ungültige Anforderung abgelehnt“.

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    Als Nächstes löst die API die Anforderung in Gerichte im Menü auf und berechnet den Gesamtpreis.

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > Beachten Sie, wie die API erwartet, dass der Client das Gericht anhand eines Teils seines Namens anstelle seiner ID angeben kann. Dies liegt daran, dass große Sprachmodelle besser mit Wörtern funktionieren als mit Zahlen. Darüber hinaus verfügt Copilot vor dem Aufrufen der API zum Aufgeben der Bestellung über den Namen des Gerichts, der als Teil der Eingabeaufforderung des Benutzenden verfügbar ist. Wenn Copilot anhand seiner ID auf ein Gericht verweisen musste, müsste es zuerst abrufen, was zusätzliche API-Anforderungen erfordern würde, und Copilot kann dies aktuell nicht tun.

    Wenn die API bereit ist, gibt sie eine Antwort mit einem Gesamtpreis, einer neu zusammengestellten Auftrags-ID und einem Status zurück.

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

