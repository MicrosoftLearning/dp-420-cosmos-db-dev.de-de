---
lab:
  title: Anpassen des bereitgestellten Durchsatzes mithilfe eines Azure CLI-Skripts
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Anpassen des bereitgestellten Durchsatzes mithilfe eines Azure CLI-Skripts

Die Azure-Befehlszeilenschnittstelle setzt sich aus einer Reihe von Befehlen zusammen, mit denen Sie verschiedene Ressourcen in Azure verwalten können. Azure Cosmos DB verfügt über eine umfangreiche Befehlsgruppe, mit der verschiedene Facetten eines Azure Cosmos DB-Kontos verwaltet werden können, unabhängig von der ausgewählten API.

In diesem Lab erstellen Sie ein Azure Cosmos DB-Konto, eine Datenbank und einen Container mithilfe der Azure-Befehlszeilenschnittstelle. Anschließend nehmen Sie Anpassungen am bereitgestellten Durchsatz mithilfe der Azure-Befehlszeilenschnittstelle vor.

## Anmelden bei der Azure CLI

Bevor Sie die Azure-Befehlszeilenschnittstelle verwenden, müssen Sie zuerst ihre Version überprüfen und sich mit Ihren Azure-Anmeldeinformationen anmelden.

1. Starten Sie **Visual Studio Code**.

1. Öffnen Sie das Menü **Terminal**, und wählen Sie dann **Neues Terminal** aus, um eine neue Terminalinstanz zu öffnen.

1. Zeigen Sie die Version der Azure CLI mithilfe des folgenden Befehls an:

    ```
    az --version
    ```

1. Zeigen Sie die am häufigsten verwendeten Azure CLI-Befehlsgruppen mithilfe des folgenden Befehls an:

    ```
    az --help
    ```

1. Installieren Sie die TLS/SSL-Zertifikate, bevor Sie sich bei Azure anmelden:

    ```
    CD "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    ```

1. Beginnen Sie mit dem folgenden Befehl die interaktive Anmeldeprozedur für die Azure CLI:

    ```
    az login
    ```

1. Die Azure CLI öffnet automatisch ein Webbrowserfenster oder ein Tab. Melden Sie sich in der Browserinstanz mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, bei der Azure CLI an.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

1. Überprüfen Sie, ob Ihr Lab-Anbieter eine Ressourcengruppe für Sie erstellt hat. Falls ja schreiben Sie ihren Namen auf, da Sie diesen im nächsten Abschnitt benötigen.

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    Dieser Befehl kann mehrere Ressourcengruppennamen zurückgeben.

1. (Optional) ***Wenn keine Ressourcengruppe für Sie erstellt wurde***, wählen Sie einen Ressourcengruppennamen aus, und erstellen Sie eine. *Beachten Sie, dass einige Lab-Umgebungen möglicherweise gesperrt sind und Sie einen Admin benötigen, um die Ressourcengruppe für Sie zu erstellen.*

    i. Abrufen des Namens Ihres Standorts, der Ihnen am nächsten liegt, aus dieser Liste

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. Erstellen Sie die Ressourcengruppe.  *Beachten Sie, dass einige Lab-Umgebungen möglicherweise gesperrt sind und Sie einen Admin benötigen, um die Ressourcengruppe für Sie zu erstellen.*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

## Erstellen eines Azure Cosmos DB-Kontos mithilfe der Azure-Befehlszeilenschnittstelle

Die Befehlsgruppe **cosmosdb** enthält grundlegende Befehle zum Erstellen und Verwalten von Azure Cosmos DB-Konten mithilfe der Befehlszeilenschnittstelle. Da ein Azure Cosmos DB-Konto über einen adressierbaren URI verfügt, ist es wichtig, einen global eindeutigen Namen für Ihr neues Konto zu erstellen, auch wenn Sie es über ein Skript erstellen.

