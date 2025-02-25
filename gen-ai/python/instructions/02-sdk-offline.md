---
title: 02 - Konfigurieren des Azure Cosmos DB Python SDK für die Offline-Entwicklung
lab:
  title: 02 - Konfigurieren des Azure Cosmos DB Python SDK für die Offline-Entwicklung
  module: Configure the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 5
parent: Python SDK labs
---

# Konfigurieren des Azure Cosmos DB Python SDK für die Offline-Entwicklung

Der Azure Cosmos DB-Emulator ist ein lokales Tool, das zu Entwicklungs- und Testzwecken den Azure Cosmos DB-Dienst emuliert. Der Emulator unterstützt die NoSQL-API und kann bei der Entwicklung von Code mit dem Azure SDK für Python anstelle des Clouddienstes verwendet werden.

In diesem Lab stellen Sie eine Verbindung zum Azure Cosmos DB Emulator über das Azure SDK für Python her.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Lab-Coderepository für **Copilots mit Azure Cosmos DB erstellen** noch nicht geklont und Ihre lokale Umgebung eingerichtet haben, lesen Sie dazu die Anleitung [Lokale Lab-Umgebung einrichten](00-setup-lab-environment.md).

## Ausführen des Azure Cosmos DB-Emulators

Wenn Sie eine gehostete Lab-Umgebung verwenden, sollte der Emulator dort bereits installiert sein. Andernfalls lesen Sie die [Installationsanleitung](https://docs.microsoft.com/azure/cosmos-db/local-emulator), um den Azure Cosmos DB-Emulator zu installieren. Sobald der Emulator gestartet ist, können Sie die Verbindungszeichenfolge abrufen und sie verwenden, um mit dem Azure SDK für Python eine Verbindung zum Emulator herzustellen.

> &#128161; Sie können optional den [neuen Linux-basierten Azure Cosmos DB Emulator (in der Vorschau)](https://learn.microsoft.com/azure/cosmos-db/emulator-linux) installieren, der als Docker-Container verfügbar ist. Er unterstützt die Ausführung auf einer Vielzahl von Prozessoren und Betriebssystemen.

1. Starten Sie den **Azure Cosmos DB-Emulator**.

    > 💡 Wenn Sie Windows verwenden, wird der Azure Cosmos DB Emulator sowohl an die Windows-Taskleiste als auch an das Startmenü angeheftet. Wenn er nicht über die angehefteten Symbole gestartet wird, versuchen Sie, ihn durch Doppelklick auf die Datei **C:\Programme\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** zu öffnen.

1. Warten Sie, bis der Emulator Ihren Standardbrowser öffnet, und navigieren Sie zur Landing Page **https://localhost:8081/_explorer/index.html**.

1. Beachten Sie im Bereich **Schnellstart** die **Primäre Verbindungszeichenfolge**. Sie werden diese Verbindungszeichenfolge später verwenden.

> &#128221; Manchmal wird die Landing Page nicht erfolgreich geladen, obwohl der Emulator läuft. Wenn dies geschieht, können Sie die bekannte Verbindungszeichenfolge verwenden, um eine Verbindung mit dem Emulator herzustellen. Die bekannte Verbindungszeichenfolge lautet: `AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## Installieren der Azure-Cosmos-Bibliothek

Die **azure-cosmos**-Bibliothek ist auf **PyPI** für eine einfache Installation in Ihren Python-Projekten verfügbar.

1. In **Visual Studio Code**, im Bereich **Explorer**, suchen Sie den Ordner **python/02-sdk-offline**.

1. Öffnen Sie das Kontextmenü für den Ordner **python/02-sdk-offline** und wählen Sie dann **Öffnen im integrierten Terminal**, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Dieser Befehl öffnet das Terminal, wobei das Startverzeichnis bereits auf den Ordner **python/02-sdk-offline** eingestellt ist.

1. Erstellen und aktivieren Sie eine virtuelle Umgebung, um Abhängigkeiten zu verwalten:

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. Installieren Sie das Paket [azure-cosmos][pypi.org/project/azure-cosmos] mit dem folgenden Befehl:

   ```bash
   pip install azure-cosmos
   ```

## Verbindung zum Emulator über das Python-SDK herstellen

1. In **Visual Studio Code**, im Bereich **Explorer**, suchen Sie den Ordner **python/02-sdk-offline**.

1. Öffnen Sie die leere Python-Datei mit dem Namen **script.py**.

1. Fügen Sie den folgenden Code hinzu, um eine Verbindung zum Emulator herzustellen, eine Datenbank zu erstellen und ihre ID zu drucken:

   ```python
   from azure.cosmos import CosmosClient, PartitionKey
   
   # Connection string for the Azure Cosmos DB Emulator
   connection_string = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
    
   # Initialize the Cosmos client
   client = CosmosClient.from_connection_string(connection_string)
    
   # Create a database
   database_name = "cosmicworks"
   database = client.create_database_if_not_exists(id=database_name)
    
   # Print the database ID
   print(f"New Database: Id: {database.id}")
   ```

1. **Speichern** der **script.py** Datei.

## Ausführen des Skripts

1. Verwenden Sie dasselbe Terminalfenster in **Visual Studio Code**, das Sie zum Einrichten der Python-Umgebung für dieses Lab verwendet haben. Wenn Sie es schließen, öffnen Sie das Kontextmenü für den Ordner **python/02-sdk-offline** und wählen Sie dann **Öffnen im integrierten Terminal** aus, um eine neue Terminalinstanz zu öffnen.

1. Führen Sie das Skript mithilfe des Befehls `python` aus:

   ```bash
   python script.py
   ```

1. Das Skript erstellt eine Datenbank mit dem Namen `cosmicworks` im Emulator. Die Ausgabe sollte etwa folgendermaßen aussehen:

   ```text
   New Database: Id: cosmicworks
   ```

## Erstellen und Anzeigen eines neuen Containers

Sie können das Skript erweitern, um einen Container innerhalb der Datenbank zu erstellen.

### Aktualisierter Code

1. Ändern Sie die `script.py`-Datei, um den folgenden Code am Ende der Datei für die Erstellung eines Containers hinzuzufügen:

   ```python
   # Create a container
   container_name = "products"
   partition_key_path = "/categoryId"
   throughput = 400
    
   container = database.create_container_if_not_exists(
       id=container_name,
       partition_key=PartitionKey(path=partition_key_path),
       offer_throughput=throughput
   )
    
   # Print the container ID
   print(f"New Container: Id: {container.id}")
   ```

### Führen Sie das aktualisierte Skript aus

1. Führen Sie das aktualisierte Skript mit dem folgenden Befehl aus:

   ```bash
   python script.py
   ```

1. Das Skript erstellt einen Container namens `products` im Emulator. Die Ausgabe sollte etwa folgendermaßen aussehen:

   ```text
   New Container: Id: products
   ```

### Bestätigen der Ergebnisse:

1. Wechseln Sie zu dem Browser, in dem der Data Explorer des Emulators geöffnet ist.

1. Aktualisieren Sie die **NoSQL API**, um die neue **cosmicworks**-Datenbank und den **Produkte**-Container zu sehen.

## Beenden des Azure Cosmos DB-Emulators

Es ist wichtig, den Emulator zu beenden, wenn Sie ihn nicht mehr verwenden, um Systemressourcen freizugeben. Führen Sie die folgenden Schritte für Ihr Betriebssystem aus:

### Auf macOS oder Linux:

Wenn Sie den Emulator in einem Terminalfenster gestartet haben, gehen Sie wie folgt vor:

1. Suchen Sie das Terminalfenster, in dem der Emulator ausgeführt wird.

1. Drücken Sie `Ctrl + C`, um den Emulatorprozess zu beenden.

Alternativ können Sie den Emulatorprozess auch manuell beenden:

1. Öffnen Sie ein neues Terminalfenster.

1. Verwenden Sie den folgenden Befehl, um den Emulatorprozess zu finden:

   ```bash
   ps aux | grep CosmosDB.Emulator
   ```

Identifizieren Sie die **PID** (Prozess-ID) des Emulatorprozesses in der Ausgabe. Verwenden Sie den Befehl Beenden, um den Emulatorprozess zu beenden:

```bash
kill <PID>
```

### Unter Windows:

1. Suchen Sie das Azure Cosmos DB Emulator-Symbol im Windows-Infobereich (in der Nähe der Uhr in der Taskleiste).

1. Klicken Sie mit der rechten Maustaste auf das Symbol des Emulators, um das Kontextmenü zu öffnen.

1. Wählen Sie **Beenden**, um den Emulator herunterzufahren.

> 💡 Es kann eine Minute dauern, bis alle Instanzen des Emulators beendet sind.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
