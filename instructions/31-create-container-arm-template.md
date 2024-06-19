---
lab:
  title: "Erstellen eines Azure Cosmos\_DB for NoSQL-Containers mithilfe von Azure\_Resource\_Manager-Vorlagen"
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# Erstellen eines Azure Cosmos DB for NoSQL-Containers mithilfe von Azure Resource Manager-Vorlagen

Azure Resource Manager-Vorlagen sind JSON-Dateien, die die Infrastruktur, die Sie auf Azure bereitstellen wollen, deklarativ definieren. Azure Resource Manager-Vorlagen sind eine gängige Infrastruktur-as-Code-Lösung zum Bereitstellen von Diensten in Azure. Bicep geht noch einen Schritt weiter, indem es eine einfacher zu lesende domänenspezifische Sprache definiert, die zur Erstellung von JSON-Vorlagen verwendet werden kann.

In diesem Lab erstellen Sie ein neues Azure Cosmos DB-Konto, eine Datenbank und einen Container mithilfe einer Azure Resource Manager-Vorlage. Zunächst erstellen Sie die Vorlage aus rohem JSON, dann erstellen Sie die Vorlage mit der domänenspezifischen Sprache von Bicep.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALTTASTE+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

## Azure Cosmos DB for NoSQL-Ressourcen mit Azure Resource Manager-Vorlagen erstellen

Der Ressourcenanbieter **Microsoft.DocumentDB** im Azure Resource Manager ermöglicht die Bereitstellung von Konten, Datenbanken und Containern mithilfe von JSON-Dateien. Die Dateien mögen zwar komplex sein, aber sie folgen einem vorhersehbaren Format und können mit Hilfe einer Visual Studio Code-Erweiterung geschrieben werden.

> &#128161; Wenn Sie nicht weiterkommen und einen Syntaxfehler in Ihrer Vorlage nicht nachvollziehen können, verwenden Sie diese [Azure Resource Manager-Lösungsvorlage][github.com/arm-template-guide] als Leitfaden.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **31-create-container-arm-template**.

1. Öffnen Sie die Datei **deploy.json**.

1. Sehen Sie sich die leere Azure Resource Manager-Vorlage an:

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. Fügen Sie in das Array **resources** ein neues JSON-Objekt ein, um ein neues Azure Cosmos DB-Konto zu erstellen:

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    Das Objekt ist mit den folgenden Einstellungen konfiguriert:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Ressourcentyp** | *Microsoft.DocumentDB/databaseAccounts* |
    | **API-Version** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *eindeutige Zeichenfolge, die aus dem Kontonamen generiert wird*  |
    | **Location** | *Der aktuelle Speicherort der Ressourcengruppe* |
    | **Kontoangebotstyp** | *Standard* |
    | **Standorte** | *Nur USA, Westen* |

1. Speichern Sie die Datei **deploy.json**.

1. Öffnen Sie das Kontextmenü für den Ordner **31-create-container-arm-template**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **31-create-container-arm-template** festgelegt ist.

