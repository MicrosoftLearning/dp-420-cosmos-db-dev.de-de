---
lab:
  title: Konfigurieren des SDK für Azure Cosmos DB for NoSQL für Offlineentwicklung
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Konfigurieren des SDK für Azure Cosmos DB for NoSQL für Offlineentwicklung

Der Azure Cosmos DB-Emulator ist ein lokales Tool, das zu Entwicklungs- und Testzwecken den Azure Cosmos DB-Dienst emuliert. Der Emulator unterstützt NoSQL und kann anstelle des Clouddiensts beim Entwickeln von Code mithilfe des Azure SDK für .NET verwendet werden.

In diesem Lab stellen Sie vom Azure SDK für .NET aus eine Verbindung mit dem Azure Cosmos DB-Emulator her.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie die [Dokumentation „Erste Schritte“][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALTTASTE+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Ausführen des Azure Cosmos DB-Emulators

In Ihrer Umgebung sollte der Emulator bereits vorinstalliert sein. Andernfalls lesen Sie die [Installationsanleitung][docs.microsoft.com/azure/cosmos-db/local-emulator], um den Azure Cosmos DB-Emulator zu installieren. Nachdem der Emulator gestartet wurde, können Sie die Verbindungszeichenfolge abrufen und verwenden, um unter Verwendung des Azure SDK für .NET oder eines anderen SDK Ihrer Wahl eine Verbindung mit dem Emulator herzustellen.

1. Starten Sie den **Azure Cosmos DB-Emulator**.

    > &#128221; Möglicherweise werden Sie aufgefordert, Administratorzugriff zum Starten des Emulators zu gewähren. In der Labumgebung verfügt das **Administratorkonto** über dasselbe Kennwort wie das Konto **Student**.

    > &#128161; Der Azure Cosmos DB-Emulator ist sowohl an die Windows-Taskleiste als auch an das Startmenü angeheftet. ***Wenn sich der Emulator nicht über die angehefteten Symbole starten lässt, versuchen Sie, ihn zu öffnen, indem Sie auf die Datei*** **C:\Programme\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** ***doppelklicken***. Beachten Sie, dass es 20 bis 30 Sekunden dauert, bis der Emulator gestartet wird.

1. Warten Sie, bis der Emulator Ihren Standardbrowser automatisch öffnet und zur Startseite **localhost:8081/_explorer/index.html** navigiert.

1. Navigieren Sie auf der Startseite des **Azure Cosmos DB-Emulators** zum Bereich **Schnellstart**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die erforderlich sind, um vom SDK aus eine Verbindung mit dem Konto herzustellen. Speziell:

    > &#128221; Beachten Sie das Feld **Primäre Verbindungszeichenfolge**. Sie verwenden diesen Wert der **Verbindungszeichenfolge** später in dieser Übung.

1. Navigieren Sie zum Bereich **Explorer**.

1. Beachten Sie im **Daten-Explorer**, dass in der Navigationsstruktur **NoSQL-API** keine Knoten vorhanden sind.

1. Lassen Sie diese Registerkarte geöffnet, und wechseln Sie zu **Visual Studio Code**.

## Herstellen einer Verbindung mit dem Emulator über das SDK

Die Bibliothek **Microsoft.Azure.Cosmos** wurde bereits in dem .NET-Skript vorinstalliert, das Sie in dieser Übung verwenden werden. Überdies wurde bereits ein Teil des Codebausteins geschrieben, um Ihnen Zeit zu sparen. Sie müssen den Wert der Verbindungszeichenfolge im Textbaustein aktualisieren und ein paar Codezeilen schreiben, um das Skript abzuschließen.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **05-sdk-offline**.

1. Öffnen Sie die Codedatei **script.cs** im Ordner **05-sdk-offline**.

1. Aktualisieren Sie die vorhandene Variable namens **connectionString** mit dem Wert der **Verbindungszeichenfolge** des Azure Cosmos DB-Emulators.
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; Der URI für den Emulator ist in der Regel ***localhost:[port]***, wobei SSL verwendet und der Standardport auf **8081** festgelegt wird.

    > &#128221; *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* ist der Standardschlüssel für alle Installationen des Emulators. Dieser Schlüssel kann mithilfe von Befehlszeilenoptionen geändert werden.

1. Rufen Sie die [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]-Methode der Variable **client** asynchron auf, wobei Sie den Namen der neuen Datenbank (**cosmicworks**), die Sie im Emulator erstellen möchten, übergeben und das Ergebnis in einer Variablen vom Typ [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] speichern:

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. Verwenden Sie die integrierte statische Methode **Console.WriteLine**, um die[ID][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] Eigenschaft der Database-Klasse mit einem Header mit dem Titel **Ne Database** zu drucken:

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **05-sdk-offline**, und wählen Sie dann die Option **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **05-sdk-offline** festgelegt ist.

1. Fügen Sie mithilfe des folgenden Befehls das Paket [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] aus NuGet hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```

1. Schließen Sie das integrierte Terminal.

## Anzeigen der Änderungen im Emulator

Nachdem Sie nun eine neue Datenbank im Azure Cosmos DB-Emulator erstellt haben, verwenden Sie den **Daten-Explorer** online, um die neue NoSQL-API-Datenbank im Emulator zu beobachten.

1. Wechseln Sie zurück zu Ihrem Browser.

1. Navigieren Sie auf der Startseite des **Azure Cosmos DB-Emulators** zum Bereich **Explorer**.

1. Aktualisieren Sie im **Daten-Explorer** die **NOSQL-API**, um den neuen Datenbankknoten **cosmicworks** innerhalb der Navigationsstruktur zu beobachten.

1. Wechseln Sie zurück zu **Visual Studio Code**.

## Erstellen und Anzeigen eines neuen Containers

Die Erstellung eines neuen Containers ähnelt dem Muster, das für die Erstellung einer neuen Datenbank verwendet wird. Der hier beschriebene Code ist unabhängig davon relevant, ob Sie Ressourcen in der Cloud oder im Emulator erstellen, Sie müssen lediglich die Verbindungszeichenfolge ändern. Sie erweitern die Skriptdatei, um einen neuen Container zusammen mit der Datenbank zu erstellen.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **05-sdk-offline**.

1. Öffnen Sie die Codedatei **script.cs** erneut im Ordner **05-sdk-offline**.

1. Rufen Sie die Methode [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] der Variablen **database** asynchron auf und übergeben Sie den Namen des neuen Containers (**products**), den Partitionsschlüsselpfad (**/category/name**) und den Durchsatz (**400**), den Sie in der Datenbank **cosmicworks** erstellen möchten, und speichern Sie das Ergebnis in einer Variablen vom Typ [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]:

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name", 400);
    ```

1. Verwenden Sie die integrierte statische Methode **Console.WriteLine**, um die[ID][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] Eigenschaft der Container-Klasse mit einem Header mit dem Titel **New Container** zu drucken:

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **05-sdk-offline**, und wählen Sie dann die Option **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

1. Wechseln Sie zu Ihrem Browser.

1. Navigieren Sie auf der Startseite des **Azure Cosmos DB-Emulators** zum Bereich **Explorer**.

1. Aktualisieren Sie im **Daten-Explorer**die **SQL-API**, um den neuen Containerknoten **products** im Datenbankknoten **cosmicworks** zu beobachten.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

## Beenden des Azure Cosmos DB-Emulators

Es ist wichtig, den Emulator zu beenden, wenn Sie ihn nicht mehr verwenden, da er Systemressourcen in Ihrer Umgebung verbrauchen kann. Sie verwenden das Taskleistensymbol, um den Emulator und alle laufenden Instanzen zu beenden.

 Navigieren Sie zum Emulatorsymbol im Windows-Infobereich, öffnen Sie das Kontextmenü, und wählen Sie dann **Beenden** aus, um den Emulator zu beenden.

> &#128221; Es kann unter Umständen eine Minute dauern, bis alle Instanzen des Emulators beendet worden sind.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1