1. Kehren Sie zu der Terminalinstanz zurück, die bereits in **Visual Studio Code** geöffnet ist.

1. Zeigen Sie die häufigsten Azure CLI-Befehle im Zusammenhang mit **Azure Cosmos DB** mithilfe des folgenden Befehls an:

    ```
    az cosmosdb --help
    ```

1. Erstellen Sie eine neue Variable mit dem Namen **suffix** mit dem PowerShell-Cmdlet [Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] mithilfe des folgenden Befehls:

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; Das Cmdlet Get-Random generiert einen zufälligen Integer zwischen 0 und 1.000.000. Dies ist nützlich, da die verwendeten Dienste einen global eindeutigen Namen erfordern.

1. Erstellen Sie mithilfe des folgenden Befehls einen weiteren neuen Variablennamen, **accountName**, mithilfe der hartcodierten Zeichenfolge **csms** und Variablenersetzung, um den Wert der Variable **$suffix** zu injizieren:

    ```
    $accountName="csms$suffix"
    ```

1. Erstellen Sie einen weiteren neuen Variablennamen, **resourceGroup**, mithilfe des Namens der Ressourcengruppe, die Sie zuvor in diesem Lab erstellt oder angezeigt haben, mithilfe des folgenden Befehls:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Wenn Ihre Ressourcengruppe beispielsweise **dp420** heißt, ist der Befehl **$resourceGroup="dp420"**.

1. Verwenden Sie das Cmdlet **echo**, um den Wert der Variablen **$accountName** und **$resourceGroup** mithilfe des folgenden Befehls in die Terminalausgabe zu schreiben:

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. Zeigen Sie die Optionen für **az cosmosdb create** mithilfe des folgenden Befehls an:

    ```
    az cosmosdb create --help
    ```

1. Erstellen Sie ein neues Azure Cosmos DB-Konto mit den vordefinierten Variablen und dem folgenden Befehl:

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. Warten Sie, bis die Ausführung des Befehls **create** abgeschlossen wurde und eine Rückgabe erfolgt ist, bevor Sie mit diesem Lab fortfahren.

    > &#128161; Der Befehl **create** kann bis zum Abschluss im Durchschnitt zwischen zwei und zwölf Minuten dauern.

## Erstellen von Azure Cosmos DB for NoSQL-Ressourcen mit der Azure-Befehlszeilenschnittstelle

Die Befehlsgruppe **cosmosdb sql** enthält Befehle zum Verwalten von NoSQL-API-spezifischen Ressourcen für Azure Cosmos DB. Sie können jederzeit das Flag **--help** verwenden, um die Optionen für diese Befehlsgruppen zu überprüfen.

1. Kehren Sie zu der Terminalinstanz zurück, die bereits in **Visual Studio Code** geöffnet ist.

1. Zeigen Sie die häufigsten Azure CLI-Befehlsgruppen im Zusammenhang mit **Azure Cosmos DB for NoSQL** mithilfe des folgenden Befehls an:

    ```
    az cosmosdb sql --help
    ```

1. Zeigen Sie die Azure CLI-Befehle zum Verwalten von **Azure Cosmos DB for NoSQL**-Datenbanken mithilfe des folgenden Befehls an:

    ```
    az cosmosdb sql database --help
    ```

1. Erstellen Sie eine neue Azure Cosmos DB-Datenbank mit den vordefinierten Variablen, dem Datenbanknamen **cosmicworks** und dem folgenden Befehl:

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Warten Sie, bis die Ausführung des Befehls **create** abgeschlossen wurde und eine Rückgabe erfolgt ist, bevor Sie mit diesem Lab fortfahren.

1. Zeigen Sie die Azure CLI-Befehle zum Verwalten von **Azure Cosmos DB for NoSQL**-Containern mithilfe des folgenden Befehls an:

    ```
    az cosmosdb sql container --help
    ```

