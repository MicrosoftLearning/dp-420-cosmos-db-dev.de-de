---
lab:
  title: Erstellen und Aktualisieren von Dokumenten mit dem SDK für Azure Cosmos DB for NoSQL
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Erstellen und Aktualisieren von Dokumenten mit dem SDK für Azure Cosmos DB for NoSQL

Die Klasse [Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] enthält eine Reihe von Membermethoden zum Erstellen, Abrufen, Aktualisieren und Löschen von Elementen in einem Azure Cosmos DB for NoSQL-Container. Zusammen führen diese Methoden einige der häufigsten CRUD-Vorgänge in verschiedenen Elementen innerhalb von NoSQL-API-Containern aus.

In diesem Lab verwenden Sie das SDK, um alltägliche CRUD-Vorgänge für ein Element in einem Azure Cosmos DB for NoSQL-Container auszuführen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie die [Dokumentation „Erste Schritte“][code.visualstudio.com/docs/getstarted].

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
    | **Workloadtyp** | **Weiterbildung** |
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

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in diese Lab.

1. Wechseln Sie zurück zu **Visual Studio Code**.

## Herstellen einer Verbindung mit einem Azure Cosmos DB for NoSQL-Konto mithilfe des SDKs

Mit den Anmeldeinformationen des neu erstellten Kontos verbinden Sie sich mit den SDK-Klassen und erstellen eine neue Datenbank- und Containerinstanz. Anschließend überprüfen Sie mit Daten-Explorer, ob die Instanzen im Azure-Portal vorhanden sind.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **06-sdk-crud**.

1. Öffnen Sie das Kontextmenü für den Ordner **06-sdk-crud**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **06-sdk-crud** festgelegt ist.

