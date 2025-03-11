---
lab:
  title: "Durchsuchen von Daten mit Azure\_KI-Suche und Azure Cosmos\_DB for NoSQL"
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Durchsuchen von Daten mit Azure KI-Suche und Azure Cosmos DB for NoSQL

Die Azure KI-Suche kombiniert eine Suchmaschine als Dienst mit einer umfassenden Integration mit KI-Funktionen, um die Informationen im Suchindex zu erweitern.

In diesem Lab erstellen Sie einen Azure KI-Suchindex, der Daten automatisch in einem Azure Cosmos DB for NoSQL-Container indiziert und die Daten mithilfe der Azure Cognitive Services-Übersetzerfunktionalität erweitert.

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

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die erforderlich sind, um vom SDK aus eine Verbindung mit dem Konto herzustellen. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY CONNECTION STRING**. Sie verwenden diesen Wert der **Verbindungszeichenfolge** später in dieser Übung.

1. Wählen Sie im Ressourcenmenü **Data Explorer** aus.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Neu erstellen* &vert; *``cosmicworks``* |
    | **Container-ID** | *``products``* |
    | **Partitionsschlüssel** | *``/categoryId``* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

## Seeding Ihres Azure Cosmos DB for NoSQL-Kontos mit Beispieldaten

Sie werden ein Befehlszeilen-Hilfsprogramm verwenden, das eine **cosmicworks**-Datenbank und einen **products**-Container erstellt.

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie **Neuer Terminal** aus.

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

1. Schließen Sie **Visual Studio Code**.

## Erstellen einer Azure KI-Suche-Ressource

Bevor Sie mit dieser Übung fortfahren, müssen Sie zuerst eine neue Azure KI-Suchinstanz erstellen.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *KI-Suche*, und erstellen Sie dann eine neue Kontoressource für die **Azure KI-Suche** mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |

    > &#128221; Ihre Labumgebungen haben möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten Kontoressource für die **Azure KI-Suche**.

## Erstellen von Indexer und Index für Azure Cosmos DB for NoSQL-Daten

Sie erstellen einen Indexer, der eine Teilmenge von Daten in einem bestimmten Azure Cosmos DB for NoSQL-Container stündlich indiziert.

1. Wählen Sie auf dem Ressourcenblatt für die **KI-Suche** die Option **Daten importieren** aus.

1. Wählen Sie im Schritt **Herstellen einer Verbindung mit Ihren Daten** des Assistenten zum **Importieren von Daten** in der Liste **Datenquelle** die Option **Azure Cosmos DB** aus.

1. Konfigurieren Sie die Datenquelle mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Datenquellenname** | *``products-cosmossql-source``* |
    | **Verbindungszeichenfolge** | ***Verbindungszeichenfolge** des zuvor erstellten Azure Cosmos DB for NoSQL-Kontos* |
    | **Datenbank** | *cosmicworks* |
    | **Sammlung** | *products* |

1. Geben Sie in das Feld **Abfrage** die folgende SQL-Abfrage ein, um eine materialisierte Sicht einer Teilmenge Ihrer Daten im Container zu erstellen:

    ```sql
    SELECT 
        p.id, 
        p.categoryId, 
        p.name, 
        p.price,
        p._ts
    FROM 
        products p 
    WHERE 
        p._ts > @HighWaterMark 
    ORDER BY 
        p._ts
    ```

1. Aktivieren Sie das Kontrollkästchen **Abfrageergebnisse sortiert nach _ts**.

    > &#128221; Mit diesem Kontrollkästchen wird die Azure KI-Suche informiert, dass die Abfrage Ergebnisse nach dem Feld **_ts** sortiert. Diese Art von Sortierung ermöglicht die inkrementelle Fortschrittsverfolgung. Wenn der Indexer fehlschlägt, kann er seinen Vorgang direkt beim selben **_ts**-Wert fortsetzen, da die Ergebnisse nach dem Zeitstempel sortiert werden.

1. Klicken Sie auf **Weiter: Kognitive Skills hinzufügen**.

1. Wählen Sie **Springen zu: Zielindex anpassen** aus.

1. Konfigurieren Sie im Schritt **Zielindex anpassen** des Assistenten den Index mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Indexname** | *``products-index``* |
    | **Schlüssel** | *id* |

1. Konfigurieren Sie in der Feldtabelle die Optionen **Abrufbar**, **Filterbar**, **Sortierbar**, **In Facets einteilbar** und **Suchbar** für jedes Feld mithilfe der folgenden Tabelle:

    | **Feld** | **Abrufbar** | **Filterbar** | **Sortierbar** | **In Facets einteilbar** | **Durchsuchbar** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | **categoryId** | &#10004; | &#10004; | &#10004; | &#10004; | |
    | **name** | &#10004; | &#10004; | &#10004; | | &#10004; (Englisch – Microsoft) |
    | **price** | &#10004; | &#10004; | &#10004; | &#10004; | |

1. Klicken Sie auf **Next: Erstellen eines Indexers**.

1. Konfigurieren Sie im Schritt **Indexer erstellen** des Assistenten den Indexer mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Name** | *``products-cosmosdb-indexer``* |
    | **Zeitplan** | *Stündlich* |

