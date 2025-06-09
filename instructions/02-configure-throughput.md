---
lab:
  title: "Konfigurieren des Durchsatzes für Azure Cosmos\_DB for NoSQL über das Azure-Portal"
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# Konfigurieren des Durchsatzes für Azure Cosmos DB for NoSQL über das Azure-Portal

Eines der wichtigsten Dinge, die Sie verstehen müssen, ist die Konfiguration des Durchsatzes in Azure Cosmos DB for NoSQL. Um einen Azure Cosmos DB for NoSQL-Container zu erstellen, müssen Sie zuerst ein Konto und dann eine Datenbank erstellen, in dieser Reihenfolge.

In diesem Lab stellen Sie den Durchsatz mithilfe verschiedener Methoden im Daten-Explorer bereit. Sie stellen den Durchsatz manuell oder mithilfe der Autoskalierung auf Datenbank- und Containerebene bereit.

## Erstellen eines serverlosen Kontos

Beginnen Sie einfach mit dem Erstellen eines serverlosen Kontos. Es gibt hier nicht viel zu konfigurieren, da alles serverlos ist. Wenn Sie die Datenbank und den Container erstellen, müssen Sie überhaupt keinen Durchsatz bereitstellen. Sie werden dies erkennen, während dieses Konto erstellt wird.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie in der Kategorie **Azure-Dienste** die Option **Ressource erstellen** und dann **Azure Cosmos DB** aus.

    > &#128161; Alternativ können Sie das Menü **&#8801;** erweitern und **Alle Dienste** in der Kategorie **Datenbanken**, dann **Azure Cosmos DB** und dann **Erstellen** auswählen.

1. Wählen Sie im Bereich **API-Option auswählen** die Option **Erstellen** im Abschnitt **Azure Cosmos DB for NoSQL** aus.

1. Beachten Sie im Bereich **Azure Cosmos DB-Konto erstellen** die Registerkarte **Grundlegendes**.

1. Geben Sie auf der Registerkarte **Grundlagen** die folgenden Werte für die jeweilige Einstellung ein:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Workloadtyp** | **Weiterbildung** |
    | **Abonnement** | **Verwenden Sie Ihr bereits vorhandenes Azure-Abonnement.** *Alle Ressourcen müssen einer Ressourcengruppe angehören. Jede Ressourcengruppe muss einem Abonnement angehören.* |
    | **Ressourcengruppe** | **Verwenden Sie eine vorhandene Ressourcengruppe, oder erstellen Sie eine neue.** *Alle Ressourcen müssen einer Ressourcengruppe angehören.*|
    | **Account Name** |  **Geben Sie einen global eindeutigen Namen ein.** *Der global eindeutige Name des Kontos Dieser Name wird als Teil der DNS-Adresse für Anforderungen verwendet.  Das Portal überprüft den Namen in Echtzeit.* |
    | **Location** | **Wählen Sie eine verfügbare Region aus.** *Wählen Sie die geografische Region aus, in der Ihre Datenbank anfänglich gehostet werden soll.* |
    | **Kapazitätsmodus** | **Auswählen von „Serverlos“** |

1. Wählen Sie **Überprüfen + erstellen** aus, um zur Registerkarte **Überprüfen + erstellen** zu navigieren, und wählen Sie dann **Erstellen** aus.

    > &#128221; Es kann 10–15 Minuten dauern, bis das Azure Cosmos DB for NoSQL-Konto einsatzbereit ist.

1. Sehen Sie sich den Bereich **Bereitstellung** an. Wenn die Bereitstellung abgeschlossen ist, wird der Bereich mit der Meldung **Bereitstellung erfolgreich** aktualisiert.

1. Wählen Sie immer noch im Bereich **Bereitstellung** die Option **Zu Ressource wechseln** aus.

1. Wählen Sie im Bereich **Azure Cosmos DB-Konto** im Ressourcenmenü **Daten-Explorer** aus.

1. Erweitern Sie im Bereich **Daten-Explorer** die Option **Neuer Container**, und wählen Sie dann **Neue Datenbank** aus.

1. Geben Sie im Popup **Neue Datenbank** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *`cosmicworks`* |

1. Beachten Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Vorhandene verwenden* &vert; *cosmicworks* |
    | **Container-ID** | *`products`* |
    | **Partitionsschlüssel** | *`/category/name`* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie danach den Containerknoten **products** in der Hierarchie.

1. Kehren Sie zur **Startseite** des Azure-Portals zurück.

## Erstellen eines bereitgestellten Kontos

Jetzt wird ein Konto für bereitgestellten Durchsatz mit herkömmlicheren Konfigurationsoptionen erstellt. Diese Art von Konto eröffnet zahlreiche Konfigurationsoptionen, die ein wenig überwältigend sein können. Sie werden einige Beispiele für Datenbank- und Containerpaare durchlaufen, die hier möglich sind.

