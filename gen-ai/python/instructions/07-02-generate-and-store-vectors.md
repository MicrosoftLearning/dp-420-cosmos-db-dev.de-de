---
title: 07.2 – Generieren von Vektoreinbettungen mit Azure OpenAI und Speichern in Azure Cosmos DB für NoSQL
lab:
  title: 07.2 – Generieren von Vektoreinbettungen mit Azure OpenAI und Speichern in Azure Cosmos DB für NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 11
parent: Python SDK labs
---

# Generieren Sie Vektoreinbettungen mit Azure OpenAI und speichern Sie in Azure Cosmos DB für NoSQL

Azure OpenAI bietet Zugriff auf die fortschrittlichen Sprachmodelle von OpenAI, einschließlich der Modelle `text-embedding-ada-002`, `text-embedding-3-small` und `text-embedding-3-large`. Durch die Nutzung eines dieser Modelle können Sie Vektordarstellungen von Textdaten generieren, die in einem Vektorspeicher wie Azure Cosmos DB für NoSQL gespeichert werden können. Dies ermöglicht effiziente und genaue Ähnlichkeitssuchen und verbessert die Fähigkeit eines Copilots erheblich, relevante Informationen abzurufen und kontextreiche Interaktionen bereitzustellen.

In dieser Übung erstellen Sie einen Azure-OpenAI-Dienst und stellen ein Einbettungsmodell bereit. Anschließend erstellen Sie mithilfe von Python-Code Azure OpenAI- und Cosmos DB-Clients, die mit ihren jeweiligen Python-SDKs Vektordarstellungen von Produktbeschreibungen generieren und in Ihre Datenbank schreiben.

> &#128721; Die vorherige Übung in diesem Modul ist Voraussetzung für diese Übung. Wenn Sie diese Übung noch nicht abgeschlossen haben, beenden Sie sie bitte, bevor Sie fortfahren, da sie die für diese Übung erforderliche Infrastruktur bereitstellt.

## Erstellen eines Azure OpenAI-Diensts

Azure OpenAI bietet REST-API-Zugriff auf die leistungsstarken Sprachmodelle von OpenAI. Diese Modelle können problemlos an Ihre spezifische Aufgabe angepasst werden, einschließlich, aber nicht beschränkt auf die Erstellung von Inhalten, die Zusammenfassung, die Bildanalyse, die semantische Suche und die Übersetzung von natürlicher Sprache in Code.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

2. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

3. Wählen Sie **Ressource erstellen**, suchen Sie nach *Azure OpenAI* und erstellen Sie dann eine neue **Azure OpenAI**-Ressource mit den folgenden Einstellungen, wobei alle übrigen Einstellungen auf ihren Standardwerten belassen werden:

    | Einstellung | Wert |
    | ------- | ----- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Region** | *Wählen Sie eine verfügbare Region aus der [Liste der unterstützenden Regionen](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings), die das `text-embedding-3-small` Modell* unterstützt. |
    | **Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Tarif** | *Wählen Sie Standard 0* aus |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

4. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren.

## Bereitstellen eines Einbettungsmodells

Um Azure OpenAI zur Erstellung von Einbettungen zu verwenden, müssen Sie zunächst eine Instanz des gewünschten Einbettungsmodells in Ihrem Dienst bereitstellen.

1. Navigieren Sie zu Ihrem neu erstellten Azure OpenAI-Dienst im Azure-Portal (``portal.azure.com``).

2. Starten Sie auf der Seite **Übersicht** des Azure OpenAI Service **Azure AI Foundry**, indem Sie in der Symbolleiste auf den Link **Zum Azure AI Foundry-Portal** klicken.

3. Wählen Sie in Azure AI Foundry die Option **Bereitstellungen** im linken Menü aus.

4. Wählen Sie auf der Seite **Modellbereitstellungen** die Option **Modell bereitstellen** und wählen Sie dann **Basismodell bereitstellen** aus der Dropdown-Liste aus.

