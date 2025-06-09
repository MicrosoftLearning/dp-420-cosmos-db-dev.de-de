---
lab:
  title: "Verarbeiten von Änderungsfeedereignissen mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Verarbeiten von Änderungsfeedereignissen mit dem SDK für Azure Cosmos DB for NoSQL

Der Azure Cosmos DB for NoSQL-Änderungsfeed ist der Schlüssel zum Erstellen zusätzlicher Anwendungen, die von Ereignissen von der Plattform gesteuert werden. Das .NET SDK für die Azure Cosmos DB for NoSQL enthält eine Reihe von Klassen, mit denen Sie Ihre Anwendungen erstellen können, die in den Änderungsfeed integriert sind, und auf Benachrichtigungen über Vorgänge in Ihren Containern lauschen.

In diesem Lab verwenden Sie die Funktionalität des Änderungsfeedprozessors im .NET SDK, um eine Anwendung zu erstellen, die benachrichtigt wird, wenn ein Vorgang zum Erstellen oder Aktualisieren eines Elements im angegebenen Container ausgeführt wird.

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

    1. Beachten Sie das Feld **PRIMARY CONNECTION STRING**. Sie verwenden diesen Wert der **Verbindungszeichenfolge** später in dieser Übung.

1. Wählen Sie im Ressourcenmenü **Data Explorer** aus.

1. Erweitern Sie im Bereich **Daten-Explorer** die Option **Neuer Container**, und wählen Sie dann **Neue Datenbank** aus.

1. Geben Sie im Popup **Neue Datenbank** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *``cosmicworks``* |
    | **Durchsatz bereitstellen** | enabled |
    | **Datenbank-Durchsatz** | **Manuell** |
    | **Erforderliche Datenbank-RU/s** | ``1000`` |

1. Beachten Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Vorhandene verwenden* &vert; *cosmicworks* |
    | **Container-ID** | *``products``* |
    | **Partitionsschlüssel** | *``/category/name``* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** erneut die Option **Neuer Container** aus.

1. Geben Sie im Popup **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Vorhandene verwenden* &vert; *cosmicworks* |
    | **Container-ID** | *``productslease``* |
    | **Partitionsschlüssel** | *``/partitionKey``* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **productslease** in der Hierarchie.

1. Kehren Sie zu **Visual Studio Code** zurück.

## Implementieren des Änderungsfeedprozessors im .NET SDK

Die Klasse **Microsoft.Azure.Cosmos.Container** wird mit einer Reihe von Methoden ausgeliefert, um den Änderungsfeedprozessor fließend zu erstellen. Zunächst benötigen Sie einen Verweis auf Ihren überwachten Container, Ihren Leasecontainer und einen Delegat in C\# (um jeden Batch von Änderungen zu verarbeiten).

1. Navigieren Sie im Bereich **Explorer** zum Ordner **13-change-feed**.

1. Öffnen Sie die Codedatei **product.cs**.

1. Beobachten Sie die Klasse **Product** und die entsprechenden Eigenschaften. Insbesondere verwendet diese Übung die Eigenschaften **id** und **name**.

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

    > &#128221; Wenn Ihr Schlüssel beispielsweise lautet: **fDR2ci9QgkdkvERTQ==**, lautet die C#-Anweisung: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Verwenden Sie die Methode **GetContainer** der Variable **client**, um den vorhandenen Container mithilfe des Namens der Datenbank (*cosmicworks*) und den Namen des Containers (*products*) abzurufen und das Ergebnis in einer Variablen mit dem Namen **sourceContainer** vom Typ **Container** zu speichern:

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. Verwenden Sie die Methode **GetContainer** der Variable **client**, um den vorhandenen Container mithilfe des Namens der Datenbank (*cosmicworks*) und den Namen des Containers (*productslease*) abzurufen und das Ergebnis in einer Variablen mit dem Namen **leaseContainer** vom Typ **Container** zu speichern:

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. Erstellen Sie eine neue Delegatvariable mit dem Namen **handleChanges** vom Typ [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] verwenden Sie eine leere asynchrone anonyme Funktion mit zwei Eingabeparametern:

    1. Ein Parameter mit dem Namen **changes** vom Typ **IReadOnlyCollection\<Product\>**.
    
    1. Ein Parameter mit dem Namen **cancellationToken** vom Typ **CancellationToken**.

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. Verwenden Sie innerhalb der anonymen Funktion die integrierte statische Methode **Console.WriteLine**, um die unformatierte Zeichenfolge **START\tHandling batch of changes...** auszugeben:

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. Erstellen Sie weiterhin innerhalb der anonymen Funktion eine foreach-Schleife, die die Variable **changes** durchläuft, indem Sie die Variable **product** verwenden, um eine Instanz vom Typ **Product** darzustellen:

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. Verwenden Sie in der foreach-Schleife der anonymen Funktion die integrierte asynchrone statische Methode **Console.WriteLineAsync**, um die Eigenschaften **id** und **name** der Variable **product** auszugeben:

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. Erstellen Sie außerhalb der foreach-Schleife und der anonymen Funktion eine neue Variable mit dem Namen **builder**, die das Ergebnis des Aufrufens von [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] für die Variable **sourceContainer** mit den folgenden Parametern speichert:

    | **Parameter** | **Wert** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. Rufen Sie die Methode [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] mit einem Parameter von **consoleApp**, der Methode [WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] mit einem Parameter von **leaseContainer** auf, und die Methode [build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] fließend auf der Variable **builder**, die das Ergebnis in einer Variablen mit dem Namen **processor** der Art [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor] speichert:

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. Rufen Sie asynchron die [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync] der Variable **processor** auf:

    ```
    await processor.StartAsync();
    ```