1. Erstellen Sie einen neuen Azure Cosmos DB-Container mit den vordefinierten Variablen, dem Datenbanknamen **cosmicworks**, dem Containernamen **products** und dem folgenden Befehl:

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Warten Sie, bis die Ausführung des Befehls **create** abgeschlossen wurde und eine Rückgabe erfolgt ist, bevor Sie mit diesem Lab fortfahren.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **Ressourcengruppen** und dann die Ressourcengruppe aus, die Sie zuvor in diesem Lab erstellt oder angezeigt haben. Wählen Sie dann die **Azure Cosmos DB**-Kontoressource mit dem Präfix **csms** aus, die Sie in diesem Lab erstellt haben.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie den neuen Containerknoten **products** in der **NoSQL API**-Navigationsstruktur.

1. Wählen Sie den Containerknoten **products** in der **NoSQL API** Navigationsstruktur aus, und wählen Sie dann **Skalieren & Einstellungen** aus.

1. Beachten Sie die Werte auf der Registerkarte **Skalieren**. Beachten Sie insbesondere, dass die Option **Manuell** im Abschnitt **Durchsatz** ausgewählt ist und dass der bereitgestellte Durchsatz auf **400** RU/s festgelegt ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

## Anpassen des Durchsatzes eines vorhandenen Containers mithilfe der Azure-Befehlszeilenschnittstelle

Die Azure-Befehlszeilenschnittstelle kann verwendet werden, um einen Container zwischen der manuellen und automatischen Bereitstellung des Durchsatzes zu migrieren. Wenn der Container den Durchsatz mit Autoskalierung verwendet, kann die Befehlszeilenschnittstelle verwendet werden, um den maximal zulässigen Durchsatzwert dynamisch anzupassen.

1. Kehren Sie zu der Terminalinstanz zurück, die bereits in **Visual Studio Code** geöffnet ist.

1. Zeigen Sie die Azure CLI-Befehle zum Verwalten des **Azure Cosmos DB for NoSQL**-Containerdurchsatzes mithilfe des folgenden Befehls an:

    ```
    az cosmosdb sql container throughput --help
    ```

1. Migrieren Sie den Containerdurchsatz für **products** von der manuellen Bereitstellung zur Autoskalierung mithilfe des folgenden Befehls:

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Warten Sie, bis die Ausführung des Befehls **migrate** abgeschlossen wurde und eine Rückgabe erfolgt ist, bevor Sie mit diesem Lab fortfahren.

1. Fragen Sie den Container **products** ab, um den minimal möglichen Durchsatzwert mithilfe des folgenden Befehls zu ermitteln:

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Aktualisieren Sie den maximalen Durchsatz mit Autoskalierung des Containers **products** vom aktuellen Standardwert **1.000** auf den neuen Wert **5.000** mit dem folgenden Befehl:

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. Warten Sie, bis die Ausführung des Befehls **update** abgeschlossen wurde und eine Rückgabe erfolgt ist, bevor Sie mit diesem Lab fortfahren.

1. Schließen Sie **Visual Studio Code**.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **Ressourcengruppen** und dann die Ressourcengruppe aus, die Sie zuvor in diesem Lab erstellt oder angezeigt haben. Wählen Sie dann die **Azure Cosmos DB**-Kontoressource mit dem Präfix **csms** aus, die Sie in diesem Lab erstellt haben.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie den neuen Containerknoten **products** in der **NoSQL API**-Navigationsstruktur.

1. Wählen Sie den Containerknoten **products** in der **NoSQL API** Navigationsstruktur aus, und wählen Sie dann **Skalieren & Einstellungen** aus.

1. Beachten Sie die Werte auf der Registerkarte **Skalieren**. Beachten Sie insbesondere, dass die Option **Autoskalierung** im Abschnitt **Durchsatz** ausgewählt ist und dass der bereitgestellte Durchsatz auf **5.000** RU/s festgelegt ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
