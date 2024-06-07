---
lab:
  title: "Ausführen einer Abfrage mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# Ausführen einer Abfrage mit dem SDK für Azure Cosmos DB for NoSQL

Mit der neuesten Version des .NET SDK für Azure Cosmos DB for NoSQL ist das Abfragen eines Containers und das asynchrone Durchlaufen von Resultsets mit den neuesten bewährten Methoden und Sprachfeatures von C# einfacher denn je.

Diese Bibliothek verfügt über spezielle Funktionen, die das Abfragen von Azure Cosmos DB mithilfe von [https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet] erleichtern.

In diesem Lab verwenden Sie einen asynchronen Datenstrom, um ein großes Resultset zu durchlaufen, das von Azure Cosmos DB for NoSQL zurückgegeben wird. Sie verwenden das .NET SDK, um Ergebnisse abzufragen und zu durchlaufen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALTTASTE+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Wenn Sie ein Azure Cosmos DB-Konto zum ersten Mal bereitstellen, wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Sobald die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen und verwenden, um unter Verwendung des Azure SDK für .NET oder einem anderen SDK Ihrer Wahl eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Cosmos DB*, und erstellen Sie dann eine neue**Azure Cosmos DB for NoSQL**-Kontoressource mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Account Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Kapazitätsmodus** | *Bereitgestellter Durchsatz* |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die erforderlich sind, um vom SDK aus eine Verbindung mit dem Konto herzustellen. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Kehren Sie zu **Visual Studio Code** zurück.

## Versorgen des Azure Cosmos DB for NoSQL-Kontos mit Daten

Das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] stellt Beispieldaten für ein beliebiges Azure Cosmos DB for NoSQL-Konto bereit. Das Tool ist ein Open-Source-Tool und über NuGet verfügbar. Sie installieren dieses Tool in der Azure Cloud Shell und verwenden es dann, um Seed-Werte für Ihre Datenbank zu generieren.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neues Terminal** aus, um eine neue Terminalinstanz zu öffnen.

1. Installieren Sie das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] für den globalen Einsatz auf Ihrem Computer.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Die Ausführung dieses Befehls kann einige Minuten dauern. Dieser Befehl gibt die Warnmeldung (*Tool 'cosmicworks' is already installed') aus, wenn Sie die neueste Version dieses Tools in der Vergangenheit bereits installiert haben.

1. Führen Sie „cosmicworks“ aus, um das Seeding für Ihr Azure Cosmos DB-Konto mit den folgenden Befehlszeilenoptionen durchzuführen:

    | **Option** | **Wert** |
    | ---: | :--- |
    | **--endpoint** | *Der Endpunktwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--key** | *Der Schlüsselwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** und Ihr Schlüssel **fDR2ci9QgkdkvERTQ==** lautet, dann lautet der Befehl: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Warten Sie, bis der Befehl **cosmicworks** das Konto mit einer Datenbank, einem Container und Elementen aufgefüllt hat.

1. Schließen Sie das integrierte Terminal.

## Durchlaufen der Ergebnisse einer SQL-Abfrage mithilfe des SDKs

Sie verwenden nun einen asynchronen Datenstrom, um eine einfach verständliche foreach-Schleife über paginierte Ergebnisse von Azure Cosmos DB zu erstellen. Im Hintergrund verwaltet das SDK den Feed-Iterator und stellt sicher, dass nachfolgende Anforderungen ordnungsgemäß aufgerufen werden.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **09-execute-query-sdk**.

1. Öffnen Sie die Codedatei **product.cs**.

1. Beobachten Sie die Klasse **Product** und die entsprechenden Eigenschaften. Insbesondere verwendet dieses Lab die Eigenschaften **id**, **name** und **price**.

1. Öffnen Sie im Bereich **Explorer** von **Visual Studio Code** die Codedatei **script.cs**.

1. Aktualisieren Sie die vorhandene Variable mit dem Namen **endpoint**, wobei ihr Wert auf den **Endpunkt** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** lautet, dann lautet die C#-Anweisung: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Aktualisieren Sie die vorhandene Variable namens **key**, wobei ihr Wert auf den **Schlüssel** (key) des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** lautet, dann lautet die C#-Anweisung **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Nun fügen wir am Ende der Datei **script.cs** zusätzlichen Code an, erstellen eine neue Variable namens **sql** vom Typ *string* mit dem Wert **SELECT * FROM products p**:

    ```
    string sql = "SELECT * FROM products p";
    ```

1. Erstellen Sie eine neue Variable vom Typ [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition], wobei Sie dem Konstruktor die Variable **sql** als Parameter übergeben:

    ```
    QueryDefinition query = new (sql);
    ```

1. Erstellen Sie eine neue **while**-Schleife, indem Sie die generische [GetItemQueryIterator-Methode][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] der [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]-Klasse aufrufen, wobei Sie die Variable **query** als Parameter übergeben. Dann werden die Ergebnisse durchlaufen:

    ```
    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
    }
    ```

1. In der **while**-Schleife wird das nächste Ergebnis asynchron gelesen. Verwenden Sie die integrierte statische Methode **Console.WriteLine**, um die Eigenschaften **id**, **name** und **price** der Variablen **product** zu formatieren und zu drucken:

    ```
    FeedResponse<Product> response = await feed.ReadNextAsync();
    foreach (Product product in response)
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(
        queryDefinition: query
    );

    while (feed.HasMoreResults)
    {
        FeedResponse<Product> response = await feed.ReadNextAsync();
        foreach (Product product in response)
        {
            Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
        }
    }
    ```

1. **Speichern** Sie die Datei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **09-execute-query-sdk**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Fügen Sie das Paket [Microsoft.Azure.Cosmos](nuget.org/packages/microsoft.azure.cosmos/3.22.1) aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```

1. Das Skript gibt nun jedes Produkt im Container aus.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet]: https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
