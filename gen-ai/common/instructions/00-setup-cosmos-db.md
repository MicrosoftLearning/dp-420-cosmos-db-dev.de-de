---
title: Einrichten von Azure Cosmos DB
lab:
  title: Einrichten von Azure Cosmos DB
  module: Setup
layout: default
nav_order: 3
parent: Common setup instructions
---

# Einrichten von Azure Cosmos DB

In dieser Übung erstellen Sie ein Azure Cosmos DB for NoSQL-Konto, das Sie in den Lab-Modulen verwenden werden, und gewähren Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen. Dadurch können Sie die Azure-Authentifizierung für den Zugriff auf die Datenbank von Ihrem Labcode aus nutzen und müssen keine Schlüssel speichern und verwalten.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Wenn Sie ein Azure Cosmos DB-Konto zum ersten Mal einrichten, wählen Sie aus, welche APIs das Konto unterstützen soll. Sobald die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen und sie verwenden, um sich mit dem Azure SDK für Python oder einem anderen SDK Ihrer Wahl mit dem Azure Cosmos DB for NoSQL-Konto zu verbinden.

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
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die erforderlich sind, um vom SDK aus eine Verbindung mit dem Konto herzustellen. Speziell:

    1. Kopieren Sie das **URI**-Feld und speichern Sie es für später in einem Text-Editor. Sie verwenden diesen **Endpunktwert** später in dieser Übung.

1. Lassen Sie die Browser-Registerkarte für den nächsten Schritt geöffnet.

## Geben Sie Ihrer Benutzeridentität die RBAC-Rolle Cosmos DB Built-in Data Contributor

Als letzte Aufgabe in dieser Übung werden Sie Ihrer Microsoft Entra ID-Benutzeridentität Zugriff auf die Verwaltung von Daten in Ihrem Azure Cosmos DB for NoSQL-Konto gewähren, indem Sie ihr die RBAC-Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen. So können Sie die Azure-Authentifizierung nutzen, um von Ihrem Code aus auf die Datenbank zuzugreifen, und müssen keine Schlüssel speichern und verwalten.

> &#128221; Die Verwendung der rollenbasierten Zugriffskontrolle (RBAC) von Microsoft Entra ID für die Authentifizierung gegenüber Azure-Diensten wie Azure Cosmos DB bietet mehrere primäre Vorteile gegenüber schlüsselbasierten Methoden. Entra ID RBAC erhöht die Sicherheit durch präzise, auf die Benutzerrollen zugeschnittene Zugriffskontrollen und reduziert so effektiv das Risiko eines unberechtigten Zugriffs. Außerdem wird die Benutzerverwaltung rationalisiert, indem Administrierende in die Lage versetzt werden, Berechtigungen dynamisch zuzuweisen und zu ändern, ohne dass sie kryptografische Schlüssel verteilen und verwalten müssen. Darüber hinaus verbessert dieser Ansatz die Compliance und die Überprüfbarkeit, da er mit den Unternehmensrichtlinien übereinstimmt und eine umfassende Zugriffsüberwachung und -überprüfung ermöglicht. Durch die Rationalisierung der sicheren Zugriffsverwaltung bietet Entra ID RBAC eine effizientere und skalierbare Lösung für die Nutzung von Azure-Diensten.

1. Öffnen Sie in der Symbolleiste des [Azure-Portals](https://portal.azure.com) eine Cloud Shell.

    ![Das Cloud Shell-Symbol wird in der Symbolleiste des Azure-Portals hervorgehoben.](media/azure-portal-toolbar-cloud-shell.png)

1. Stellen Sie an der Cloud Shell-Eingabeaufforderung sicher, dass Ihr Übungsabonnement für nachfolgende Befehle verwendet wird, indem Sie `az account set -s <SUBSCRIPTION_ID>` ausführen und dabei das Platzhalter-Token `<SUBSCRIPTION_ID>` durch die ID des Abonnements ersetzen, das Sie für diese Übung verwenden.

1. Kopieren Sie die Ausgabe des obigen Befehls zur Verwendung als `<PRINCIPAL_OBJECT_ID>`-Token im folgenden `az cosmosdb sql role assignment create`-Befehl.

1. Als nächstes werden Sie die Definitions-ID der Rolle **Cosmos DB Built-in Data Contributor** abrufen. Führen Sie den folgenden Befehl aus und ersetzen Sie dabei die Token `<RESOURCE_GROUP_NAME>` und `<COSMOS_DB_ACCOUNT_NAME>`.

    ```bash
    az cosmosdb sql role definition list --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>"
    ```

    Überprüfen Sie die Ausgabe, und suchen Sie die Rollendefinition namens **Integrierter Mitwirkender an Cosmos DB-Daten**. Die Ausgabe enthält den eindeutigen Bezeichner der Rollendefinition in der `name`-Eigenschaft. Notieren Sie diesen Wert, da er für den Zuweisungsschritt im nächsten Schritt benötigt wird.

1. Sie sind nun bereit, sich der Rollendefinition **Cosmos DB Built-in Data Contributor** zuzuweisen. Geben Sie den folgenden Befehl an der Eingabeaufforderung ein und achten Sie darauf, die Token `<RESOURCE_GROUP_NAME>` und `<COSMOS_DB_ACCOUNT_NAME>` zu ersetzen.

    > &#128221; In dem unten stehenden Befehl wird `role-definition-id` auf `00000000-0000-0000-0000-000000000002` gesetzt, was der Standardwert für die Rollendefinition **Cosmos DB Built-in Data Contributor** ist. Wenn der Wert, den Sie aus dem Befehl `az cosmosdb sql role definition list` abgerufen haben, abweicht, ersetzen Sie den Wert im folgenden Befehl vor der Ausführung. Der Befehl `az ad signed-in-user show` ruft die Objekt-ID des angemeldeten Entra-ID-Benutzenden ab.

    ```bash
    az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
    ```

1. Wenn der Befehl ausgeführt wird, können Sie lokal Code ausführen, um Daten in Ihre Cosmos DB NoSQL-Datenbank einzufügen und mit ihnen zu interagieren.

1. Schließen Sie die Cloud Shell.
