---
lab:
  title: "Verarbeiten von Azure Cosmos\_DB for NoSQL-Daten mit Azure Functions"
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# Verarbeiten von Azure Cosmos DB for NoSQL-Daten mit Azure Functions

Der Azure Cosmos DB-Trigger für Azure Functions wird mithilfe eines Änderungsfeedprozessors implementiert. Mit diesem Wissen können Sie Funktionen erstellen, die auf Erstellungs- und Aktualisierungsvorgänge in Ihrem Azure Cosmos DB for NoSQL-Container reagieren. Wenn Sie einen Änderungsfeedprozessor manuell implementiert haben, ist das Setup für Azure Functions ähnlich.

In dieser Übung erhalten Sie Informationen zu folgenden Vorgängen:

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Wenn Sie ein Azure Cosmos DB-Konto zum ersten Mal bereitstellen, wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Sobald die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen und verwenden, um unter Verwendung des Azure SDK für .NET oder einem anderen SDK Ihrer Wahl eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Cosmos DB*, und erstellen Sie dann eine neue**Azure Cosmos DB for NoSQL-Kontoressource** mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe*. |
    | **Account Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Kapazitätsmodus** | *Serverlos* |

    > &#128221; Ihre Labumgebungen haben möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die zum Herstellen einer Verbindung mit dem Konto aus dem SDK erforderlich sind. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMARY KEY**. Sie verwenden diesen **Schlüsselwert** später in dieser Übung.

1. Wählen Sie im Ressourcenmenü **Data Explorer** aus.

1. Erweitern Sie im Bereich **Daten-Explorer** die Option **Neuer Container**, und wählen Sie dann **Neue Datenbank** aus.

1. Geben Sie im Popup **Neue Datenbank** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *``cosmicworks``* |

1. Beachten Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Popup **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Verwenden vorhandener* &vert; *cosmicworks* |
    | **Container-ID** | *``products``* |
    | **Partitionsschlüssel** | *``/categoryId``* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** erneut die Option **Neuer Container** aus.

1. Geben Sie im Popup **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Verwenden vorhandener* &vert; *cosmicworks* |
    | **Container-ID** | *``productslease``* |
    | **Partitionsschlüssel** | *``/id``* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **productslease** in der Hierarchie.

1. Kehren Sie zur **Startseite** des Azure-Portals zurück.

## Erstellen von Application Insight

Bevor Sie die *Azure Functions-Anwendung*erstellen, müssen Sie eine *Azure Application Insight*-Instanz aktivieren, damit Sie die Anwendungsfunktion überwachen können. Application Insight benötigt zunächst einen *Log Analytics-Arbeitsbereich*.

1. Suchen Sie im Suchfeld nach **Log Analytics-Arbeitsbereich**.

1. Wählen Sie **+ Hinzufügen** aus, um einen neuen *Log Analytics*-Arbeitsbereich zu erstellen.

1. Geben Sie im Dialogfeld des** Log Analytics-Arbeitsbereichs** die folgenden Werte für jede Einstellung ein, wählen Sie **Überprüfen + Erstellen** aus, und wählen Sie dann **Erstellen** aus:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe*. |
    | **Name** | *``lab14laworkspace``* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |

1. Nachdem Ihr *Log Analytics-Arbeitsbereich* erstellt wurde, suchen Sie im Suchfeld nach **Application Insights**.

1. Wählen Sie **+ Erstellen**, um eine neue *Application Insight*-Instanz zu erstellen.

1. Geben Sie im Dialogfeld **Application Insights** die folgenden Werte für jede Einstellung ein, wählen Sie **Überprüfen + Erstellen** aus, und wählen Sie dann **Erstellen** aus:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement (beide Einträge)** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe*. |
    | **Name** | *``lab14appinsight``* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Log Analytics-Arbeitsbereich** | *lab14laworkspace* |

Sie sollten jetzt auf Ihre Anwendung zugreifen können.

## Erstellen einer Azure-Funktions-App unter Verwendung der Azure Cosmos DB-Triggerfunktion

Bevor Sie mit dem Schreiben von Code beginnen können, müssen Sie die Azure Functions-Ressource und die abhängigen Ressourcen (Application Insights, Storage) mithilfe des Erstellungsassistenten erstellen.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Funktionen*, und erstellen Sie dann eine neue**Funktions-App** mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe*. |
    | **Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Veröffentlichen** | *Code* |
    | **Runtimestapel** | *.NET* |
    | **Version** | *6 (LTS) In-Process* |
    | **Region** | *Wählen Sie eine verfügbare Region aus.* |
    | **Speicherkonto** | *Erstellen eines neuen Speicherkontos* |

    > &#128221; Ihre Labumgebungen haben möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Functions**-Kontoressource, und navigieren Sie zum Bereich **Funktionen**.

1. Wählen Sie im Bereich **Funktionen** **+ Erstellen** aus.

1. Erstellen Sie im Popup **Funktion erstellen** eine neue Funktion mit den folgenden Einstellungen, sodass alle verbleibenden Einstellungen ihren Standardwerten bleiben:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Entwicklungsumgebung** | *Im Portal entwickeln* |
    | **Auswählen einer Vorlage** | *Azure Cosmos DB-Trigger* |
    | **Neue Funktion** | *``ItemsListener``* |
    | **Cosmos DB-Kontoverbindung** | Wählen Sie *Neu* &vert; *Azure Cosmos DB-Konto* &vert; *Azure Cosmos DB-Konto, das Sie zuvor erstellt haben* |
    | **Datenbankname** | *``cosmicworks``* |
    | **Sammlungsname** | *``products``* |
    | **Sammlungsname für Leases** | *``productslease``* |
    | **Leasesammlung erstellen, sofern nicht vorhanden** | *Nein* |