1. Verwenden Sie die integrierten statischen Methoden **Console.WriteLine** und **Console.ReadKey**, um die Ausgabe in der Konsole wiederzugeben und die Anwendung auf einen Tastendruck warten zu lassen:

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. Rufen Sie asynchron die [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] der Variable **processor** auf:

    ```
    await processor.StopAsync();
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **Speichern** Sie die Datei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **13-change-feed**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]:

    ```
    dotnet run
    ```

1. Lassen Sie sowohl **Visual Studio Code** und das Terminal geöffnet.

    > &#128221; Sie werden ein anderes Tool verwenden, um Elemente in Ihrem Azure Cosmos DB for NoSQL-Container zu erzeugen. Sobald Sie die Elemente erstellt haben, kehren Sie zu diesem Terminal zurück, um sich die Ausgabe anzusehen. Schließen Sie das Terminal nicht vorzeitig.

## Legen Sie Ihr Azure Cosmos DB for NoSQL-Konto mit Beispieldaten an.

Sie werden ein Befehlszeilen-Dienstprogramm verwenden, das eine **cosmicworks**-Datenbank und einen **products**-Container erstellt. Das Tool erstellt dann eine Reihe von Elementen, die Sie mit dem Änderungsfeed-Prozessor in Ihrem Terminalfenster beobachten können.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Geteilter Terminal** aus, um daneben ein neues Terminal mit Ihrer vorhandenen Instanz zu öffnen.

1. Installieren Sie das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] für den globalen Einsatz auf Ihrem Computer.

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; Die Ausführung dieses Befehls kann einige Minuten dauern. Dieser Befehl gibt die Warnmeldung (*Tool 'cosmicworks' is already installed') aus, wenn Sie die neueste Version dieses Tools in der Vergangenheit bereits installiert haben.

1. Führen Sie „cosmicworks“ aus, um das Seeding für Ihr Azure Cosmos DB-Konto mit den folgenden Befehlszeilenoptionen durchzuführen:

    | **Option** | **Wert** |
    | ---: | :--- |
    | **-c** | *Die Verbindungszeichenfolge, die Sie zuvor in diesem Lab überprüft haben* |
    | **Anzahl der Mitarbeitenden** | *Der Befehl „cosmicworks“ füllt Ihre Datenbank mit Containern für Mitarbeitende und Produkte mit jeweils 1000 und 200 Elementen, sofern nicht anders angegeben.* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** und Ihr Schlüssel **fDR2ci9QgkdkvERTQ==** lautet, dann lautet der Befehl: ``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. Warten Sie, bis der Befehl **cosmicworks** das Konto mit einer Datenbank, einem Container und Gegenständen aufgefüllt hat.

1. Beobachten Sie die Terminalausgabe ihrer .NET-Anwendung. Das Terminal gibt eine **Erkannten Vorgang**-Nachricht für jede Änderung aus, die mit dem Änderungsfeed an sie gesendet wurde.

1. Schließen Sie beide integrierte Terminals.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