1. Wählen Sie **Übermitteln** aus, um die Datenquelle, den Index und den Indexer zu erstellen.

    > &#128221; Möglicherweise müssen Sie ein Umfrage-Pop-up nach dem Erstellen des ersten Indexers schließen.

1. Navigieren Sie auf dem Ressourcenblatt für die **KI-Suche** zur Registerkarte **Indexer**, um das Ergebnis des ersten Indizierungsvorgangs zu beobachten.

1. Warten Sie, bis der Indexer **products-cosmosdb-indexer** den Status **Success** aufweist, bevor Sie mit dieser Aufgabe fortfahren.

    > &#128221; Möglicherweise müssen Sie die Option **Aktualisieren** verwenden, um das Blatt zu aktualisieren, wenn es nicht automatisch aktualisiert wird.

1. Navigieren Sie zur Registerkarte **Indizes**, und wählen Sie dann den Index **products-index** aus.

## Validieren des Indexes mit Beispielsuchabfragen

Nachdem sich jetzt Ihre materialisierte Sicht der Azure Cosmos DB for NoSQL-Daten im Suchindex befindet, können Sie einige grundlegende Abfragen ausführen, die die Vorteile der Features in der Azure KI-Suche nutzen.

> &#128221; Dieses Lab ist nicht dafür vorgesehen, die Syntax der Azure KI-Suche zu vermitteln. Diese Abfragen wurden kuratiert, um einige der Features zu präsentieren, die im Suchindex und in der Engine verfügbar sind.

1. Wählen Sie auf der Registerkarte **Such-Explorer** zunächst das Pulldown-Menü **Ansicht** und dann die **JSON-Ansicht** aus.

1. Beachten Sie im **JSON-Abfrage-Editor** die Syntax der standardmäßigen JSON-Suchabfrage, die alle möglichen Ergebnisse mithilfe des (Platzhalter)-Operators **\*** zurückgibt.

   ```json
    {
      "search": "*",
      "count": true
    }
   ```

1. Wählen Sie die Schaltfläche **Suchen** aus, um die Suche auszuführen.

1. Beachten Sie, dass diese Suchanfrage alle möglichen Ergebnisse zurückgibt und auch ein Metadatenfeld enthält, das die Gesamtzahl der Ergebnisse angibt, auch wenn sie nicht alle auf derselben Seite enthalten sind.

1. Geben Sie im Bereich **JSON-Abfrage-Editor** die folgende Abfrage ein, und wählen Sie dann die Schaltfläche **Suchen** aus:

    ```json
    {
        "search": "touring 3000"
    }
    ```

1. Beachten Sie, dass diese Suchabfrage Ergebnisse zurückgibt, die entweder die Ausdrücke **touring** oder **3000** enthalten, wobei Ergebnisse höher bewertet werden, die beide Ausdrücke enthalten. Die Ergebnisse werden dann nach dem Feld **@search.score** in absteigender Reihenfolge sortiert.

1. Geben Sie im Bereich **JSON-Abfrage-Editor** die folgende Abfrage ein, und wählen Sie dann die Schaltfläche **Suchen** aus:

    ```json
    {
        "search": "blue"
        , "count": true
        , "top": 6
    }
    ```

1. Beachten Sie, dass diese Suchabfrage jeweils nur einen Satz von sechs Ergebnissen zurückgibt, obwohl mehr Übereinstimmungen serverseitig vorhanden sind.

1. Geben Sie im Bereich **JSON-Abfrage-Editor** die folgende Abfrage ein, und wählen Sie dann die Schaltfläche **Suchen** aus:

    ```json
    {
        "search": "mountain"
        , "count": true
        , "top": 25
        , "skip": 50
    }
    ```

1. Beachten Sie, dass diese Suchabfrage die ersten 50 Ergebnisse überspringt und einen Satz von 25 Ergebnissen zurückgibt. Wenn dies eine paginierte Ansicht in einer clientseitigen Anwendung wäre, könnten Sie daraus ableiten, dass dies die dritte „Seite“ der Ergebnisse ist.

1. Geben Sie im Bereich **JSON-Abfrage-Editor** die folgende Abfrage ein, und wählen Sie dann die Schaltfläche **Suchen** aus:

    ```json
    {
        "search": "touring"
        , "count": true
        , "filter": "price lt 500"
    }
    ```

1. Beachten Sie, dass diese Suchabfrage nur Ergebnisse zurückgibt, bei denen der Wert des numerischen Preisfelds kleiner als 500 ist.

1. Geben Sie im Bereich **JSON-Abfrage-Editor** die folgende Abfrage ein, und wählen Sie dann die Schaltfläche **Suchen** aus:

    ```json
    {
        "search": "road"
        , "count": true
        , "top": 15
        , "facets": ["price,interval:500"]
    }
    ```

1. Beachten Sie, dass diese Suchabfrage eine Sammlung von Facetdaten zurückgibt, die angibt, wie viele Elemente zu jeder Kategorie gehören, auch wenn sie nicht alle auf der aktuellen Ergebnisseite vorhanden sind. In diesem Beispiel werden die übereinstimmenden Elemente in Intervallen von 500 in numerische Preiskategorien unterteilt. Dies wird in der Regel verwendet, um Filter und Navigationshilfen in clientseitigen Anwendungen aufzufüllen.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
