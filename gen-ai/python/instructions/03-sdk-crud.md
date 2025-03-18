---
lab:
  title: 03 - Erstellen und aktualisieren Sie Dokumente mit dem Azure Cosmos DB for NoSQL SDK
  module: Implement Azure Cosmos DB for NoSQL point operations
---

# Erstellen und Aktualisieren von Dokumenten mit dem SDK für Azure Cosmos DB for NoSQL

Die `azure-cosmos`-Bibliothek enthält Methoden zum Erstellen, Abrufen, Aktualisieren und Löschen (CRUD) von Elementen in einem Azure Cosmos DB for NoSQL-Container. Zusammengenommen führen diese Methoden einige der häufigsten „CRUD“-Operationen für verschiedene Elemente in NoSQL-API-Containern aus.

In diesem Lab verwenden Sie das Python SDK, um alltägliche CRUD-Operationen an einem Element in einem Azure Cosmos DB for NoSQL-Container durchzuführen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Erstellen von Copilots mit Azure Cosmos DB** noch nicht geklont und Ihre lokale Umgebung noch nicht eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#install-the-azure-cosmos-library) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Installieren der Azure-Cosmos-Bibliothek

Die **azure-cosmos**-Bibliothek ist auf **PyPI** für eine einfache Installation in Ihren Python-Projekten verfügbar.

1. Navigieren Sie in **Visual Studio Code** im Bereich **„Explorer“** zum Ordner **„python/03-sdk-crud“**.

1. Öffnen Sie das Kontextmenü für den Ordner **python/03-sdk-crud** und wählen Sie dann **In integriertem Terminal öffnen**, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **python/03-sdk-crud** eingestellt ist.

1. Erstellen und aktivieren Sie eine virtuelle Umgebung, um Abhängigkeiten zu verwalten:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Installieren Sie das Paket [azure-cosmos][pypi.org/project/azure-cosmos] mit dem folgenden Befehl:

   ```bash
   pip install azure-cosmos
   ```

1. Da wir die asynchrone Version des SDKs verwenden, müssen wir auch die `asyncio`-Bibliothek installieren:

   ```bash
   pip install asyncio
   ```

1. Für die asynchrone Version des SDK ist auch die `aiohttp`-Bibliothek erforderlich. Installieren Sie sie mithilfe des folgenden Befehls:

   ```bash
   pip install aiohttp
   ```

1. Installieren Sie die [azure-identity][pypi.org/project/azure-identity]-Bibliothek, die es uns ermöglicht, die Azure-Authentifizierung zu verwenden, um sich mit dem Azure Cosmos DB-Arbeitsbereich zu verbinden, indem Sie den folgenden Befehl verwenden:n:

   ```bash
   pip install azure-identity
   ```

## Verwenden Sie die Azure-Cosmos-Bibliothek

Mit den Anmeldeinformationen des neu erstellten Kontos verbinden Sie sich mit den SDK-Klassen und erstellen eine neue Datenbank- und Containerinstanz. Anschließend verwenden Sie den Daten-Explorer, um zu validieren, ob die Instanzen im Azure-Portal vorhanden sind.

1. Navigieren Sie in **Visual Studio Code** im Bereich **„Explorer“** zum Ordner **„python/03-sdk-crud“**.

1. Öffnen Sie die leere Python-Datei mit dem Namen **script.py**.

1. Fügen Sie die folgende `import` Anweisung hinzu, um die Klasse **PartitionKey** zu importieren:

   ```python
   from azure.cosmos import PartitionKey
   ```

