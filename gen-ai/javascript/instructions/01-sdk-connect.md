---
title: 01 - Stellen Sie mit dem SDK eine Verbindung zu Azure Cosmos DB für NoSQL her
lab:
  title: 01 - Stellen Sie mit dem SDK eine Verbindung zu Azure Cosmos DB für NoSQL her
  module: Use the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 4
parent: JavaScript SDK labs
---

# Herstellen einer Verbindung mit Azure Cosmos DB for NoSQL über das SDK

Das Azure SDK für JavaScript (Node.js & Browser) ist eine Suite von Client-Bibliotheken, die eine konsistente Entwickleroberfläche für die Interaktion mit vielen Azure-Diensten bietet. Client-Bibliotheken sind Pakete, die Sie verwenden würden, um diese Ressourcen zu nutzen und mit ihnen zu interagieren.

In diesem Lab stellen Sie mithilfe des Azure SDK für JavaScript eine Verbindung zu einem Azure Cosmos DB for NoSQL-Konto her.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die **Build-Copilots mit Azure Cosmos DB**-Labs auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#import-the-azurecosmos-library) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Importieren der @azure/cosmos-Bibliothek

Die **@azure/cosmos**-Bibliothek ist auf **npm** verfügbar, um sie einfach in Ihre JavaScript-Projekte zu installieren.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Erkunden** zum Ordner **javascript/01-sdk-connect**.

1. Öffnen Sie das Kontextmenü für den Ordner **javascript/01-sdk-connect** und wählen Sie dann **In integriertem Terminal öffnen**, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **javascript/01-sdk-connect** eingestellt ist.

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

1. Navigieren Sie in **Visual Studio Code** im Bereich **Erkunden** zum Ordner **javascript/01-sdk-connect**.

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

1. Fügen Sie eine `async`-Funktion mit dem Namen **Standard** hinzu, um Kontoeigenschaften zu lesen und zu drucken:

    ```javascript
    async function main() {
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }
    ```

1. Rufen Sie die **Standard**-Funktion auf:

    ```javascript
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
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }

    main().catch((error) => console.error(error));
    ```

1. **Speichern** der **script.js**-Datei.

## Testen des Skripts

Da der JavaScript-Code für die Verbindung mit dem Azure Cosmos DB for NoSQL-Konto nun abgeschlossen ist, können Sie das Skript testen. Dieses Skript druckt die Standardkonsistenzstufe und den Namen des ersten schreibbaren Bereichs. Bei der Erstellung des Kontos haben Sie einen Speicherort angegeben und sollten erwarten, dass derselbe Speicherortwert als Ergebnis dieses Skripts angezeigt wird.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **javascript/01-sdk-connect** und wählen Sie dann **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Bevor Sie das Skript ausführen, müssen Sie sich mit dem Befehl `az login` bei Azure anmelden. Führen Sie im Fenster Terminal Folgendes aus:

    ```bash
    az login
    ```

1. Führen Sie das Skript mithilfe des Befehls `node` aus:

    ```bash
    node script.js
    ```

1. Das Skript gibt nun die Standard-Konsistenzstufe des Kontos und den ersten beschreibbaren Bereich aus. Wenn beispielsweise die Standard-Konsistenzstufe für das Konto **Sitzung** lautet und der erste beschreibbare Bereich **Ost-USA** war, würde das Skript Folgendes ausgeben:

    ```text
    Consistency Policy: Session
    Primary Region: East US
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
