---
lab:
  title: 07.1 - Aktivieren der Vektorsuche für Azure Cosmos DB for NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Aktivieren der Vektorsuche für Azure Cosmos DB for NoSQL

Azure Cosmos DB for NoSQL bietet eine effiziente Vektorindizierung und Suchfunktion, die entwickelt wurde, um hochdimensionale Vektoren effizient und genau in beliebiger Größenordnung zu speichern und abzufragen. Um diese Fähigkeit zu nutzen, müssen Sie Ihr Konto aktivieren, um das Feature *Vektorsuche für NoSQL API*zu verwenden.

In diesem Lab werden Sie ein Azure Cosmos DB for NoSQL-Konto erstellen und die Vektorsuche-Funktion aktivieren, um eine Datenbank für die Verwendung als Vektorspeicher vorzubereiten.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Erstellen von Copilots mit Azure Cosmos DB** noch nicht geklont und Ihre lokale Umgebung noch nicht eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#enable-vector-search-for-nosql-api) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Aktivieren der Vektorsuche für NOSQL API

In dieser Aufgabe aktivieren Sie die *Vektorsuche für das NoSQL API Feature* in Ihrem Azure Cosmos DB-Konto mithilfe der Azure CLI.

1. Öffnen Sie in der Symbolleiste des [Azure-Portals](https://portal.azure.com) eine Cloud Shell.

    ![Das Cloud Shell-Symbol wird in der Symbolleiste des Azure-Portals hervorgehoben.](media/07-azure-portal-toolbar-cloud-shell.png)

2. Stellen Sie an der Cloud Shell-Eingabeaufforderung sicher, dass Ihr Übungsabonnement für nachfolgende Befehle verwendet wird, indem Sie `az account set -s <SUBSCRIPTION_ID>` ausführen und dabei das Platzhalter-Token `<SUBSCRIPTION_ID>` durch die ID des Abonnements ersetzen, das Sie für diese Übung verwenden.

3. Aktivieren Sie das Feature *Vektorsuche für NoSQL API*, indem Sie den folgenden Befehl in der Azure Cloud Shell ausführen und dabei die Token `<RESOURCE_GROUP_NAME>` und `<COSMOS_DB_ACCOUNT_NAME>` durch den Namen Ihrer Ressourcengruppe bzw. den Namen Ihres Azure Cosmos DB-Kontos ersetzen.

     ```bash
     az cosmosdb update \
       --resource-group <RESOURCE_GROUP_NAME> \
       --name <COSMOS_DB_ACCOUNT_NAME> \
       --capabilities EnableNoSQLVectorSearch
     ```

4. Warten Sie auf die erfolgreiche Ausführung des Befehls, bevor Sie die Cloud Shell beenden.

5. Schließen Sie die Cloud Shell.

## Erstellen einer Datenbank und eines Containers zum Hosting von Vektoren

1. Wählen Sie **Data Explorer** aus dem linken Menü Ihres Azure Cosmos DB-Kontos im [Azure Portal](https://portal.azure.com) und wählen Sie dann **Neuer Container**.

2. Im Dialogfeld **Neuer Container**:
   1. Wählen Sie unter **Datenbank-ID** die Option **Neu erstellen** und geben Sie „CosmicWorks“ in das Feld Datenbank-ID ein.
   2. Geben Sie in das Feld **Container-ID** den Namen „Produkte“ ein.
   3. Weisen Sie „/category_id“ als **Partitionsschlüssel zu.**

      ![Screenshot der oben angegebenen Einstellungen für neue Container, die in den Dialog eingegeben wurden.](media/07-azure-cosmos-db-new-container.png)

   4. Scrollen Sie zum Ende des Dialogs **Neuer Container**, erweitern Sie **Container-Vektor-Richtlinie** und wählen Sie **Vektoreinbettung hinzufügen**.

   5. Legen Sie im Abschnitt **Container-Vektor-Richtlinie** folgende Einstellungen fest:

      | Einstellung | Wert |
      | ------- | ----- |
      | **Pfad** | Geben Sie */Embedding* ein. |
      | **Datentyp** | Wählen Sie *float32*. |
      | **Distance-Funktion** | Wählen Sie *Kosinus*. |
      | **Dimensionen** | Geben Sie *1536* ein, um der Anzahl der Dimensionen zu entsprechen, die das Modell `text-embedding-3-small` von OpenAI erzeugt. |
      | **Indextyp** | Wählen Sie *diskANN*. |
      | **Quantisierungsbyte-Größe** | Lassen Sie dieses Feld leer. |
      | **Indizierung der Suchlistengröße** | Nehmen Sie den Standardwert von *100*an. |

      ![Screenshot der oben angegebenen Container-Vektor-Richtlinie, die in das Dialogfeld Neuer Container eingegeben wurde.](media/07-azure-cosmos-db-container-vector-policy.png)

   6. Wählen Sie **OK** aus, um die Datenbank und den Container zu erstellen.

   7. Warten Sie, bis der Container erstellt wurde, bevor Sie fortfahren. Es kann einige Minuten dauern, bis der Container fertig ist.
