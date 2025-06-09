---
lab:
  title: "Überprüfen der Standardindizierungsrichtlinie für einen Azure Cosmos\_DB for NoSQL-Container mithilfe des Portals"
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB for NoSQL
---

# Überprüfen der Standardindizierungsrichtlinie für einen Azure Cosmos DB for NoSQL-Container mithilfe des Portals

Jeder Container in Azure Cosmos DB verfügt über eine Indizierungsrichtlinie, die dem Dienst vorgibt, wie Elemente innerhalb des Containers indiziert werden. Standardmäßig indiziert diese Indizierungsrichtlinie jede Eigenschaft eines jeden Elements. Die Standardindizierungsrichtlinie erleichtert die ersten Schritte mit Azure Cosmos DB, da Sie sich zu Beginn eines Projekts keine Gedanken über Indizierung, Leistung und Verwaltung machen müssen.

In diesem Lab werden Sie die Standardindizierungsrichtlinie für einige Container mit dem Daten-Explorer beobachten und bearbeiten.

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

    1. Beachten Sie das Feld **PRIMARY CONNECTION STRING**. Sie verwenden diesen Wert der **Verbindungszeichenfolge** später in dieser Übung.

## Versorgen des Azure Cosmos DB for NoSQL-Kontos mit Daten

Das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] stellt Beispieldaten für ein beliebiges Azure Cosmos DB for NoSQL-Konto bereit. Das Tool ist ein Open-Source-Tool und über NuGet verfügbar. Sie installieren dieses Tool in der Azure Cloud Shell und verwenden es dann, um Seed-Werte für Ihre Datenbank zu generieren.

1. Starten Sie **Visual Studio Code**.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neuer Terminal** aus, um eine neue Terminal-Instanz zu öffnen.

    > &#128221; Wenn Sie noch nicht mit der Visual Studio Code-Schnittstelle vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

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

## Anzeigen und Bearbeiten der Standardindizierungsrichtlinie

Wenn ein Container mittels Code, Portal oder Tool erstellt wird, wird die Indizierungsrichtlinie auf einen intelligenten Standardwert festgelegt, sofern Sie keinen anderen Wert angeben. Sie werden diese Standardindizierungsrichtlinie beobachten und Änderungen an der Richtlinie vornehmen.

1. Kehren Sie zu Ihrem Webbrowser zurück.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und beobachten Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API**.

1. Wählen Sie den Containerknoten **products** in der **NOSQL API** Navigationsstruktur aus, und wählen Sie dann **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle Dokumente zurückgibt, bei denen der **Name** **HL Headset** entspricht:

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse der Abfrage.

1. Wählen Sie auf der Registerkarte **Abfrage** die Option **Abfragestatistiken** aus.

1. Beobachten Sie den Wert des Felds **Anforderungsgebühr** im Abschnitt **Abfragestatistik**.

    > &#128221; Da derzeit alle Pfade indiziert sind, sollte diese Abfrage relativ effizient sein.

1. Wählen Sie im Containerknoten **products** der Navigationsstruktur **NOSQL API** die Option **Einstellungen** aus.

1. Beobachten Sie die Standardindizierungsrichtlinie im Abschnitt **Indizierungsrichtlinie**:

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

    > &#128221; Mit dieser Standardrichtlinie werden alle möglichen Pfade mit Ausnahme von **_etag** indiziert.

1. Ersetzen Sie im Editor den Inhalt der Indizierungsrichtlinie, um nur den Pfad **/price** zu indizieren:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. Klicken Sie auf **Speichern**, um die Änderungen zu übernehmen.

1. Wählen Sie **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle Dokumente zurückgibt, bei denen der **Name** **HL Headset** entspricht:

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse der Abfrage.

1. Wählen Sie auf der Registerkarte **Abfrage** die Option **Abfragestatistiken** aus.

1. Beobachten Sie den Wert des Felds **Anforderungsgebühr** im Abschnitt **Abfragestatistik**.

    > &#128221; Da die Eigenschaft **Name** nicht indiziert wird, hat sich die Anforderungsgebühr erhöht.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle Dokumente zurückgibt, in denen der Wert von **price** größer als **3.000** USD ist:

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse der Abfrage.

1. Wählen Sie auf der Registerkarte **Abfrage** die Option **Abfragestatistiken** aus.

1. Beobachten Sie den Wert des Felds **Anforderungsgebühr** im Abschnitt **Abfragestatistik**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
