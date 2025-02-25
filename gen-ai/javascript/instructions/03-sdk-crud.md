---
title: 03 - Erstellen und aktualisieren Sie Dokumente mit dem Azure Cosmos DB for NoSQL SDK
lab:
  title: 03 - Erstellen und aktualisieren Sie Dokumente mit dem Azure Cosmos DB for NoSQL SDK
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: JavaScript SDK labs
---

# Erstellen und Aktualisieren von Dokumenten mit dem SDK für Azure Cosmos DB for NoSQL

Die `@azure/cosmos` Bibliothek enthält Methoden zum Erstellen, Abrufen, Aktualisieren und Löschen (CRUD) von Elementen in einem Azure Cosmos DB for NoSQL-Container. Zusammengenommen führen diese Methoden einige der häufigsten „CRUD“-Operationen für verschiedene Elemente in NoSQL-API-Containern aus.

In dieser Übung verwenden Sie das JavaScript SDK, um alltägliche CRUD-Operationen an einem Element in einem Azure Cosmos DB for NoSQL-Container durchzuführen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die **Build-Copilots mit Azure Cosmos DB**-Labs auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#import-the-azurecosmos-library) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Importieren der @azure/cosmos-Bibliothek

Die **@azure/cosmos**-Bibliothek ist auf **npm** verfügbar, um sie einfach in Ihre JavaScript-Projekte zu installieren.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **javascript/03-sdk-crud**.

1. Öffnen Sie das Kontextmenü für den Ordner **javascript/03-sdk-crud** und wählen Sie dann **In integriertem Terminal öffnen**, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **javascript/03-sdk-crud** eingestellt ist.

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

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **javascript/03-sdk-crud**.

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
    const { CosmosClient } = require("@azure/cosmos");
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

## Ausführen von Erstellungs- und Lesepunktvorgängen für Elemente mit dem SDK

Sie werden nun das Methodenset in der Klasse **Container** verwenden, um allgemeine Operationen an Elementen innerhalb eines NoSQL-API-Containers durchzuführen.

1. Kehren Sie zu **Visual Studio Code** zurück. Falls sie noch nicht geöffnet ist, öffnen Sie die **script.js**-Codedatei im **javascript/03-sdk-crud**-Ordner.

1. Erstellen Sie einen neuen Produktartikel und weisen Sie ihn einer Variablen mit dem Namen **Sattel** mit den folgenden Eigenschaften zu. Stellen Sie sicher, dass Sie den folgenden Code in die `main` Funktion einfügen:

    | Eigenschaft | Wert |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **Tags** | *{ tan, new, crisp }* |

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };
    ```

1. Rufen Sie die [`create`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-create) Methode der **Elemente der** Klasse des Containers auf und übergeben Sie die Variable **saddle** als Methodenparameter:

    ```javascript
    const { resource: item } = await container
        .items.create(saddle);
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const saddle = {
            id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
            categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
            name: "Road Saddle",
            price: 45.99,
            tags: ["tan", "new", "crisp"]
        };
    
        const { resource: item } = await container
                .items.create(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

    ```bash
    node script.js
    ```

1. Beobachten Sie das neue Element im **Daten-Explorer**.

1. Kehren Sie zu **Visual Studio Code** zurück.

1. Kehren Sie zur Registerkarte „Editor“ zurück, um die **script.js**-Codedatei zu erhalten.

1. Löschen Sie die folgenden Codezeilen:

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };

    const { resource: item } = await container
            .items.create(saddle);
    ```

1. Erstellen Sie eine String-Variable mit dem Namen **item_id** und dem Wert **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

    ```javascript
    const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Erstellen Sie eine String-Variable mit dem Namen **partition_key** und dem Wert **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Rufen Sie die [`read`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-read)Methode der **Item**-Klasse des Containers auf und übergeben Sie die Variablen **itemId** und **partitionKey** als Methodenparameter:

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();
    ```

    > &#128161; Mit der `read` Methode können Sie einen Punktlesevorgang für ein Element im Container ausführen. Die Methode erfordert die Parameter `itemId` und `partitionKey`, um den zu lesenden Artikel zu identifizieren. Im Gegensatz zur Ausführung einer Abfrage mit der SQL-Abfragesprache von Cosmos DB, um das einzelne Element zu finden, ist die `read`-Methode eine effizientere und kostengünstigere Möglichkeit, ein einzelnes Element abzurufen. Point Reads können die Daten direkt lesen und benötigen keine Abfrage-Engine, um die Anfrage zu verarbeiten.

1. Drucken Sie das Sattelobjekt mithilfe einer formatierten Ausgabezeichenfolge:

    ```javascript
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();
    
        console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

    ```bash
    node script.js
    ```

