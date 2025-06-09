---
lab:
  title: Erstellen einer gespeicherten Prozedur mit dem Azure-Portal
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# Erstellen einer gespeicherten Prozedur mit dem Azure-Portal

Gespeicherte Prozeduren sind eine von mehreren Möglichkeiten, wie Sie Geschäftslogik serverseitig in Azure Cosmos DB ausführen können. Mit einer gespeicherten Prozedur können Sie grundlegende CRUD-Vorgänge (Create, Read, Update, Delete) mit einem Container für mehrere Dokumente innerhalb eines einzelnen Transaktionsbereichs ausführen.

In diesem Lab erstellen Sie eine gespeicherte Prozedur, die ein Dokument in Ihrem Container erstellt. Anschließend verwenden Sie eine SQL-Abfrage, um die Ergebnisse der gespeicherten Prozedur zu überprüfen.

## Erstellen einer gespeicherten Prozedur

Gespeicherte Prozeduren werden in dem in die Sprache integriertem JavaScript erstellt und unterstützen die Ausführung grundlegender CRUD-Vorgänge innerhalb des Datenbankmoduls. Die Ausführung von JavaScript im Datenbankmodul wird mithilfe des serverseitigen JavaScript-SDK für Azure Cosmos DB und einer Reihe von Hilfsmethoden ermöglicht.

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

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Daten-Explorer**.

1. Wählen Sie im **Daten-Explorer** die Option **Neuer Container** aus, und erstellen Sie dann einen neuen Container mit den folgenden Einstellungen, wobei Sie alle verbleibenden Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Datenbank-ID** | *Neu erstellen* &vert; *``cosmicworks``* |
    | **Share throughput across containers** (Durchsatz zwischen Containern freigeben) | *Wählen Sie diese Option aus* |
    | **Datenbank-Durchsatz** | *Manuell* &vert; *400* |
    | **Container-ID** | *``products``* |
    | **Indizierung** | *Automatisch* |
    | **Partitionsschlüssel** | *``/categoryId``* |

1. Erweitern Sie im **Data Explorer** den Datenbankknoten **cosmicworks**, und wählen Sie dann den neuen Containerknoten **products** in der Navigationsstruktur **NOSQL-API** aus.

1. Wählen Sie **Neue gespeicherte Prozedur** aus.

1. Geben Sie im Feld **ID der gespeicherten Prozedur** den Wert **createDoc**ein.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue JavaScript-Funktion namens **createDoc** ohne Eingabeparameter:

    ```
    function createDoc() {
        
    }
    ```

1. Rufen Sie in der Funktion **createDoc** die integrierte [getContext][azure.github.io/azure-cosmosdb-js-server/global.html]-Methode auf, und speichern Sie das Ergebnis in einer Variablen namens **Kontext**:

    ```
    var context = getContext();
    ```

1. Rufen Sie die [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html]-Methode des Kontextobjekts auf, und speichern Sie das Ergebnis in einer Variablen namens **container**:

    ```
    var container = context.getCollection();
    ```

1. Erstellen Sie ein neues Objekt namens **doc** mit zwei Eigenschaften:

    | **Eigenschaft** | **Farbe** |
    | ---: | :--- |
    | **Name** | *erstes Dokument* |
    | **Kategorie-ID** | *demo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. Rufen Sie die **createDocument**-Methode des Containerobjekts auf, wobei Sie das Ergebnis des Aufrufs der **getSelfLink**-Methode des Containerobjekts und des neuen Dokuments als Parameter übergeben:

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. Nachdem Sie damit fertig sind, sollte ihre gespeicherter Prozedur jetzt Folgenden Code enthalten:

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. Wählen Sie **Speichern** aus, um die Änderungen der gespeicherten Prozedur zu speichern.

1. Wählen Sie **Ausführen** aus, und führen Sie dann die gespeicherte Prozedur mit den folgenden Eingabeparametern aus:

    | **Einstellung** | **Schlüssel** | **Wert** |
    | ---: | :--- | :--- |
    | **Partitionsschlüsselwert** | *Zeichenfolge* | *demo* |

1. Beachten Sie das leere Ergebnis. Die gespeicherte Prozedur wurde zwar erfolgreich ausgeführt, doch hat der JavaScript-Code keine für Menschen lesbare Antwort zurückgegeben.

