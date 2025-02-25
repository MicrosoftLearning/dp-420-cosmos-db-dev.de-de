---
title: 06 - Paginieren produktübergreifender Abfrageergebnisse mit dem Azure Cosmos DB for NoSQL SDK
lab:
  title: 06 - Paginieren produktübergreifender Abfrageergebnisse mit dem Azure Cosmos DB for NoSQL SDK
  module: Author complex queries with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 9
parent: JavaScript SDK labs
---

# Paginieren produktübergreifender Abfrageergebnisse mit dem SDK für Azure Cosmos DB for NoSQL

Azure Cosmos DB-Abfragen weisen in der Regel mehrere Ergebnisseiten auf. Die Paginierung erfolgt automatisch serverseitig, wenn Azure Cosmos DB nicht alle Abfrageergebnisse in einem einzigen Ausführungslauf zurückgeben kann. In vielen Anwendungen werden Sie mithilfe des SDK Code schreiben wollen, um Ihre Abfrageergebnisse in Stapeln auf eine performante Weise zu verarbeiten.

In diesem Lab erstellen Sie einen Feediterator, der in einer Schleife verwendet werden kann, um das gesamte Resultset zu durchlaufen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und zum [nächsten Abschnitt](#create-azure-cosmos-db-database-and-container-with-sample-data) übergehen. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Azure Cosmos DB Datenbank und Container mit Beispieldaten erstellen

Wenn Sie bereits eine Azure Cosmos DB-Datenbank mit dem Namen **cosmicworks-full** und einen Container darin mit dem Namen **Produkte** erstellt haben, der mit Beispieldaten vorgeladen ist, können Sie ihn für dieses Lab verwenden und zum [nächsten Abschnitt](#import-the-azurecosmos-library) übergehen. Führen Sie andernfalls die folgenden Schritte aus, um eine neue Beispieldatenbank und einen neuen Container zu erstellen.

<details markdown=1>
<summary markdown="span"><strong>Klicken Sie hier, um die Schritte zur Erstellung der Datenbank und des Containers mit Beispieldaten zu erweitern/zu reduzieren</strong></summary>

1. Navigieren Sie innerhalb der neu erstellten **Azure Cosmos DB**-Kontoressource zum Bereich **Data Explorer**.

1. Wählen Sie im **Daten-Explorer** auf der Startseite **Schnellstart starten** aus.

1. Geben Sie im Formular **Neuer Container** die folgenden Werte ein:

    - **Datenbank-ID**: `cosmicworks-full`
    - **Container-ID**: `products`
    - **Partitionsschlüssel**: `/categoryId`
    - **Analysespeicher**: `Off`

1. Wählen Sie **OK**, um den neuen Container zu erstellen. Dieser Vorgang dauert ein oder zwei Minuten, während er die Ressourcen erstellt und den Container mit Beispielproduktdaten vorlädt.

1. Lassen Sie die Browserregisterkarte geöffnet, da wir später dorthin zurückkehren werden.

1. Wechseln Sie zurück zu **Visual Studio Code**.

</details>

## Importieren der @azure/cosmos-Bibliothek

Die **@azure/cosmos**-Bibliothek ist auf **npm** verfügbar, um sie einfach in Ihre JavaScript-Projekte zu installieren.

1. In **Visual Studio Code**, im **Explorer**-Bereich, suchen Sie den Ordner **javascript/06-sdk-pagination**.

1. Öffnen Sie das Kontextmenü für den Ordner **javascript/06-sdk-pagination** und wählen Sie dann **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Dieser Befehl öffnet das Terminal, wobei das Startverzeichnis bereits auf den Ordner **javascript/06-sdk-pagination** eingestellt ist.

1. Initialisieren eines neuen Node.js-Projekts:

    ```bash
    npm init -y
    ```

1. Installieren Sie das Paket [@azure/cosmos][npmjs.com/package/@azure/cosmos] mit dem folgenden Befehl:

    ```bash
    npm install @azure/cosmos
    ```

1. Installieren Sie die [@azure/identity][npmjs.com/package/@azure/identity]-Bibliothek, die es uns ermöglicht, die Azure-Authentifizierung zu verwenden, um sich mit dem Azure Cosmos DB-Arbeitsbereich zu verbinden, indem Sie den folgenden Befehl verwenden:

    ```bash
    npm install @azure/identity
    ```

## Durchlaufen der Ergebnisse einer SQL-Abfrage mithilfe des SDKs

Beim Verarbeiten von Abfrageergebnissen müssen Sie sicherstellen, dass der Code alle Ergebnisseiten durchläuft und prüft, ob weitere Seiten vorhanden sind, bevor Sie weitere Anforderungen erstellen.

1. In **Visual Studio Code**, im **Explorer**-Bereich, suchen Sie den Ordner **javascript/06-sdk-pagination**.

1. Öffnen Sie die leere JavaScript-Datei mit dem Namen **script.js**.

1. Fügen Sie die folgenden `require` Anweisungen hinzu, um die Bibliotheken **@azure/cosmos** und **@azure/identity** zu importieren:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. Fügen Sie Variablen mit den Namen **Endpunkt** und **Anmeldeinformation** hinzu und setzen Sie den Wert **Endpunkt** auf den **Endpunkt** des Azure Cosmos DB-Kontos, das Sie zuvor erstellt haben. Die Variable **Anmeldeinformation** sollte auf eine neue Instanz der Klasse **DefaultAzureCredential** gesetzt werden:

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; Zum Beispiel, wenn Ihr Endpunkt **https://dp420.documents.azure.com:443/** ist, würde die Anweisung **const endpoint = „https://dp420.documents.azure.com:443/“ lauten;**.

1. Fügen Sie eine neue Variable namens **Client** hinzu und initialisieren Sie sie als neue Instanz der Klasse **CosmosClient** unter Verwendung der Variablen **Endpunkt** und **Anmeldeinformation**:

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. Erstellen Sie eine neue Methode mit dem Namen **paginateResults** und Code, um diese Methode auszuführen, wenn Sie das Skript ausführen. In dieser Methode fügen Sie den Code zur Abfrage des Containers hinzu:

    ```javascript
    async function paginateResults() {
        // Query the container
    }

    paginateResults().catch((error) => {
        console.error(error);
    });
    ```

1. Fügen Sie innerhalb der Methode **paginateResults** den folgenden Code hinzu, um eine Verbindung zur Datenbank und zum Container herzustellen, die Sie zuvor erstellt haben::

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. Erstellen Sie eine Abfragezeichenfolge-Variable nanmens `sql` und dem Wert `SELECT * FROM products p`.

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. Erstellen Sie eine neue Variable namens `options` und setzen Sie sie auf ein Objekt, dessen Eigenschaft `enableCrossPartitionQuery` auf `true` gesetzt ist. Diese Eigenschaft ermöglicht das Senden von mehr als einer Anfrage zur Ausführung der Abfrage im Azure Cosmos DB-Dienst. Es ist mehr als eine Anforderung erforderlich, wenn die Abfrage nicht auf einen einzigen Partitionsschlüsselwert beschränkt ist. Setzen Sie die Eigenschaft `maxItemCount` auf `50`, um die Anzahl der Elemente pro Seite zu begrenzen:

    ```javascript
    const options = {
        enableCrossPartitionQuery: true,
        maxItemCount: 50 // Set the maximum number of items per page
    };
    ```

1. Rufen Sie die Methode [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) mit den Variablen `sql` und `options` als Parameter für den Konstruktor auf. Diese Methode gibt einen Iterator zurück, der verwendet werden kann, um die nächste Seite der Ergebnisse abzurufen:

    ```javascript
    const iterator = container.items.query(query, options);
    ```

1. Durchlaufen Sie die paginierten Ergebnisse und drucken Sie die `id`, `name` und `price` der einzelnen Elemente. Die `iterator.getAsyncIterator`-Methode gibt einen asynchronen Iterator zurück, der zum Abrufen der `for await...of`-Schleife verwendet werden kann, um jede Seite der Ergebnisse abzurufen. Diese Schleife behandelt automatisch die asynchrone Iteration über die Seiten.

    ```javascript
    for await (const page of iterator.getAsyncIterator()) {
        page.resources.forEach(product => {
            console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
        });
    }
    ```

1. Ihre **script.js** Datei sollte jetzt so aussehen:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function paginateResults() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products WHERE products.price > 500";

        const options = {
            enableCrossPartitionQuery: true,
            maxItemCount: 50 // Set the maximum number of items per page
        };
        
        const iterator = container.items.query(query, options);

        for await (const page of iterator.getAsyncIterator()) {
            page.resources.forEach(product => {
                console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
            });
        }
    }
    
    paginateResults().catch((error) => {
        console.error(error);
    });
    ```

1. **Speichern** der **script.js**-Datei.

1. Bevor Sie das Skript ausführen, müssen Sie sich mit dem Befehl `az login` bei Azure anmelden. Führen Sie im Fenster Terminal Folgendes aus:

    ```bash
    az login
    ```

1. Führen Sie das Skript aus, um die Datenbank und den Container zu erstellen:

    ```bash
    node script.js
    ```

1. Das Skript gibt nun Seiten mit jeweils 50 Elementen aus.

    > &#128161; Die Abfrage ergibt Hunderte von übereinstimmenden Elementen im Produktcontainer.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
