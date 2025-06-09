---
lab:
  title: Herstellen einer Verbindung mit Azure Cosmos DB for NoSQL über das SDK
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# Herstellen einer Verbindung mit Azure Cosmos DB for NoSQL über das SDK

Das Azure SDK für .NET ist eine Suite von Bibliotheken, die eine konsistente Entwicklerschnittstelle für die Interaktion mit vielen Azure-Diensten bietet. Das Azure SDK für .NET basiert auf der .NET-Standard 2.0-Spezifikation und stellt dadurch sicher, dass es in Anwendungen für .NET Framework (4.6.1 oder höher), .NET Core (2.1 oder höher) und .NET (5 oder höher) verwendet werden kann.

In diesem Lab stellen Sie eine Verbindung mit einem Azure Cosmos DB for NoSQL-Konto mithilfe des Azure SDK für .NET her.

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
    | **Begrenzen des Gesamtdurchsatzes, der für dieses Konto bereitgestellt werden kann** | *Nicht aktiviert* |

    > &#128221; Ihre Labumgebungen haben möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die erforderlich sind, um vom SDK aus eine Verbindung mit dem Konto herzustellen. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Lassen Sie die Browserregisterkarte geöffnet, da Sie später dorthin zurückkehren.

## Anzeigen der Microsoft.Azure.Cosmos-Bibliothek auf NuGet

Die NuGet-Website enthält einen durchsuchbaren Index von Paketen, die zum Importieren in Ihre .NET-Anwendungen verfügbar sind. Um Vorabversionen von Paketen wie **Microsoft.Azure.Cosmos** zu importieren, können Sie über die NuGet-Website die entsprechenden Versionen und Befehle abrufen, um das Paket in Ihre Anwendungen zu importieren.

1. Navigieren Sie auf der neuen Browserregisterkarte zur NuGet-Website (``nuget.org``).

1. Überprüfen Sie die Beschreibung von NuGet, dem Paket-Manager für .NET, und dessen Funktionen.

1. Suchen Sie nach der Bibliothek **Microsoft.Azure.Cosmos** auf NuGet.org.

1. Wählen Sie die Registerkarte **.NET CLI** aus, um den Befehl zu sehen, der zum Importieren der neuesten Version dieser Bibliothek in ein .NET-Projekt erforderlich ist.

    > &#128161; Dieser Befehl muss nicht aufgezeichnet werden. Sie verwenden später in dieser Übung eine bestimmte Version der Bibliothek.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

## Importieren der Microsoft.Azure.Cosmos-Bibliothek in ein .NET-Projekt

Die .NET-CLI enthält einen Befehl zum Hinzufügen eines [Pakets][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] zum Importieren von Paketen aus einem vorkonfigurierten Paketfeed. Eine .NET-Installation verwendet NuGet als Standardpaketfeed.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **04-sdk-connect**.

1. Öffnen Sie das Kontextmenü für den Ordner **04-sdk-connect**, und wählen Sie dann **Im integrierten Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **04-sdk-connect** festgelegt ist.

1. Fügen Sie mithilfe des folgenden Befehls das Paket [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos] aus NuGet hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Schließen Sie das integrierte Terminal.

## Verwenden der Microsoft.Azure.Cosmos-Bibliothek

Nachdem die Azure Cosmos DB-Bibliothek aus dem Azure SDK für .NET importiert wurde, können Sie ihre Klassen sofort im Namespace [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] verwenden, um eine Verbindung mit einem Azure Cosmos DB for NoSQL-Konto herzustellen. Die [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]-Klasse ist die Kernklasse, die verwendet wird, um die erste Verbindung mit einem Azure Cosmos DB for NoSQL-Konto herzustellen.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **04-sdk-connect**.

1. Öffnen Sie die leere Codedatei **script.cs**.

1. Fügen Sie Blöcke für die integrierten Namespaces **System** und **System.Linq** hinzu:

    ```
    using System;
    using System.Linq;
    ```

1. Fügen Sie einen using-Block für den Namespace [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] hinzu:

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. Fügen Sie eine Variable vom Typ **string** mit dem Namen **endpoint** hinzu, wobei der Wert auf den **Endpunkt** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** lautet, dann lautet die C#-Anweisung: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**.

1. Fügen Sie eine Variable vom Typ **string** mit dem Namen **key** hinzu, wobei der Wert auf den **Schlüssel** des zuvor erstellten Azure Cosmos DB-Kontos festgelegt wird.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; Wenn Ihr Schlüssel beispielsweise **fDR2ci9QgkdkvERTQ==** lautet, dann lautet die C#-Anweisung: **string key = "fDR2ci9QgkdkvERTQ==";**.

1. Fügen Sie eine neue Variable mit dem Namen **client** vom Typ [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] mithilfe der Variablen **endpoint** und **key** im Konstruktor hinzu:
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. Fügen Sie eine neue Variable mit dem Namen **account** vom Typ [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] mithilfe des asynchronen Ergebnisses des Aufrufs der [ReadAccountAsync-][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]-Methode der Variable **client** hinzu:

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. Verwenden Sie die integrierte statische **Console.WriteLine**-Methode, um die[Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]-Eigenschaft der AccountProperties-Klasse mit einem Header mit dem Titel **Account Name** zu drucken:

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. Verwenden Sie die integrierte statische **Console.WriteLine-**-Methode, um die [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]-Eigenschaft der AccountProperties-Klasse abzufragen, und drucken Sie dann die [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]-Eigenschaft des ersten Ergebnisses mit einem Header mit dem Titel **Primary Region**:

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. Nachdem Sie fertig sind, sollte die Codedatei jetzt folgenden Code enthalten:
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **Speichern** Sie die Codedatei **script.cs**.

## Testen des Skripts

Nachdem der .NET-Code zum Herstellen einer Verbindung mit dem Azure Cosmos DB for NoSQL-Konto abgeschlossen ist, können Sie das Skript testen. Dieses Skript druckt den Namen des Kontos und den Namen der ersten beschreibbaren Region. Als Sie das Konto erstellt haben, haben Sie einen Speicherort angegeben, und Sie sollten davon ausgehen, dass derselbe Speicherortwert als Ergebnis dieses Skripts gedruckt wird.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **04-sdk-connect**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```

1. Dieses Skript gibt nun den Namen des Kontos und die erste beschreibbare Region aus. Wenn Sie beispielsweise das Konto **dp420** genannt haben und die erste beschreibbare Region **USA, Westen 2** war, gibt das Skript Folgendes aus:

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