1. Beachten Sie die Ausgabe auf dem Terminal. Beachten Sie insbesondere den formatierten Ausgabetext mit der ID, dem Namen und dem Preis des Elements.

## Ausführen von Aktualisierungs- und Löschpunktvorgängen mit dem SDK

Beim Erlernen des SDK ist es nicht ungewöhnlich, ein Online-Azure Cosmos DB-Konto oder den Emulator zu verwenden, um ein Element zu aktualisieren und während der Ausführung eines Vorgangs zwischen dem Daten-Explorer und der IDE Ihrer Wahl hin und her zu wechseln und zu überprüfen, ob Ihre Änderung übernommen wurde. Hier werden Sie genau dies tun, indem Sie ein Element mithilfe des SDK aktualisieren und löschen.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Wählen Sie das einzige Element im Container aus, und beobachten Sie dann die Werte der Eigenschaften **name** und **price** des Elements.

    | **Eigenschaft** | **Farbe** |
    | ---: | :--- |
    | **Name** | *Road Saddle* |
    | **Preis** | *45,99 USD* |

    > &#128221; Zu diesem Zeitpunkt sollten diese Werte nicht geändert worden sein, seit Sie das Element erstellt haben. Sie werden die Werte in dieser Übung ändern.

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zur Registerkarte „Editor“ zurück, um die **script.js**-Codedatei zu erhalten.

1. Löschen Sie die folgenden Codezeilen:

    ```javascript
    console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    ```

1. Ändern Sie die Variable **saddle**, indem Sie den Wert der Eigenschaft „price“ auf **32,55** festlegen:

    ```javascript
    // Update the item
    saddle.price = 32.55;
    ```

1. Ändern Sie die Variable **saddle** erneut, indem Sie den Wert der Eigenschaft **namet** auf **Road LL Sattel** festlegen:

    ```javascript
    saddle.name = "Road LL Saddle";
    ```

1. Rufen Sie die [`replace`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-replace) Methode der **Item**-Klasse des Containers auf und übergeben Sie die **Saddle**-Variable als Methodenparameter:

    ```javascript
    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();

        // Update the item
        saddle.price = 32.55;
        saddle.name = "Road LL Saddle";
    
        await container.item(saddle.id, partitionKey).replace(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

    ```bash
    node script.js
    ```

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Wählen Sie das einzige Element im Container aus, und beobachten Sie dann die Werte der Eigenschaften **name** und **price** des Elements.

    | **Eigenschaft** | **Farbe** |
    | ---: | :--- |
    | **Name** | *Road LL Saddle* |
    | **Preis** | *32,55 USD* |

    > &#128221; Jetzt sollten diese Werte geändert worden sein, seit Sie das Element beobachtet haben.

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zur Registerkarte „Editor“ zurück, um die **script.js**-Codedatei zu erhalten.

1. Löschen Sie die folgenden Codezeilen:

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();

    // Update the item
    saddle.price = 32.55;
    saddle.name = "Road LL Saddle";

    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. Rufen Sie die [`delete`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-delete) Methode der **Item**-Klasse des Containers auf und übergeben Sie die Variablen **itemId** und **partitionKey** als Methodenparameter:

    ```javascript
    // Delete the item
    await container.item(itemId, partitionKey).delete();
    ```

1. Speichern Sie das Skript, und führen Sie es erneut aus:

    ```bash
    node script.js
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Beachten Sie, dass die Elementliste jetzt leer ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
