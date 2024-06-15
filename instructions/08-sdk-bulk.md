---
lab:
  title: "Massenverschieben von Dokumenten mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# Massenverschieben von Dokumenten mit dem SDK für Azure Cosmos DB for NoSQL

Am einfachsten lernen Sie das Ausführen eines Massenvorgangs, indem Sie viele Dokumente an ein Azure Cosmos DB for NoSQL-Konto in der Cloud pushen. Mithilfe der Massenfeatures des SDK kann dies mit geringfügiger Hilfe vom Namespace [System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks] erreicht werden.

In diesem Lab verwenden Sie die Bibliothek [Bogus][nuget.org/packages/bogus/33.1.1] von NuGet, um fiktive Daten zu generieren und diese in ein Azure Cosmos DB-Konto zu platzieren.

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

1. Navigieren Sie weiterhin in der neu erstellten **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Wählen Sie im **Daten-Explorer** die Option **Neuer Container** aus, und erstellen Sie dann einen neuen Container mit den folgenden Einstellungen, wobei Sie alle verbleibenden Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Datenbank-ID** | *Neu erstellen* &vert; *`cosmicworks`* |
    | **Freigeben des Durchsatzes für mehrere Container** | *Nicht auswählen* |
    | **Container-ID** | *`products`* |
    | **Partitionsschlüssel** | *`/categoryId`* |
    | **Containerdurchsatz** | *Autoskalierung* &vert; *`4000`* |

1. Kehren Sie zu **Visual Studio Code** zurück.

1. Navigieren Sie im Bereich **Explorer** zum Ordner **08-sdk-bulk**.

1. Öffnen Sie die Codedatei **script.cs** im Ordner **08-sdk-bulk**.

    > &#128221; Die Bibliothek **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** wurde bereits aus NuGet importiert.

1. Suchen Sie die Variable **string** mit dem Namen **endpoint**. Legen Sie ihren Wert auf den **endpoint** des zuvor erstellten Azure Cosmos DB-Kontos fest.
  
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

1. Öffnen Sie das Kontextmenü für den Ordner **08-sdk-bulk**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **08-sdk-bulk** festgelegt ist.

1. Fügen Sie das Paket [Microsoft.Azure.Cosmos](nuget.org/packages/microsoft.azure.cosmos/3.22.1) aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Erstellen Sie das Projekt mithilfe des Befehls [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

1. Schließen Sie das integrierte Terminal.

## Masseneinfügen von 25.000 Dokumenten

Steigen Sie direkt ein, indem Sie versuchen, viele Dokumente einzufügen, um zu sehen, wie dies funktioniert. Bei unseren internen Tests kann dies ungefähr 1–2 Minuten dauern, wenn der virtuelle Labcomputer und das Azure Cosmos DB for NoSQL-Konto geografisch relativ nah beieinander liegen.

1. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Erstellen Sie eine neue Instanz von [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] mit dem Klassennamen **options**, wobei die **AllowBulkExecution**-Eigenschaft auf den Wert **true** festgelegt ist:

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. Erstellen Sie eine neue Instanz der **CosmosClient**-Klasse mit dem Namen **client**, und übergeben Sie Variablen **endpoint**, **key** und **options** als Konstruktorparameter:

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. Verwenden Sie die [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]-Methode der Variable **client**, um den vorhandenen Container mithilfe des Namens der Datenbank (*cosmicworks*) und des Namens des Containers (*products*) abzurufen:

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Verwenden Sie diesen speziellen Beispielcode, um **25.000** fiktive Produkte mithilfe der **Faker**-Klasse aus der Bibliothek „Bogus“ zu generieren, die aus NuGet importiert wurde.

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; Die Bibliothek [Bogus][nuget.org/packages/bogus/33.1.1] ist eine Open-Source-Bibliothek, die zum Entwerfen fiktiver Daten zum Testen von Benutzeroberflächenanwendungen verwendet wird. Sie eignet sich hervorragend zum Entwickeln von Anwendungen für den Massenimport/-export.

1. Erstellen Sie eine neue generische **List<>** vom Typ **Task** mit dem Namen **concurrentTasks**:

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. Erstellen Sie eine foreach-Schleife, die die zuvor in dieser Anwendung generierte Liste der Produkte durchläuft:

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. Erstellen Sie in der foreach-Schleife einen **Task** zum asynchronen Einfügen eines Produkts in Azure Cosmos DB for NoSQL. Achten Sie dabei darauf, den Partitionsschlüssel explizit anzugeben und den Task der Liste der Aufgaben mit dem Namen **concurrentTasks** hinzuzufügen:

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. Warten Sie nach der foreach-Schleife asynchron auf das Ergebnis von **Task.WhenAll** in der Variable **concurrentTasks**:

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. Verwenden Sie die integrierte statische **Console.WriteLine**-Methode, um die statische Meldung **Massenaufgaben abgeschlossen** in der Konsole zu drucken:

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **08-sdk-bulk**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Die Anwendung sollte unbeaufsichtigt im Hintergrund ausgeführt werden, und es sollte ungefähr ein bis zwei Minuten dauern, bis die Ausführung im Hintergrund abgeschlossen ist.

1. Schließen Sie das integrierte Terminal.

## Einsehen der Ergebnisse

Nachdem Sie nun 25.000 Elemente an Azure Cosmos DB gesendet haben, sehen Sie sich den Daten-Explorer an.

1. Kehren Sie zum Webbrowser zurück, und navigieren Sie zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie dann den Containerknoten **products** in der **NOSQL API**-Navigationsstruktur.

1. Erweitern Sie den Knoten **products**, und wählen Sie dann den Knoten **Items** aus. Beachten Sie die Liste der Elemente in Ihrem Container.

1. Wählen Sie den Containerknoten **products** in der **NoSQL-API**-Navigationsstruktur aus, und wählen Sie dann **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die die Anzahl aller mit dem Massenvorgang erstellten Dokumente zurückgibt:

    ```
    SELECT COUNT(1) FROM items
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Anzahl der Elemente in Ihrem Container.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
