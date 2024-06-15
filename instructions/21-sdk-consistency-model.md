---
lab:
  title: "Konfigurieren von Konsistenzmodellen im Portal und im SDK von Azure Cosmos\_DB for NoSQL"
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Konfigurieren von Konsistenzmodellen im Portal und im SDK von Azure Cosmos DB for NoSQL

Die Standardkonsistenzstufe für neue Azure Cosmos DB for NoSQL-Konten ist die Sitzungskonsistenz. Diese Standardeinstellung kann für alle zukünftigen Anforderungen geändert werden. Auf einer individuellen Anforderungsebene können Sie einen Schritt weitergehen und das Konsistenzniveau für diese spezifische Anforderung lockern.

In diesem Lab werden wir die Standardkonsistenzstufe für ein Azure Cosmos DB for NoSQL-Konto konfigurieren und dann eine Konsistenzstufe für einen einzelnen Vorgang mithilfe des SDK konfigurieren.

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
    | **Globale Verteilung** &vert; **Georedundanz** | *Aktivieren* |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Daten global replizieren**.

1. Fügen Sie im Bereich **Daten global replizieren** dem Konto zwei zusätzliche Lesebereiche hinzu, und **speichern** Sie dann Ihre Änderungen.

    > &#128221; In einigen Schritten werden Sie aufgefordert, die Konsistenzstufe auf Stark zu ändern. Beachten Sie jedoch, dass die starke Konsistenz für Konten mit Regionen, die sich über mehr als 5000 Meilen (8000 Kilometer) erstrecken, aufgrund der hohen Schreiblatenz standardmäßig blockiert ist. Achten Sie darauf, dass Sie Regionen auswählen, die näher beieinander liegen.  Wenn Sie diese Funktion in einer Produktivumgebung aktivieren möchten, wenden Sie sich bitte an den Support.

1. Warten Sie, bis die Replikationsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

    > &#128221; Dieser Vorgang kann ca. 5–10 Minuten dauern. Navigieren Sie dann zum Bereich **Standardkonsistenz**.

1. Navigieren Sie auf dem Ressourcenblatt zum Bereich **Standardkonsistenz**.

1. Wählen Sie im Bereich **Standardkonsistenz** die Option **Stark** aus, und **Speichern** Sie Ihre Änderungen.

1. Warten Sie, bis die Änderung der Standardkonsistenzstufe dauerhaft ist, bevor Sie mit dieser Aufgabe fortfahren.

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

Mit den Anmeldeinformationen des neu erstellten Kontos verbinden Sie sich mit den SDK-Klassen und erstellen eine neue Datenbank- und Containerinstanz. Anschließend verwenden Sie den Daten-Explorer, um zu überprüfen, ob die Instanzen im Azure-Portal vorhanden sind.

1. Navigieren Sie im Bereich **Explorer** zum Ordner **21-sdk-consistency-model**.

1. Öffnen Sie das Kontextmenü für den Ordner **21-sdk-consistency-model**, und wählen Sie **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **21-sdk-consistency-model** festgelegt ist.

1. Erstellen Sie das Projekt mithilfe des Befehls [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; Möglicherweise wird eine Compilerwarnung angezeigt, dass die Variablen **endpoint** und **key** aktuell nicht verwendet werden. Sie können diese Warnung getrost ignorieren, da Sie diese Variablen in dieser Aufgabe verwenden werden.

1. Schließen Sie das integrierte Terminal.

1. Öffnen Sie die Codedatei **product.cs**.

1. Sehen Sie sich den **Produktdatensatz** und die entsprechenden Eigenschaften an. Insbesondere verwendet dieses Lab die Eigenschaften **id**, **name** und **categoryId**.

1. Öffnen Sie im Bereich **Explorer** von **Visual Studio Code** die Codedatei **script.cs**.

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

## Konfigurieren der Konsistenzstufe für einen Punktvorgang

Die Klasse **ItemRequestOptions** enthält Konfigurationseigenschaften für die einzelnen Anforderungen. Mit dieser Klasse lockern Sie die Konsistenzstufe von der aktuellen Standardeinstellung „Stark" auf die spätere Konsistenz.

1. Erstellen Sie eine Zeichenfolgenvariable mit dem Namen **id** und dem Wert **7d9273d9-5d91-404c-bb2d-126abb6e4833**:

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. Erstellen Sie eine Zeichenfolgenvariable mit dem Namen **categoryId** und dem Wert **78d204a2-7d64-4f4a-ac29-9bfc437ae959**:

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. Erstellen Sie eine Variable vom Typ **PartitionKey** mit dem Namen **partitionKey** und übergeben Sie die Variable **categoryId** als Konstruktorparameter:

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. Rufen Sie asynchron die generische Methode **ReadItemAsync\<\>** der Variable **container** auf und übergeben Sie die Variablen **id** und **partitionKey** als Methodenparameter, wobei **Product** als generischer Typ verwendet wird. Speichern Sie das Ergebnis in einer Variablen mit dem Namen **response** vom Typ **ItemResponse\<Product\>**:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. Rufen Sie die statische Methode **Console.WriteLine** auf, um die Anforderungsgebühr mithilfe einer formatierten Ausgabezeichenfolge auszugeben:

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **21-sdk-consistency-model** und wählen Sie dann **Im integrierten Terminal öffnen**, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und starten Sie das Projekt mit dem Befehl **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Sehen Sie sich die Ausgabe im Terminal an. Die Anforderungsgebühr (in RUs) sollten in der Konsole ausgegeben werden.

    > &#128221; Die aktuelle Anforderungsgebühr sollte **2 RUs** sein. Das liegt an der starken Konsistenz, die ein Lesen von mindestens zwei Replikaten erfordert, um sicherzustellen, dass sie über den aktuellsten Schreibvorgang verfügen.

1. Schließen Sie das integrierte Terminal.

1. Kehren Sie zur Editor-Registerkarte für die Codedatei **script.cs** zurück.

1. Löschen Sie die folgenden Codezeilen:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Erstellen Sie eine neue Variable mit dem Namen **options** vom Typ [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] und legen Sie die Eigenschaft [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] auf den Enumerationswert [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel] fest:

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. Rufen Sie asynchron die generische Methode **ReadItemAsync\<\>** der Variable **container** auf und übergeben Sie die Variablen **id**, **partitionKey** und **options** als Methodenparameter, wobei **Product** als generischer Typ verwendet wird. Speichern Sie das Ergebnis in einer Variablen mit dem Namen **response** vom Typ **ItemResponse\<Product\>**:

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. Rufen Sie die statische Methode **Console.WriteLine** auf, um die Anforderungsgebühr mithilfe einer formatierten Ausgabezeichenfolge auszugeben:

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **21-sdk-consistency-model** und wählen Sie dann **Im integrierten Terminal öffnen**, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und starten Sie das Projekt mit dem Befehl **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Sehen Sie sich die Ausgabe im Terminal an. Die Anforderungsgebühr (in RUs) sollten in der Konsole ausgegeben werden.

    > &#128221; Die aktuelle Anforderungsgebühr sollte **1 RUs** sein. Dies ist darauf zurückzuführen, dass für die endgültige Konsistenz nur ein einziges Replikat gelesen werden muss.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