1. Fügen Sie mithilfe des folgenden Befehls das Paket [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] aus NuGet hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Erstellen Sie das Projekt mithilfe des Befehls [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Schließen Sie das integrierte Terminal.

1. Öffnen Sie die Codedatei **script.cs** im Ordner **06-sdk-crud**.

    > &#128221; Die Bibliothek **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** wurde bereits aus NuGet importiert.

1. Suchen Sie die **Zeichenfolgenvariable** mit dem Namen **endpoint**. Legen Sie ihren Wert auf den **endpoint** des zuvor erstellten Azure Cosmos DB-Kontos fest.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** lautet, dann lautet die C#-Anweisung **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Suchen Sie die **string**-Variable mit dem Namen **key**. Legen Sie ihren Wert auf den **key** des zuvor erstellten Azure Cosmos DB-Kontos fest.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** lautet, dann lautet die C#-Anweisung: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Rufen Sie die CreateDatabaseIfNotExistsAsync-Methode der Variablen **client** asynchron auf, wobei Sie den Namen der neuen Datenbank (**cosmicworks**), die Sie erstellen möchten, übergeben und das Ergebnis in einer Variablen vom Typ **Database** speichern:

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Rufen Sie die **CreateContainerIfNotExistsAsync**-Methode der Variablen **database** asynchron auf, wobei Sie den Namen des neuen Containers (**products**), den Partitionsschlüsselpfad (**/categoryId**) und den Durchsatz (**400**), den Sie innerhalb der Datenbank **cosmicworks** erstellen möchten, übergeben und das Ergebnis in einer Variablen vom Typ **Container** speichern:
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **06-sdk-crud**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Schließen Sie das integrierte Terminal.

1. Wechseln Sie zu Ihrem Webbrowserfenster.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und beobachten Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

## Ausführen von Erstellungs- und Lesepunktvorgängen für Elemente mit dem SDK

Sie verwenden nun den Satz asynchroner Methoden in der Microsoft.Azure.Cosmos.Container-Klasse, um allgemeine Vorgänge für Elemente in einem NoSQL-API-Container auszuführen. Diese Vorgänge werden alle mit dem asynchronen Programmiermodell der Aufgabe in C# durchgeführt.

1. Kehren Sie zu **Visual Studio Code** zurück. Öffnen Sie die Codedatei **product.cs** im Ordner **06-sdk-crud**.

    > &#128221; Schließen Sie den Editor mit der Datei **script.cs** nicht.

1. Beachten Sie die **Product**-Klasse in dieser Codedatei. Diese Klasse stellt ein Produktelement dar, das in diesem Container gespeichert und bearbeitet wird.

1. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Erstellen Sie ein neues Objekt vom Typ **Product** namens **saddle** mit den folgenden Eigenschaften:

    | Eigenschaft | Wert |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **Tags** | *{ tan, new, crisp }* |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. Rufen Sie die generische Methode [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] der Variablen **container** asynchron auf, wobei Sie die Variable **saddle** als Methodenparameter übergeben und **Product** als generischen Typ verwenden:

    ```
    await container.CreateItemAsync<Product>(saddle);
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **06-sdk-crud**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Schließen Sie das integrierte Terminal.

1. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Löschen Sie die folgenden Codezeilen:

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. Erstellen Sie eine Zeichenfolgenvariable namens **id** mit dem Wert **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**:

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. Erstellen Sie eine Zeichenfolgenvariable mit dem Namen **categoryId** und dem Wert **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. Erstellen Sie eine Variable vom Typ [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] mit dem Namen **partitionKey** und übergeben Sie die Variable **categoryId** als Konstruktorparameter:

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Rufen Sie asynchron die generische Methode [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] der Variable **container** auf, und übergeben Sie die Variablen **id** und **partitionKey** als Methodenparameter, wobei **Product** als generischer Typ verwendet wird. Speichern Sie das Ergebnis in einer Variablen mit dem Namen **saddle** vom Typ **Product**:

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Rufen Sie die statische Methode **Console.WriteLine** auf, um das saddle-Objekt mithilfe einer formatierten Ausgabezeichenfolge auszugeben:

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **06-sdk-crud**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Beachten Sie die Ausgabe auf dem Terminal. Beachten Sie insbesondere den formatierten Ausgabetext mit der ID, dem Namen und dem Preis des Elements.

1. Schließen Sie das integrierte Terminal.

## Ausführen von Aktualisierungs- und Löschpunktvorgängen mit dem SDK

Beim Erlernen des SDK ist es nicht ungewöhnlich, ein Online-Azure Cosmos DB SDK-Konto oder den Emulator zu verwenden, um ein Element zu aktualisieren und zwischen dem Daten-Explorer und Ihrer IDE der Wahl hin- und herzuwechseln, während Sie einen Vorgang ausführen und überprüfen, ob Ihre Änderung übernommen wurde. Hier werden Sie genau dies tun, indem Sie ein Element mithilfe des SDK aktualisieren und löschen.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Wählen Sie das einzige Element im Container aus, und beobachten Sie dann die Werte der Eigenschaften **name** und **price** des Elements.

    | **Eigenschaft** | **Farbe** |
    | ---: | :--- |
    | **Name** | *Road Saddle* |
    | **Preis** | *45,99 USD* |

    > &#128221; Zu diesem Zeitpunkt sollten diese Werte nicht geändert worden sein, seit Sie das Element erstellt haben. Sie werden die Werte in dieser Übung ändern.

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Löschen Sie die folgenden Codezeilen:

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. Ändern Sie die Variable **saddle**, indem Sie den Wert der Eigenschaft „price“ auf **32,55** festlegen:

    ```
    saddle.price = 32.55d;
    ```

1. Ändern Sie die Variable **saddle** erneut, indem Sie den Wert der Eigenschaft **namet** auf **Road LL Sattel** festlegen:

    ```
    saddle.name = "Road LL Saddle";
    ```

1. Rufen Sie die generische Methode [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] der Variablen **container** asynchron auf, wobei Sie die Variable **saddle** als Methodenparameter übergeben und **Product** als generischen Typ verwenden:

    ```
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **06-sdk-crud**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Wählen Sie das einzige Element im Container aus, und beobachten Sie dann die Werte der Eigenschaften **name** und **price** des Elements.

    | **Eigenschaft** | **Farbe** |
    | ---: | :--- |
    | **Name** | *Road LL Saddle* |
    | **Preis** | *32,55 USD* |

    > &#128221; Jetzt sollten diese Werte geändert worden sein, seit Sie das Element beobachtet haben.

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Löschen Sie die folgenden Codezeilen:

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. Rufen Sie die generische Methode [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] der Variablen **container** asynchron auf, wobei Sie die Variablen **id** und **partitionkey** als Methodenparameter übergeben und **Product** als generischen Typ verwenden:

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **06-sdk-crud**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und erweitern Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Knoten **Items** aus. Beachten Sie, dass die Elementliste jetzt leer ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
