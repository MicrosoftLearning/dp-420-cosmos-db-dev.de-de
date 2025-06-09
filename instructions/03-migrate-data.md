---
lab:
  title: Migrieren vorhandener Daten mit Azure Data Factory
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Migrieren vorhandener Daten mit Azure Data Factory

In Azure Data Factory wird Azure Cosmos DB als Quelle der Datenerfassung als auch als Ziel (Senke) der Datenausgabe unterstützt.

In diesem Lab werden wir Azure Cosmos DB mithilfe eines hilfreichen Befehlszeilen-Hilfsprogramms auffüllen und dann Azure Data Factory verwenden, um eine Teilmenge von Daten aus einem Container in einen anderen zu verschieben.

## Erstellen und Seeding Ihres Azure Cosmos DB for NoSQL-Kontos

Sie werden ein Befehlszeilen-Hilfsprogramm verwenden, das eine **cosmicworks**-Datenbank und einen **products**-Container bei **4.000** Anforderungseinheiten pro Sekunde (RU/s) erstellt. Nach der Erstellung skalieren Sie den Durchsatz auf 400 RU/s herunter.

Um den products-Container zu begleiten, erstellen Sie einen **flatproducts**-Container manuell, der das Ziel der ETL-Transformation und des Ladevorgangs am Ende dieses Labs ist.

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

    1. Beachten Sie das Feld **PRIMARY CONNECTION STRING**. Sie verwenden diesen Wert der **Verbindungszeichenfolge** später in dieser Übung.

1. Lassen Sie die Browserregisterkarte geöffnet, da wir später dorthin zurückkehren werden.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie noch nicht mit der Visual Studio Code-Schnittstelle vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted]

1. Öffnen Sie in **Visual Studio Code** das Menü **Terminal**, und wählen Sie dann **Neues Terminal** aus, um eine neue Terminalinstanz zu öffnen.

