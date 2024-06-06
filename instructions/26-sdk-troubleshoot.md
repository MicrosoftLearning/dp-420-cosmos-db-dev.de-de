---
lab:
  title: "Problembehandlung für eine Anwendung mit dem SDK für Azure Cosmos\_DB for NoSQL"
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Problembehandlung für eine Anwendung mit dem SDK für Azure Cosmos DB for NoSQL

In Azure Cosmos DB gibt es verschiedenste Antwortcodes, die uns helfen, Probleme zu beheben, die mit unseren verschiedenen Betriebstypen auftreten könnten. Wichtig ist vor allem, die richtige Fehlerbehandlung beim Erstellen von Apps für Azure Cosmos DB zu programmieren.

In diesem Lab erstellen wir ein menügesteuertes Programm, mit dem wir eines von zwei Dokumenten einfügen oder löschen können. Der Hauptzweck dieses Labs ist die Einführung in die Verwendung einiger der am häufigsten verwendeten Antwortcodes und deren Anwendung im Fehlerbehandlungscode unserer App.  Obwohl wir die Fehlerbehandlung für mehrere Antwortcodes programmieren, lösen wir nur zwei verschiedene Arten von Bedingungen aus.  Darüber hinaus wird die Fehlerbehandlung je nach Antwortcode keine komplexen Aktionen ausführen. Sie zeigt entweder eine Meldung auf dem Bildschirm an oder wartet 10 Sekunden und wiederholt den Vorgang ein weiteres Mal. 

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie noch nicht mit der Visual Studio Code-Benutzeroberfläche vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALT+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Beim ersten Bereitstellen eines Azure Cosmos DB-Kontos wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Nachdem die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen. Verwenden Sie den Endpunkt und den Schlüssel, um programmgesteuert eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen. Verwenden Sie den Endpunkt und den Schlüssel für die Verbindungszeichenfolgen des Azure SDK for .NET oder eines anderen SDK.

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

    1. Beachten Sie das Feld **PRIMÄRSCHLÜSSEL**. Sie verwenden diesen **Schlüsselwert** später in diesem Lab.

1. Minimieren Sie das Browserfenster, schließen Sie es jedoch nicht. Wir kehren ein paar Minuten nach dem Starten einer Hintergrundworkload in den nächsten Schritten zum Azure-Portal zurück.


## Importieren der Microsoft.Azure.Cosmos-Bibliothek in ein .NET-Skript

Die .NET-CLI enthält einen Befehl zum Hinzufügen eines [Pakets][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] zum Importieren von Paketen aus einem vorkonfigurierten Paketfeed. Eine .NET-Installation verwendet NuGet als Standardpaketfeed.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **26-sdk-troubleshoot**.

1. Öffnen Sie das Kontextmenü für den Ordner **26-sdk-troubleshoot**, und wählen Sie dann die Option **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **26-sdk-troubleshoot** festgelegt ist.

1. Fügen Sie mithilfe des folgenden Befehls das Paket [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] aus NuGet hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

## Führen Sie ein Skript aus, um menügesteuerte Optionen zum Einfügen und Löschen von Dokumenten zu erstellen.

Bevor wir unsere Anwendung ausführen können, müssen wir sie mit unserem Azure Cosmos DB-Konto verknüpfen. 

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **26-sdk-troubleshoot**.

1. Öffnen Sie die **Program.cs**-Codedatei.

1. Aktualisieren Sie die vorhandene Variable mit dem Namen **endpoint**, wobei ihr Wert auf den **Endpunkt** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** ist, dann lautet die C#-Anweisung: **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Aktualisieren Sie die vorhandene Variable mit dem Namen **key**, wobei ihr Wert auf den **Schlüssel** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** ist, dann lautet die C#-Anweisung: **private static readonly string key = "fDR2ci9QgkdkvERTQ==";**.

