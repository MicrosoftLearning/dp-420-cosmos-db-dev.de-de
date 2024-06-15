---
lab:
  title: Erstellen eines Azure Cosmos DB for NoSQL-Kontos
  module: Module 1 - Get started with Azure Cosmos DB for NoSQL
---

# Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Bevor Sie zu tief in Azure Cosmos DB eintauchen, ist es wichtig, die Grundlagen für die Erstellung der Ressourcen, die Sie am häufigsten verwenden werden, zu verstehen. In den meisten Szenarien müssen Sie mit dem Erstellen von Konten, Datenbanken, Containern und Elementen vertraut sein. In einem realen Szenario sollten Sie auch einige grundlegende Abfragen „zur Hand“ haben, um zu prüfen, ob Sie alle Ihre Ressourcen richtig erstellt haben.

In diesem Lab erstellen Sie ein neues Azure Cosmos DB-Konto mit NoSQL. Anschließend verwenden Sie den Daten-Explorer, um eine Datenbank, einen Container und zwei Elemente zu erstellen. Schließlich fragen Sie die Datenbank nach den von Ihnen erstellten Elementen ab.

## Erstellen eines neuen Azure Cosmos DB-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Wenn Sie ein Azure Cosmos DB-Konto zum ersten Mal bereitstellen, wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**).

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie in der Kategorie **Azure-Dienste** die Option **Ressource erstellen** und dann **Azure Cosmos DB** aus.

    > &#128161; Alternativ können Sie das Menü **&#8801;** erweitern und **Alle Dienste** in der Kategorie **Datenbanken**, dann **Azure Cosmos DB** und dann **Erstellen** auswählen.

1. Wählen Sie im Bereich **API-Option auswählen** die Option **Erstellen** im Abschnitt **Azure Cosmos DB for NoSQL** aus.

1. Beachten Sie im Bereich **Azure Cosmos DB-Konto erstellen** die Registerkarte **Grundlegendes**.

1. Geben Sie auf der Registerkarte **Grundlagen** die folgenden Werte für die jeweilige Einstellung ein:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Abonnement** | **Verwenden Sie Ihr bereits vorhandenes Azure-Abonnement.** *Alle Ressourcen müssen einer Ressourcengruppe angehören. Jede Ressourcengruppe muss einem Abonnement angehören.* |
    | **Ressourcengruppe** | **Verwenden Sie eine vorhandene Ressourcengruppe, oder erstellen Sie eine neue.** *Alle Ressourcen müssen einer Ressourcengruppe angehören.* |
    | **Account Name** | **Geben Sie einen global eindeutigen Namen ein.** *Der global eindeutige Name des Kontos Dieser Name wird als Teil der DNS-Adresse für Anforderungen verwendet.  Das Portal überprüft den Namen in Echtzeit.* |
    | **Location** | **Wählen Sie eine verfügbare Region aus.** *Wählen Sie die geografische Region aus, in der Ihre Datenbank anfänglich gehostet werden soll.* |
    | **Kapazitätsmodus** | **Bereitgestellter Durchsatz** |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | **Nicht anwenden** |

1. Wählen Sie **Überprüfen + erstellen** aus, um zur Registerkarte **Überprüfen + erstellen** zu navigieren, und wählen Sie dann **Erstellen** aus.

    > &#128221; Es kann 10–15 Minuten dauern, bis das Azure Cosmos DB for NoSQL-Konto einsatzbereit ist.

1. Sehen Sie sich den Bereich **Bereitstellung** an. Wenn die Bereitstellung abgeschlossen ist, wird der Bereich mit der Meldung **Bereitstellung erfolgreich** aktualisiert.

1. Wählen Sie immer noch im Bereich **Bereitstellung** die Option **Zu Ressource wechseln** aus.

## Verwenden des Daten-Explorers, um eine neue Datenbank und einen Container zu erstellen

Der Daten-Explorer ist Ihr primäres Tool zum Verwalten der Azure Cosmos DB for NoSQL-Datenbank und -Container im Azure-Portal. Sie erstellen eine einfache Datenbank und einen Standardcontainer, die in dieser Übung verwendet werden sollen.

1. Wählen Sie im Bereich **Azure Cosmos DB-Konto** im Ressourcenmenü **Daten-Explorer** aus.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *cosmicworks* |
    | **Freigeben des Durchsatzes für mehrere Container** | *Unckecked* |
    | **Container-ID** | *products* |
    | **Partitionsschlüssel** | */categoryId* |
    | **Containerdurchsatz (Autoskalierung)** | *Manuell* |
    | **RUs/Sek.** | *400* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

## Verwenden des Daten-Explorers zum Erstellen neuer Elemente

Der Daten-Explorer enthält auch eine Reihe von Features zum Abfragen, Erstellen und Verwalten von Elementen in einem Azure Cosmos DB for NoSQL-Container. Sie erstellen zwei grundlegende Elemente mit unformatiertem JSON im Daten-Explorer.

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**. Erweitern Sie den Containerknoten **products** und wählen Sie dann **Elemente** aus.

1. Wählen Sie in der Befehlsleiste **Neues Element** aus, und ersetzen Sie im Editor das Platzhalter-JSON-Element durch den folgenden Inhalt:

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. Wählen Sie in der Befehlsleiste **Speichern** aus, um das erste JSON-Element hinzuzufügen:

1. Kehren Sie zur Registerkarte **Elemente ** zurück, und wählen Sie in der Befehlsleiste **Neues Element** aus. Ersetzen Sie im Editor das JSON-Platzhalterelement durch den folgenden Inhalt:

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. Wählen Sie in der Befehlsleiste **Speichern** aus, um das zweite JSON-Element hinzuzufügen:

1. Sehen Sie sich auf der Registerkarte **Elemente** die beiden neuen Elemente im Bereich **Elemente** an.

## Verwenden des Daten-Explorers zum Ausgeben einer einfachen Abfrage

Der Daten-Explorer verfügt über einen integrierten Abfrage-Editor, der verwendet wird, um Abfragen zu erstellen, die Ergebnisse zu beobachten und die Auswirkungen in Bezug auf Anforderungseinheiten pro Sekunde (RU/s) zu messen.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neue SQL-Abfrage** aus.

1. Wählen Sie auf der Registerkarte „Abfrage“ die Option **Abfrage ausführen** aus, um eine Standardabfrage anzuzeigen, die alle Elemente ohne Filter auswählt.

1. Löschen Sie den Inhalt des Editorbereichs.

1. Ersetzen Sie die Platzhalterabfrage durch den folgenden Inhalt:

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; Diese Abfrage wählt alle Elemente aus, bei denen der Wert von **price** größer als 500 USD ist.

1. Klicken Sie auf **Abfrage ausführen**.

1. Beobachten Sie die Ergebnisse der Abfrage, die ein einzelnes JSON-Element und alle zugehörigen Eigenschaften enthalten sollten.

1. Wählen Sie auf der Registerkarte **Abfrage** die Option **Abfragestatistiken** aus.

1. Beobachten Sie auf der Registerkarte **Abfrage** den Wert des Felds **Anforderungsgebühr** im Abschnitt **Abfragestatistik**.

    > &#128221; In der Regel liegt die Anforderungsgebühr für diese einfache Abfrage zwischen 2 und 3 RU/s, wenn der Container klein ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.