1. Wählen Sie in der Kategorie **Azure-Dienste** die Option **Ressource erstellen** und dann **Azure Cosmos DB** aus.

    > &#128161; Alternativ können Sie das Menü **&#8801;** erweitern und **Alle Dienste** in der Kategorie **Datenbanken**, dann **Azure Cosmos DB** und dann **Erstellen** auswählen.

1. Wählen Sie im Bereich **API-Option auswählen** die Option **Erstellen** im Abschnitt **Azure Cosmos DB for NoSQL** aus.

1. Beachten Sie im Bereich **Azure Cosmos DB-Konto erstellen** die Registerkarte **Grundlegendes**.

1. Geben Sie auf der Registerkarte **Grundlagen** die folgenden Werte für die jeweilige Einstellung ein:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Workloadtyp** | **Weiterbildung** |
    | **Abonnement** | **Verwenden Sie Ihr bereits vorhandenes Azure-Abonnement.** *Alle Ressourcen müssen einer Ressourcengruppe angehören. Jede Ressourcengruppe muss einem Abonnement angehören.* |
    | **Ressourcengruppe** | **Verwenden Sie eine vorhandene Ressourcengruppe, oder erstellen Sie eine neue.** *Alle Ressourcen müssen einer Ressourcengruppe angehören.*|
    | **Account Name** |  **Geben Sie einen global eindeutigen Namen ein.** *Der global eindeutige Name des Kontos Dieser Name wird als Teil der DNS-Adresse für Anforderungen verwendet.  Das Portal überprüft den Namen in Echtzeit.* |
    | **Location** | **Wählen Sie eine verfügbare Region aus.** *Wählen Sie die geografische Region aus, in der Ihre Datenbank anfänglich gehostet werden soll.* |
    | **Kapazitätsmodus** | **Bereitgestellter Durchsatz** |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | **Nicht anwenden** |
    | **Begrenzen des Gesamtdurchsatzes, der für dieses Konto bereitgestellt werden kann** | **Nicht aktiviert** |

1. Wählen Sie **Überprüfen + erstellen** aus, um zur Registerkarte **Überprüfen + erstellen** zu navigieren, und wählen Sie dann **Erstellen** aus.

    > &#128221; Es kann 10–15 Minuten dauern, bis das Azure Cosmos DB for NoSQL-Konto einsatzbereit ist.

1. Sehen Sie sich den Bereich **Bereitstellung** an. Wenn die Bereitstellung abgeschlossen ist, wird der Bereich mit der Meldung **Bereitstellung erfolgreich** aktualisiert.

1. Wählen Sie immer noch im Bereich **Bereitstellung** die Option **Zu Ressource wechseln** aus.

1. Wählen Sie im Bereich **Azure Cosmos DB-Konto** im Ressourcenmenü **Daten-Explorer** aus.

1. Erweitern Sie im Bereich **Daten-Explorer** die Option **Neuer Container**, und wählen Sie dann **Neue Datenbank** aus.

1. Geben Sie im Popup **Neue Datenbank** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *`nothroughputdb`* |
    | **Durchsatz bereitstellen** | *Nicht aktiviert* |

1. Beachten Sie im Bereich **Daten-Explorer** den Datenbankknoten **nothroughputdb** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Verwenden vorhandener* &vert; *nothroughputdb* |
    | **Container-ID** | *`requiredthroughputcontainer`* |
    | **Partitionsschlüssel** | *`/primarykey`* |
    | **Containerdurchsatz** | *Manuell* |
    | **RUs/Sek.** | *`400`* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **nothroughputdb**, und beachten Sie danach den Containerknoten **requiredthroughputcontainer** in der Hierarchie.

1. Erweitern Sie im Bereich **Daten-Explorer** die Option **Neuer Container**, und wählen Sie dann **Neue Datenbank** aus.

1. Geben Sie im Popup **Neue Datenbank** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *`manualthroughputdb`* |
    | **Durchsatz bereitstellen** | *Überprüft* |
    | **Datenbank-Durchsatz** | *Manuell* |
    | **RUs/Sek.** | *`400`* |

1. Beachten Sie im Bereich **Daten-Explorer** den Datenbankknoten **manualthroughputdb** in der Hierarchie.

1. Wählen Sie im Bereich **Daten-Explorer** die Option **Neuer Container** aus.

1. Geben Sie im Pop-up **Neuer Container** die folgenden Werte für die jeweilige Einstellung ein, und wählen Sie dann **OK** aus:

    | **Einstellung** | **Wert** |
    | --: | :-- |
    | **Datenbank-ID** | *Verwenden vorhandener* &vert; *manualthroughputdb* |
    | **Container-ID** | *`childcontainer`* |
    | **Partitionsschlüssel** | *`/primarykey`* |
    | **Bereitstellen des dedizierten Durchsatzes für diesen Container** | *Überprüft* |
    | **Containerdurchsatz** | *Manuell* |
    | **RUs/Sek.** | *`1000`* |

1. Erweitern Sie im Bereich **Daten-Explorer** den Datenbankknoten **manualthroughputdb**, und beachten Sie danach den Containerknoten **childcontainer** in der Hierarchie.
