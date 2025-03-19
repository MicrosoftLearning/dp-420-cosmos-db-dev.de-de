---
lab:
  title: 02 - Konfigurieren des Azure Cosmos DB JavaScript SDK für die Offline-Entwicklung
  module: Configure the Azure Cosmos DB for NoSQL SDK
---

# Konfigurieren des Azure Cosmos DB JavaScript SDK für die Offline-Entwicklung

Der Azure Cosmos DB-Emulator ist ein lokales Tool, das zu Entwicklungs- und Testzwecken den Azure Cosmos DB-Dienst emuliert. Der Emulator unterstützt die NoSQL API und kann bei der Entwicklung von Code mit dem Azure SDK für JavaScript anstelle des Clouddienstes verwendet werden.

In dieser Lab stellen Sie eine Verbindung zum Azure Cosmos DB Emulator über das Azure SDK für JavaScript her.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Ausführen des Azure Cosmos DB-Emulators

Wenn Sie eine gehostete Lab-Umgebung verwenden, sollte der Emulator dort bereits installiert sein. Andernfalls lesen Sie die [Installationsanleitung](https://docs.microsoft.com/azure/cosmos-db/local-emulator), um den Azure Cosmos DB-Emulator zu installieren. Sobald der Emulator gestartet ist, können Sie die Verbindungszeichenfolge abrufen und sie verwenden, um mit dem Azure SDK für JavaScript eine Verbindung zum Emulator herzustellen.

> &#128161; Sie können optional den [neuen Linux-basierten Azure Cosmos DB Emulator (in der Vorschau)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux) installieren, der als Docker-Container verfügbar ist. Er unterstützt die Ausführung auf einer Vielzahl von Prozessoren und Betriebssystemen.

1. Starten Sie den **Azure Cosmos DB-Emulator**.

    > 💡 Wenn Sie Windows verwenden, wird der Azure Cosmos DB Emulator sowohl an die Windows-Taskleiste als auch an das Startmenü angeheftet. Wenn er nicht über die angehefteten Symbole gestartet wird, versuchen Sie, ihn durch Doppelklick auf die Datei **C:\Programme\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** zu öffnen.

1. Warten Sie, bis der Emulator Ihren Standardbrowser öffnet, und navigieren Sie zur Landing Page **https://localhost:8081/_explorer/index.html**.

1. Beachten Sie im Bereich **Schnellstart** die **Primäre Verbindungszeichenfolge**. Sie werden diese Verbindungszeichenfolge später verwenden.

> &#128221; Manchmal wird die Landing Page nicht erfolgreich geladen, obwohl der Emulator läuft. Wenn dies geschieht, können Sie die bekannte Verbindungszeichenfolge verwenden, um eine Verbindung mit dem Emulator herzustellen. Die bekannte Verbindungszeichenfolge lautet: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## Importieren der @azure/cosmos-Bibliothek

Die **@azure/cosmos**-Bibliothek ist auf **npm** verfügbar, um sie einfach in Ihre JavaScript-Projekte zu installieren.

1. In **Visual Studio Code**, im **Explorer**-Bereich, durchsuchen Sie den **javascript/02-sdk-offline**-Ordner.

1. Öffnen Sie das Kontextmenü für den Ordner **javascript/02-sdk-offline** und wählen Sie dann **Öffnen im integrierten Terminal**, um eine neue Terminalinstanz zu öffnen.

    > 💡 Dieser Befehl öffnet das Terminal, wobei das Startverzeichnis bereits auf den Ordner **javascript/02-sdk-offline** eingestellt ist.

1. Initialisieren eines neuen Node.js-Projekts:

    ```bash
    npm init -y
    ```

1. Installieren Sie das Paket [@azure/cosmos][npmjs.com/package/@azure/cosmos] mit dem folgenden Befehl:

    ```bash
    npm install @azure/cosmos
    ```

## Verbindung zum Emulator über das JavaScript-SDK

1. In **Visual Studio Code**, im **Explorer**-Bereich, durchsuchen Sie den **javascript/02-sdk-offline**-Ordner.

1. Öffnen Sie die leere JavaScript-Datei mit dem Namen **script.js**.

1. Fügen Sie den folgenden Code hinzu, um eine Verbindung zum Emulator herzustellen, eine Datenbank zu erstellen und ihre ID zu drucken:

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    
    // Connection string for the Azure Cosmos DB Emulator
    const endpoint = "https://127.0.0.1:8081/";
    const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    // Initialize the Cosmos client
    const client = new CosmosClient({ endpoint, key });
    
    async function main() {
        // Create a database
        const databaseName = "cosmicworks";
        const { database } = await client.databases.createIfNotExists({ id: databaseName });
    
        // Print the database ID
        console.log(`New Database: Id: ${database.id}`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **Speichern** der **script.js**-Datei.

## Ausführen des Skripts

1. Verwenden Sie dasselbe Terminalfenster in **Visual Studio Code**, das Sie zur Installation der Bibliothek für dieses Lab verwendet haben. Wenn Sie es schließen, öffnen Sie das Kontextmenü für den Ordner **javascript/02-sdk-offline** und wählen Sie dann **Öffnen im integrierten Terminal**, um eine neue Terminalinstanz zu öffnen.

1. Führen Sie das Skript mithilfe des Befehls `node` aus:

    ```bash
    node script.js
    ```

1. Das Skript erstellt eine Datenbank mit dem Namen `cosmicworks` im Emulator. Die Ausgabe sollte etwa folgendermaßen aussehen:

    ```text
    New Database: Id: cosmicworks
    ```

## Erstellen und Anzeigen eines neuen Containers

Sie können das Skript erweitern, um einen Container innerhalb der Datenbank zu erstellen.

### Aktualisierter Code

1. Ändern Sie die Datei`script.js`zum**Ersetzen**der folgenden unteren Zeile der Datei`main().catch((error) => console.error(error));`um einen Container zu erstellen:

```javascript
async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

Die Datei `script.js` sollte jetzt wie folgt aussehen:

```javascript
const { CosmosClient } = require("@azure/cosmos");
process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

// Connection string for the Azure Cosmos DB Emulator
const endpoint = "https://127.0.0.1:8081/";
const key = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";

// Initialize the Cosmos client
const client = new CosmosClient({ endpoint, key });

async function main() {
    // Create a database
    const databaseName = "cosmicworks";
    const { database } = await client.databases.createIfNotExists({ id: databaseName });

    // Print the database ID
    console.log(`New Database: Id: ${database.id}`);
}

async function createContainer() {
    const containerName = "products";
    const partitionKeyPath = "/categoryId";
    const throughput = 400;

    const { container } = await client.database("cosmicworks").containers.createIfNotExists({
        id: containerName,
        partitionKey: { paths: [partitionKeyPath] },
        throughput: throughput
    });

    // Print the container ID
    console.log(`New Container: Id: ${container.id}`);
}

main()
    .then(createContainer)
    .catch((error) => console.error(error));
```

### Führen Sie das aktualisierte Skript aus

1. Führen Sie das aktualisierte Skript mit dem folgenden Befehl aus:

    ```bash
    node script.js
    ```

1. Das Skript erstellt einen Container namens `products` im Emulator. Die Ausgabe sollte etwa folgendermaßen aussehen:

    ```text
    New Database: Id: cosmicworks
    New Container: Id: products
    ```

### Bestätigen der Ergebnisse:

1. Wechseln Sie zu dem Browser, in dem der Data Explorer des Emulators geöffnet ist.

1. Aktualisieren Sie die **NoSQL API**, um die neue **cosmicworks**-Datenbank und den **Produkte**-Container zu sehen.

## Beenden des Azure Cosmos DB-Emulators

Es ist wichtig, den Emulator zu beenden, wenn Sie ihn nicht mehr verwenden, um Systemressourcen freizugeben. Führen Sie die folgenden Schritte für Ihr Betriebssystem aus:

### Auf macOS oder Linux:

Wenn Sie den Emulator in einem Terminalfenster gestartet haben, gehen Sie wie folgt vor:

1. Suchen Sie das Terminalfenster, in dem der Emulator ausgeführt wird.

1. Drücken Sie `Ctrl + C`, um den Emulatorprozess zu beenden.

Alternativ können Sie den Emulatorprozess auch manuell beenden:

1. Öffnen Sie ein neues Terminalfenster.

1. Verwenden Sie den folgenden Befehl, um den Emulatorprozess zu finden:

    ```bash
    ps aux | grep CosmosDB.Emulator
    ```

Identifizieren Sie die **PID** (Prozess-ID) des Emulatorprozesses in der Ausgabe. Verwenden Sie den Befehl Beenden, um den Emulatorprozess zu beenden:

```bash
kill <PID>
```

### Unter Windows:

1. Suchen Sie das Azure Cosmos DB Emulator-Symbol im Windows-Infobereich (in der Nähe der Uhr in der Taskleiste).

1. Klicken Sie mit der rechten Maustaste auf das Symbol des Emulators, um das Kontextmenü zu öffnen.

1. Wählen Sie **Beenden**, um den Emulator herunterzufahren.

> 💡 Es kann eine Minute dauern, bis alle Instanzen des Emulators beendet sind.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