5. Wählen Sie aus der Modellliste `text-embedding-3-small` aus.

    > &#128161; Sie können die Liste filtern, um nur *Embeddings*-Modelle anzuzeigen, indem Sie den Filter „Inferenzaufgaben“ verwenden.

    > &#128221; Wenn Sie das `text-embedding-3-small` Modell nicht sehen, haben Sie möglicherweise eine Azure-Region ausgewählt, die dieses Modell derzeit nicht unterstützt. In diesem Fall können Sie das `text-embedding-ada-002` Modell für diese Übung verwenden. Beide Modelle generieren Vektoren mit 1536 Dimensionen, sodass keine Änderungen an der Containervektorrichtlinie erforderlich sind, die Sie für den `Products` Container in Azure Cosmos DB definiert haben.

6. Wählen Sie „**Bestätigen**“, um das Modell bereitzustellen.

7. Notieren Sie sich auf der Seite „**Model deployments**“ in Azure AI Foundry den **Namen** der `text-embedding-3-small` Modellbereitstellung, da Sie diesen später in dieser Übung benötigen werden.

## Setzen Sie ein Chat-Abschlussmodell um.

Zusätzlich zum Einbettungsmodell benötigen Sie ein Chat-Abschlussmodell für Ihren Copilot. Sie verwenden das große Sprachmodell von OpenAI, `gpt-4o` um Antworten von Ihrem Copilot zu generieren.

1. Während Sie sich noch auf der Seite **Bereitstellung von Modellen** in Azure AI Foundry befinden, wählen Sie erneut die Schaltfläche **Modell bereitstellen** und wählen Sie dann **Basismodell bereitstellen** aus dem Dropdown-Menü aus.

2. Wählen Sie das Chat-Abschlussmodell **gpt-4o** aus der Liste aus.

3. Wählen Sie „**Bestätigen**“, um das Modell bereitzustellen.

4. Hinweis: Auf der Seite **Modellbereitstellungen** in Azure AI Foundry  finden Sie den **Namen** der `gpt-4o`-Modellbereitstellung, den Sie später in dieser Übung benötigen werden.

## Zuweisen der Rolle „OpenAI-Benutzer RBAC“ für kognitive Dienste

Damit Ihre Benutzeridentität mit dem Azure OpenAI Service interagieren kann, können Sie Ihrem Konto die Rolle **Cognitive Services OpenAI-Benutzer** zuweisen. Azure OpenAI Service unterstützt die rollenbasierte Zugriffssteuerung von Azure (Azure RBAC), ein Autorisierungssystem zum Verwalten des individuellen Zugriffs auf Azure-Ressourcen. Mithilfe der Azure RBAC können Sie verschiedenen Teammitgliedern unterschiedliche Berechtigungsebenen basierend auf ihren Anforderungen für ein bestimmtes Projekt zuweisen.

> &#128221; Microsoft Entra IDs rollenbasierte Zugriffssteuerung (RBAC) für die Authentifizierung bei Azure-Diensten wie Azure OpenAI erhöht die Sicherheit durch präzise, auf Benutzerrollen zugeschnittene Zugriffskontrollen und reduziert so effektiv das Risiko unbefugter Zugriffe. Die Optimierung der sicheren Zugriffsverwaltung mithilfe von Entra ID RBAC ist eine effizientere und skalierbarere Lösung für die Nutzung von Azure-Diensten.

1. Navigieren Sie im Azure-Portal (``portal.azure.com``) zu Ihrer Azure OpenAI-Ressource.

2. Wählen Sie im linken Navigationsbereich **Zugriffssteuerung (IAM)** aus.

3. Wählen Sie **Hinzufügen** und dann **Rollenzuweisung hinzufügen** aus.

4. Wählen Sie auf der Registerkarte **Rolle** die Rolle **Cognitive Services OpenAI-Benutzer** aus und klicken Sie dann auf **Weiter**.

5. Wählen Sie auf der Registerkarte **Mitglieder** die Option „Zugriff zuweisen“ für einen Benutzenden, eine Gruppe oder einen Dienstprinzipal und wählen Sie **Mitglieder auswählen**.

6. Suchen Sie im Dialogfeld **Mitglieder auswählen** nach Ihrem Namen oder Ihrer E-Mail-Adresse und wählen Sie Ihr Konto aus.