## Implementieren von Funktionscode in .NET

Die zuvor erstellte Funktion ist ein C#-Skript, das im Portal bearbeitet wird. Sie verwenden nun das Portal, um eine kurze Funktion zu schreiben, um den eindeutigen Bezeichner jedes Elements auszugeben, das in den Container eingefügt oder aktualisiert wurde.

1. Navigieren Sie im Bereich **ItemsListener** &vert; **Funktion** zum Bereich **Code + Test**.

1. Löschen Sie im Editor für das Skript **run.csx** den Inhalt des Editorbereichs.

1. Verweisen Sie im Editorbereich auf die Bibliothek **Microsoft.Azure.DocumentDB.Core**:

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. Fügen Sie Blöcke für die Namespaces **System**, **System.Collections.Generic** und [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] hinzu:

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. Erstellen Sie eine neue statische Methode mit dem Namen **Run** mit zwei Parametern:

    1. Ein Parameter mit dem Namen **input** vom Typ **IReadOnlyList\<\>** mit einem generischen Typ von [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document].

    1. Ein Parameter mit dem Namen **log** vom Typ **ILogger**.

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. Rufen Sie in der Methode **Run** die Methode **LogInformation** der Variable **log** auf und übergeben Sie eine Zeichenfolge, die die Anzahl der Elemente im aktuellen Batch berechnet:

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. Erstellen Sie weiterhin innerhalb der Funktion **Run** eine foreach-Schleife, die die Variable **input**changes durchläuft, indem Sie die Variable **item** verwenden, um eine Instanz vom Typ **Document** darzustellen:

    ```
    foreach(Document item in input)
    {
    }
    ```

1. Rufen Sie innerhalb der foreach-Schleife in der Methode **Run** die Methode **LogInformation** der Variable **log** auf und übergeben Sie eine Zeichenfolge, die die Eigenschaft [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] der Variable **item** ausgibt:

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. Nachdem Sie fertig sind, sollte Ihre Codedatei jetzt Folgendes enthalten:
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. Erweitern Sie den Abschnitt **Protokolle**, um eine Verbindung mit den Streamingprotokollen für die aktuelle Funktion herzustellen.

    > &#128161; Es kann einige Sekunden dauern, bis eine Verbindung mit dem Streamingprotokolldienst hergestellt wird. Sobald Sie verbunden sind, wird eine Meldung in der Protokollausgabe angezeigt.

1. **Speichern** Sie den aktuellen Funktionscode.

1. Beobachten Sie das Ergebnis der C#-Codekompilierung. Sie sollten davon ausgehen, dass am Ende der Protokollausgabe die Meldung **Kompilierung erfolgreich** angezeigt bekommen.

    > &#128221; Möglicherweise werden Warnmeldungen in der Protokollausgabe angezeigt. Diese Warnungen wirken sich nicht auf dieses Lab aus.

1. **Maximieren** Sie den Protokollabschnitt, um das Ausgabefenster zu erweitern bis es den maximal verfügbaren Platz ausfüllt.

    > &#128221; Sie werden ein anderes Tool verwenden, um Elemente in Ihrem Azure Cosmos DB for NoSQL-Container zu erzeugen. Sobald Sie die Elemente erstellt haben, kehren Sie zu diesem Browserfenster zurück, um sich die Ausgabe anzusehen. Schließen Sie das Browserfenster nicht vorzeitig.

## Legen Sie Ihr Azure Cosmos DB for NoSQL-Konto mit Beispieldaten an.

Sie werden ein Befehlszeilen-Dienstprogramm verwenden, das eine **cosmicworks**-Datenbank und einen **products**-Container erstellt. Das Tool erstellt dann eine Reihe von Elementen, die Sie mit dem Änderungsfeed-Prozessor in Ihrem Terminalfenster beobachten können.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie noch nicht mit der Visual Studio Code-Schnittstelle vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neuer Terminal** aus, um eine neue Terminal-Instanz zu öffnen.

1. Installieren Sie das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] für den globalen Einsatz auf Ihrem Computer.

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; Diese Ausführung dieses Befehls dauert möglicherweise einige Minuten. Dieser Befehl gibt die Warnmeldung („Das Tool „cosmicworks“ ist bereits installiert“) aus, wenn Sie die neueste Version dieses Tools bereits in der Vergangenheit installiert haben.

1. Führen Sie cosmicworks aus, um Ihr Azure Cosmos DB-Konto mit den folgenden Befehlszeilenoptionen zu starten:

    | **Option** | **Wert** |
    | ---: | :--- |
    | **--endpoint** | *Der Endpunktwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--key** | *Der Schlüsselwert, den Sie zuvor in diesem Lab kopiert haben* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; Wenn Ihr Endpunkt beispielsweise lautet **https&shy;://dp420.documents.azure.com:443/** und Ihr Schlüssel lautet: **fDR2ci9QgkdkvERTQ==**, dann lautet der Befehl: ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. Warten Sie, bis der Befehl **cosmicworks** das Konto mit einer Datenbank, einem Container und Gegenständen aufgefüllt hat.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

1. Kehren Sie zum aktuell geöffneten Browserfenster oder zum Tab zurück, in dem der Abschnitt „Azure Functions-Protokoll“ erweitert ist.

1. Beobachten Sie die Protokollausgabe ihrer Funktion. Das Terminal gibt eine **Erkannten Vorgang**-Nachricht für jede Änderung aus, die mit dem Änderungsfeed an sie gesendet wurde. Die Vorgänge werden in Gruppen von ~ 100 Vorgängen zusammengefasst.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
