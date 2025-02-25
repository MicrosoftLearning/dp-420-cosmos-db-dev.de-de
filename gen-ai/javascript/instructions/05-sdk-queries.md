---
title: 05 - Führen Sie eine Abfrage mit dem Azure Cosmos DB for NoSQL SDK aus
lab:
  title: 05 - Führen Sie eine Abfrage mit dem Azure Cosmos DB for NoSQL SDK aus
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: JavaScript SDK labs
---

# Ausführen einer Abfrage mit dem SDK für Azure Cosmos DB for NoSQL

Die neueste Version des JavaScript SDK für Azure Cosmos DB für NoSQL vereinfacht die Abfrage eines Containers und das Durchlaufen von Ergebnismengen mithilfe der modernen Funktionen von JavaScript.

Die `@azure/cosmos`-Bibliothek verfügt über eine integrierte Funktionalität, die die Abfrage von Azure Cosmos DB effizient und unkompliziert macht.

In dieser Übung verwenden Sie einen Iterator, um eine große Ergebnismenge zu verarbeiten, die von Azure Cosmos DB für NoSQL zurückgegeben wurde. Sie verwenden das JavaScript SDK, um Ergebnisse abzufragen und zu durchlaufen.

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

1. Navigieren Sie in **Visual Studio Code** im Bereich **Erkunden** zum Ordner **javascript/05-sdk-queries**.

1. Öffnen Sie das Kontextmenü für den Ordner **javascript/05-sdk-queries** und wählen Sie dann **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **javascript/05-sdk-queries** eingestellt ist.

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

Mit den Anmeldedaten des neu erstellten Kontos stellen Sie eine Verbindung zu den SDK-Klassen und zu der Datenbank und dem Container her, die Sie in einem früheren Schritt bereitgestellt haben, und durchlaufen die Ergebnisse einer SQL-Abfrage mithilfe des SDK.

Sie werden nun einen Iterator verwenden, um eine einfach zu verstehende Schleife über paginierte Ergebnisse aus Azure Cosmos DB zu erstellen. Hinter den Kulissen verwaltet das SDK den Feed-Iterator und stellt sicher, dass nachfolgende Anfragen korrekt aufgerufen werden.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Erkunden** zum Ordner **javascript/05-sdk-queries**.

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

1. Erstellen Sie eine neue Methode mit dem Namen **queryContainer** und einen Code, um diese Methode auszuführen, wenn Sie das Skript ausführen. In dieser Methode fügen Sie den Code zur Abfrage des Containers hinzu:

    ```javascript
    async function queryContainer() {
        // Query the container
    }

    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. Fügen Sie in der **queryContainer**-Methode den folgenden Code hinzu, um eine Verbindung zu der Datenbank und dem Container herzustellen, die Sie zuvor erstellt haben:

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. Erstellen Sie eine Abfragezeichenfolge-Variable nanmens `sql` und dem Wert `SELECT * FROM products p`.

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. Rufen Sie die [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1)-Methode mit der `sql`-Variable als Parameter für den Konstruktor auf. Der Parameter `enableCrossPartitionQuery`, wenn er auf `true` gesetzt ist, ermöglicht das Senden von mehr als einer Anfrage, um die Abfrage im Azure Cosmos DB-Dienst auszuführen. Es ist mehr als eine Anforderung erforderlich, wenn die Abfrage nicht auf einen einzigen Partitionsschlüsselwert beschränkt ist.

    ```javascript
    const iterator = container.items.query(
        query,
        { enableCrossPartitionQuery: true }
    );
    ```

1. Durchlaufen Sie die paginierten Ergebnisse und drucken Sie die `id`, `name` und `price` der einzelnen Elemente:

    ```javascript
    while (iterator.hasMoreResults()) {
        const { resources } = await iterator.fetchNext();
        for (const item of resources) {
            console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
        }
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

    async function queryContainer() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products p";
    
        const iterator = container.items.query(
            query,
            { enableCrossPartitionQuery: true }
        );
        
        while (iterator.hasMoreResults()) {
            const { resources } = await iterator.fetchNext();
            for (const item of resources) {
                console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
            }
        }
    }
    
    queryContainer().catch((error) => {
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

1. Das Skript wird nun jedes Produkt im Container ausgeben.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
