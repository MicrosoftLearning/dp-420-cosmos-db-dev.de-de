---
lab:
  title: "Batchverarbeitung mehrerer Punktvorgänge mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Batchverarbeitung mehrerer Punktvorgänge mit dem SDK für Azure Cosmos DB for NoSQL

Die Klassen [TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] und [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] bilden gemeinsam den Schlüssel zum Verfassen und Dekompilieren von Vorgängen in einem einzigen logischen Schritt. Mithilfe dieser Klassen können Sie Ihren Code schreiben, um mehrere Vorgänge auszuführen, und dann ermitteln, ob sie serverseitig erfolgreich abgeschlossen wurden.

In dieser Übung verwenden Sie das SDK, um zwei Vorgänge auszuführen, bei denen Sie versuchen, zwei Elemente als einzelne logische Einheit zu erstellen.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie die [Dokumentation „Erste Schritte“][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALTTASTE+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos und Konfigurieren des SDK-Projekts

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

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Kehren Sie zu **Visual Studio Code** zurück.

1. Navigieren Sie im Bereich **Explorer** zum Ordner **07-sdk-batch**.

1. Öffnen Sie die Codedatei **script.cs** im Ordner **07-sdk-batch**.

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

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** lautet, dann lautet die C#-Anweisung **string key = "fDR2ci9QgkdkvERTQ==";**.

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie das Kontextmenü für den Ordner **07-sdk-batch**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **07-sdk-batch** festgelegt ist.

1. Fügen Sie mithilfe des folgenden Befehls das Paket [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] aus NuGet hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Erstellen Sie das Projekt mithilfe des Befehls [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Schließen Sie das integrierte Terminal.

## Erstellen eines transaktionalen Batches

Als Erstes erstellen Sie einen einfachen Transaktionsbatch, der zwei fiktive Produkte erstellt. Dieser Batch fügt einen getragenen Sattel und eine rostige Griffleiste in den Container mit demselben Kategoriebezeichner „used accessories“ ein. Beide Elemente weisen denselben logischen Partitionsschlüssel auf, sodass sichergestellt ist, dass der Batchvorgang erfolgreich abläuft.

1. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Erstellen Sie eine Variable vom Typ **Product** mit dem Namen **saddle** und dem eindeutigen Bezeichner **0120**, dem Namen **Worn Saddle** sowie der Kategorie-ID **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Erstellen Sie eine Variable vom Typ **Product** mit dem Namen **handlebar** und dem eindeutigen Bezeichner **012A**, dem Namen **Rusty Handlebar** sowie der Kategorie-ID **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Erstellen Sie eine Variable vom Typ **PartitionKey** mit dem Namen **partitionKey**, und übergeben Sie den Wert **9603ca6c-9e28-4a02-9194-51cdb7fea816** als Konstruktorparameter:

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Rufen Sie die [CreateTransactionalBatch-][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]-Methode der Variable **container** auf, und übergeben Sie dabei die Variable **partitionkey** als Methodenparameter. Verwenden Sie die Fluent-Syntax zum Aufrufen der generischen [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]-Methoden, und übergeben Sie dabei die Variablen **saddle** und **handlebar** als Elemente, die in einzelnen Vorgängen erstellt werden sollen. Speichern Sie das Ergebnis in einer Variable mit dem Namen **batch** vom Typ **TransactionalBatch**:

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. Rufen Sie in einer using-Anweisung die **ExecuteAsync**-Methode der Variable **Batch** auf, und speichern Sie das Ergebnis in einer Variable vom Typ **TransactionalBatchResponse** mit dem Namen **response**:

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Rufen Sie die statische **Console.WriteLine-**-Methode auf, um den Wert der **StatusCode**-Eigenschaft der Variable **response** auszugeben:

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
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

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **07-sdk-batch**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Beachten Sie die Ausgabe auf dem Terminal. Der Statuscode sollte **OK** lauten.

1. Schließen Sie das integrierte Terminal.

## Erstellen eines fehlerhaften transaktionalen Batch

Nun erstellen Sie einen Transaktionsbatch, der absichtlich einen Fehler erzeugt. Dieser Batch versucht, zwei Elemente mit unterschiedlichen logischen Partitionsschlüsseln einzufügen. Es werden ein flackerndes Stroboskoplicht in der Kategorie „used accessories“ und einen neuen Helm in der Kategorie „pristine accessories“ erstellt. Laut Definition sollte dies eine ungültige Anforderung sein und beim Ausführen dieser Transaktion einen Fehler zurückgeben.

1. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Löschen Sie die folgenden Codezeilen:

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. Erstellen Sie eine Variable vom Typ **Product** mit dem Namen **light** und dem eindeutigen Bezeichner **012B**, dem Namen **Flickering Strobe Light** sowie der Kategorie-ID **9603ca6c-9e28-4a02-9194-51cdb7fea816**:

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Erstellen Sie eine Variable vom Typ **Product** mit dem Namen **Helm** und dem eindeutigen Bezeichner **012C**, dem Namen **New Helmet** und dem Kategoriebezeichner **0feee2e4-687a-4d69-b64e-be36afc33e74**:

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. Erstellen Sie eine Variable vom Typ **PartitionKey** mit dem Namen **partitionKey**, und übergeben Sie den Wert **9603ca6c-9e28-4a02-9194-51cdb7fea816** als Konstruktorparameter:

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. Rufen Sie die **CreateTransactionalBatch-**-Methode der Variable **container** auf, und übergeben Sie dabei die Variable **partitionkey** als Methodenparameter. Verwenden Sie die Fluent-Syntax zum Aufrufen der generischen **CreateItem<>**-Methoden, und übergeben Sie dabei die Variablen **light** und **helmet** als Elemente, die in einzelnen Vorgängen erstellt werden sollen. Speichern Sie das Ergebnis in einer Variable mit dem Namen **batch** vom Typ **TransactionalBatch**:

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. Rufen Sie in einer using-Anweisung die **ExecuteAsync**-Methode der Variable **Batch** auf, und speichern Sie das Ergebnis in einer Variable vom Typ **TransactionalBatchResponse** mit dem Namen **response**:

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. Rufen Sie die statische **Console.WriteLine-**-Methode auf, um den Wert der **StatusCode**-Eigenschaft der Variable **response** auszugeben:

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
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

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **07-sdk-batch**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Beachten Sie die Ausgabe auf dem Terminal. Der Statuscode sollte entweder **Ungültige Anforderung** oder **Konflikt** lauten. Dies ist aufgetreten, da nicht alle Elemente innerhalb der Transaktion denselben Partitionsschlüsselwert wie der transaktionale Batch verwendeten.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