7. Wählen Sie auf der Registerkarte **Überprüfen und zuweisen** die Option **Überprüfen und zuweisen** aus, um die Rolle zuzuweisen.

## Erstellen einer virtuellen Python-Umgebung

Virtuelle Umgebungen in Python sind für die Aufrechterhaltung eines sauberen und organisierten Entwicklungsbereichs unerlässlich, da sie es einzelnen Projekten ermöglichen, ihre eigenen, von anderen isolierten Abhängigkeiten zu haben. Dies verhindert Konflikte zwischen verschiedenen Projekten und gewährleistet die Konsistenz in Ihrem Entwicklungs-Workflow. Durch die Verwendung virtueller Umgebungen können Sie Paketversionen einfach verwalten, Abhängigkeitskonflikte vermeiden und dafür sorgen, dass Ihre Projekte reibungslos laufen. Dies ist eine bewährte Methode, die Ihre Programmierumgebung stabil und zuverlässig hält und Ihren Entwicklungsprozess effizienter und weniger anfällig für Probleme macht.

1. Öffnen Sie mit Visual Studio Code den Ordner, in den Sie das Lab-Coderepository für das Lernmodul **Erstellen von Copilots mit Azure Cosmos DB** geklont haben.

2. Öffnen Sie in Visual Studio Code ein neues Terminalfenster und wechseln Sie das Verzeichnis zum Ordner `python/07-build-copilot`.

3. Erstellen Sie eine virtuelle Umgebung mit dem Namen `.venv`, indem Sie den folgenden Befehl in der Eingabeaufforderung des Terminals ausführen:

    ```bash
    python -m venv .venv 
    ```

    Mit dem obigen Befehl wird ein `.venv`-Ordner unter dem `07-build-copilot`-Ordner erstellt, der eine dedizierte Python-Umgebung für die Übungen in diesem Lab bereitstellt.

4. Aktivieren Sie die virtuelle Umgebung, indem Sie den entsprechenden Befehl für Ihr Betriebssystem und Ihre Shell aus der folgenden Tabelle auswählen und ihn an der Eingabeaufforderung des Terminals ausführen.

    | Plattform | Shell | Befehl zum Aktivieren der virtuellen Umgebung |
    | -------- | ----- | --------------------------------------- |
    | POSIX | Bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

5. Installieren Sie die in `requirements.txt` definierten Bibliotheken:

    ```bash
    pip install -r requirements.txt
    ```

    Die Datei `requirements.txt` enthält eine Reihe von Python-Bibliotheken, die Sie in diesem Lab verwenden werden.

    | Bibliothek | Version | Beschreibung |
    | ------- | ------- | ----------- |
    | `azure-cosmos` | 4.9.0 | Azure Cosmos DB SDK für Python – Clientbibliothek |
    | `azure-identity` | 1.19.0 | Azure Identity SDK für Python |
    | `fastapi` | 0.115.5 | Web-Framework für die Erstellung von APIs mit Python |
    | `openai` | 1.55.2 | Ermöglicht den Zugriff auf die Azure OpenAI REST API von Python-Apps aus. |
    | `pydantic` | 2.10.2 | Datenvalidierung mit Python-Typ-Hinweisen. |
    | `requests` | 2.32.3 | Senden von HTTP-Anforderungen |
    | `streamlit` | 1.40.2 | Wandelt Python-Skripte in interaktive Web-Apps um. |
    | `uvicorn` | 0.32.1 | Eine ASGI-Webserver-Implementierung für Python. |
    | `httpx` | 0.27.2 | Ein HTTP-Client der nächsten Generation für Python. |

## Fügen Sie eine Python-Funktion hinzu, um Text zu vektorisieren

Das Python SDK für Azure OpenAI bietet Zugriff auf synchrone und asynchrone Klassen, die zur Erstellung von Einbettungen für Textdaten verwendet werden können. Diese Funktionalität kann in einer Funktion in Ihrem Python-Code zusammengefasst werden.

