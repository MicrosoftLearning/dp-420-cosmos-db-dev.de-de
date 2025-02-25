---
title: 04 - Fassen Sie mehrere Punktoperationen mit dem Azure Cosmos DB for NoSQL SDK zusammen
lab:
  title: 04 - Fassen Sie mehrere Punktoperationen mit dem Azure Cosmos DB for NoSQL SDK zusammen
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
layout: default
nav_order: 7
parent: JavaScript SDK labs
---

# Führen Sie mehrere Punktoperationen zusammen mit dem Azure Cosmos DB for NoSQL SDK aus

Die `TransactionalBatch`-Klasse im JavaScript-SDK für Azure Cosmos DB bietet Funktionen zum Verfassen und Ausführen von Batch-Vorgängen innerhalb desselben logischen Partitionsschlüssels. Mit diesem Feature können Sie mehrere Vorgänge in einer einzigen Transaktion durchführen und sicherstellen, dass entweder alle oder keiner der Vorgänge abgeschlossen werden.

In diesem Lab verwenden Sie das JavaScript SDK, um Vorgänge mit zwei Elementen in einem Batch durchzuführen, bei denen Sie versuchen, zwei Elemente als eine einzige logische Einheit zu erstellen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die **Build-Copilots mit Azure Cosmos DB**-Labs auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#import-the-azurecosmos-library) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Importieren der @azure/cosmos-Bibliothek

Die **@azure/cosmos**-Bibliothek ist auf **npm** verfügbar, um sie einfach in Ihre JavaScript-Projekte zu installieren.

1. In **Visual Studio Code**, im Bereich **Explorer**, suchen Sie den Ordner **javascript/04-sdk-batch**.

1. Öffnen Sie das Kontextmenü für den Ordner **javascript/04-sdk-batch** und wählen Sie dann **in integriertem Terminal öffnen**, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Dieser Befehl öffnet das Terminal, wobei das Startverzeichnis bereits auf den Ordner **javascript/04-sdk-batch** eingestellt ist.

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

## Verwenden Sie die @azure/cosmos-Bibliotheken

Sobald die Azure Cosmos DB-Bibliothek aus dem Azure SDK für JavaScript importiert wurde, können Sie ihre Klassen sofort verwenden, um eine Verbindung zu einem Azure Cosmos DB for NoSQL-Konto herzustellen. Die Klasse **CosmosClient** ist die Kernklasse, die verwendet wird, um die erste Verbindung zu einem Azure Cosmos DB for NoSQL-Konto herzustellen.

1. In **Visual Studio Code**, im Bereich **Explorer**, suchen Sie den Ordner **javascript/04-sdk-batch**.

1. Öffnen Sie die leere JavaScript-Datei mit dem Namen **script.js**.

1. Fügen Sie die folgenden `require` Anweisungen hinzu, um die Bibliotheken **@azure/cosmos** und **@azure/identity** zu importieren:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
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

1. Fügen Sie den folgenden Code hinzu, um eine Datenbank und einen Container zu erstellen, falls diese noch nicht vorhanden sind:

    ```javascript
    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. Ihre **script.js** Datei sollte jetzt so aussehen:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
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

1. Wechseln Sie zu Ihrem Webbrowserfenster.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und beobachten Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

## Erstellen eines transaktionalen Batches

Lassen Sie uns zunächst einen einfachen transaktionalen Batch erstellen, der zwei fiktive Produkte zum Container hinzufügt. Diese Stapel wird einen abgenutzten Sattel und einen rostigen Lenker in den Behälter mit der gleichen „gebrauchtes Zubehör“-Kategoriebeschreibung einfügen. Beide Elemente haben denselben logischen Partitionsschlüssel, so dass ein erfolgreicher Batch-Vorgang gewährleistet ist.

1. Kehren Sie zu **Visual Studio Code** zurück. Wenn sie noch nicht geöffnet ist, öffnen Sie die Code-Datei **script.js** im Ordner **javascript/04-sdk-batch**.

1. Definieren Sie die beiden Produktelemente, die in den transaktionalen Batch eingefügt werden sollen:

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    ```

1. Erstellen Sie einen transaktionalen Batch für denselben logischen Partitionsschlüssel, und fügen Sie die Elemente hinzu:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = container.items.batch(partitionKey)
        .create(saddle)
        .create(handlebar);
    ```

1. Führen Sie den Batch aus und drucken Sie den Status des Vorgangs:

    ```javascript
    const response = await batch.execute();
    console.log(`Status: ${response.statusCode}`);
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: saddle },
            { operationType: BulkOperationType.Create, resourceBody: handlebar },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

    ```bash
    node script.js
    ```

1. Die Ausgabe sollte einen erfolgreichen Statuscode für jede Operation anzeigen.

## Erstellen eines fehlerhaften transaktionalen Batch

Nun erstellen Sie einen Transaktionsbatch, der absichtlich einen Fehler erzeugt. Dieser Batch versucht, zwei Elemente mit unterschiedlichen logischen Partitionsschlüsseln einzufügen. Wir werden ein flackerndes Stroboskoplicht in der Kategorie „gebrauchtes Zubehör“ und einen neuen Helm in der Kategorie „tadelloses Zubehör“ erstellen. Laut Definition sollte dies eine ungültige Anforderung sein und beim Ausführen dieser Transaktion einen Fehler zurückgeben.

1. Kehren Sie zur Registerkarte „Editor“ zurück, um die **script.js**-Codedatei zu erhalten.

1. Löschen Sie die folgenden Codezeilen:

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };

    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: saddle },
        { operationType: BulkOperationType.Create, resourceBody: handlebar },
    ];
    ```

1. Modifiziere das Skript, um ein neues **flackerndes Stroboskoplicht** und einen **neuen Helm** mit unterschiedlichen Partitionsschlüsselwerten zu erstellen

    ```javascript
    const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    ```

1. Erstellen Sie eine String-Variable mit dem Namen **partition_key** und dem Wert **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Erstellen Sie einen neuen Stapel mit den Artikeln **Licht** und **Helm**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: light },
        { operationType: BulkOperationType.Create, resourceBody: helmet },
    ];
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: light },
            { operationType: BulkOperationType.Create, resourceBody: helmet },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

    ```bash
    node script.js
    ```

1. Beachten Sie die Ausgabe vom Terminal. Der Statuscode der Elemente sollte entweder **424** für **Fehlerhafte Abhängigkeit** oder **400** für **Fehlerhafte Anforderung** lauten. Dies lag daran, dass nicht alle Elemente innerhalb der Transaktion denselben Partitionsschlüsselwert hatten wie der transaktionale Batch.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
