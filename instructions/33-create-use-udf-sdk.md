---
lab:
  title: Implementieren und anschließendes Verwenden benutzerdefinierter Funktionen mit dem SDK
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Implementieren und anschließendes Verwenden benutzerdefinierter Funktionen mit dem SDK

Das .NET SDK für Azure Cosmos DB for NoSQL kann verwendet werden, um serverseitige Programmierkonstrukte direkt in einem Container zu verwalten und aufzurufen. Beim Vorbereiten eines neuen Containers kann es sinnvoll sein, UDFs mithilfe des .NET SDK direkt in einem Container zu veröffentlichen, anstatt die Aufgaben manuell mithilfe des Daten-Explorers auszuführen.

In diesem Lab erstellen Sie eine neue UDF mit dem .NET SDK und verwenden dann den Daten-Explorer, um zu überprüfen, ob die UDF ordnungsgemäß funktioniert.

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

    1. Beachten Sie das Feld **PRIMÄRSCHLÜSSEL**. Sie verwenden diesen **Schlüsselwert** später in diesem Lab.

    1. Beachten Sie das Feld **PRIMARY CONNECTION STRING**. Sie verwenden diesen Wert der **Verbindungszeichenfolge** später in dieser Übung.

1. Öffnen Sie **Visual Studio Code**, ohne das Browserfenster zu schließen.

## Versorgen des Azure Cosmos DB for NoSQL-Kontos mit Daten

Das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] stellt Beispieldaten für ein beliebiges Azure Cosmos DB for NoSQL-Konto bereit. Das Tool ist ein Open-Source-Tool und über NuGet verfügbar. Sie installieren dieses Tool in der Azure Cloud Shell und verwenden es dann, um Seed-Werte für Ihre Datenbank zu generieren.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neues Terminal** aus, um eine neue Terminalinstanz zu öffnen.

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

1. Warten Sie, bis der Befehl **cosmicworks** das Konto mit einer Datenbank, einem Container und Elementen aufgefüllt hat.

1. Schließen Sie das integrierte Terminal.

## Erstellen einer benutzerdefinierten Funktion (User-Defined Function, UDF) über das .NET SDK

Die [Container-][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]-Klasse im .NET SDK enthält die [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]-Eigenschaft zum Ausführen von CRUD-Vorgängen über gespeicherte Prozeduren, UDFs und Trigger direkt aus dem SDK. Sie verwenden diese Eigenschaft, um eine neue UDF zu erstellen und diese dann an einen Azure Cosmos DB for NoSQL-Container zu übertragen. Die mit dem SDK erstellte UDF berechnet den Preis des Produkts mit der Steuer, sodass SQL-Abfragen für die Produkte mit dem Preis mit Steuer ausgeführt werden können.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **33-create-use-udf-sdk**.

1. Öffnen Sie die Codedatei **script.cs**.

1. Fügen Sie einen using-Block für den Namespace [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts] hinzu:

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

1. Aktualisieren Sie die vorhandene Variable mit dem Namen **endpoint**, wobei ihr Wert auf den **Endpunkt** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** lautet, dann lautet die C#-Anweisung: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Aktualisieren Sie die vorhandene Variable namens **key**, wobei ihr Wert auf den **Schlüssel** (key) des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** lautet, dann lautet die C#-Anweisung: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Erstellen Sie mithilfe des leeren Standardkonstruktors eine neue Variable vom Typ [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] mit dem Namen „props“:

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. Legen Sie die [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]-Eigenschaft der Variable **props** auf den Wert **tax** fest:

    ```
    props.Id = "tax";
    ```

1. Legen Sie die [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]-Eigenschaft der Variable **props** auf den Wert **props.Body = "function tax(i) { return i * 1.25; }";** fest:

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. Rufen Sie asynchron die [Skripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]-Methode der Variable **container** auf, wobei Sie die Variable **props** als Parameter übergeben, und speichern Sie das Ergebnis in einer Variable mit dem Namen **udf** vom Typ [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]:

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. Verwenden Sie die integrierte statische Methode **Console.WriteLine**, um die[Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]-Eigenschaft der UserDefinedFunctionResponse-Klasse mit einem Header mit dem Titel **Created UDF** zu drucken:

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **Speichern** Sie die Datei **script.cs**.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **33-create-use-udf-sdk**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```

1. Das Skript gibt jetzt den Namen der neu erstellten UDF aus:

    ```
    Created UDF [tax]
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

## Testen der UDF mit dem Daten-Explorer

Nachdem nun im Azure Cosmos DB-Container eine neue UDF erstellt wurde, verwenden Sie den Daten-Explorer, um zu überprüfen, ob die UDF erwartungsgemäß funktioniert.

1. Kehren Sie zu Ihrem Webbrowser zurück.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und beobachten Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Containerknoten **products** in der **NOSQL API** Navigationsstruktur aus, und wählen Sie dann **Neue SQL-Abfrage** aus.

1. Wählen Sie auf der Registerkarte „Abfrage“ die Option **Abfrage ausführen** aus, um eine Standardabfrage anzuzeigen, die alle Elemente ohne Filter auswählt.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle Dokumente mit zwei projizierten Preiswerten zurückgibt. Der erste Wert ist der unverarbeitete Preiswert aus dem Container, und der zweite Wert ist der Preiswert, der von der UDF berechnet wurde:

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Sehen Sie sich die Dokumente an, und vergleichen Sie die enthaltenen Felder **price** und **priceWithTax**.

    > &#128221; Das Feld **priceWithTax** sollte einen Wert aufweisen, der um 25 % größer als das Feld **price** ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