1. Navigieren Sie im Bereich **Erkunden** in Visual Studio Code zum Ordner `python/07-build-copilot/api/app` und öffnen Sie die darin enthaltene Datei `main.py`.

    > &#128221; Diese Datei dient als Einstiegspunkt für eine Back-End-Python-API, die Sie in der nächsten Übung erstellen werden. In dieser Übung stellen Sie eine Handvoll asynchroner Funktionen bereit, mit denen Daten mit Einbettungen in Azure Cosmos DB importiert werden können, die von der API genutzt werden.

2. Um das asynchrone Azure OpenAI SDK für Python zu verwenden, importieren Sie die Bibliothek, indem Sie den folgenden Code am Anfang der `main.py`-Datei einfügen:

   ```python
   from openai import AsyncAzureOpenAI
   ```

3. Sie greifen asynchron auf Azure OpenAI und Cosmos DB zu, indem Sie die Azure-Authentifizierung und die Entra-ID-RBAC-Rollen verwenden, die Sie zuvor Ihrer Benutzeridentität zugewiesen haben. Fügen Sie die folgende Zeile unter der `openai`-Import-Anweisung am Anfang der Datei hinzu, um die erforderlichen Klassen aus der `azure-identity`-Bibliothek zu importieren:

   ```python
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   ```

    > &#128221; Um sicherzustellen, dass Sie über Ihre API sicher mit Azure-Diensten interagieren können, verwenden Sie das Azure Identity SDK für Python. Mit diesem Ansatz müssen Sie keine Schlüssel aus dem Code speichern oder mit ihnen interagieren, sondern können stattdessen die RBAC-Rollen nutzen, die Sie Ihrem Konto in den vorherigen Übungen für den Zugriff auf Azure Cosmos DB und Azure OpenAI zugewiesen haben.

4. Erstellen Sie Variablen, um die Azure OpenAI API-Version und den Endpunkt zu speichern, und ersetzen Sie das `<AZURE_OPENAI_ENDPOINT>`-Token durch den Endpunktwert für Ihren Azure OpenAI Service. Erstellen Sie außerdem eine Variable für den Namen Ihrer Bereitstellung des Einbettungsmodells. Fügen Sie den folgenden Code unter den `import`-Anweisungen in der Datei ein:

   ```python
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   ```

    Wenn Ihr Name für das Bereitstellen der Einbettung abweicht, aktualisieren Sie den der Variablen zugewiesenen Wert entsprechend.

    > &#128161; Die API-Version von `2024-10-21` war zum Zeitpunkt der Erstellung dieses Artikels die neueste GA-Version. Sie können diese oder eine neue Version verwenden, falls eine verfügbar ist. Die Dokumentation der API-Spezifikationen enthält eine [Tabelle mit den neuesten API-Versionen](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs).

    > &#128221; Der `EMBEDDING_DEPLOYMENT_NAME` ist der **Name** des Werts, den Sie nach dem Bereitstellen des `text-embedding-3-small`-Modells in Azure AI Foundry angegeben haben. Wenn Sie darauf zurückgreifen müssen, starten Sie Azure AI Foundry, navigieren Sie zur Seite **Bereitstellen** und suchen Sie die Bereitstellung, deren **Modellname** `text-embedding-3-small` lautet. Kopieren Sie dann den Wert des Feldes **Name** dieses Elements. Wenn Sie das `text-embedding-ada-002`-Modell bereitgestellt haben, verwenden Sie den Namen für diese Bereitstellung.

5. Verwenden Sie die Azure Identity SDK für Pythons `DefaultAzureCredential`-Klasse, um mithilfe der Microsoft Entra ID RBAC-Authentifizierung asynchrone Anmeldeinformationen für den Zugriff auf Azure OpenAI und Azure Cosmos DB zu erstellen, indem Sie den folgenden Code unter den Variablendeklarationen einfügen:

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

6. Um die Erstellung von Einbettungen zu handhaben, fügen Sie Folgendes ein, das eine Funktion hinzufügt, um Einbettungen mithilfe eines Azure OpenAI-Clients zu generieren:

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

    Für die Erstellung des Azure OpenAI-Clients ist der Wert `api_key` nicht erforderlich, da ein Bearertoken mithilfe der Klasse `get_bearer_token_provider` des Azure Identity SDK abgerufen wird.