1. Speichern Sie die Datei .

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```
    > &#128221; Das ist ein sehr einfaches Programm.  Es wird ein Menü mit fünf Optionen angezeigt, wie unten dargestellt. Zwei Optionen sind zum Einfügen eines vordefinierten Dokuments, zwei zum Löschen eines vordefinierten Dokuments und eine Option zum Beenden des Programms.

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## Es ist an der Zeit, Dokumente einzufügen und zu löschen.

1. Drücken Sie **1** und dann die **EINGABETASTE**, um das erste Dokument einzufügen. Das Programm fügt das erste Dokument ein und gibt die folgende Meldung zurück.

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. Drücken Sie erneut **1** und dann die **EINGABETASTE**, um das erste Dokument einzufügen. Dieses Mal stürzt das Programm mit einer Ausnahme ab. Im Fehlerstapel finden wir den Grund für den Programmfehler. Wie wir aus der aus dem Fehlerstapel extrahierten Nachricht erkennen können, wird ein Ausnahmefehler „Konflikt (409)“ angezeigt.

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. Da wir ein Dokument einfügen, müssen wir die Liste der üblichen [Statuscodes für die Dokumenterstellung][/rest/api/cosmos-db/create-a-document#status-codes] überprüfen, die beim Erstellen eines Dokuments zurückgegeben werden. Die Beschreibung dieses Codes lautet: *Die für das neue Dokument angegebene ID wird bereits von einem vorhandenen Dokument verwendet*. Das ist logisch, da wir gerade erst die Menüoption zum Erstellen desselben Dokuments ausgeführt haben.

1. Wenn wir uns näher mit dem Stapel befassen, können wir sehen, dass diese Ausnahme von Zeile 100 aufgerufen wurde und diese wiederum von Zeile 64 aufgerufen wurde.

    ```
    at Program.CreateDocument1(Container Customer) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 100   
   at Program.CompleteTaskOnCosmosDB(String consoleinputcharacter, Container container) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 64
    ```

1. Durch Überprüfung der Zeile 100 stellt sich heraus, dass der Fehler wie erwartet durch den *CreateItemAsync*-Vorgang verursacht wurde. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. Darüber hinaus ist bei der Überprüfung der Zeilen 100 bis 103 ersichtlich, dass dieser Code keine Fehlerbehandlung aufweist. Das müssen wir beheben. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. Wir müssen entscheiden, was unser Fehlerbehandlungscode tun soll. Durch Überprüfen der [Statuscodes für die Dokumenterstellung][/rest/api/cosmos-db/create-a-document#status-codes] können wir für jeden möglichen Statuscode für diesen Vorgang Fehlerbehandlungscode erstellen.  In dieser Übung werden wir aus dieser Liste nur die Statuscodes 403 und 409 berücksichtigen.  Alle anderen zurückgegebenen Statuscodes zeigen lediglich die Systemfehlermeldung an.

    > &#128221; Beachten Sie, dass in diesem Lab keine 403-Ausnahmen generiert werden, obwohl wir einen Task zur Fehlerbehandlung für 403-Ausnahmen programmieren.

1. Fügen wir die Fehlerbehandlung für die Funktion **CompleteTaskOnCosmosDB** hinzu. Suchen Sie die **while**-Schleife in der Funktion **Main** in Zeile **45**, und umschließen Sie die Aufrufe von **CompleteTaskOnCosmosDB** mit einem Fehlerbehandlungscode. Wir ersetzen die **CompleteTaskOnCosmosDB**-Anweisung in Zeile **47** durch den folgenden Code.  Der erste, was wir an diesem neuen Code bemerken, ist, dass wir mit **catch** eine Ausnahme vom Typ **CosmosException**-Klasse erfassen.  Diese Klasse enthält die Eigenschaft **StatusCode**, die den Statuscode für den Anforderungsabschluss aus dem Azure Cosmos DB-Dienst zurückgibt. Die **StatusCode**-Eigenschaft ist vom Typ **System.Net.HttpStatusCode**. Wir können diesen Wert verwenden und mit den Feldnamen aus dem [HTTP-Statuscode][dotnet/api/system.net.httpstatuscode] für .NET vergleichen.  

    ```C#
        try
        {
            await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. Speichern Sie die Datei. Da das Menüprogramm abgestürzt ist, müssen wir es erneut ausführen. Führen Sie also den Befehl aus:

    ```
    dotnet run
    ```
 
1. Drücken Sie erneut **1** und dann die **EINGABETASTE**, um das erste Dokument einzufügen. Dieses Mal stürzt das Programm nicht ab, sondern gibt eine besser verständliche Meldung darüber aus, was passiert ist.

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    ```

1. Dieser Code hat die Fehlerbehandlung für *403*- und *409*-Ausnahmen hinzugefügt. Nun fügen wir nun Code für einige gängige Kommunikationsarten von Ausnahmen hinzu. Es gibt drei allgemeine Kommunikationsarten von Ausnahmen: *429*, *503* und *408* oder zu viele Anforderungen, Dienst nicht verfügbar und Anforderungstimeout. Um Zeile *66* sollte nun eine **default**-Anweisung vorhanden sein. Fügen Sie also den folgenden Code direkt nach der vorherigen **break;**-Anweisung und direkt vor der **default**-Anweisung hinzu.  Der Code überprüft, ob eine dieser Kommunikationsausnahmen gefunden wird, und wenn ja, wartet er 10 Sekunden und versucht dann, das Dokument ein weiteres Mal einzufügen.  Fügen Sie Folgendes zum Code hinzu:

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; Beachten Sie, dass in diesem Lab kein Fehler mit dieser Ausnahme generiert wird, wenn eine 429-, 503- oder 408-Ausnahme auftritt.

1. Unsere **Main**-Funktion sollte nun etwa wie folgt aussehen:

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                 try
                 {
                     await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                 }
                 catch (CosmosException e)
                 {
                     switch (e.StatusCode.ToString())
                     {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                     }
                }
                

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. Beachten Sie, dass die **CreateDocument2**-Funktion auch durch die oben genannten Änderungen behoben wird.

1. Schließlich muss für die Funktionen **DeleteDocument1** und **DeleteDocument2** auch der folgende Code durch den richtigen Fehlerbehandlungscode ersetzt werden, ähnlich wie bei der Funktion **CreateDocument1**. Der einzige Unterschied zwischen diesen Funktionen besteht neben der Verwendung von **DeleteItemAsync** anstelle von **CreateItemAsync** darin, dass [Löschstatuscodes][/rest/api/cosmos-db/delete-a-document] anders sind als die Einfügestatuscodes. Bei Löschvorgängen ist nur der Statuscode **404** relevant, der bedeutet, dass das Dokument nicht gefunden wurde. Fügen wir der Fehlerbehandlung des Funktionsaufrufs **CompleteTaskOnCosmosDB** einen weiteren Fall hinzu.  In der **Main**-Funktion muss der folgende Code oberhalb **default**-Falls hinzugefügt werden:

    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1. Speichern Sie die Datei .

1. Nachdem Sie alle Funktionen angepasst haben, testen Sie mehrmals alle Menüoptionen, um sicherzustellen, dass Ihre App eine Meldung zurückgibt, wenn eine Ausnahme auftritt, und nicht abstürzt.  Wenn Ihre App abstürzt, beheben Sie die Fehler, und führen Sie den Befehl erneut aus:

    ```
    dotnet run
    ```


1. Sobald Sie fertig sind, sollten Ihre `Main`-Codes ungefähr wie folgt aussehen.

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## Zusammenfassung

Die ordnungsgemäße Fehlerbehandlung muss zum gesamten Code hinzugefügt werden. Während die Fehlerbehandlung in diesem Code einfach ist, sollten Sie die Grundlagen zu den Azure Cosmos DB-Ausnahmekomponenten nun kennen, mit denen Sie robuste Fehlerbehandlungslösungen in Ihrem Code erstellen können.


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

