---
lab:
  title: "Paginieren produktübergreifender Abfrageergebnisse mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Paginieren produktübergreifender Abfrageergebnisse mit dem SDK für Azure Cosmos DB for NoSQL

Azure Cosmos DB-Abfragen weisen in der Regel mehrere Ergebnisseiten auf. Die Paginierung erfolgt automatisch serverseitig, wenn Azure Cosmos DB nicht alle Abfrageergebnisse in einem einzigen Ausführungslauf zurückgeben kann. In vielen Anwendungen werden Sie mithilfe des SDK Code schreiben wollen, um Ihre Abfrageergebnisse in Stapeln auf eine performante Weise zu verarbeiten.

In diesem Lab erstellen Sie einen Feediterator, der in einer Schleife verwendet werden kann, um das gesamte Resultset zu durchlaufen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALT+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Wenn Sie ein Azure Cosmos DB-Konto zum ersten Mal bereitstellen, wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Sobald die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen und verwenden, um unter Verwendung des Azure SDK für .NET oder einem anderen SDK Ihrer Wahl eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Cosmos DB*, und erstellen Sie dann eine neue**Azure Cosmos DB for NoSQL**-Kontoressource mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe*. |
    | **Account Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Kapazitätsmodus** | *Bereitgestellter Durchsatz* |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die zum Herstellen einer Verbindung mit dem Konto über das SDK erforderlich sind. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in diese Lab.

1. Kehren Sie zu **Visual Studio Code** zurück.

## Versorgen des Azure Cosmos DB for NoSQL-Kontos mit Daten

Das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] stellt Beispieldaten für ein beliebiges Azure Cosmos DB for NoSQL-Konto bereit. Das Tool ist ein Open-Source-Tool und über NuGet verfügbar. Sie installieren dieses Tool in der Azure Cloud Shell und verwenden es dann, um Seed-Werte für Ihre Datenbank zu generieren.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neuer Terminal** aus, um eine neue Terminal-Instanz zu öffnen.

1. Installieren Sie das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] für den globalen Einsatz auf Ihrem Computer.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Diese Ausführung dieses Befehls dauert möglicherweise einige Minuten. Dieser Befehl gibt die Warnmeldung („Das Tool „cosmicworks“ ist bereits installiert“) aus, wenn Sie die neueste Version dieses Tools bereits in der Vergangenheit installiert haben.

1. Führen Sie cosmicworks aus, um Ihr Azure Cosmos DB-Konto mit den folgenden Befehlszeilenoptionen zu starten:

    | **Option** | **Wert** |
    | ---: | :--- |
    | **--endpoint** | *Der Endpunktwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--key** | *Der Schlüsselwert, den Sie weiter zuvor in diesem Lab kopiert haben* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** lautet und Ihr Schlüssel **fDR2ci9QgkdkvERTQ==** lautet, dann lautet der Befehl: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Warten Sie, bis der Befehl **cosmicworks** das Konto mit einer Datenbank, einem Container und Elementen aufgefüllt hat.

1. Schließen Sie das integrierte Terminal.

## Paginieren durch kleine Resultsets einer SQL-Abfrage unter Verwendung des SDK

Beim Verarbeiten von Abfrageergebnissen müssen Sie sicherstellen, dass der Code alle Ergebnisseiten durchläuft und prüft, ob weitere Seiten vorhanden sind, bevor Sie weitere Anforderungen erstellen.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **10-paginate-results-sdk**.

1. Öffnen Sie die Codedatei **product.cs**.

1. Beobachten Sie die Klasse **Product** und die entsprechenden Eigenschaften. Insbesondere verwendet dieses Lab die Eigenschaften **id**, **name** und **price**.

1. Öffnen Sie im Bereich **Explorer** von **Visual Studio Code** die Codedatei **script.cs**.

1. Aktualisieren Sie die vorhandene Variable mit dem Namen **endpoint**, wobei ihr Wert auf den **Endpunkt** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt ist.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** lautet, dann lautet die C#-Anweisung: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Aktualisieren Sie die vorhandene Variable namens **key**, wobei ihr Wert auf den **Schlüssel** (key) des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** lautet, dann lautet die C#-Anweisung: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Erstellen Sie eine neue Variable namens **sql** vom Typ *string* mit dem Wert **SELECT p.id, p.name, p.price FROM products p**:

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. Erstellen Sie eine neue Variable vom Typ [QueryDefinition][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition], wobei Sie dem Konstruktor die Variable **sql** als Parameter übergeben:

    ```
    QueryDefinition query = new (sql);
    ```

1. Erstellen Sie mithilfe des leeren Standardkonstruktors eine neue Variable vom Typ [QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] mit dem Namen **options**:

    ```
    QueryRequestOptions options = new ();
    ```

1. Legen Sie die Eigenschaft [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] der Variablen **options** auf den Wert **50** fest:

    ```
    options.MaxItemCount = 50;
    ```

1. Erstellen Sie eine neue Variable namens **iterator** vom Typ [FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1], indem Sie die generische Methode [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] der Klasse [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] aufrufen, wobei Sie die Variablen **query** und **options** als Parameter übergeben:

    ```
    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);
    ```

1. Erstellen Sie eine **while**-Schleife, in der die Eigenschaft [HasMoreResults][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults] der Variablen **iterator** überprüft wird:

    ```
    while (iterator.HasMoreResults)
    {
        
    }
    ```

1. Rufen Sie innerhalb der **while**-Schleife die Methode [ReadNextAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]der Variablen **iterator**auf, die das Ergebnis in einer Variablen namens**products** vom Typ [ FeedResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1] unter Verwendung der Klasse **Product** speichert:

    ```
    FeedResponse<Product> products = await iterator.ReadNextAsync();
    ```

1. Erstellen Sie dann innerhalb der **while**-Schleife eine neue **foreach**-Schleife, in der die Variable **products** durchlaufen und mithilfe der Variablen **product** eine Instanz vom Typ **Product** dargestellt wird:

    ```
    foreach (Product product in products)
    {

    }
    ```

1. Verwenden Sie in der **foreach**-Schleife die integrierte statische Methode **Console.WriteLine**, um die Eigenschaften **id**, **name** und **price** der Variablen **product** zu formatieren und zu drucken:

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. Verwenden Sie in der **while**-Schleife die integrierte statische Methode **Console.WriteLine**, um die Nachricht *Press any key to get more results* zu drucken:

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. Verwenden Sie in der **while**-Schleife noch die integrierte statische Methode **Console.ReadKey**, um auf die nächste Tastatureingabe zu warten:

    ```
    Console.ReadKey();
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    QueryDefinition query = new (sql);

    QueryRequestOptions options = new ();
    options.MaxItemCount = 50;

    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);

    while (iterator.HasMoreResults)
    {
        FeedResponse<Product> products = await iterator.ReadNextAsync();
        foreach (Product product in products)
        {
            Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
        }

        Console.WriteLine("Press any key for next page of results");
        Console.ReadKey();        
    }
    ```

1. **Speichern** Sie die Datei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **10-paginate-results-sdk**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```

1. Das Skript gibt nun den ersten Satz von 50 Elementen aus, die der Abfrage entsprechen. Drücken Sie eine beliebige Taste, um den nächsten Satz von 50 Elementen abzurufen, bis die Abfrage alle übereinstimmenden Elemente durchlaufen hat.

    > &#128161; Die Abfrage ergibt Hunderte von übereinstimmenden Elementen im Produktcontainer.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
