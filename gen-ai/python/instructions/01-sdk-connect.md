---
title: 01 - Stellen Sie mit dem SDK eine Verbindung zu Azure Cosmos DB für NoSQL her
lab:
  title: 01 - Stellen Sie mit dem SDK eine Verbindung zu Azure Cosmos DB für NoSQL her
  module: Use the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 4
parent: Python SDK labs
---

# Herstellen einer Verbindung mit Azure Cosmos DB for NoSQL über das SDK

Das Azure SDK für Python ist eine Suite von Client-Bibliotheken, die eine einheitliche Entwickleroberfläche für die Interaktion mit vielen Azure-Diensten bietet. Client-Bibliotheken sind Pakete, die Sie verwenden würden, um diese Ressourcen zu nutzen und mit ihnen zu interagieren.

In dieser Übung stellen Sie mithilfe des Azure SDK für Python eine Verbindung zu einem Azure Cosmos DB for NoSQL-Konto her.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Wenn Sie bereits ein Azure Cosmos DB for NoSQL-Konto für die Labs **Copilots mit Azure Cosmos DB erstellen** auf dieser Website erstellt haben, können Sie es für dieses Lab verwenden und mit dem [nächsten Abschnitt](#install-the-azure-cosmos-library) fortfahren. Andernfalls sehen Sie sich die Anweisungen zum [Einrichten von Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) an, um ein Azure Cosmos DB for NoSQL-Konto zu erstellen, das Sie in den Labmodulen verwenden werden, und gewähren Sie Ihrer Benutzeridentität Zugriff auf die Verwaltung von Daten im Konto, indem Sie ihr die Rolle **Cosmos DB integrierter Daten-Mitwirkender** zuweisen.

## Installieren der Azure-Cosmos-Bibliothek

Die **azure-cosmos**-Bibliothek ist auf **PyPI** für eine einfache Installation in Ihren Python-Projekten verfügbar.

1. Navigieren Sie in **Visual Studio Code** im Bereich **„Explorer“** zum Ordner **„python/01-sdk-connect“**.

1. Öffnen Sie das Kontextmenü für den Ordner **python/01-sdk-connect** und wählen Sie dann **„In integriertem Terminal öffnen**“, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **python/01-sdk-connect** eingestellt ist.

1. Erstellen und aktivieren Sie eine virtuelle Umgebung, um Abhängigkeiten zu verwalten:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Installieren Sie das Paket [azure-cosmos][pypi.org/project/azure-cosmos] mit dem folgenden Befehl:

   ```bash
   pip install azure-cosmos
   ```

1. Installieren Sie die [azure-identity][pypi.org/project/azure-identity]-Bibliothek, die es uns ermöglicht, die Azure-Authentifizierung zu verwenden, um sich mit dem Azure Cosmos DB-Arbeitsbereich zu verbinden, indem Sie den folgenden Befehl verwenden:n:

   ```bash
   pip install azure-identity
   ```

1. Schließen Sie das integrierte Terminal.

## Verwenden Sie die Azure-Cosmos-Bibliothek

Sobald die Azure Cosmos DB-Bibliothek aus dem Azure SDK für Python importiert wurde, können Sie deren Klassen sofort verwenden, um eine Verbindung zu einem Azure Cosmos DB for NoSQL-Konto herzustellen. Die Klasse **CosmosClient** ist die Kernklasse, die verwendet wird, um die erste Verbindung zu einem Azure Cosmos DB for NoSQL-Konto herzustellen.

1. Navigieren Sie in **Visual Studio Code** im Bereich **„Explorer“** zum Ordner **„python/01-sdk-connect“**.

1. Öffnen Sie die leere Python-Datei mit dem Namen **script.py**.

1. Fügen Sie die folgende `import` Anweisung hinzu, um die Klassen **CosmosClient** und **DefaultAzureCredential** zu importieren:

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential
   ```

1. Fügen Sie Variablen mit den Namen **Endpunkt** und **Anmeldeinformation** hinzu und setzen Sie den Wert **Endpunkt** auf den **Endpunkt** des Azure Cosmos DB-Kontos, das Sie zuvor erstellt haben. Die Variable **Anmeldeinformation** sollte auf eine neue Instanz der Klasse **DefaultAzureCredential** gesetzt werden:

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221; Zum Beispiel, wenn Ihr Endpunkt ist: **https://dp420.documents.azure.com:443/**, würde die Anweisung lauten: **Endpunkt = "https://dp420.documents.azure.com:443/"**.

1. Fügen Sie eine neue Variable namens **Client** hinzu und initialisieren Sie sie als neue Instanz der Klasse **CosmosClient** unter Verwendung der Variablen **Endpunkt** und **Anmeldeinformation**:

   ```python
   client = CosmosClient(endpoint, credential=credential)
   ```

1. Fügen Sie eine Funktion mit dem Namen **„main**“ hinzu, um Kontoeigenschaften zu lesen und zu drucken:

   ```python
   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
   ```

1. Ihre Datei**script.py** sollte nun wie folgt aussehen:

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   client = CosmosClient(endpoint, credential=credential)

   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
    ```

1. **Speichern** der **script.py** Datei.

## Testen des Skripts

Nachdem der Python-Code für die Verbindung zum Azure Cosmos DB for NoSQL-Konto vollständig ist, können Sie das Skript testen. Dieses Skript druckt die Standardkonsistenzstufe und den Namen des ersten schreibbaren Bereichs. Bei der Erstellung des Kontos haben Sie einen Speicherort angegeben und sollten erwarten, dass derselbe Speicherortwert als Ergebnis dieses Skripts angezeigt wird.

1. Öffnen Sie in **Visual Studio Code** das Kontextmenü für den Ordner **python/01-sdk-connect** und wählen Sie dann **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

1. Bevor Sie das Skript ausführen, müssen Sie sich mit dem Befehl `az login` bei Azure anmelden. Führen Sie im Fenster Terminal Folgendes aus:

   ```bash
   az login
   ```

1. Führen Sie das Skript mithilfe des Befehls `python` aus:

   ```bash
   python script.py
   ```

1. Das Skript gibt nun die Standardkonsistenzstufe und den ersten schreibbaren Bereich aus. Wenn beispielsweise die Standard-Konsistenzstufe für das Konto **„Sitzung**“ lautet und der erste beschreibbare Bereich **Ost-USA** war, würde das Skript Folgendes ausgeben:

   ```text
   Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
   Primary Region: East US
   ```

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.

[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