## Implementieren bewährter Methoden für eine gespeicherte Prozedur

Obwohl die zuvor in dieser Übung erstellte gespeicherte Prozedur grundlegende Funktionen aufweist, fehlen einige gängige Fehlerbehandlungstechniken, die in allen gespeicherten Prozeduren implementiert werden sollten. Zunächst geht die gespeicherte Prozedur davon aus, dass sie immer Zeit zum Abschließen des Vorgangs hat, und überprüft den Rückgabewert der **createDocument**-Methode nicht, um sicherzustellen, dass sie genügend Zeit hat. Zweitens geht die gespeicherte Prozedur davon aus, dass alle Dokumente erfolgreich eingefügt werden, ohne dies zu überprüfen oder potenzielle Fehlermeldungen auszulösen. Schließlich gibt die gespeicherte Prozedur das neu erstellte Dokument nicht als HTTP-Antwort auf die Anforderung zurück, die ursprünglich die gespeicherte Prozedur aufgerufen hat. Sie nehmen diese drei Änderungen an der gespeicherten Prozedur vor, um gängige bewährte Methoden zu implementieren.

1. Kehren Sie zum Editor für die gespeicherte Prozedur **createDoc** zurück.

1. Suchen Sie Zeile 1 im Code, in der die **createDoc**-Funktion definiert wird:

    ```
    function createDoc() {
    ```

    Aktualisieren Sie die Codezeile, um einen Parameter namens **title** einzufügen:

    ```
    function createDoc(title) {
    ```

1. Suchen Sie Zeile 5 im Code, in der die Eigenschaft **name** des **doc**-Objekts festgelegt wird:

    ```
    name: 'first document',
    ```

    Aktualisieren Sie die Codezeile, sodass der Wert des Parameters **title** verwendet wird:

    ```
    name: title,
    ```

1. Suchen Sie Zeile 8 im Code, in der die **createDocument**-Methode aufgerufen wird:

    ```
    container.createDocument(
    ```

    Aktualisieren Sie die Codezeile, um das Ergebnis des Methodenaufrufs in einer Variablen namens **accepted** zu speichern.

    ```
    var accepted = container.createDocument(
    ```

1. Fügen Sie nach dem Aufruf der **createDocument**-Methode eine neue Codezeile hinzu, um den Wert der Variablen **accepted** zu überprüfen und die Methode zurückzugeben, wenn sie nicht den Wert „true“ hat:

    ```
    if (!accepted) return;
    ```

1. Fügen Sie dem **createDocument**-Methodenaufruf schließlich einen dritten Parameter hinzu, der eine Funktion ist, die zwei Parameter namens **error** und **newDoc** akzeptiert. Diese Funktion soll überprüfen, ob der Fehler null ist, und dann den Parameter newDoc auf den Antworttext der gespeicherten Prozedur festlegen:

    ```
    ,
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. Nachdem Sie damit fertig sind, sollte ihre gespeicherter Prozedur jetzt Folgenden Code enthalten:

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. Wählen Sie **Aktualisieren** aus, um die Änderungen an der gespeicherten Prozedur zu speichern.

1. Wählen Sie **Ausführen** aus, und führen Sie dann die gespeicherte Prozedur mit den folgenden Eingabeparametern aus:

    | **Einstellung** | **Schlüssel** | **Wert** |
    | ---: | :--- | :--- |
    | **Partitionsschlüsselwert** | *Zeichenfolge* | *demo* |
    | **Eingabeparameter** | *Zeichenfolge* | *zweites Dokument* |

1. Beachten Sie das JSON-Ergebnis. Nachdem die gespeicherte Prozedur erfolgreich ausgeführt wurde, wurde das neu erstellte Dokument als Antwort auf die ursprüngliche HTTP-Anforderung zurückgegeben.

## Abfragedokumente

Zum Schluss verwenden Sie den Daten-Explorer, um eine SQL-Abfrage auszugeben, die die beiden in diesem Lab erstellten Dokumente zurückgibt.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und wählen Sie dann den Containerknoten **products** in der Navigationsstruktur **NOSQL-API** aus.

1. Wählen Sie **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle Dokumente zurückgibt, bei denen **categoryId** gleich **demo** ist:

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die beiden Dokumente, die in diesem Lab als Ergebnisse der Ausführung dieser Abfrage erstellt wurden.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