1. Installieren Sie die TLS/SSL-Zertifikate, bevor Sie sich bei Azure anmelden:

    ```
    $CurrentDirectory=$pwd
    CD "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    CD $CurrentDirectory
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

1. Erstellen Sie einen neuen Variablennamen **resourceGroup** mithilfe des Namens der Ressourcengruppe, die Sie zuvor in diesem Lab erstellt oder angezeigt haben, mithilfe des folgenden Befehls:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Wenn Ihre Ressourcengruppe beispielsweise **dp420** heißt, ist der Befehl **$resourceGroup="dp420"**.

1. Verwenden Sie das Cmdlet **echo**, um den Wert der Variable **$resourceGroup** mithilfe des folgenden Befehls in die Terminalausgabe zu schreiben:

    ```
    echo $resourceGroup
    ```

1. Stellen Sie die Azure Resource Manager-Vorlage mit dem Befehl [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] bereit:

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Lassen Sie das integrierte Terminal geöffnet, und kehren Sie zum Editor für die Datei **deploy.json** zurück.

1. Fügen Sie in das Array **resources** ein weiteres neues JSON-Objekt ein, um eine neue Azure Cosmos DB for NoSQL-Datenbank zu erstellen:

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    Das Objekt ist mit den folgenden Einstellungen konfiguriert:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Ressourcentyp** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API-Version** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *eindeutige Zeichenfolge, die aus dem Kontonamen generiert wird* &amp; */cosmicworks*  |
    | **Resource id** | *cosmicworks* |
    | **Abhängigkeiten** | *databaseAccount, der zuvor in der Vorlage erstellt wurde* |

1. Speichern Sie die Datei **deploy.json**.

1. Kehren Sie zum integrierten Terminal zurück.

1. Stellen Sie die Azure Resource Manager-Vorlage mit dem Befehl **az deployment group create** bereit:

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Lassen Sie das integrierte Terminal geöffnet, und kehren Sie zum Editor für die Datei **deploy.json** zurück.

1. Fügen Sie in das Array **resources** ein weiteres neues JSON-Objekt ein, um einen neuen Azure Cosmos DB for NoSQL-Container zu erstellen:

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    Das Objekt ist mit den folgenden Einstellungen konfiguriert:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Ressourcentyp** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **API-Version** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *eindeutige Zeichenfolge, die aus dem Kontonamen generiert wird* &amp; */cosmicworks/products*  |
    | **Resource id** | *products* |
    | **Durchsatz** | *400* |
    | **Partitionsschlüssel** | */categoryId* |
    | **Abhängigkeiten** | *Konto und Datenbank, die zuvor in der Vorlage erstellt wurden* |

1. Speichern Sie die Datei **deploy.json**.

1. Kehren Sie zum integrierten Terminal zurück.

1. Stellen Sie die letzte Azure Resource Manager-Vorlage mit dem Befehl **az deployment group create** bereit:

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. Schließen Sie das integrierte Terminal.

## Sehen Sie sich die bereitgestellten Azure Cosmos DB-Ressourcen an

Sobald Ihre Azure Cosmos DB for NoSQL-Ressourcen bereitgestellt wurde, können Sie im Azure-Portal zu den Ressourcen navigieren. Mit dem Daten-Explorer überprüfen Sie, ob das Konto, die Datenbank und der Container ordnungsgemäß bereitgestellt und konfiguriert wurden.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **Ressourcengruppen** aus, und wählen Sie dann die Ressourcengruppe aus, die Sie zuvor in diesem Lab erstellt oder angezeigt haben. Wählen Sie dann diev**Azure Cosmos DB-Kontoressource** aus mit dem Präfix **csmsarm**, die Sie in diesem Lab erstellt haben.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie den neuen Containerknoten **products** in der **NoSQL API**-Navigationsstruktur.

1. Wählen Sie den Containerknoten **products** in der **NoSQL API** Navigationsstruktur aus, und wählen Sie dann **Skalieren & Einstellungen** aus.

1. Beobachten Sie die Werte im Abschnitt **Skalieren**. Beachten Sie insbesondere, dass die Option **Manuell** im Abschnitt **Durchsatz** ausgewählt ist und dass der bereitgestellte Durchsatz auf **400** RU/s festgelegt ist.

1. Beobachten Sie die Werte im Abschnitt **Einstellungen**. Beachten Sie insbesondere, dass der Wert **Partitionsschlüssel** auf **/categoryId** festgelegt ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

## Azure Cosmos DB for NoSQL-Ressourcen mit Bicep-Vorlagen erstellen

Bicep ist eine effiziente domänenspezifische Sprache, die die Bereitstellung von Azure-Ressourcen einfacher und leichter macht als Azure Resource Manager-Vorlagen. Sie werden genau die gleiche Ressource mit Bicep und einem anderen Namen bereitstellen, um die Unterschied\[e\] zu verdeutlichen.

> &#128161; Wenn Sie nicht weiterkommen und einen Syntaxfehler in Ihrer Vorlage nicht nachvollziehen können, verwenden Sie diese [Bicep-Lösungsvorlage][github.com/bicep-template-guide] als Leitfaden.

1. Navigieren Sie in **Visual Studio Code** im Bereich **Explorer** zum Ordner **31-create-container-arm-template**.

1. Öffnen Sie die leere Datei **deploy.bicep**.

1. Fügen Sie in die Datei ein neues Objekt ein, um ein neues Azure Cosmos DB-Konto zu erstellen:

    ```
    param location string = resourceGroup().location
    
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    Das Objekt ist mit den folgenden Einstellungen konfiguriert:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Alias** | *Konto* |
    | **Name** | *csmsarm* &amp; *eindeutige Zeichenfolge, die aus dem Kontonamen generiert wird* |
    | **Ressourcentyp** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API-Version** | *2021-05-15* |
    | **Location** | *Der aktuelle Speicherort der Ressourcengruppe* |
    | **Kontoangebotstyp** | *Standard* |
    | **Standorte** | *Nur USA, Westen* |

1. Speichern Sie die Datei **deploy.bicep**.