1. Installieren Sie das Befehlszeilentool [cosmicworks][nuget.org/packages/cosmicworks] für den globalen Einsatz auf Ihrem Computer.

    ```powershell
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

1. Wechseln Sie zurück zum Webbrowser, öffnen Sie eine neue Registerkarte, und navigieren Sie zum Azure-Portal (``portal.azure.com``).

1. Wählen Sie zunächst die Option **Ressourcengruppen** und dann die Ressourcengruppe aus, die Sie zuvor in diesem Lab erstellt oder angezeigt haben. Wählen Sie dann die **Azure Cosmos DB-Kontoressource** aus, die Sie in diesem Lab erstellt haben.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie zunächst im **Daten-Explorer** den Datenbankknoten **cosmicworks** und dann den Containerknoten **products**, und wählen Sie anschließend die Option **Elemente** aus.

1. Beachten Sie die verschiedenen JSON-Elemente im **products**-Container, und wählen Sie sie aus. Dies sind die Elemente, die vom Befehlszeilentool erstellt wurden, das in den vorherigen Schritten verwendet wurde.

1. Wählen Sie den **Skalierungsknoten** aus. Wählen Sie auf der Registerkarte **Skalieren** die Option **Manuell**, aktualisieren Sie die Einstellung **erforderlichen Durchsatz** von **4000 RU/s** auf **400 RU/s** und speichern Sie Ihre Änderungen mit **Speichern****.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Vorhandene verwenden* &vert; *cosmicworks* |
    | **Container-ID** | *`flatproducts`* |
    | **Partitionsschlüssel** | *`/category`* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **flatproducts** in der Hierarchie.

1. Kehren Sie zur **Startseite** des Azure-Portals zurück.

## Erstellen einer Azure Data Factory-Ressource

Da nun die Azure Cosmos DB for NoSQL-Ressourcen vorhanden sind, erstellen Sie eine Azure Data Factory-Ressource, und konfigurieren Sie alle erforderlichen Komponenten und Verbindungen, um eine einmalige Datenverschiebung von einem NoSQL-API-Container in einen anderen durchzuführen, um Daten zu extrahieren, zu transformieren und in einen anderen NoSQL-API-Container zu laden.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Data Factory*, und erstellen Sie dann eine neue**Data Factory**-Ressource mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Region** | *Wählen Sie eine verfügbare Region aus.* |
    | **Version** | *Version 2* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Data Factory**-Ressource, und wählen Sie **Studio starten** aus.

    > &#128161; Alternativ können Sie zu (``adf.azure.com/home``) navigieren, Ihre neu erstellte Data Factory-Ressource auswählen und dann das Symbol der Startseite auswählen.

1. Von der Startseite aus. Wählen Sie die Option **Erfassen** aus, um den Schnellassistenten zu starten, um einen Vorgang zum einmaligen Kopieren von Daten im großen Maßstab auszuführen und zum Schritt **Eigenschaften** des Assistenten zu wechseln.

1. Beginnen Sie mit dem Schritt **Eigenschaften** des Assistenten, und wählen Sie im Abschnitt **Aufgabentyp** die Option **Integrierte Kopieraufgabe** aus.

1. Wählen Sie im Abschnitt **Aufgabenhäufigkeit oder Aufgabenzeitplan** zunächst die Option **Jetzt einmal ausführen** und dann die Option **Weiter** aus, um zum Schritt **Quelle** des Assistenten zu wechseln.

1. Wählen Sie im Schritt **Quelle** des Assistenten in der Liste **Quelltyp** die Option **Azure Cosmos DB for NoSQL** aus.

1. Wählen Sie im Abschnitt **Verbindung** die Option **+ Neue Verbindung** aus.

1. Konfigurieren Sie im Pop-up **Neue Verbindung (Azure Cosmos DB for NoSQL)** die neue Verbindung mit den folgenden Werten, und wählen Sie dann **Erstellen** aus:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Name** | *`CosmosSqlConn`* |
    | **Herstellen einer Verbindung über Integration Runtime** | *AutoResolveIntegrationRuntime* |
    | **Authentifizierungsmethode** | *Kontoschlüssel* &vert; *Verbindungszeichenfolge* |
    | **Kontoauswahlmethode** | *Im Azure-Abonnement* |
    | **Azure-Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Der Azure Cosmos DB-Kontoname** | *Ihr vorhandener Azure Cosmos DB-Kontoname, den Sie zuvor in diesem Lab ausgewählt haben* |
    | **Datenbankname** | *cosmicworks* |

1. Wählen Sie im Abschnitt **Quelldatenspeicher** im Abschnitt **Quelltabellen** die Option **Abfrage verwenden** aus.

1. Wählen Sie in der Liste **Tabellenname** die Option **products** aus.

1. Löschen Sie im **Abfrage**-Editor den vorhandenen Inhalt, und geben Sie die folgende Abfrage ein:

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. Wählen Sie die Option **Vorschaudaten** aus, um die Gültigkeit der Abfrage zu testen. Wählen Sie **Weiter** aus, um zum Schritt **Ziel** des Assistenten zu wechseln.

1. Wählen Sie im Schritt **Ziel** des Assistenten in der Liste **Zieltyp** die Option **Azure Cosmos DB for NoSQL** aus.

1. Wählen Sie in der Liste **Verbindung** die Option **CosmosSqlConn** aus.

1. Wählen Sie zunächst in der Liste **Ziel** die Option **flatproducts** und dann **Weiter** aus, um zum Schritt **Einstellungen** des Assistenten zu wechseln.

1. Geben Sie im Schritt **Einstellungen** des Assistenten im Feld **Aufgabenname** den Text **`FlattenAndMoveData`** ein.

1. Belassen Sie alle verbleibenden Felder auf ihren standardmäßigen leeren Werten bei, und wählen Sie dann **Weiter** aus, um zum letzten Schritt des Assistenten zu wechseln.

1. Überprüfen Sie die **Zusammenfassung** der Schritte, die Sie im Assistenten ausgewählt haben, und wählen Sie dann **Weiter** aus.

1. Beachten Sie die verschiedenen Schritte in der Bereitstellung. Wenn die Bereitstellung abgeschlossen ist, wählen Sie **Fertigstellen** aus.

1. Kehren Sie zur Registerkarte „Browser“ mit Ihrem **Azure Cosmos DB-Konto** zurück, und navigieren Sie zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, wählen Sie zunächst den Containerknoten **flatproducts**und dann die Option **Neue SQL-Abfrage** aus.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Erstellen Sie eine neue SQL-Abfrage, die alle Dokumente zurückgibt, bei denen der **Name** **HL Headset** entspricht:

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        flatproducts p
    WHERE
        p.name = 'HL Headset'
    ```

1. Klicken Sie auf **Abfrage ausführen**.

1. Beachten Sie die Ergebnisse der Abfrage.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
