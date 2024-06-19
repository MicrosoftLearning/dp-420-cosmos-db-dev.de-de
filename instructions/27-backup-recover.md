---
lab:
  title: Wiederherstellen einer Datenbank oder eines Containers anhand eines Wiederherstellungspunkts
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Wiederherstellen einer Datenbank oder eines Containers anhand eines Wiederherstellungspunkts 

Azure erstellt automatisch verschlüsselte Sicherungen Ihrer Daten. Diese Sicherungen werden in zwei Sicherungsmodi ausgeführt: den Modi **Periodisch** und **Fortlaufend**.

In diesem Lab führen Sie **Sicherungen** und **Wiederherstellungen** unter Verwendung des fortlaufenden Sicherungsmodus durch. Zunächst erstellen Sie ein Azure Cosmos DB-Konto. Anschließend erstellen Sie zwei Container und fügen ihnen einige Dokumente hinzu. Als Nächstes aktualisieren Sie einige Dokumente in diesen Containern. Schließlich erstellen Sie Wiederherstellungen des Kontos zu einem Zeitpunkt vor der jeweiligen Löschung.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Beim ersten Bereitstellen eines Azure Cosmos DB-Kontos wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Nachdem die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen. Verwenden Sie den Endpunkt und den Schlüssel, um programmgesteuert eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen. Verwenden Sie den Endpunkt und den Schlüssel für die Verbindungszeichenfolgen des Azure SDK for .NET oder eines anderen SDK.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Cosmos DB*, und erstellen Sie dann eine neue**Azure Cosmos DB for NoSQL**-Kontoressource mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Account Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Kapazitätsmodus** | *Bereitgestellter Durchsatz* |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |
    | **Globale Verteilung** TAB | Schreibvorgänge in mehreren Regionen deaktivieren |

    > &#128221; Beachten Sie, dass Sie den Modus **Fortlaufend** (7-Tage-Modus) während der Erstellung des Azure Cosmos DB-Kontos aktivieren können, indem Sie ihn auf der Registerkarte **Sicherungsrichtlinie** auswählen. In diesem Lab können Sie dieses Feature wahlweise während der Kontoerstellung oder nach der Erstellung des Kontos im optionalen Abschnitt unten aktivieren. **Wenn Sie das Feature jedoch <ins>*nach*</ins> der Erstellung des Kontos* aktivieren, kann dies mehr als 5 Minuten in Anspruch nehmen*.**

    > &#128221; Beachten Sie, dass *[Konten mit Schreibvorgängen in mehreren Regionen derzeit nicht für fortlaufende Sicherungen unterstützt werden][/azure/cosmos-db/continuous-backup-restore-introduction]*.

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

## Hinzufügen von einer Datenbank und zwei Containern zum Konto

Erstellen wir eine Datenbank und zwei Container.

1. Navigieren Sie im Azure-Portal zur Seite mit Ihrem Azure Cosmos DB-Konto.

1. Fügen Sie im **Daten-Explorer** einen neuen Container mit den folgenden Einstellungen hinzu:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Datenbank-ID** | *Neu erstellen* Name: *`Sales`* |
    | **Share throughput across containers** (Durchsatz zwischen Containern freigeben) | *Nicht auswählen* |
    | **Container-ID** | *`customer`* |
    | **Partitionsschlüssel** | *`/id`* |
    | **Containerdurchsatz (400 – unbegrenzte RU/s)** | *Manueller* Durchsatz: *400*|

1. Fügen Sie im **Daten-Explorer** einen neuen Container mit den folgenden Einstellungen hinzu:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Datenbank-ID** | * Vorhandenen Namen verwenden*: *Vertrieb* |
    | **Container-ID** | *`salesOrder`* |
    | **Partitionsschlüssel** | *`/id`* |
    | **Containerdurchsatz (400 – unbegrenzte RU/s)** | *Manueller* Durchsatz: *400*|

