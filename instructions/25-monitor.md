---
lab:
  title: "Verwenden von Azure Monitor zum Analysieren eines Azure Cosmos\_DB for NoSQL-Kontos"
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Verwenden von Azure Monitor zum Analysieren eines Azure Cosmos DB for NoSQL-Kontos

Azure Monitor ist ein umfassender Stapelüberwachungsdienst in Azure, der umfangreiche Features zum Überwachen von Azure-Ressourcen bereitstellt.  Azure Cosmos DB erstellt Überwachungsdaten mithilfe von Azure Monitor.  Azure Monitor erfasst die Metriken und Telemetriedaten von Cosmos DB.

In dieser Übung führen Sie eine simulierte Workload für Azure Cosmos DB-Container aus und analysieren, wie sich diese Workload auf das Azure Cosmos DB-Konto auswirkt.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALTTASTE+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Beim ersten Bereitstellen eines Azure Cosmos DB-Kontos wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Nachdem die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen. Verwenden Sie den Endpunkt und den Schlüssel, um programmgesteuert eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen. Verwenden Sie den Endpunkt und den Schlüssel für die Verbindungszeichenfolgen des Azure SDK for .NET oder eines anderen SDK.

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
    | **Begrenzen des Gesamtdurchsatzes, der für dieses Konto bereitgestellt werden kann** | *Deaktivieren* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die erforderlich sind, um vom SDK aus eine Verbindung mit dem Konto herzustellen. Speziell:

    1. Beachten Sie das Feld **URI**. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

    1. Beachten Sie das Feld **PRIMÄRSCHLÜSSEL**. Sie verwenden diesen **Schlüsselwert** später in diesem Lab.

1. Minimieren Sie das Browserfenster, schließen Sie es jedoch nicht. Wir kehren ein paar Minuten nach dem Starten einer Hintergrundworkload in den nächsten Schritten zum Azure-Portal zurück.


## Importieren der Microsoft.Azure.Cosmos- und Newtonsoft.Json-Bibliotheken in ein .NET-Skript

Die .NET-CLI enthält einen Befehl zum Hinzufügen eines [Pakets][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] zum Importieren von Paketen aus einem vorkonfigurierten Paketfeed. Eine .NET-Installation verwendet NuGet als Standardpaketfeed.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **25-monitor**.

1. Öffnen Sie das Kontextmenü für den Ordner **25-monitor**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **25-monitor** festgelegt ist.

