---
lab:
  title: Verbinden verschiedener Regionen mit dem SDK für Azure Cosmos DB for NoSQL
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Verbinden verschiedener Regionen mit dem SDK für Azure Cosmos DB for NoSQL

Wenn Sie Georedundanz für ein Azure Cosmos DB for NoSQL-Konto aktivieren, können Sie das SDK verwenden, um Daten aus Regionen in jeder von Ihnen konfigurierten Reihenfolge zu lesen. Diese Technik ist von Vorteil, wenn Sie Ihre Leseanforderungen über alle verfügbaren Leseregionen hinweg verteilen.

In diesem Lab konfigurieren Sie die CosmosClient-Klasse, um eine Verbindung mit Leseregionen in einer Fallbackreihenfolge herzustellen, die Sie manuell konfigurieren.

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
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Account Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Kapazitätsmodus** | *Bereitgestellter Durchsatz* |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Daten global replizieren**.

1. Fügen Sie im Bereich **Daten global replizieren** dem Konto zwei zusätzliche Leseregionen hinzu, und **Speichern** Sie dann Ihre Änderungen.

1. Warten Sie, bis die Replikationsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

    > &#128221; Dieser Vorgang kann ungefähr 5–10 Minuten dauern.

1. Notieren Sie sich die Namen der **Schreibregion** (primär) und der beiden **Leseregionen**. Sie verwenden diese Regionsnamen später in dieser Übung.

    > &#128221; Wenn Ihre primäre Region beispielsweise **Europa, Norden** ist und Ihre beiden sekundären Leseregionen **USA, Osten 2** und **Südafrika, Norden** sind; Sie werden alle drei Namen so erfassen, wie sie sind.

1. Navigieren Sie im Ressourcenblatt zum Bereich **Daten-Explorer**.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Neu erstellen* &vert; *``cosmicworks``* |
    | **Freigeben des Durchsatzes für mehrere Container** | *Nicht auswählen* |
    | **Container-ID** | *``products``* |
    | **Partitionsschlüssel** | *``/categoryId``* |
    | **Containerdurchsatz** | *Manuell* &vert; *400* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

1. Erweitern Sie zunächst im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks** und dann den Containerknoten **products**, und wählen Sie anschließend die Option **Elemente** aus.

1. Wählen Sie noch im Bereich **Daten-Explorer** die Option **Neues Element** aus der Befehlsleiste aus. Ersetzen Sie im Editor das JSON-Platzhalterelement durch den folgenden Inhalt:

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Wählen Sie in der Befehlsleiste die Option **Speichern** aus, um das JSON-Element hinzuzufügen:

1. Sehen Sie sich auf der Registerkarte **Elemente** das neue Element im Bereich **Elemente** an.

1. Navigieren Sie im Ressourcenblatt zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die zum Herstellen einer Verbindung mit dem Konto im SDK erforderlich sind. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Kehren Sie zu **Visual Studio Code** zurück.

## Herstellen einer Verbindung mit einem Azure Cosmos DB for NoSQL-Konto mithilfe des SDKs

Mithilfe der Anmeldeinformationen des neu erstellten Kontos stellen Sie eine Verbindung mit den SDK-Klassen her und greifen Sie auf die Datenbank- und Containerinstanz aus einer anderen Region zu.

1. Navigieren Sie im Bereich **Explorer** zum Ordner **20-sdk-regions**.

1. Öffnen Sie das Kontextmenü für den Ordner **20-sdk-regions**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **20-sdk-regions** festgelegt ist.

1. Erstellen Sie das Projekt mithilfe des Befehls [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; Möglicherweise wird eine Compilerwarnung angezeigt, dass die Variablen **endpoint** und **key** aktuell nicht verwendet werden. Sie können diese Warnung getrost ignorieren, da Sie diese Variablen in dieser Aufgabe verwenden werden.

1. Schließen Sie das integrierte Terminal.

1. Öffnen Sie die Codedatei **script.cs** im Ordner **20-sdk-regions**.

    > &#128221; Die Bibliothek **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** wurde bereits aus NuGet vorab importiert.

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

## Konfigurieren des .NET SDK mit einer bevorzugten Regionsliste

Die **CosmosClientOptions**-Klasse enthält eine Eigenschaft zum Konfigurieren der Liste der Regionen, mit denen Sie eine Verbindung mit dem SDK herstellen möchten. Die Liste wird nach Failoverpriorität sortiert und versucht, eine Verbindung mit den einzelnen Regionen in der von Ihnen konfigurierten Reihenfolge herzustellen.

1. Erstellen Sie eine neue Variable vom generischen Typ **List\<string\>**, die eine Liste der Regionen enthält, die Sie mit Ihrem Konto konfiguriert haben, beginnend mit der dritten Region und endend mit der ersten (primären) Region. Wenn Sie beispielsweise Ihr Azure Cosmos DB for NoSQL-Konto in der Region **USA, Westen** erstellt und dann **Südafrika, Norden** und schließlich **Asien, Osten**hinzugefügt haben, dann würde Ihre list-Variable Folgendes enthalten:

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; Alternativ können Sie auch die statische Klasse [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] verwenden, die integrierte Zeichenfolgeneigenschaften für verschiedene Azure-Regionen einbezieht.

1. Erstellen Sie eine neue Instanz von **CosmosClientOptions** mit dem Klassennamen **options**, wobei die [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]-Eigenschaft auf die **regions**-Variable festgelegt ist:

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    ```

1. Erstellen Sie eine neue Instanz der **CosmosClient**-Klasse mit dem Namen **client**, die die Variablen **endpoint**, **key** und **options** als Konstruktorparameter übergibt:

    ```
    using CosmosClient client = new (endpoint, key, options); 
    ```

1. Verwenden Sie die Methode **GetContainer** der Variable **client**, um den vorhandenen Container mithilfe des Namens der Datenbank (*cosmicworks*) und des Namens des Containers (*products*) abzurufen:

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Verwenden Sie die [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]-Methode der Variable **container**, um ein bestimmtes Element vom Server abzurufen und das Ergebnis in einer Variablen mit dem Namen **response** des Nullable-Typs [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] zu speichern:

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. Rufen Sie die statische Methode **Console.WriteLine** auf, um den aktuellen Elementbezeichner und die JSON-Diagnosedaten auszugeben:

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **20-sdk-regions**, und wählen Sie dann die Option **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Beachten Sie die Ausgabe vom Terminal. Der Name des Containers und die JSON-Diagnosedaten sollten in der Konsolenausgabe ausgegeben werden.

1. Überprüfen Sie die JSON-Diagnosedaten. Suchen Sie nach einer Eigenschaft namens **HttpResponseStats** und einer untergeordneten Eigenschaft mit dem Namen **RequestUri**. Der Wert dieser Eigenschaft sollte ein URI sein, der den Namen und die Region enthält, die Sie zuvor in diesem Lab konfiguriert haben.

    > &#128221; Wenn Ihr Kontoname beispielsweise **dp420** lautet und die erste Region, die Sie konfiguriert haben, **Asien, Osten** ist, dann lautet der Wert der JSON-Eigenschaft **dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products**.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
