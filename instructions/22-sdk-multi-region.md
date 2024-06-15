---
lab:
  title: "Verbinden eines Kontos für Schreibvorgänge in mehreren Regionen mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# Verbinden eines Kontos für Schreibvorgänge in mehreren Regionen mit dem SDK für Azure Cosmos DB for NoSQL

Die **CosmosClientBuilder**-Klasse ist eine Fluent-Klasse, die entwickelt wurde, um den SDK-Client zu erstellen, um eine Verbindung mit Ihrem Container herzustellen und Vorgänge auszuführen. Mithilfe des Generators können Sie eine bevorzugte Anwendungsregion für Schreibvorgänge konfigurieren, wenn Ihr Azure Cosmos DB for NoSQL-Konto bereits für Schreibvorgänge in mehreren Regionen konfiguriert ist.

In diesem Lab konfigurieren Sie ein Azure Cosmos DB for NoSQL-Konto mit mehreren Regionen und aktivieren Schreibvorgänge in mehreren Regionen. Anschließend verwenden Sie das SDK, um Vorgänge für eine bestimmte Region auszuführen.

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

1. Fügen Sie im Bereich **Daten global replizieren** mindestens eine zusätzliche Region zum Konto hinzu.

1. Aktivieren Sie weiterhin im Bereich **Daten global replizieren** die Option **Schreibvorgänge in mehreren Regionen**, und **Speichern** Sie dann Ihre Änderungen.

1. Warten Sie, bis die Replikationsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

    > &#128221; Dieser Vorgang kann ungefähr 5–10 Minuten dauern.

1. Beachten Sie mindestens eine der zusätzlichen Regionen, die Sie erstellt haben. Sie verwenden diesen Regionswert später in dieser Übung.

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

1. Navigieren Sie im Ressourcenblatt zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die zum Herstellen einer Verbindung mit dem Konto im SDK erforderlich sind. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Kehren Sie zu **Visual Studio Code** zurück.

## Herstellen einer Verbindung mit einem Azure Cosmos DB for NoSQL-Konto mithilfe des SDKs

Mit den Anmeldeinformationen des neu erstellten Kontos verbinden Sie sich mit den SDK-Klassen und erstellen eine neue Datenbank- und Containerinstanz. Anschließend verwenden Sie den Daten-Explorer, um zu validieren, ob die Instanzen im Azure-Portal vorhanden sind.

1. Navigieren Sie im Bereich **Explorer** zum Ordner **22-sdk-multi-region**.

1. Öffnen Sie das Kontextmenü für den Ordner **22-sdk-multi-region**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **22-sdk-multi-region** festgelegt ist.

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

## Konfigurieren der Schreibregion für das SDK

Die Fluent-Methode **WithApplicationRegion** wird verwendet, um die bevorzugte Region für die nachfolgenden Vorgänge mithilfe der Klasse „builder“ zu konfigurieren.

1. Erstellen Sie eine neue Instanz der [CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]-Klasse mit dem Namen **builder**, die die Variablen **endpoint** und **key** als Konstruktorparameter übergibt:

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. Erstellen Sie eine neue Variable namens **region** vom Typ **string** mit dem Namen der zusätzlichen Region, die Sie zuvor im Lab erstellt haben. Wenn Sie beispielsweise Ihr Azure Cosmos DB for NoSQL-Konto in der Region **USA, Osten** erstellt und anschließend **Brasilien, Süden** hinzugefügt haben, dann würde Ihre Variable „string“ Folgendes enthalten:

    ```
    string region = "Brazil South"; 
    ```

    > &#128161; Alternativ können Sie auch die statische Klasse [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] verwenden, die integrierte Zeichenfolgeneigenschaften für verschiedene Azure-Regionen einbezieht.

1. Rufen Sie die [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]-Methode mit einem Parameter der **Region** und die Fluent-Methode [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] für die **builder**-Variable auf, die das Ergebnis in einer Variable mit dem Namen **client** vom Typ **CosmosClient** speichert, die in einer using-Anweisung eingeschlossen wird:

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. Verwenden Sie die Methode **GetContainer** der Variable **client**, um den vorhandenen Container mithilfe des Namens der Datenbank (*cosmicworks*) und des Namens des Containers (*products*) abzurufen:

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. Erstellen Sie zwei **string**-Variablen mit dem Namen **id** und **categoryId**, indem Sie einen neuen **Guid**-Wert generieren und dann das Ergebnis als Zeichenfolge speichern:

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. Erstellen Sie eine neue Variable mit dem Namen **item** vom Typ **Product**, die die Variable **id**, einen Zeichenfolgenwert **Polished Bike Frame** und die **categoryId**-Variable als Konstruktorparameter übergibt:

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. Rufen Sie asynchron die **CreateItemAsync\<\>**-Methode der **container**-Variable auf, die die Variable **item** als Parameter übergibt und das Ergebnis in einer Variablen mit dem Namen **response** speichert:

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. Rufen Sie die statische Methode **Console.WriteLine** auf, um den HTTP-Statuscode der Antwort und die Anforderungsgebühr (in Anforderungseinheiten) auszugeben:

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";    

    CosmosClientBuilder builder = new (endpoint, key);            
    
    string region = "West Europe";
    
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);
    
    var response = await container.CreateItemAsync<Product>(item);
    
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **22-sdk-multi-region**, und wählen Sie die Option **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Beachten Sie die Ausgabe vom Terminal. Der HTTP-Statuscode und die Anforderungsgebühr (in RUs) sollten in der Konsole ausgegeben werden.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
