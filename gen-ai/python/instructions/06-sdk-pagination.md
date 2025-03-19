---
lab:
  title: 06 - Paginieren produktübergreifender Abfrageergebnisse mit dem Azure Cosmos DB for NoSQL SDK
  module: Author complex queries with the Azure Cosmos DB for NoSQL
---

# Paginieren produktübergreifender Abfrageergebnisse mit dem SDK für Azure Cosmos DB for NoSQL

Azure Cosmos DB-Abfragen weisen in der Regel mehrere Ergebnisseiten auf. Die Paginierung erfolgt automatisch serverseitig, wenn Azure Cosmos DB nicht alle Abfrageergebnisse in einem einzigen Ausführungslauf zurückgeben kann. In vielen Anwendungen werden Sie mithilfe des SDK Code schreiben wollen, um Ihre Abfrageergebnisse in Stapeln auf eine performante Weise zu verarbeiten.

In diesem Lab erstellen Sie einen Feediterator, der in einer Schleife verwendet werden kann, um das gesamte Resultset zu durchlaufen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Erstellen von Copilots mit Azure Cosmos DB** noch nicht geklont und Ihre lokale Umgebung noch nicht eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#create-azure-cosmos-db-database-and-container-with-sample-data) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

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

1. In **Visual Studio Code**, im Bereich **Explorer**, suchen Sie den Ordner **python/06-sdk-pagination**.

1. Öffnen Sie das Kontextmenü für den Ordner **python/06-sdk-pagination** und wählen Sie dann **In integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Dieser Befehl öffnet das Terminal, wobei das Startverzeichnis bereits auf den Ordner **python/06-sdk-pagination** eingestellt ist.

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

## Paginieren durch kleine Resultsets einer SQL-Abfrage unter Verwendung des SDK

Beim Verarbeiten von Abfrageergebnissen müssen Sie sicherstellen, dass der Code alle Ergebnisseiten durchläuft und prüft, ob weitere Seiten vorhanden sind, bevor Sie weitere Anforderungen erstellen.

1. In **Visual Studio Code**, im Bereich **Explorer**, suchen Sie den Ordner **python/06-sdk-pagination**.

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

1. Erstellen Sie eine neue Variable namens **sql** vom Typ *Zeichenfolge* mit dem Wert **SELECT * FROM products WHERE products.price > 500**:

   ```python
   sql = "SELECT * FROM products WHERE products.price > 500"
   ```

1. Rufen Sie die Methode [`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items)mit der Variablen `sql` als Parameter für den Konstruktor auf. Setzen Sie den Wert `max_item_count` auf `50`, um die Anzahl der auf jeder Seite zurückgegebenen Elemente zu begrenzen.

   ```python
   iterator = container.query_items(
       query=sql,
       max_item_count=50  # Set maximum items per page
   )
   ```

1. Erstellen Sie eine asynchrone **for**-Schleife, die asynchron die [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page)-Methode für das Iterator-Objekt aufruft. Diese Methode gibt bei jedem Aufruf eine Seite mit Ergebnissen zurück.

   ```python
   async for page in iterator.by_page():
   ```

1. Durchlaufen Sie innerhalb der asynchronen **For**-Schleife asynchron über die paginierten Ergebnisse und drucken Sie die `id`, `name` und `price` der einzelnen Elemente.

   ```python
   async for product in page:
       print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")
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
           # Get database and container clients
           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products WHERE products.price > 500"
        
           iterator = container.query_items(
               query=sql,
               max_item_count=50  # Set maximum items per page
           )
        
           async for page in iterator.by_page():
               async for product in page:
                   print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")

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

1. Das Skript gibt nun Seiten mit jeweils 50 Elementen aus.

    > &#128161; Die Abfrage ergibt Hunderte von übereinstimmenden Elementen im Produktcontainer.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
