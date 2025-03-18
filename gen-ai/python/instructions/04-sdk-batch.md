---
lab:
  title: 04 - Fassen Sie mehrere Punktoperationen mit dem Azure Cosmos DB for NoSQL SDK zusammen
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# Führen Sie mehrere Punktoperationen zusammen mit dem Azure Cosmos DB for NoSQL SDK aus

Das `azure-cosmos` Python SDK bietet die `execute_item_batch` Methode, mehrere Punktoperationen in einem einzigen logischen Schritt auszuführen. Auf diese Weise können Entwickler mehrere Vorgänge effizient bündeln und ermitteln, ob sie erfolgreich serverseitig abgeschlossen wurden.

In dieser Übung verwenden Sie das Python SDK, um Stapelverarbeitungsvorgänge mit zwei Elementen durchzuführen, die sowohl erfolgreiche als auch fehlerhafte Transaktionsstapel veranschaulichen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Erstellen von Copilots mit Azure Cosmos DB** noch nicht geklont und Ihre lokale Umgebung noch nicht eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#install-the-azure-cosmos-library) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Installieren der Azure-Cosmos-Bibliothek

Die **azure-cosmos**-Bibliothek ist auf **PyPI** für eine einfache Installation in Ihren Python-Projekten verfügbar.

1. Navigieren Sie in **Visual Studio Code** im Bereich **„Explorer“** zum Ordner **„python/04-sdk-batch“**.

1. Öffnen Sie das Kontextmenü für den Ordner **python/04-sdk-batch** und wählen Sie dann **„In integriertem Terminal öffnen**“, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **python/04-sdk-batch** eingestellt ist.

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
       asyncio.run(main())
   ```

1. Ihre Datei**script.py** sollte nun wie folgt aussehen:

   ```python
   from azure.cosmos import exceptions, PartitionKey
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

## Erstellen eines transaktionalen Batches

Lassen Sie uns zunächst einen einfachen Transaktionsstapel erstellen, der zwei fiktive Produkte herstellt. Diese Stapel wird einen abgenutzten Sattel und einen rostigen Lenker in den Behälter mit der gleichen „gebrauchtes Zubehör“-Kategoriebeschreibung einfügen. Beide Elemente weisen denselben logischen Partitionsschlüssel auf, sodass sichergestellt ist, dass der Batchvorgang erfolgreich abläuft.

1. Kehren Sie zu **Visual Studio Code** zurück. Wenn sie noch nicht geöffnet ist, öffnen Sie die Code-Datei **script.py** im Ordner **python/04-sdk-batch**.

1. Erstellen Sie zwei Wörterbücher, die Produkte darstellen: einen **abgenutzten Sattel** und einen **verrosteten Lenker**. Beide Elemente verwenden denselben Partitionsschlüsselwert von **„9603ca6c-9e28-4a02-9194-51cdb7fea816“**.

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   ```

1. Definieren Sie den Partitionsschlüsselwert.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Erstellen Sie einen Batch mit den beiden Elementen.

   ```python
   batch = [saddle, handlebar]
   ```

1. Führen Sie den Batch mit der Methode `execute_item_batch` des Objekts `container` aus und drucken Sie die Antwort für jedes Element im Batch.

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
   ```python
   from azure.cosmos import exceptions, PartitionKey
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

           saddle = ("create", (
               {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           handlebar = ("create", (
               {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [saddle, handlebar]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

   ```bash
   python script.py
   ```

1. Die Ausgabe sollte einen erfolgreichen Statuscode für jede Operation anzeigen.

## Erstellen eines fehlerhaften transaktionalen Batch

Nun erstellen Sie einen Transaktionsbatch, der absichtlich einen Fehler erzeugt. Dieser Batch versucht, zwei Elemente mit unterschiedlichen logischen Partitionsschlüsseln einzufügen. Wir werden ein flackerndes Stroboskoplicht in der Kategorie „gebrauchtes Zubehör“ und einen neuen Helm in der Kategorie „tadelloses Zubehör“ erstellen. Laut Definition sollte dies eine ungültige Anforderung sein und beim Ausführen dieser Transaktion einen Fehler zurückgeben.

1. Kehren Sie zur Editor-Registerkarte für die Code-Datei **script.py** zurück.

1. Löschen Sie die folgenden Codezeilen:

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

   batch = [saddle, handlebar]
   ```

1. Modifiziere das Skript, um ein neues **flackerndes Stroboskoplicht** und einen **neuen Helm** mit unterschiedlichen Partitionsschlüsselwerten zu erstellen

   ```python
   light = ("create", (
       {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   helmet = ("create", (
       {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
   ))
   ```

1. Definieren Sie den Partitionsschlüsselwert für den Batch.

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. Erstellen Sie einen neuen Batch, der die beiden Elemente enthält.

   ```python
   batch = [light, helmet]
   ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

   ```python
   from azure.cosmos import exceptions, PartitionKey
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

           light = ("create", (
               {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           helmet = ("create", (
               {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [light, helmet]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **Speichern** Sie das Skript, und führen Sie es erneut aus:

   ```bash
   python script.py
   ```

1. Beachten Sie die Ausgabe vom Terminal. Der Statuscode für das zweite Element („Neuer Helm“) sollte **400** für **Ungültige Anforderung** sein. Dies lag daran, dass nicht alle Elemente innerhalb der Transaktion denselben Partitionsschlüsselwert hatten wie der transaktionale Batch.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
