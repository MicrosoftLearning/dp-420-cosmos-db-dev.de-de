---
lab:
  title: "Optimieren der Indizierungsrichtlinie eines Azure Cosmos\_DB for NoSQL-Containers für eine bestimmte Abfrage"
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# Optimieren der Indizierungsrichtlinie eines Azure Cosmos DB for NoSQL-Containers für eine Abfrage

Wenn Sie ein Azure Cosmos DB for NoSQL-Konto planen, kann das Wissen um die beliebtesten Abfragen uns helfen, die Indizierungsrichtlinie so zu optimieren, dass Abfragen so leistungsfähig wie möglich sind.

In dieser Übung verwenden wir den Daten-Explorer, um SQL-Abfragen mit der Standardindizierungsrichtlinie und einer Indizierungsrichtlinie zu testen, die einen zusammengesetzten Index enthält.

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

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in diese Lab.

1. Öffnen Sie **Visual Studio Code**.

## Legen Sie Ihr Azure Cosmos DB for NoSQL-Konto mit Beispieldaten an.

Sie werden ein Befehlszeilen-Dienstprogramm verwenden, das eine **cosmicworks**-Datenbank und einen **products**-Container erstellt. Das Tool erstellt dann eine Reihe von Elementen, die Sie mit dem Änderungsfeed-Prozessor in Ihrem Terminalfenster beobachten können.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neues Terminal** aus, um ein neues Terminal zu öffnen.

1. Installieren Sie das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] für den globalen Einsatz auf Ihrem Computer.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Die Ausführung dieses Befehls kann einige Minuten dauern. Dieser Befehl gibt die Warnmeldung (*Tool 'cosmicworks' is already installed') aus, wenn Sie die neueste Version dieses Tools in der Vergangenheit bereits installiert haben.

1. Führen Sie „cosmicworks“ aus, um das Seeding für Ihr Azure Cosmos DB-Konto mit den folgenden Befehlszeilenoptionen durchzuführen:

    | **Option** | **Wert** |
    | ---: | :--- |
    | **--endpoint** | *Der Endpunktwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--key** | *Der Schlüsselwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise **https&shy;://dp420.documents.azure.com:443/** und Ihr Schlüssel **fDR2ci9QgkdkvERTQ==** lautet, dann lautet der Befehl: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Warten Sie, bis der Befehl **cosmicworks** das Konto mit einer Datenbank, einem Container und Elementen aufgefüllt hat.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**, und kehren Sie zu Ihrem Browser zurück.

## Ausführen von SQL-Abfragen und Messen der Gebühr für Anforderungseinheiten

Bevor Sie die Indizierungsrichtlinie ändern, führen Sie zunächst einige SQL-Beispielabfragen aus, um eine Grundgebühr für Anforderungseinheiten abzurufen, die in RU ausgedrückt wird.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und wählen Sie zunächst den Containerknoten **products** und dann **Neue SQL-Abfrage** aus.

1. Wählen Sie **Abfrage ausführen** aus, um die Standardabfrage auszuführen:

    ```
    SELECT * FROM c
    ```

1. Beachten Sie die Ergebnisse der Abfrage. Wählen Sie **Abfragestatistik-** aus, um die Gebühr für Anforderungseinheiten in RU anzuzeigen.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle drei Werte von allen Dokumenten zurückgibt:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p    
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse und Statistiken der Abfrage. Die Gebühr für Anforderungseinheiten entspricht fast der ersten Abfrage.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die drei Werte von allen Dokumenten zurückgibt, sortiert nach **categoryName**:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse und Statistiken der Abfrage. Die Gebühr für Anforderungseinheiten wurde aufgrund der **ORDER BY**-Klausel erhöht.

## Erstellen eines zusammengesetzten Indexes in der Indizierungsrichtlinie

Jetzt müssen Sie einen zusammengesetzten Index erstellen, wenn Sie Ihre Elemente mithilfe mehrerer Eigenschaften sortieren. In dieser Aufgabe erstellen Sie einen zusammengesetzten Index zum Sortieren von Elementen nach dem categoryName und dann nach dem eigentlichen Namen.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und wählen Sie zunächst den Containerknoten **products** und dann **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die die Ergebnisse zuerst in absteigender Reihenfolge nach **categoryName** sortiert und dann in aufsteigender Reihenfolge nach **price**:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Bei der Abfrage sollte der Fehler auftreten: **Die ORDER BY-Abfrage verfügt nicht über einen entsprechenden zusammengesetzten Index für die Bereitstellung**.

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
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und wählen Sie zunächst den Containerknoten **products** und dann **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die die Ergebnisse zuerst in absteigender Reihenfolge nach **categoryName** sortiert und dann in aufsteigender Reihenfolge nach **price**:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse und Statistiken der Abfrage. Dieses Mal können Sie nach dem Abschluss der Abfrage wieder die RU-Gebühren überprüfen.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die die Ergebnisse zuerst in absteigender Reihenfolge nach **categoryName** sortiert, dann in aufsteigender Reihenfolge nach **name** und schließlich in aufsteigender Reihenfolge nach **price**:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Bei der Abfrage sollte der Fehler auftreten: **Die ORDER BY-Abfrage verfügt nicht über einen entsprechenden zusammengesetzten Index für die Bereitstellung**.

1. Erweitern Sie zunächst im **Daten-Explorer** den Datenbankknoten **cosmicworks** und dann den Containerknoten **products**, und wählen Sie dann wieder die Option **Einstellungen** aus.

1. Navigieren Sie auf der Registerkarte **Einstellungen** zum Abschnitt **Indizierungsrichtlinie**.

1. Ersetzen Sie die Indizierungsrichtlinie durch dieses geänderte JSON-Objekt, und **Speichern** Sie dann die Änderungen:

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und wählen Sie zunächst den Containerknoten **products** und dann **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die die Ergebnisse zuerst in absteigender Reihenfolge nach **categoryName** sortiert, dann in aufsteigender Reihenfolge nach **name** und schließlich in aufsteigender Reihenfolge nach **price**:

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse und Statistiken der Abfrage. Dieses Mal können Sie nach dem Abschluss der Abfrage wieder die RU-Gebühren überprüfen.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