1. Öffnen Sie das Kontextmenü für den Ordner **31-create-container-arm-template**, und wählen Sie **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Erstellen Sie einen neuen Variablennamen **resourceGroup** mithilfe des Namens der Ressourcengruppe, die Sie zuvor in diesem Lab erstellt oder angezeigt haben, mithilfe des folgenden Befehls:

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; Wenn Ihre Ressourcengruppe beispielsweise **dp420** heißt, ist der Befehl **$resourceGroup="dp420"**.

1. Stellen Sie die Bicep-Vorlage mit dem Befehl **az deployment group create** bereit:

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Lassen Sie das integrierte Terminal geöffnet, und kehren Sie zum Editor für die Datei **deploy.bicep** zurück.

1. Fügen Sie in die Datei ein neues Objekt ein, um eine neue Azure Cosmos DB-Datenbank zu erstellen:

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    Das Objekt ist mit den folgenden Einstellungen konfiguriert:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Parent** | *Konto, das zuvor in der Vorlage erstellt wurde* |
    | **Alias** | *Datenbank* |
    | **Name** | *cosmicworks*  |
    | **Ressourcentyp** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API-Version** | *2021-05-15* |
    | **Resource id** | *cosmicworks* |

1. Speichern Sie die Datei **deploy.bicep**.

1. Kehren Sie zum integrierten Terminal zurück.

1. Stellen Sie die Bicep-Vorlage mit dem Befehl **az deployment group create** bereit:

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Lassen Sie das integrierte Terminal geöffnet, und kehren Sie zum Editor für die Datei **deploy.bicep** zurück.

1. Fügen Sie in die Datei ein neues Objekt ein, um einen neuen Azure Cosmos DB-Container zu erstellen:

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    Das Objekt ist mit den folgenden Einstellungen konfiguriert:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Parent** | *Datenbank, die zuvor in der Vorlage erstellt wurde* |
    | **Alias** | *Container* |
    | **Name** | *products*  |
    | **Resource id** | *products* |
    | **Durchsatz** | *400* |
    | **Partitionsschlüsselpfad** | */categoryId* |

1. Speichern Sie die Datei **deploy.bicep**.

1. Kehren Sie zum integrierten Terminal zurück.

1. Stellen Sie die letzte Bicep-Vorlage mit dem Befehl **az deployment group create** bereit:

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

## Sehen Sie sich die Ergebnisse der Bicep-Vorlagenbereitstellung an

Bicep-Bereitstellungen können mit vielen der gleichen Techniken validiert werden wie Azure Resource Manager-Bereitstellungen. Sie überprüfen nicht nur, ob Ihr Konto, Ihre Datenbank und Ihr Container erfolgreich bereitgestellt wurden, sondern sehen auch den Verlauf der Bereitstellung aller sechs Bereitstellungen.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie die Option **Ressourcengruppen**, und wählen Sie dann die Ressourcengruppe, die Sie zuvor in diesem Lab erstellt oder angesehen haben.

1. Navigieren Sie innerhalb der Ressourcengruppe zum Bereich **Bereitstellungen**.

1. Sehen Sie sich die sechs Bereitstellungen aus den Azure Resource Manager-Vorlagen und Bicep-Dateien an.

1. Navigieren Sie noch innerhalb der Ressourcengruppe zum Bereich **Übersicht**.

1. Wählen Sie weiterhin innerhalb der Ressourcengruppe die **Azure Cosmos DB-Kontoressource** aus mit dem Präfix **csmsbicep**, die Sie in diesem Lab erstellt haben.

1. Navigieren Sie in der **Azure Cosmos DB**-Kontoressource zum Bereich **Daten-Explorer**.

1. Erweitern Sie im **Daten-Explorer** den Datenbankknoten **cosmicworks**, und beachten Sie den neuen Containerknoten **products** in der **NoSQL API**-Navigationsstruktur.

1. Wählen Sie den Containerknoten **products** in der **NoSQL API** Navigationsstruktur aus, und wählen Sie dann **Skalieren & Einstellungen** aus.

1. Beobachten Sie die Werte im Abschnitt **Skalieren**. Beachten Sie insbesondere, dass die Option **Manuell** im Abschnitt **Durchsatz** ausgewählt ist und dass der bereitgestellte Durchsatz auf **400** RU/s festgelegt ist.

1. Beobachten Sie die Werte im Abschnitt **Einstellungen**. Beachten Sie insbesondere, dass der Wert **Partitionsschlüssel** auf **/categoryId** festgelegt ist.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