1. Fügen Sie die folgenden `import` Anweisungen hinzu, um die asynchrone **CosmosClient** Klasse, die **DefaultAzureCredential**-Klasse und die **asyncio**-Bibliothek zu importieren:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio
   ```

1. Fügen Sie Variablen mit den Namen **Endpunkt** und **Anmeldeinformation** hinzu und setzen Sie den Wert **Endpunkt** auf den **Endpunkt** des Azure Cosmos DB-Kontos, das Sie zuvor erstellt haben. Die Variable **Anmeldeinformation** sollte auf eine neue Instanz der Klasse **DefaultAzureCredential** gesetzt werden:

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Zum Beispiel, wenn Ihr Endpunkt ist: **https://dp420.documents.azure.com:443/**, würde die Anweisung lauten: **Endpunkt = "https://dp420.documents.azure.com:443/"**.

1. Jede Interaktion mit Cosmos DB beginnt mit einer Instanz der `CosmosClient`. Um den asynchronen Client zu nutzen, müssen wir die Schlüsselwörter async/await verwenden, die nur innerhalb von asynchronen Methoden verwendet werden können. Erstellen Sie eine neue asynchrone Methode namens **Standard** und fügen Sie den folgenden Code hinzu, um eine neue Instanz der asynchronen Klasse **CosmosClient** unter Verwendung der Variablen **Endpunkt** und **Anmeldeinformation** zu erstellen:

   ```python
   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
   ```

    > &#128161; Da wir den asynchrone Clientn **CosmosClient** verwenden, muss man ihn auch aufwärmen und schließen, um ihn richtig zu nutzen. Wir empfehlen die Verwendung der `async with`-Schlüsselwörter, wie im obigen Code gezeigt, um Ihre Clients zu starten – diese Schlüsselwörter erzeugen einen Kontextmanager, der den Client automatisch aufwärmt, initialisiert und bereinigt, sodass Sie das nicht tun müssen.

1. Fügen Sie den folgenden Code hinzu, um eine Datenbank und einen Container zu erstellen, falls diese noch nicht vorhanden sind:

   ```python
   # Create database
   database = await client.create_database_if_not_exists(id="cosmicworks")
    
   # Create container
   container = await database.create_container_if_not_exists(
       id="products",
       partition_key=PartitionKey(path="/categoryId"),
       offer_throughput=400
   )
   ```

1. Fügen Sie unterhalb der Methode `main` den folgenden Code ein, um die Methode `main` unter Verwendung der Bibliothek `asyncio` auszuführen:

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. Ihre Datei**script.py** sollte nun wie folgt aussehen:

   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Speichern** der **script.py** Datei.

1. Bevor Sie das Skript ausführen, müssen Sie sich mit dem Befehl `az login` bei Azure anmelden. Führen Sie im Fenster Terminal Folgendes aus:

   ```bash
   az login
   ```

1. Führen Sie das Skript aus, um die Datenbank und den Container zu erstellen:

   ```bash
   python script.py
   ```

1. Wechseln Sie zu Ihrem Webbrowserfenster.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und beobachten Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

## Ausführen von Erstellungs- und Lesepunktvorgängen für Elemente mit dem SDK

Sie werden nun die Methoden in der **ContainerProxy**-Klasse verwenden, um allgemeine Vorgänge für Elemente in einem NoSQL-API-Container auszuführen.

1. Kehren Sie zu **Visual Studio Code** zurück. Wenn sie noch nicht geöffnet ist, öffnen Sie die **script.py**-Codedatei im Ordner **python/03-sdk-crud**.

1. Erstellen Sie einen neuen Produktartikel und weisen Sie ihn einer Variablen mit dem Namen **Sattel** mit den folgenden Eigenschaften zu:

    | Eigenschaft | Wert |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **Tags** | *{ tan, new, crisp }* |

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
   ```

1. Rufen Sie die [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item)-Methode der **Container-** Variablen auf, indem Sie die **Saddle-** Variable als Methodenparameter übergeben:

   ```python
   await container.create_item(body=saddle)
   ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )
        
           saddle = {
               "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
               "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
               "name": "Road Saddle",
               "price": 45.99,
               "tags": ["tan", "new", "crisp"]
           }
            
           await container.create_item(body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

   ```bash
   python script.py
   ```

1. Beobachten Sie das neue Element im **Daten-Explorer**.

1. Kehren Sie zu **Visual Studio Code** zurück.

1. Kehren Sie zur Editor-Registerkarte für die Code-Datei **script.py** zurück.

1. Löschen Sie die folgenden Codezeilen:

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
    
   await container.create_item(body=saddle)
   ```

1. Erstellen Sie eine String-Variable mit dem Namen **item_id** und dem Wert **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

   ```python
   item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
   ```

