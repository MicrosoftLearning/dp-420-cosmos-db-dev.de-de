---
lab:
  title: "Optimieren der Indizierungsrichtlinie eines Azure Cosmos\_DB for NoSQL-Containers für allgemeine Vorgänge"
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimieren der Indizierungsrichtlinie eines Azure Cosmos DB for NoSQL-Containers für allgemeine Vorgänge

Für schreibintensive Workloads oder Workloads mit großen JSON-Objekten kann es vorteilhaft sein, die Indizierungsrichtlinie auf die Indexeigenschaften zu optimieren, von denen Sie wissen, dass Sie sie in Ihren Abfragen verwenden werden.

In diesem Lab verwenden wir eine .NET-Testanwendung, um ein großes JSON-Element in einen Azure Cosmos DB for NoSQL-Container zunächst mithilfe der Standardindizierungsrichtlinie einzufügen und dann mithilfe einer Indexrichtlinie, die leicht abgestimmt wurde.

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
    | **Kapazitätsmodus** | *Serverlos* |

    > &#128221; Ihre Labumgebungen haben möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Daten-Explorer**.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Neu erstellen* &vert; *``cosmicworks``* |
    | **Container-ID** | *``products``* |
    | **Partitionsschlüssel** | *``/categoryId``* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

1. Navigieren Sie im Ressourcenblatt zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die zum Herstellen einer Verbindung mit dem Konto im SDK erforderlich sind. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Kehren Sie zu **Visual Studio Code** zurück.

## Ausführen der .NET-Testanwendung mithilfe der Standardindizierungsrichtlinie

Dieses Lab verfügt über eine vordefinierte .NET-Testanwendung, die ein großes JSON-Objekt verwendet und ein neues Element im Azure Cosmos DB für NoSQL-Container erstellt. Sobald der einzelne Schreibvorgang abgeschlossen ist, gibt die Anwendung den eindeutigen Bezeichner des Elements und die RU-Gebühr an das Konsolenfenster aus.

1. Navigieren Sie im Bereich **Explorer** zum Ordner **23-index-optimization**.

1. Öffnen Sie das Kontextmenü für den Ordner **23-index-optimization**, und wählen Sie dann die Option **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **23-index-optimization** festgelegt ist.

1. Erstellen Sie das Projekt mithilfe des Befehls [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build]:

    ```
    dotnet build
    ```

    > &#128221; Möglicherweise wird eine Compilerwarnung angezeigt, dass die Variablen **endpoint** und **keys** aktuell nicht verwendet werden. Sie können diese Warnung getrost ignorieren, da Sie diese Variablen in dieser Aufgabe verwenden werden.

1. Schließen Sie das integrierte Terminal.

1. Öffnen Sie die Codedatei **script.cs**.

1. Suchen Sie die **string**-Variable mit dem Namen **endpoint**. Legen Sie ihren Wert auf den **endpoint** des zuvor erstellten Azure Cosmos DB-Kontos fest.
  
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

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **23-index-optimization**, und wählen Sie dann die Option **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen und Ausführen des Projekts mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]**:

    ```
    dotnet run
    ```

1. Beachten Sie die Ausgabe vom Terminal. Der eindeutige Bezeichner des Elements und die Anforderungsgebühr des Vorgangs (in RUs) sollten in der Konsole ausgegeben werden.

1. Erstellen Sie das Projekt, und führen Sie es mindestens zwei weitere Male mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** aus. Beachten Sie die RU-Gebühr in der Konsolenausgabe:

    ```
    dotnet run
    ```

1. Lassen Sie das integrierte Terminal offen.

    > &#128221; Sie werden dieses Terminal später in dieser Übung wiederverwenden. Es ist wichtig, das Terminal offen zu lassen, damit Sie die ursprünglichen und aktualisierten RU-Gebühren vergleichen können.

## Aktualisieren der Indizierungsrichtlinie und erneutes Ausführen der .NET-Anwendung

In diesem Lab-Szenario wird davon ausgegangen, dass sich unsere zukünftigen Abfragen hauptsächlich auf die Eigenschaften „name“ und „categoryName“ konzentrieren. Um unser großes JSON-Element zu optimieren, schließen Sie alle anderen Felder aus dem Index aus, indem Sie eine Indizierungsrichtlinie erstellen, die beginnt, indem alle Pfade ausgeschlossen werden. Anschließend bezieht die Richtlinie selektiv bestimmte Pfade ein.

1. Kehren Sie zu Ihrem Webbrowser zurück.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie zunächst im **Daten-Explorer** den Datenbankknoten **cosmicworks** und dann den Containerknoten **products**, und wählen Sie anschließend die Option **Einstellungen** aus.

1. Navigieren Sie auf der Registerkarte **Einstellungen** zum Abschnitt **Indizierungsrichtlinie**.

1. Beachten Sie die Standardindizierungsrichtlinie:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. Ersetzen Sie die Indizierungsrichtlinie durch dieses geänderte JSON-Objekt, und **Speichern** Sie dann die Änderungen:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zum geöffneten Terminal zurück.

1. Erstellen Sie das Projekt, und führen Sie es mindestens zwei weitere Male mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** aus. Beachten Sie die neue RU-Gebühr in der Konsolenausgabe, die deutlich geringer sein sollte als die ursprüngliche Gebühr. Da Sie nicht alle Elementeigenschaften indizieren, sind die Kosten für Ihre Schreibvorgänge beim Aktualisieren des Indexes deutlich geringer. Dadurch können für Sie jedoch erhebliche Kosten entstehen, wenn Ihre Lesevorgänge Eigenschaften abfragen müssen, die nicht indiziert sind.  

    ```
    dotnet run
    ```

    > &#128221; Wenn keine aktualisierte RU-Gebühr angezeigt wird, müssen Sie möglicherweise einige Minuten warten.

1. Kehren Sie zu Ihrem Webbrowser zurück.

    > &#128221; Wenn die Seite **Indexrichtlinie** nicht geöffnet ist, wechseln Sie zum **Daten-Explorer**, erweitern Sie zunächst den Datenbankknoten **cosmicworks**und dann den Containerknoten **Produkte**, wählen Sie die Option **Einstellungen** aus, und navigieren Sie zum Abschnitt **Indizierungsrichtlinie**.

1. Ersetzen Sie die Indizierungsrichtlinie durch dieses geänderte JSON-Objekt, und **Speichern** Sie dann die Änderungen:

    ```
    {
      "indexingMode": "none"
    }
    ```

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Kehren Sie zu **Visual Studio Code** zurück. Kehren Sie zum geöffneten Terminal zurück.

1. Erstellen Sie das Projekt, und führen Sie es mindestens zwei weitere Male mithilfe des Befehls **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** aus. Beachten Sie die neue RU-Gebühr in der Konsolenausgabe, die viel geringer sein sollte als die ursprüngliche Gebühr.  Wie kann das sein? Da dieses Skript die RUs misst, wenn Sie das Element schreiben, fallen bei der Auswahl, keinen Index zu verwenden, keine Unkosten an, um diesen Index zu erhalten. Die Kehrseite hierfür ist, dass Ihre Lesevorgänge sehr kostspielig sind, obwohl Ihre Schreibvorgänge weniger RUs generieren.

    ```
    dotnet run
    ```

    > &#128221; Wenn keine aktualisierte RU-Gebühr angezeigt wird, müssen Sie möglicherweise einige Minuten warten.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