7. Die `main.py`-Datei sollte nun wie folgt aussehen:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

8. Speichern Sie die Datei `main.py`.

## Testen der Einbettungsfunktion

Um sicherzustellen, dass die `generate_embeddings`-Funktion in der `main.py`-Datei ordnungsgemäß funktioniert, fügen Sie am Ende der Datei einige Codezeilen hinzu, damit sie direkt ausgeführt werden kann. Diese Zeilen ermöglichen es Ihnen, die `generate_embeddings`-Funktion über die Befehlszeile auszuführen und den einzubettenden Text zu übergeben.

1. Fügen Sie einen **Hauptschutzblock** mit einem Aufruf an `generate_embeddings` am Ende der Datei `main.py` hinzu:

   ```python
   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

    > &#128221; Der `if __name__ == "__main__":`-Block wird in Python allgemein als **Standardwache** oder **Einstiegspunkt** bezeichnet. Es stellt sicher, dass ein bestimmter Code nur ausgeführt wird, wenn das Skript direkt ausgeführt wird, und nicht, wenn es als Modul in ein anderes Skript importiert wird. Diese Vorgehensweise hilft bei der Organisation von Code und macht ihn wiederverwendbarer und modularer.

2. Speichern Sie die Datei `main.py`, die nun wie folgt aussehen sollte:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

3. Öffnen Sie in Visual Studio Code ein neues integriertes Terminalfenster.

4. Bevor Sie die API ausführen, die Anfragen an Azure OpenAI sendet, müssen Sie sich mit dem Befehl `az login` bei Azure anmelden. Führen Sie im Fenster Terminal Folgendes aus:

   ```bash
   az login
   ```

5. Schließen Sie den Anmeldevorgang in Ihrem Browser ab.

6. Wechseln Sie in der Eingabeaufforderung des Terminals in das Verzeichnis `python/07-build-copilot`.

7. Stellen Sie sicher, dass das integrierte Terminalfenster in Ihrer virtuellen Python-Umgebung ausgeführt wird, indem Sie Ihre virtuelle Umgebung mit einem Befehl aus der folgenden Tabelle aktivieren und den entsprechenden Befehl für Ihr Betriebssystem und Ihre Shell auswählen.

    | Plattform | Shell | Befehl zum Aktivieren der virtuellen Umgebung |
    | -------- | ----- | --------------------------------------- |
    | POSIX | Bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

8. Wechseln Sie an der Terminal-Eingabeaufforderung in das Verzeichnis `api/app` und führen Sie dann den folgenden Befehl aus:

   ```python
   python main.py "Hello, world!"
   ```

9. Beobachten Sie die Ausgabe im Terminalfenster. Sie sollten ein Array mit Fließkommazahlen sehen, das die Vektordarstellung von „Hello World!“ ist. string. Es sollte in etwa so aussehen wie die folgende verkürzte Ausgabe:

   ```bash
   [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
   ```

## Erstellen einer Funktion zum Schreiben von Daten in Azure Cosmos DB

Mit dem Azure Cosmos DB SDK for Python können Sie eine Funktion erstellen, die das Hochladen von Dokumenten in Ihre Datenbank ermöglicht. Ein Upsert-Vorgang aktualisiert einen Datensatz, wenn ein Match gefunden wird, und fügt einen neuen Datensatz ein, wenn dies nicht der Fall ist.

1. Kehren Sie zu der in Visual Studio Code geöffneten Datei `main.py` zurück und importieren Sie die asynchrone `CosmosClient`-Klasse aus dem Azure Cosmos DB SDK für Python, indem Sie die folgende Zeile direkt unter den `import`-Anweisungen einfügen, die sich bereits in der Datei befinden:

   ```python
   from azure.cosmos.aio import CosmosClient
   ```

2. Fügen Sie eine weitere Importanweisung hinzu, um die Klasse `Product` aus dem Modul *models* im Ordner `api/app` zu referenzieren. Die Klasse `Product` definiert die Form der Produkte im Cosmic Works-Datensatz.

   ```python
   from models import Product
   ```

3. Erstellen Sie eine neue Gruppe von Variablen mit Konfigurationswerten, die mit Azure Cosmos DB verbunden sind, und fügen Sie sie der `main.py`-Datei unter den Azure OpenAI-Variablen hinzu, die Sie zuvor eingefügt haben. Stellen Sie sicher, dass Sie das `<AZURE_COSMOSDB_ENDPOINT>`-Token durch den Endpunkt für Ihr Azure Cosmos DB-Konto ersetzen.

   ```python
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
   ```

4. Fügen Sie eine Funktion namens `upsert_product` für das Hochladen (Aktualisieren oder Einfügen) von Dokumenten in Cosmos DB hinzu, indem Sie den folgenden Code unterhalb der Funktion `generate_embeddings` in der Datei `main.py` einfügen:

   ```python
   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
   ```

5. Speichern Sie die Datei `main.py`, die nun wie folgt aussehen sollte:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"

   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
    
   if __name__ == "__main__":
       import sys
       print(generate_embeddings(sys.argv[1]))
   ```

## Vektorisieren von Beispieldaten

Sie können nun die Funktionen `generate_embeddings` und `upsert_document` gemeinsam testen. Dazu überschreiben Sie den Hauptschutzblock `if __name__ == "__main__"` mit Code, der eine Beispieldatendatei mit Cosmic Works-Produktinformationen von GitHub herunterlädt und dann das Feld `description` jedes Produkts vektorisiert und die Dokumente in den Container `Products` in Ihrer Azure Cosmos DB-Datenbank einfügt.

> &#128221; Dieser Ansatz wird verwendet, um die Techniken für die Erzeugung mit Azure OpenAI und die Speicherung von Einbettungen in Azure Cosmos DB zu demonstrieren. In einem realen Szenario wäre jedoch ein robusterer Ansatz, z. B. die Verwendung einer Azure-Funktion, die durch den Azure Cosmos DB Änderungsfeed ausgelöst wird, für die Handhabung des Hinzufügens von Einbettungen zu bestehenden und neuen Dokumenten geeigneter.

1. Überschreiben Sie in der `main.py`-Datei den Codeblock `if __name__ == "__main__":` mit folgendem Code:

   ```python
   if __name__ == "__main__":
       import asyncio
       from models import Product
       import requests
    
       async def main():
           product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
           products = [Product(**data) for data in requests.get(product_raw_data).json()]
    
           # Call the generate_embeddings function, passing in an argument from the command line.    
           for product in products:
               print(f"Generating embeddings for product: {product.name}", end="\r")
               product.embedding = await generate_embeddings(product.description)
               await upsert_product(product.model_dump())
    
           print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
           # Close open credential sessions
           await credential.close()
    
       asyncio.run(main())
   ```

2. Führen Sie die Datei `main.py` in der geöffneten integrierten Terminal-Eingabeaufforderung in Visual Studio Code erneut mit dem Befehl aus:

   ```python
   python main.py
   ```

3. Warten Sie auf den Abschluss der Codeausführung, der durch eine Meldung angezeigt wird, die besagt, dass alle Produkte mit vektorisierten Beschreibungen in den Cosmos DB-Container eingefügt wurden. Es kann bis zu 10-15 Minuten dauern, bis die Vektorisierung und das Einfügen der Daten für die 295 Datensätze im Produktdatensatz abgeschlossen sind. Wenn nicht alle Produkte eingefügt sind, können Sie `main.py` mit dem obigen Befehl erneut ausführen, um die restlichen Produkte hinzuzufügen.

## Überprüfung der eingefügten Probendaten in Cosmos DB

1. Kehren Sie zum Azure-Portal zurück (``portal.azure.com``) und navigieren Sie zu Ihrem Azure Cosmos DB-Konto.

2. Wählen Sie den **Data Explorer** für das linke Navigationsmenü.

3. Erweitern Sie die Datenbank **CosmicWorks** und den Container **Produkte** und wählen Sie dann **Elemente** unter dem Container.

4. Wählen Sie mehrere zufällige Dokumente innerhalb des Containers aus und stellen Sie sicher, dass das Feld `embedding` mit einem Vektor-Array gefüllt ist.