## Hinzufügen von Elementen zum Container

Fügen wir diesen Containern einige Dokumente hinzu.

1. Navigieren Sie im Azure-Portal zur Seite mit Ihrem Azure Cosmos DB-Konto.

1. Fügen Sie unter **Daten-Explorer** dem Container **customer** die folgenden zwei Dokumente hinzu.

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. Fügen Sie im **Daten-Explorer** dem Container **salesOrder** die folgenden drei Dokumente hinzu.

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## Ändern des Standardsicherungsmodus in „fortlaufend“ (optional, wenn das Feature während der Kontoerstellung nicht aktiviert wird)

*Wenn Sie das Feature nicht während der Azure Cosmos DB-Kontoerstellung aktiviert haben, müssen Sie dies jetzt tun.*  Der Sicherungsmodus lässt sich einfach ändern. Dazu muss lediglich eine Einstellung in **Ein** geändert werden. Wir ändern sie jetzt.

1. Navigieren Sie im Azure-Portal zur Seite mit Ihrem Azure Cosmos DB-Konto.

1. Wählen Sie im Abschnitt **Einstellungen** die Option **Sichern und Wiederherstellen** aus.

1. Wählen Sie **Ändern** neben **Sicherungsrichtlinienmodus** aus, und wählen Sie in diesem Bildschirm die Option **Fortlaufend (7 Tage)** und dann **Speichern** aus. ***Die Aktivierung dieses Features kann mehr als fünf Minuten dauern***.

    > &#128221; Beachten Sie, dass *[Konten mit Schreibvorgängen in mehreren Regionen derzeit nicht für fortlaufende Sicherungen unterstützt werden][/azure/cosmos-db/continuous-backup-restore-introduction]*. Wenn Sie beim Erstellen Ihres Azure Cosmos DB-Kontos Schreibvorgänge mit mehreren Regionen nicht deaktiviert haben, müssen Sie dies jetzt tun, da sonst die Aktivierung fortlaufender Sicherungen fehlschlägt.  Sie können Schreibvorgänge mit mehreren Regionen unter dem Abschnitt **Daten global replizieren** *Einstellungen* deaktivieren.

## Löschen eines der salesOrder-Dokumente

1. Führen Sie im **Daten-Explorer** die folgende Abfrage aus, um das aktuelle Datum und die aktuelle Uhrzeit abzurufen. Kopieren Sie diesen Zeitstempel in den Editor. Dieser Zeitstempel sollte in UTC angegeben werden.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. Suchen Sie unter **Daten-Explorer** das Dokument **salesOrder** mit der **id** `0019092E-BD25-48F5-8050-7051B2655BC5`. Löschen Sie das Dokument, vergewissern Sie sich, dass das Dokument nicht mehr vorhanden ist.

## Wiederherstellen der Datenbank an dem Punkt, bevor das salesOrder-Dokument gelöscht wurde

1. Navigieren Sie im Azure-Portal zur Seite mit Ihrem Azure Cosmos DB-Konto.

1. Wählen Sie im Abschnitt *Einstellungen* die Option **Point-in-Time-Wiederherstellung** aus. Verwenden Sie folgende Einstellungen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Wiederherstellungspunkt (UTC)** | Konvertieren Sie das Datum und die Uhrzeit entsprechend. Die Uhrzeit muss im AM/PM-Format angegeben werden.|
    | **Location** | *Wählen Sie einen verfügbaren Speicherort aus*. |
    | **Wählen Sie die Ressourcen aus, die Sie wiederherstellen möchten**. | *Ausgewählte Datenbank/ausgewählter Container* |
    | **Wiederherstellungsressource** | *salesOrder* |
    | **Wiederherstellungszielkonto** | *wählen Sie einen* ***neuen*** *Azure Cosmos DB-Kontonamen aus* |

    > &#128221; Bei Azure Cosmos DB-Wiederherstellungen erfolgt die Wiederherstellung ***nie*** in einem *vorhandenen* Konto. Sie müssen immer ein neues Azure Cosmos DB-Konto erstellen.

    > &#128221; Sie hätten sich zwar dafür entscheiden können, die gesamte Datenbank oder sogar das gesamte Konto wiederherzustellen, doch könnte die Datenbank in einer echten Produktionsumgebung sehr groß sein. In vielen Szenarien kann es schneller gehen, nur die Container oder die benötigten Datenbanken wiederherzustellen.

