---
title: 05 - Führen Sie eine Abfrage mit dem Azure Cosmos DB for NoSQL SDK aus
lab:
  title: 05 - Führen Sie eine Abfrage mit dem Azure Cosmos DB for NoSQL SDK aus
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: Python SDK labs
---

# Ausführen einer Abfrage mit dem SDK für Azure Cosmos DB for NoSQL

Die neueste Version des Python SDK für Azure Cosmos DB für NoSQL vereinfacht die Abfrage eines Containers und das Durchlaufen von Ergebnismengen mithilfe der modernen Funktionen von Python.

Die `azure-cosmos` Bibliothek verfügt über eine integrierte Funktionalität, die die Abfrage von Azure Cosmos DB effizient und unkompliziert macht.

In dieser Übung verwenden Sie einen Iterator, um eine große Ergebnismenge zu verarbeiten, die von Azure Cosmos DB für NoSQL zurückgegeben wurde. Sie werden das Python SDK verwenden, um Ergebnisse abzufragen und zu durchlaufen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und zum [nächsten Abschnitt](#create-azure-cosmos-db-database-and-container-with-sample-data) übergehen. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Azure Cosmos DB Datenbank und Container mit Beispieldaten erstellen

Wenn Sie bereits eine Azure Cosmos DB-Datenbank namens **cosmicworks-full** und einen Container darin namens **Produkte** erstellt haben, der mit Beispieldaten vorgeladen ist, können Sie ihn für dieses Lab verwenden und zum [nächsten Abschnitt](#install-the-azure-cosmos-library) übergehen. Führen Sie andernfalls die folgenden Schritte aus, um eine neue Beispieldatenbank und einen neuen Container zu erstellen.

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

## Installieren der Azure-Cosmos-Bibliothek

Die **azure-cosmos**-Bibliothek ist auf **PyPI** für eine einfache Installation in Ihren Python-Projekten verfügbar.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **python/05-sdk-queries**.

1. Öffnen Sie das Kontextmenü für den Ordner **python/05-sdk-queries** und wählen Sie dann **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **python/05-sdk-queries** eingestellt ist.

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

## Durchlaufen der Ergebnisse einer SQL-Abfrage mithilfe des SDKs

Mit den Anmeldedaten des neu erstellten Kontos stellen Sie eine Verbindung zu den SDK-Klassen und zu der Datenbank und dem Container her, die Sie in einem früheren Schritt bereitgestellt haben, und durchlaufen die Ergebnisse einer SQL-Abfrage mithilfe des SDK.

Sie werden nun einen Iterator verwenden, um eine einfach zu verstehende Schleife über paginierte Ergebnisse aus Azure Cosmos DB zu erstellen. Hinter den Kulissen verwaltet das SDK den Feed-Iterator und stellt sicher, dass nachfolgende Anfragen korrekt aufgerufen werden.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **python/05-sdk-queries**.

1. Öffnen Sie die leere Python-Datei mit dem Namen **script.py**.

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

1. Fügen Sie den folgenden Code hinzu, um eine Verbindung mit der Datenbank und dem Container herzustellen, die Sie zuvor erstellt haben:

   ```python
   database = client.get_database_client("cosmicworks-full")
   container = database.get_container_client("products")
   ```

1. Erstellen Sie eine Abfragezeichenfolge-Variable nanmens `sql` und dem Wert `SELECT * FROM products p`.

   ```python
   sql = "SELECT * FROM products p"
   ```

1. Rufen Sie die Methode [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items)mit der Variablen `sql` als Parameter für den Konstruktor auf.

   ```python
   result_iterator = container.query_items(
       query=sql
   )
   ```

1. Die Methode **query_items** gibt einen asynchronen Iterator zurück, den wir in einer Variablen namens `result_iterator` speichern. Dies bedeutet, dass jedes Objekt aus dem Iterator ein erwartbares Objekt ist und noch nicht das Abfrageergebnis enthält. Fügen Sie den folgenden Code hinzu, um eine asynchrone **For**-Schleife zu erstellen, die auf jedes Abfrageergebnis wartet, während Sie den asynchronen Iterator durchlaufen und die `id`, `name` und `price` jedes Elements ausgeben.

   ```python
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Fügen Sie unterhalb der Methode `main` den folgenden Code ein, um die Methode `main` unter Verwendung der Bibliothek `asyncio` auszuführen:

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. Ihre Datei**script.py** sollte nun wie folgt aussehen:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           result_iterator = container.query_items(
               query=sql
           )
            
           # Perform the query asynchronously
           async for item in result_iterator:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

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

1. Das Skript wird nun jedes Produkt im Container ausgeben.

## Ausführen einer Abfrage in einer logischen Partition

Im vorherigen Abschnitt haben Sie alle Elemente im Container abgefragt. Standardmäßig führt der asynchrone**CosmosClient** partitionsübergreifende Abfragen durch. Aus diesem Grund hat die von Ihnen ausgeführte Abfrage (`"SELECT * FROM products p"`) das Abfrageprogramm veranlasst, alle Partitionen im Container zu durchsuchen. Es empfiehlt sich, Abfragen immer innerhalb einer logischen Partition durchzuführen, um partitionsübergreifende Abfragen zu vermeiden. Dadurch sparen Sie letztendlich Geld und verbessern die Leistung.

In diesem Abschnitt werden Sie eine Abfrage innerhalb einer logischen Partition durchführen, indem Sie den Partitionsschlüssel in die Abfrage aufnehmen.

1. Kehren Sie zur Editor-Registerkarte für die Code-Datei **script.py** zurück.

1. Löschen Sie die folgenden Codezeilen:

   ```python
   result_iterator = container.query_items(
       query=sql
   )
    
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Ändern Sie das Skript, um eine Variable **partition_key** zu erstellen, um den Kategorie-ID-Wert für Trikots zu speichern. Füge den **partition_key** als Parameter zur Methode **query_items** hinzu. Dadurch wird sichergestellt, dass die Abfrage innerhalb der logischen Partition für die Kategorie Trikots ausgeführt wird.

   ```python
   partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

   result_iterator = container.query_items(
       query=sql,
       partition_key=partition_key
   )
   ```

1. Im vorherigen Abschnitt haben Sie eine asynchrone For-Schleife direkt auf dem asynchronen Iterator (`async for item in result_iterator:`) ausgeführt. Diesmal erstellen Sie asynchron eine vollständige Liste der tatsächlichen Abfrageergebnisse. Dieser Code führt dieselbe Aktion aus wie das Beispiel der For-Schleife, das Sie zuvor verwendet haben. Fügen Sie die folgenden Codezeilen hinzu, um eine Ergebnisliste zu erstellen und die Ergebnisse zu drucken:

   ```python
   item_list = [item async for item in result_iterator]

   for item in item_list:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. Ihre Datei**script.py** sollte nun wie folgt aussehen:

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

           result_iterator = container.query_items(
               query=sql,
               partition_key=partition_key
           )
    
           # Perform the query asynchronously
           item_list = [item async for item in result_iterator]
    
           for item in item_list:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **Speichern** der **script.py** Datei.

1. Führen Sie das Skript aus, um die Datenbank und den Container zu erstellen:

   ```bash
   python script.py
   ```

1. Das Skript gibt nun jedes Produkt innerhalb der Trikot-Kategorie aus und führt somit eine Abfrage innerhalb der Partition durch.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