1. Fügen Sie das Paket [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0] aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. Fügen Sie das Paket [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.3] aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Newtonsoft.Json --version 13.0.3
    ```

## Ausführen eines Skripts zum Erstellen der Container und der Workload

Wir sind jetzt bereit, eine Workload auszuführen, um deren Nutzung des Azure Cosmos DB-Kontos zu überwachen.  Das Skript, das wir unter der Haube ausführen werden. Dieses Skript erstellt drei Container und lädt einige Daten in diese. Das Skript führt dann einige SQL-Abfragen zufällig aus, um mehrere Benutzeranwendungen zu emulieren, die auf das Azure Cosmos DB-Konto treffen.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **25-monitor**.

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

1. Speichern Sie die Datei **Program.cs**.

1. Kehren Sie zum *integrierten Terminal* zurück.

1. Erstellen Sie das Projekt, und führen Sie es mit dem Befehl [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] aus:

    ```
    dotnet run
    ```
    > &#128221; Der erste Teil dieses Skripts erstellt die drei Container und lädt die Daten in sie. Dieser Vorgang sollte ungefähr 2 Minuten dauern. Zum Emulieren einiger ratenbegrenzender Ereignisse legt das Skript dann den bereitgestellten Durchsatz auf 400 RU/s fest. Anschließend sollten die Meldung ***Simulierte Hintergrundworkload wird erstellt. Warten Sie 5 bis 10 Minuten, und fahren Sie mit dem nächsten Schritt der Übung fort.*** angezeigt werden. Da Azure-Ressourcen laden Überwachungsdaten asynchron in Azure Monitor hoch. Wir müssen kurz warten, bevor wir mit dem Abrufen einiger Diagnosedaten in den Azure Monitor-Metriken und -Erkenntnissen beginnen. Nach 5 bis 10 Minuten fahren Sie mit dem nächsten Schritt fort. Wenn Sie weitere Diagnosedaten erfassen möchten, müssen Sie das Skript nicht nach 5 bis 10 Minuten beenden, sondern können bis zum Ende des Labs damit warten.

    > &#128221; Sie werden eine Reihe von gelb hervorgehobenen Warnungen bemerken, da der Compiler erkennt, dass das Skript viele Vorgänge synchron ausführt und nicht auf eine Antwort der Vorgänge wartet. Sie können diese Warnung ignorieren, da das erwartete Verhalten mehrere SQL-Skripts gleichzeitig ausführt.

## Verwenden von Azure Monitor zum Analysieren der Azure Cosmos DB-Kontonutzung

In diesem Teil der Übung kehren wir zum Browser zurück und überprüfen einige der Azure Monitor-Erkenntnis- und Metrikberichte.

### Berichte zu Azure Monitor-Metriken

1. Wechseln Sie zurück zum geöffneten Browserfenster, das Sie zuvor minimiert haben. Wenn Sie es geschlossen haben, öffnen Sie ein neues Konto, und wechseln Sie unter „portal.azure.com“ zu Ihrer Azure Cosmos DB-Kontoseite.

1. Wählen Sie im linken Menü von Azure Comsos DB unter *Überwachung* die Option **Metriken** aus. Sie werden feststellen, dass die Felder **Bereich** und **Metriknamespace** bereits mit den richtigen Informationen aufgefüllt wurden. In den folgenden Schritten sehen wir uns einige **Metrikoptionen** und die Features *Filter hinzufügen* und *Teilung anwenden* an.

1. Standardmäßig werden im Abschnitt *Metriken* Diagnoseinformationen für die letzten 24 Stunden angezeigt. Wir müssen präziser filtern, um die Metriken der Workload zu betrachten, die wir im vorherigen Schritt erstellt haben. Wählen Sie in der oberen rechten Ecke die Schaltfläche mit der Bezeichnung ***Ortszeit: Letzte 24 Stunden (Automatisch)*** aus. Dann wird ein Fenster mit mehreren Optionsfeldern für den Zeitbereich geöffnet.  Wählen Sie das Optionsfeld ***Letzte 30 Minuten*** und dann die Schaltfläche **Übernehmen** aus. Bei Bedarf können Sie viel präziser filtern, indem Sie das Optionsfeld *Benutzerdefiniert* auswählen und ein Start- und Enddatum sowie eine Start- und Endzeit auswählen. 

1. Nachdem wir nun einen geeigneten Zeitbereich für unsere Diagnosediagramme festgelegt haben, werfen wir einen Blick auf einige Metriken. Wir beginnen mit einer gängigen Metrik. Wählen Sie im Pulldownmenü *Metrik* die Option **Anforderungseinheiten gesamt** aus. Standardmäßig wird diese Metrik als Gesamtsumme der RUs angezeigt. Alternativ können Sie auch das Pulldownmenü „Aggregation“ in „Durchschnitt“ oder „Max“ ändern. Nachdem Sie diese beiden Aggregationen ausgecheckt haben, legen Sie sie für die folgenden Schritte wieder auf *Summe* fest.

1. Diese Metrik gibt uns eine gute Vorstellung davon, wie viele Anforderungseinheiten in unserem Azure Cosmos DB-Konto verwendet wurden. Unser aktuelles Diagramm hilft uns jedoch möglicherweise nicht bei einem Problem, wenn mehrere Datenbanken oder Container in unserem Konto vorhanden sind. Lassen Sie uns das ändern, indem Sie sich ansehen, wie der RU-Verbrauch der Datenbank war. Wählen Sie im Menü unter dem Zeichentitel die Option **Teilung anwenden** und dann im Pulldownmenü **Werte** die Option **DatabaseName** aus. Wählen Sie dann eine beliebige Stelle im Diagramm aus, um die Änderungen zu übernehmen. Die Schaltfläche **Teilen nach = DatabaseName** wird jetzt direkt über dem Diagramm eingefügt. 

1. Jetzt wissen wir, welche Datenbank den größten Teil der Arbeit erledigt. Obwohl diese Informationen gut sind, wissen wir nicht, welcher Container die ganze Arbeit erledigt.  Wählen Sie die Schaltfläche **Teilen nach = DatabaseName** aus, um die Bedingung für „Teilen“ zu ändern, und wählen Sie **CollectionName** aus dem Pulldownmenü *Werte* aus. Großartig, wir sollten jetzt Daten für unsere **customer**- und **salesOrder**-Sammlungen haben. Es gibt nur ein Problem mit diesem Diagramm: Die **salesOrder**-Sammlung ist in zwei Datenbanken vorhanden, **database-v2** und **database-v3**, sodass dieser Wert eine Aggregation dieses Sammlungsnamens in beiden Datenbanken ist.

1. Das Problem sollte einfach zu beheben sein. Wählen Sie die Schaltfläche **Filter hinzufügen** und dann im Pulldownmenü *Eigenschaft* die Option **DatabaseName** und unter *Werte* die Option **database-V3** aus.

1. Sehen wir uns zwei weitere Metriken an. Bearbeiten Sie das vorhandene Diagramm. Sie können auch ein neues Diagramm erstellen, wenn Sie möchten. Wählen Sie über dem Diagramm die Schaltfläche mit dem *Namen des Azure Cosmos DB-Kontos* und der Bezeichnung **Anforderungseinheiten gesamt** aus. Wählen Sie **Anforderungen gesamt** aus dem Pulldownmenü *Metrik* aus, und beachten Sie, dass die einzige verfügbare Aggregation *Anzahl* ist.

1. Zwei wichtige Filter können uns dabei helfen, verschiedene Arten von Problemen zu beheben. Fügen wir einen Filter mit der Eigenschaft **StatusCode** hinzu (ein ähnlicher Filter mit einem anderen Detailtyp wäre **Status**), und wählen Sie **200** und **429** für *Werte* aus. Ändern Sie die Teilung so, dass „StatusCode“ verwendet wird. Beachten Sie, dass eine große Anzahl von 429-Anforderungen, oder ratenbegrenzende Anforderungen, im Vergleich zu erfolgreichen Anforderungen mit dem Status 200, vorliegt. Die 429-Ausnahmen sind aufgetreten, da das Skript Tausende von Anforderungen pro Sekunde sendet. Wir haben den bereitgestellten Durchsatz jedoch auf 400 RU/s festgelegt. *Diese große Anzahl von 429-Ausnahmen im Vergleich zu erfolgreichen Anforderungen sollte in einer Produktionsumgebung nicht normal sein. In einer Produktionsumgebung sollten 429-Ausnahmen in einem fehlerfreien Azure Cosmos DB-Konto selten auftreten.*  Sie können die *Eigenschaften* **StatusCode** oder **Status** auch ähnlich für die Problembehandlung bei **Anforderungseinheiten gesamt** verwenden.

1. Sehen wir uns **Anforderungen gesamt** an, aber ändern die Teilung in **OperationType**.  Diese Eigenschaft hilft uns zu bestimmen, welche Lese- oder Schreibvorgänge den Großteil der Arbeit erledigen. Auch diese Eigenschaft kann auf ähnliche Weise für **Anforderungseinheiten gesamt** verwendet werden.

1. Experimentieren Sie wie bei **Anforderungseinheiten gesamt** mit verschiedenen Filtern und Teilungsoptionen. 

1. Die endgültige Metrik, die wir in dieser Übung betrachten, ist **Normalisierter RU-Verbrauch**. Ändern Sie die Teilung in **PartitionKeyRangeId**. Anhand dieser Metrik können wir ermitteln, welche Partitionsschlüsselbereichsnutzung wärmer ist. Die Metrik teilt uns die Neigung des Durchsatzes in Richtung eines Partitionsschlüsselbereichs mit. Fahren Sie fort, und wählen Sie diese Metrik aus dem Pulldownmenü *Metrik* aus. Dieses Diagramm sollte nun ein stark fehlerhaftes System zeigen, das dauerhaft 100 % für **Normalisierter RU-Verbrauch** erreicht.

> &#128221; Wenn Sie mehrere Diagramme gleichzeitig betrachten möchten, klicken Sie oberhalb des Diagrammnamens auf die Option **+ Neues Diagramm**. 

> &#128221; Obwohl Sie die Metriken nicht direkt speichern können, können Sie ein vorhandenes Dashboard erstellen oder verwenden und dieses Diagramm hinzufügen, indem Sie auf die Schaltfläche **An Dashboard anheften** in der oberen rechten Ecke des Diagramms klicken.  Klicken Sie auf die Schaltfläche, wählen Sie die Registerkarte **Neu erstellen** aus, nennen Sie sie *DP-420 Labs*, und klicken Sie auf **Erstellen und anheften**. Um Ihre privaten Dashboards anzuzeigen, wechseln Sie in der oberen linken Ecke zum Portalmenü, und wählen Sie „Dashboard“ aus Ihren Azure-Ressourcenoptionen aus. Es kann einige Minuten dauern, bis das Dashboard das erste Mal angezeigt wird.

> &#128221; Eine weitere Möglichkeit zum Teilen Ihres Diagramms besteht darin, auf das Pulldownmenü „Teilen“ zu klicken und es als Excel-Datei herunterzuladen oder einen Link zu kopieren.

### Azure Monitor-Erkenntnisberichte

Möglicherweise müssen wir einige Zeit mit der Optimierung unserer Diagnoseberichte für Azure Monitor-Metriken verbringen.  Cosmos DB-Erkenntnisse bieten einen Überblick über die Gesamtleistung, Fehler und der Betriebsintegrität Ihrer Azure Cosmos DB-Ressourcen. Diese Erkenntnisdiagramme sind vordefinierte Diagramme, die den Metriken ähneln. Sehen wir uns einige davon an.

1. Wählen Sie im linken Menü von Azure Comsos DB unter *Überwachung* die Option **Erkenntnisse** aus. Sie werden feststellen, dass es mehrere Registerkarten von „Übersicht“ bis zu „Verwaltungsoptionen“ gibt. Wir sehen uns einige dieser **Erkenntnisdiagramme** an. Die erste Registerkarte, „Übersicht“, enthält eine Zusammenfassung der am häufigsten verwendeten Diagramme. Dazu zählen beispielsweise Diagramme wie „Anforderungen gesamt“, „Daten- und Indexnutzung“, „429-Ausnahmen“ und „Normalisierter RU-Verbrauch“.  Die meisten dieser Diagramme haben wir im vorherigen Abschnitt kennengelernt.

1. Beachten Sie oben in den Diagrammen, dass Sie den **Zeitbereich** festlegen können. Wählen Sie also entweder *15* oder *30* Minuten aus, um die Workload in dieser Übung auszuwerten.

1. In der oberen rechten Ecke *jedes* Diagramms sehen Sie eine Option, mit der Sie den ***Metrik-Explorer öffnen***. Wählen Sie nun die Option **Metrik-Explorer öffnen** für das Diagramm **Anforderungen gesamt** aus. Wenn Sie diese Option auswählen, gelangen Sie zu den Berichten der Metrik, die Sie zuvor überprüft haben. Der Vorteil, den Metrik-Explorer zu öffnen, besteht darin, dass bereits ein Teil des Diagramms für uns erstellt wurde.

1. Sie gelangen zurück zur Erkenntnisseite, indem Sie oben rechts im Metrikdiagramm auf das **X** klicken.

1. Wählen Sie die Registerkarte „Durchsatz“ aus. Diese Diagramme eignen sich gut zum Bestimmen von Durchsatzproblemen.  Sehen Sie sich das Diagramm **Normalisierter RU-Verbrauch (%) nach PartitionKeyRangeID** genau an, das verwendet werden kann, um heiße Partitionen zu erkennen.

1. Wählen Sie die Registerkarte „Anforderungen“ aus. Diese Diagramme eignen sich hervorragend, um die Anzahl der begrenzenden Ereignisse zu analysieren, die das Konto aufweist (429 im Vergleich zu 200), oder die Anzahl der Anforderungen pro Vorgangstyp zu überprüfen.  

1. Wählen Sie die Registerkarte „Speicher“ aus. Diese Diagramme zeigen uns sowohl das Wachstum unserer Sammlungen als auch die Daten- und Indexnutzung.  

1. Wählen Sie die Registerkarte „System“ aus. Wenn Ihre Anwendung häufig Kontenmetadaten erstellt, gelöscht oder abfragt, ist es möglich, dass 429-Ausnahmen vorliegen.  Diese Diagramme helfen uns zu bestimmen, ob der häufige Metadatenzugriff die Ursache unserer 429-Ausnahmen ist. Darüber hinaus können wir den Status unserer Metadatenanforderungen ermitteln.  

### Azure Monitor-Erkenntnisberichte

1. Wenn das Programm noch ausgeführt wird, wechseln Sie zurück zum Visual Studio Code-Befehlsterminal.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[nuget.org/packages/Newtonsoft.Json/13.0.1]: https://www.nuget.org/packages/Newtonsoft.Json/13.0.1