1. Erstellen Sie eine String-Variable mit dem Namen **partition_key** und dem Wert **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Rufen Sie die [`read_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item)-Methode der **Container-** Variablen auf und übergeben Sie die Variablen **item_id** und **partition_key** als Methodenparameter:

   ```python
   # Read item    
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
   ```

    > &#128161; Die `read_item`-Methode ermöglicht es Ihnen, einen Punktlesungsvorgang an einem Gegenstand im Container durchzuführen. Die Methode erfordert die Parameter `item_id` und `partition_key`, um das zu lesende Element zu identifizieren. Im Gegensatz zur Ausführung einer Abfrage mit der SQL-Abfragesprache von Cosmos DB, um das einzelne Element zu finden, ist die Methode `read_item` eine effizientere und kostengünstigere Möglichkeit, ein einzelnes Element abzurufen. Point Reads können die Daten direkt lesen und benötigen keine Abfrage-Engine, um die Anfrage zu verarbeiten.

1. Drucken Sie das Sattelobjekt mithilfe einer formatierten Ausgabezeichenfolge:

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )
       
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

   ```bash
   python script.py
   ```

1. Beachten Sie die Ausgabe auf dem Terminal. Beachten Sie insbesondere den formatierten Ausgabetext mit der ID, dem Namen und dem Preis des Elements.

## Ausführen von Aktualisierungs- und Löschpunktvorgängen mit dem SDK

Beim Erlernen des SDK ist es nicht ungewöhnlich, ein Online-Azure Cosmos DB-Konto oder den Emulator zu verwenden, um ein Element zu aktualisieren und während der Ausführung eines Vorgangs zwischen dem Daten-Explorer und der IDE Ihrer Wahl hin und her zu wechseln und zu überprüfen, ob Ihre Änderung übernommen wurde. Hier werden Sie genau dies tun, indem Sie ein Element mithilfe des SDK aktualisieren und löschen.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Wählen Sie das einzige Element im Container aus, und beobachten Sie dann die Werte der Eigenschaften **name** und **price** des Elements.

    | **Eigenschaft** | **Farbe** |
    | ---: | :--- |
    | **Name** | *Road Saddle* |
    | **Preis** | *45,99 USD* |

    > &#128221; Zu diesem Zeitpunkt sollten diese Werte nicht geändert worden sein, seit Sie das Element erstellt haben. Sie werden die Werte in dieser Übung ändern.

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zur Editor-Registerkarte für die Code-Datei **script.py** zurück.

1. Löschen Sie die folgenden Codezeilen:

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. Ändern Sie die Variable **saddle**, indem Sie den Wert der Eigenschaft „price“ auf **32,55** festlegen:

   ```python
   saddle["price"] = 32.55
   ```

1. Ändern Sie die Variable **saddle** erneut, indem Sie den Wert der Eigenschaft **namet** auf **Road LL Sattel** festlegen:

   ```python
   saddle["name"] = "Road LL Saddle"
   ```

1. Rufen Sie die [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item)-Methode der **Container-** Variablen auf, indem Sie die Variablen **item_id** und **saddle** als Methodenparameter übergeben:

   ```python
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )
        
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           saddle["price"] = 32.55
           saddle["name"] = "Road LL Saddle"
    
           await container.replace_item(item=item_id, body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

   ```bash
   python script.py
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

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zur Editor-Registerkarte für die Code-Datei **script.py** zurück.

1. Löschen Sie die folgenden Codezeilen:

   ```python
   # Read item
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
    
   saddle["price"] = 32.55
   saddle["name"] = "Road LL Saddle"
    
   await container.replace_item(item=item_id, body=saddle)
   ```

1. Rufen Sie die [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item)-Methode der **Container-** Variablen auf, indem Sie die Variablen **item_id** und **partition_key** als Methodenparameter übergeben:

   ```python
   # Delete the item
   await container.delete_item(item=item_id, partition_key=partition_key)
   ```

1. Speichern Sie das Skript, und führen Sie es erneut aus:

   ```bash
   python script.py
   ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Beachten Sie, dass die Elementliste jetzt leer ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