1. Diese Wiederherstellung kann 15 Minuten oder länger dauern. Gehen Sie zum nächsten Abschnitt, und lassen Sie diese Wiederherstellung im Hintergrund laufen.

## Löschen des Containers „customer“

1. Führen Sie im **Daten-Explorer** die folgende Abfrage aus, um das aktuelle Datum und die aktuelle Uhrzeit abzurufen. Kopieren Sie diesen Zeitstempel in den Editor.

    ```
    SELECT GetCurrentDateTime ()
    ```

1. Löschen Sie den Container **customer**.

## Wiederherstellen der Datenbank an dem Punkt, bevor das salesOrder-Dokument gelöscht wurde

1. Navigieren Sie im Azure-Portal zur Seite mit Ihrem Azure Cosmos DB-Konto.

1. Wählen Sie im Abschnitt *Einstellungen* die Option **Point-in-Time-Wiederherstellung** aus. Verwenden Sie folgende Einstellungen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Location** | *Wählen Sie einen verfügbaren Speicherort aus*. |
    | **Wiederherstellungspunkt (UTC)** | Konvertieren Sie das Datum und die Uhrzeit entsprechend. Die Uhrzeit muss im AM/PM-Format angegeben werden.|
    | **Wählen Sie die Ressourcen aus, die Sie wiederherstellen möchten**. | *Ausgewählte Datenbank/ausgewählter Container* |
    | **Wiederherstellungsressource** | *`customer`* |
    | **Wiederherstellungszielkonto** | *wählen Sie einen* ***neuen*** *Azure Cosmos DB-Kontonamen aus* |

    > &#128221; Bei Azure Cosmos DB-Wiederherstellungen erfolgt die Wiederherstellung ***nie*** in einem *vorhandenen* Konto. Sie müssen immer ein neues Azure Cosmos DB-Konto erstellen.

    > &#128221; Sie hätten sich zwar dafür entscheiden können, die gesamte Datenbank oder sogar das gesamte Konto wiederherzustellen, doch könnten die Datenbanken in einer echten Produktionsumgebung sehr groß sein. In vielen Szenarien kann es schneller gehen, nur die Container oder die benötigten Datenbanken wiederherzustellen.

1. Diese Wiederherstellung kann 15 Minuten oder länger dauern. Gehen Sie zum nächsten Abschnitt, und lassen Sie diese Wiederherstellung im Hintergrund laufen.

## Überprüfen der wiederhergestellten Daten

Wiederherstellungen können je nach Größe der Datenbank und anderen Faktoren sehr viel Zeit in Anspruch nehmen. Sobald die Wiederherstellung des Azure Cosmos DB-Kontos abgeschlossen ist, führen Sie die folgenden Schritte aus:

1. Vergewissern Sie sich bei der erste Wiederherstellung, ob das dritte Dokument wiederhergestellt wurde.

1. Bei der zweiten Wiederherstellung sollten wir die Kundentabelle wiederhergestellt haben.

## Bereinigung

1. Löschen Sie die beiden neuen Azure Cosmos DB-Konten, die von den Kontowiederherstellungen erstellt wurden.

1. Löschen Sie die Datenbank „Sales“, und löschen Sie bei Bedarf das ursprüngliche Azure Cosmos DB-Konto.

[/azure/cosmos-db/continuous-backup-restore-introduction]:https://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction

