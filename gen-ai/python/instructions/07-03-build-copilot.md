---
lab:
  title: 07.3 - Erstellen eines Copilots mit Python und Azure Cosmos DB for NoSQL
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Erstellen eines Copilots mit Python und Azure Cosmos DB for NoSQL

Durch die Nutzung der vielseitigen Programmierfähigkeiten von Python und der skalierbaren NoSQL-Datenbank und Vektorsuchfunktionen von Azure Cosmos DB können Sie leistungsstarke und effiziente KI-Copilots erstellen und komplexe Workflows rationalisieren.

In diesem Lab erstellen Sie einen Copilot mit Python und Azure Cosmos DB for NoSQL und erstellen eine Backend-API, die die für die Interaktion mit Azure-Diensten (Azure OpenAI und Azure Cosmos DB) erforderlichen Endpunkte bereitstellt, sowie eine Frontend-Benutzeroberfläche, um die Interaktion der Benutzenden mit Copilot zu erleichtern. Der Copilot wird Benutzenden von Cosmic Works als Assistent bei der Verwaltung und Suche nach Fahrradprodukten zur Seite stehen. Insbesondere wird der Copilot es Benutzenden ermöglichen, Rabatte auf Produktkategorien anzuwenden und zu entfernen, Produktkategorien nachzuschlagen, um Benutzende über die verfügbaren Produkttypen zu informieren, und die Vektorsuche zu verwenden, um Ähnlichkeitssuchen für Produkte durchzuführen.

![Ein hochrangiges Copilot-Architekturdiagramm, das eine in Python entwickelte Benutzeroberfläche mit Streamlit, einer in Python geschriebenen Backend-API, und Interaktionen mit Azure Cosmos DB und Azure OpenAI zeigt.](media/07-copilot-high-level-architecture-diagram.png)

Die Trennung der App-Funktionalität in eine eigene Benutzeroberfläche und eine Back-End-API bei der Erstellung eines Copilots in Python bietet mehrere Vorteile. Erstens verbessert es die Modularität und Wartungsfreundlichkeit, da Sie die Benutzeroberfläche oder das Back-End unabhängig voneinander aktualisieren können, ohne die jeweils andere Seite zu beeinträchtigen. Streamlit bietet eine intuitive und interaktive Oberfläche, die die Benutzerinteraktion vereinfacht, während FastAPI eine hochleistungsfähige, asynchrone Anforderungsbearbeitung und Datenverarbeitung gewährleistet. Diese Trennung fördert auch die Skalierbarkeit, da verschiedene Komponenten auf mehreren Servern bereitgestellt werden können, wodurch der Ressourceneinsatz optimiert wird. Darüber hinaus ermöglicht es bessere Sicherheitspraktiken, da die Back-End-API sensible Daten und die Authentifizierung separat handhaben kann, wodurch das Risiko der Aufdeckung von Schwachstellenaufn der Ebene der Benutzeroberfläche verringert wird. Dieser Ansatz führt zu einer robusteren, effizienteren und benutzerfreundlicheren Anwendung.

> &#128721; Die vorherigen Übungen in diesem Modul sind Voraussetzung für dieses Lab. Wenn Sie noch eine dieser Übungen abschließen müssen, tun Sie dies bitte, bevor Sie fortfahren, da sie die notwendige Infrastruktur und den Startercode für dieses Lab bereitstellen.

## Erstellen einer Back-End-API

Die Back-End-API für den Copilot erweitert seine Fähigkeiten, komplexe Daten zu verarbeiten, Erkenntnisse in Echtzeit zu liefern und eine nahtlose Verbindung zu verschiedenen Diensten herzustellen, wodurch Interaktionen dynamischer und informativer werden. Um die API für Ihren Copilot zu erstellen, werden Sie die FastAPI Python-Bibliothek verwenden. FastAPI ist ein modernes, hochleistungsfähiges Web-Framework, das es Ihnen ermöglicht, mit Python APIs zu erstellen, die auf standardmäßigen Python-Typ-Hinweisen basieren. Durch die Entkopplung des Copiloten vom Back-End mit diesem Ansatz sorgen Sie für mehr Flexibilität, Wartbarkeit und Skalierbarkeit, sodass sich der Copilot unabhängig von Änderungen im Back-End weiterentwickeln kann.

> &#128721; Die Back-End-API baut auf dem Code auf, den Sie in der vorherigen Übung zur Datei `main.py` im Ordner `python/07-build-copilot/api/app` hinzugefügt haben. Wenn Sie die vorherige Übung noch nicht abgeschlossen haben, schließen Sie sie bitte ab, bevor Sie fortfahren.

1. Öffnen Sie mit Visual Studio Code den Ordner, in den Sie das Lab-Coderepository für das Lernmodul **Erstellen von Copilots mit Azure Cosmos DB** geklont haben.

1. Navigieren Sie im Bereich **Explorer** von Visual Studio Code zum Ordner **python/07-build-copilot/api/app** und öffnen Sie die darin befindliche `main.py`-Datei.

1. Fügen Sie die folgenden Codezeilen unter den vorhandenen `import`-Anweisungen am Anfang der Datei `main.py` ein, um die Bibliotheken einzubinden, die für die Durchführung asychroner Aktionen mit FastAPI verwendet werden:

   ```python
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
   ```

1. Damit der von Ihnen erstellte `/chat`-Endpunkt Daten im Request Body empfangen kann, übergeben Sie den Inhalt über ein `CompletionRequest`-Objekt, das im Modul *Modelle* des Projekts definiert ist. Aktualisieren Sie die `from models import Product` Importanweisung am Anfang der Datei, um die `CompletionRequest`-Klasse aus dem `models`-Modul einzuschließen. Die Importanweisung sollte nun wie folgt aussehen:

   ```python
   from models import Product, CompletionRequest
   ```

1. Sie benötigen den Bereitstellungsnamen des Chat-Abschlussmodells, das Sie in Ihrem Azure OpenAI Service erstellt haben. Erstellen Sie eine Variable am Ende des Azure OpenAI-Konfigurationsvariablenblocks, um dies zu ermöglichen:

   ```python
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
   ```

    Wenn sich der Name Ihrer Abschlussbereitstellung unterscheidet, aktualisieren Sie den der Variablen zugewiesenen Wert entsprechend.

1. Die Azure Cosmos DB und Identity SDKs bieten asynchrone Methoden für die Arbeit mit diesen Diensten. Jede dieser Klassen wird in mehreren Funktionen in Ihrer API verwendet werden, so dass Sie globale Instanzen von jeder erstellen werden, so dass derselbe Client in allen Methoden gemeinsam genutzt werden kann. Fügen Sie die folgenden globalen Variablendeklarationen unterhalb des Cosmos DB-Konfigurationsvariablen-Blocks ein:

   ```python
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   ```

1. Löschen Sie die folgenden Codezeilen aus der Datei, da die bereitgestellte Funktionalität in die Funktion `lifespan` verschoben wird, die Sie im nächsten Schritt definieren werden:

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

1. Um Singleton-Instanzen der Klassen `CosmosClient` und `DefaultAzureCredentail` zu erstellen, nutzen Sie das `lifespan`-Objekt in FastAPI: Diese Methode verwaltet diese Klassen während des Lebenszyklus der API-App. Fügen Sie den folgenden Code ein, um die `lifespan` zu definieren:

   ```python
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
   ```

   In FastAPI sind Lebensdauer-Ereignisse spezielle Vorgänge, die zu Beginn und am Ende des Lebenszyklus der Anwendung ausgeführt werden. Diese Vorgänge werden ausgeführt, bevor die App mit der Bearbeitung von Anforderungen beginnt und nachdem sie beendet wurde. Dadurch eignen sie sich ideal für die Initialisierung und Bereinigung von Ressourcen, die in der gesamten Anwendung verwendet und zwischen Anforderungen gemeinsam genutzt werden. Dieser Ansatz stellt sicher, dass die notwendige Einrichtung abgeschlossen ist, bevor Anfragen bearbeitet werden, und dass die Ressourcen beim Herunterfahren ordnungsgemäß verwaltet werden.

1. Erstellen Sie mit dem folgenden Code eine Instanz der FastAPI-Klasse. Diese sollte unterhalb der Funktion `lifespan` eingefügt werden:

   ```python
   app = FastAPI(lifespan=lifespan)
   ```

   Durch den Aufruf von `FastAPI()` initialisieren Sie eine neue Instanz der FastAPI-Anwendung. Diese Instanz, die als `app` bezeichnet wird, dient als Haupteinstiegspunkt für Ihre Webanwendung. Durch die Übergabe von `lifespan` wird der Ereignishandler für die Lebensdauer an Ihre App angehängt.

1. Als Nächstes müssen Sie die Endpunkte für Ihre API erstellen. Die `api_status`-Methode wird an die Stamm-URL Ihrer API angehängt und dient als Statusmeldung, um zu zeigen, dass die API ordnungsgemäß funktioniert und läuft. Sie werden den Endpunkt `/chat` später in dieser Übung erstellen. Fügen Sie den folgenden Code unterhalb des Codes zum Erstellen des Cosmos DB-Clients, der Datenbank und des Containers ein:

   ```python
   @app.get("/")
   async def api_status():
       """Display a status message for the API"""
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

1. Überschreiben Sie den Hauptschutzblock am Ende der Datei, um den `uvicorn` ASGI (Asynchronous Server Gateway Interface)-Webserver zu starten, wenn die Datei von der Befehlszeile aus ausgeführt wird:

   ```python
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Speichern Sie die Datei `main.py`. Sie sollte nun wie folgt aussehen, einschließlich der Methoden `generate_embeddings` und `upsert_product`, die Sie in der vorangegangenen Übung hinzugefügt haben:

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product, CompletionRequest
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
    
   app = FastAPI(lifespan=lifespan)
    
   @app.get("/")
   async def api_status():
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """ Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
    
   async def generate_embeddings(text: str):
       # Create Azure OpenAI client
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
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. Um Ihre API schnell zu testen, öffnen Sie ein neues integriertes Terminalfenster in Visual Studio Code.

1. Stellen Sie sicher, dass Sie mit dem Befehl `az login` bei Azure angemeldet sind. Führen Sie Folgendes an der Terminal-Eingabeaufforderung aus:

   ```bash
   az login
   ```

1. Schließen Sie den Anmeldevorgang in Ihrem Browser ab.

1. Ändern Sie an der Terminal-Eingabeaufforderung die Verzeichnisse zu `python/07-build-copilot`.

1. Stellen Sie sicher, dass das integrierte Terminal-Fenster in Ihrer virtuellen Python-Umgebung läuft, indem Sie es mit einem Befehl aus der folgenden Tabelle aktivieren und den entsprechenden Befehl für Ihr Betriebssystem und Ihre Shell auswählen.

    | Plattform | Shell | Befehl zum Aktivieren der virtuellen Umgebung |
    | -------- | ----- | --------------------------------------- |
    | POSIX | Bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

1. Ändern Sie an der Terminal-Eingabeaufforderung die Verzeichnisse zu `api/app`, und führen Sie dann den folgenden Befehl aus, um die FastAPI-Web-App auszuführen:

   ```bash
   uvicorn main:app
   ```

1. Sollte sich eine Seite nicht automatisch öffnen, öffnen Sie ein neues Fenster oder eine neue Registerkarte des Webbrowsers und gehen Sie zu <http://127.0.0.1:8000>.

    Eine Meldung von `{"status":"ready"}` im Browserfenster zeigt an, dass Ihre API ausgeführt wird.

1. Navigieren Sie zur Swagger-Benutzeroberfläche für die API, indem Sie `/docs` an das Ende der URL anhängen: <http://127.0.0.1:8000/docs>.

    > &#128221; Die Swagger-Benutzeroberfläche ist eine interaktive, webbasierte Schnittstelle zum Erkunden und Testen von API-Endpunkten, die aus OpenAPI-Spezifikationen generiert wurden. Sie ermöglicht Fachkräften in der Entwicklung und Benutzenden das Visualisieren, Interagieren und Debuggen von API-Echtzeitaufrufen und verbessert so die Benutzerfreundlichkeit und Dokumentation.

1. Kehren Sie zu Visual Studio Code zurück und beenden Sie die API-Anwendung durch Drücken von **CTRL+C** im zugehörigen integrierten Terminalfenster.

## Einbindung von Produktdaten aus Azure Cosmos DB

Durch die Nutzung von Daten aus Azure Cosmos DB kann der Copilot komplexe Workflows rationalisieren und Benutzende bei der effizienten Erledigung von Aufgaben unterstützen. Der Copilot kann Datensätze aktualisieren und Suchwerte in Echtzeit abrufen, so dass genaue und zeitnahe Informationen gewährleistet sind. Diese Fähigkeit ermöglicht es dem Copiloten, fortgeschrittene Interaktionen bereitzustellen, wodurch die Fähigkeit der Benutzenden verbessert wird, schnell und präzise zu navigieren und Aufgaben zu erledigen.

Funktionen ermöglichen es dem Copilot des Produktmanagements, Rabatte auf Produkte innerhalb einer Kategorie anzuwenden. Diese Funktionen sind der Mechanismus, über den der Copilot die Produktdaten von Cosmic Works aus Azure Cosmos DB abruft und mit ihnen interagiert.

1. Der Copilot verwendet eine asynchrone Funktion namens `apply_discount`, um Rabatte und Verkaufspreise für Produkte innerhalb einer bestimmten Kategorie hinzuzufügen und zu entfernen. Fügen Sie den folgenden Funktionscode unterhalb der Funktion `upsert_product` am Ende der Datei `main.py` ein:

   ```python
   async def apply_discount(discount: float, product_category: str) -> str:
       """Apply a discount to products in the specified category."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
           """,
           parameters = [
               {"name": "@product_category", "value": product_category}
           ]
       )
    
       # Apply the discount to the products
       async for item in query_results:
           item['discount'] = discount
           item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
           await container.upsert_item(item)
    
       return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
   ```

    Diese Funktion führt eine Suche in Azure Cosmos DB durch, um alle Produkte innerhalb einer Kategorie zu finden und den gewünschten Rabatt auf diese Produkte anzuwenden. Es berechnet auch den Verkaufspreis des Artikels unter Verwendung des angegebenen Rabatts und fügt diesen in die Datenbank ein.

2. Als Nächstes fügen Sie eine zweite Funktion namens `get_category_names` hinzu, die der Copilot aufruft, um zu erfahren, welche Produktkategorien verfügbar sind, wenn Rabatte auf Produkte gewährt oder aufgehoben werden. Fügen Sie die nachstehende Methode unterhalb der Funktion `apply_discount` in die Datei ein:

   ```python
   async def get_category_names() -> list:
       """Retrieve the names of all product categories."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
       # Get distinct product categories
       query_results = container.query_items(
           query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
       )
       categories = []
       async for category in query_results:
           categories.append(category)
       return list(categories)
   ```

    Die Funktion `get_category_names` fragt den Container `Products` ab, um eine Liste von eindeutigen Kategorienamen aus der Datenbank zu erhalten.

3. Speichern Sie die Datei `main.py`.

## Implementieren des Chat-Endpunkts

Der `/chat`-Endpunkt auf der Back-End-API dient als Schnittstelle, über die die Front-End-Benutzeroberfläche mit Azure OpenAI-Modellen und internen Cosmic Works-Produktdaten interagiert. Dieser Endpunkt fungiert als Kommunikationsbrücke, die es der Benutzeroberfläche ermöglicht, Eingaben an den Azure OpenAI-Service zu senden, der diese Eingaben dann mithilfe ausgefeilter Sprachmodelle verarbeitet. Die Ergebnisse werden dann an das Front-End zurückgegeben, was intelligente Unterhaltungen in Echtzeit ermöglicht. Durch die Nutzung dieser Konfiguration können Fachkräfte in der Entwicklung eine nahtlose und reaktionsschnelle Benutzererfahrung gewährleisten, während das Back-End die komplexe Aufgabe der Verarbeitung natürlicher Sprache und der Generierung geeigneter Antworten übernimmt. Dieser Ansatz unterstützt auch die Skalierbarkeit und Wartbarkeit, indem er das Frontend von der zugrunde liegenden KI-Infrastruktur entkoppelt.

1. Suchen Sie den Endpunkt-Stub `/chat`, den Sie zuvor in der Datei `main.py` hinzugefügt haben.

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

    Die Funktion akzeptiert einen `CompletionRequest` als Parameter. Durch die Verwendung einer Klasse für den Eingabeparameter können mehrere Eigenschaften an den API-Endpunkt im Anforderungstext übergeben werden. Die `CompletionRequest`-Klasse ist im Modul *Models* definiert und umfasst die Eigenschaften „Benutzernachricht“, „Chatverlauf“ und „Maximaler Verlauf“. Der Chatverlauf ermöglicht es Copilot, auf frühere Aspekte des Gesprächs mit dem Benutzenden zu verweisen, sodass er den Kontext der gesamten Diskussion im Blick behält. Mit der `max_history`-Eigenschaft können Sie die Anzahl der Verlaufsmeldungen festlegen, die in den Kontext der LLM übernommen werden sollen. Auf diese Weise können Sie die Verwendung von Token für Ihre Eingabeaufforderung steuern und TPM-Beschränkungen für Anfragen vermeiden.

2. Löschen Sie zu Beginn die Zeile `raise NotImplementedError("The chat endpoint is not implemented yet.")` aus der Funktion, da Sie mit der Implementierung des Endpunkts beginnen.

3. Als Erstes werden Sie im Rahmen der Chat-Endpunkt-Methode eine Eingabeaufforderung für das System bereitstellen. Diese Eingabeaufforderung definiert die „Persona“ von Copilot und gibt vor, wie Copilot mit Benutzenden interagiert, auf Fragen antwortet und verfügbare Funktionen nutzen sollte, um Aktionen auszuführen.

   ```python
   # Define the system prompt that contains the assistant's persona.
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   """
   ```

4. Erstellen Sie als Nächstes eine Reihe von Nachrichten, die an das LLM gesendet werden sollen, und fügen Sie die Eingabeaufforderung des Systems, alle Nachrichten im Chatverlauf und die eingehende Benutzernachricht hinzu. Dieser Code sollte direkt unter der Eingabeaufforderung des Systems in der Funktion stehen:

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{"role": "system", "content": system_prompt }]
    
   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)
    
   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

    Die `messages`-Eigenschaft fasst den Verlauf des laufenden Gesprächs zusammen. Sie umfasst die gesamte Abfolge von Benutzereingaben und KI-Antworten, wodurch das Modell den Kontext beibehält. Durch den Bezug auf diese Vorgeschichte kann die KI kohärente und kontextbezogene Antworten generieren und so sicherstellen, dass die Interaktionen flüssig und dynamisch bleiben. Diese Eigenschaft ist entscheidend, damit die KI den Verlauf und die Nuancen des Gesprächs im weiteren Verlauf verstehen kann.

5. Damit Copilot die oben definierten Funktionen für die Interaktion mit Daten aus Azure Cosmos DB verwenden kann, müssen Sie eine Sammlung von „Tools“ definieren. Die LLM wird diese Tools im Rahmen ihrer Ausführung aufrufen. Azur OpenAI verwendet Funktionsdefinitionen, um strukturierte Interaktionen zwischen der KI und verschiedenen Tools oder APIs zu ermöglichen. Wenn eine Funktion definiert ist, beschreibt sie die Operationen, die sie ausführen kann, die erforderlichen Parameter und alle erforderlichen Eingaben. Um ein Array von `tools` zu erstellen, geben Sie den folgenden Code mit Funktionsdefinitionen für die zuvor definierten Methoden `apply_discount` und `get_category_names` ein:

   ```python
   # Define function calling tools
   tools = [
       {
           "type": "function",
           "function": {
               "name": "apply_discount",
               "description": "Apply a discount to products in the specified category",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "discount": {"type": "number", "description": "The percent discount to apply."},
                       "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                   },
                   "required": ["discount", "product_category"]
               }
           }
       },
       {
           "type": "function",
           "function": {
               "name": "get_category_names",
               "description": "Retrieves the names of all product categories"
           }
       }
   ]
   ```

    Durch die Verwendung von Funktionsdefinitionen stellt Azur OpenAI sicher, dass die Interaktionen zwischen der KI und externen Systemen gut organisiert, sicher und effizient sind. Dieser strukturierte Ansatz ermöglicht es der KI, komplexe Aufgaben nahtlos und zuverlässig auszuführen, wodurch ihre Gesamtleistung und die Benutzererfahrung verbessert werden.

6. Erstellen Sie einen asynchronen Azur OpenAI-Client, um Anfragen an Ihr Chat-Abschlussmodell zu senden:

   ```python
   # Create Azure OpenAI client
   aoai_client = AsyncAzureOpenAI(
       api_version = AZURE_OPENAI_API_VERSION,
       azure_endpoint = AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
   )
   ```

7. Der Chat-Endpunkt wird zwei Aufrufe an Azur OpenAI senden, um Funktionsaufrufe zu nutzen. Die erste ermöglicht dem Azur OpenAI-Client den Zugriff auf die Tools:

   ```python
   # First API call, providing the model to the defined functions
   response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages,
       tools = tools,
       tool_choice = "auto"
   )
    
   # Process the model's response and add it to the conversation history
   response_message = response.choices[0].message
   messages.append(response_message)
   ```

8. Die Antwort auf diesen ersten Aufruf enthält Informationen des LLM darüber, welche Tools oder Funktionen seiner Meinung nach erforderlich sind, um auf die Anfrage zu reagieren. Sie müssen Code einfügen, um die Ausgaben des Funktionsaufrufs zu verarbeiten, und sie in die aufgezeichneten Unterhaltungen einfügen, damit das LLM sie zur Formulierung einer Antwort über die in diesen Ausgaben enthaltenen Daten verwenden kann:

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

    Die Funktion „Aufruf in Azur OpenAI“ ermöglicht die nahtlose Integration externer APIs oder Tools direkt in die Ausgabe Ihres Modells. Wenn das Modell eine relevante Anfrage erkennt, erstellt es ein JSON-Objekt mit den erforderlichen Parametern, die Sie dann ausführen. Das Ergebnis wird an das Modell zurückgegeben, sodass es eine umfassende endgültige Antwort liefern kann, die mit externen Daten angereichert ist.

9. Um die Anfrage mit den angereicherten Daten aus Azure Cosmos DB zu vervollständigen, müssen Sie eine zweite Anfrage an Azur OpenAI senden, um eine Vervollständigung zu generieren:

   ```python
   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )
   ```

10. Schicken Sie die ausgefüllte Antwort schließlich an die Benutzeroberfläche zurück:

   ```python
   return final_response.choices[0].message.content
   ```

11. Speichern Sie die Datei `main.py`. Die `/chat` `generate_chat_completion`-Methode des Endpunkts sollte wie folgt aussehen:

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
   ```

## Erstellen einer einfachen Chat-UI

Die Streamlit-Benutzeroberfläche bietet eine Schnittstelle, über die Benutzende mit Ihrem Copilot interagieren können.

1. Die Benutzeroberfläche wird mithilfe der `index.py`-Datei definiert, die sich im `python/07-build-copilot/ui`-Ordner befindet.

2. Öffnen Sie die Datei `index.py` und fügen Sie die folgenden Importanweisungen am Anfang der Datei hinzu, um erste Schritte zu unternehmen:

   ```python
   import streamlit as st
   import requests
   ```

3. Konfigurieren Sie die in der Datei `index.py` definierte Streamlit-Seite, indem Sie die folgende Zeile unter den `import`-Anweisungen hinzufügen:

   ```python
   st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
   ```

4. Die Benutzeroberfläche interagiert mit der Back-End-API, indem sie die `requests`-Bibliothek verwendet, um Aufrufe an den `/chat`-Endpunkt zu senden, den Sie in der API definiert haben. Sie können den API-Aufruf in einer Methode kapseln, die die aktuelle Benutzernachricht und eine Liste von Nachrichten aus dem Chatverlauf erwartet.

   ```python
   async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
       """Send a message to the Copilot chat endpoint."""
       try:
           api_endpoint = "http://localhost:8000"
           request = {"message": message, "chat_history": chat_history}
           response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
           return response.json()
       except Exception as e:
           st.error(f"An error occurred: {e}")
           return""
   ```

5. Definieren Sie die `main`-Funktion, die den Einstiegspunkt für Anrufe in die Anwendung darstellt.

   ```python
   async def main():
       """Main function for the Cosmic Works Product Management Copilot UI."""
    
       st.write(
           """
           # Cosmic Works Product Management Copilot
        
           Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
        
           **Ask the copilot to apply or remove a discount on a category of products or to find products.**
           """
       )
    
       # Add a messages collection to the session state to maintain the chat history.
       if "messages" not in st.session_state:
           st.session_state.messages = []
    
       # Display message from the history on app rerun.
       for message in st.session_state.messages:
           with st.chat_message(message["role"]):
               st.markdown(message["content"])
    
       # React to user input
       if prompt := st.chat_input("What can I help you with today?"):
           with st. spinner("Awaiting the copilot's response to your message..."):
               # Display user message in chat message container
               with st.chat_message("user"):
                   st.markdown(prompt)
                
               # Send the user message to the copilot API
               response = await send_message_to_copilot(prompt, st.session_state.messages)
    
               # Display assistant response in chat message container
               with st.chat_message("assistant"):
                   st.markdown(response)
                
               # Add the current user message and assistant response messages to the chat history
               st.session_state.messages.append({"role": "user", "content": prompt})
               st.session_state.messages.append({"role": "assistant", "content": response})
   ```

6. Fügen Sie schließlich am Ende der Datei einen **Standardblock** hinzu:

   ```python
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

7. Speichern Sie die Datei `index.py`.

## Testen von Copilot über die Benutzeroberfläche

1. Kehren Sie zum integrierten Terminalfenster zurück, das Sie in Visual Studio Code für das API-Projekt geöffnet haben, und geben Sie Folgendes ein, um die API-App zu starten:

   ```bash
   uvicorn main:app
   ```

2. Öffnen Sie ein neues integriertes Terminalfenster, wechseln Sie in das Verzeichnis `python/07-build-copilot`, um Ihre Python-Umgebung zu aktivieren, wechseln Sie dann in den Ordner `ui` und führen Sie Folgendes aus, um Ihre UI-App zu starten:

   ```bash
   python -m streamlit run index.py
   ```

3. Wenn die Benutzeroberfläche nicht automatisch in einem Browserfenster geöffnet wird, starten Sie einen neuen Browser-Tab oder ein neues Browserfenster und navigieren Sie zu <http://localhost:8501>, um die Benutzeroberfläche zu öffnen.

4. Geben Sie bei der Eingabeaufforderung der Benutzeroberfläche „Rabatt anwenden“ ein und senden Sie die Nachricht.

    Da Sie Copilot mehr Details zur Verfügung stellen mussten, um zu handeln, sollte die Antwort eine Anfrage nach weiteren Informationen sein, z. B. nach dem Rabattprozentsatz, den Sie anwenden möchten, und der Kategorie der Produkte, auf die der Rabatt angewendet werden soll.

5. Um zu verstehen, welche Kategorien verfügbar sind, bitten Sie Copilot, Ihnen eine Liste der Produktkategorien zur Verfügung zu stellen.

    Copilot wird über die Funktion `get_category_names` einen Funktionsaufruf tätigen und die Konversationsnachrichten mit diesen Kategorien anreichern, damit er entsprechend reagieren kann.

6. Sie können auch nach einer spezifischeren Auswahl an Kategorien fragen, z. B. „Stell mir eine Liste mit Kategorien für Kleidung zusammen.“

7. Als Nächstes bitten Sie Copilot, einen Rabatt von 15 % auf alle Bekleidungsprodukte anzuwenden.

8. Sie können das Anwenden des Preisnachlasses überprüfen, indem Sie Ihr Azure Cosmos DB-Konto im Azure-Portal öffnen, den **Daen-Explorer** auswählen und eine Abfrage für den Container `Products` ausführen, um alle Produkte in der Kategorie „Kleidung“ anzuzeigen, z. B.:

   ```sql
   SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
   WHERE CONTAINS(LOWER(c.category_name), "clothing")
   ```

    Beachten Sie, dass jeder Eintrag in den Abfrageergebnissen einen `discount`-Wert von `0.15` hat und die `sale_price` 15 % weniger als die ursprünglichen `price` sein sollten.

9. Kehren Sie zu Visual Studio Code zurück und stoppen Sie die API-App, indem Sie im Terminalfenster, in dem die App ausgeführt wird, **STRG+C** drücken. Sie können die Benutzeroberfläche geöffnet lassen.

## Integrieren der Vektorsuche

Bisher haben Sie Copilot die Möglichkeit gegeben, Aktionen zum Anwenden von Rabatten auf Produkte durchzuführen, aber er hat noch keine Kenntnis von den in der Datenbank gespeicherten Produkten. In dieser Aufgabe werden Sie Vektorsuchfunktionen hinzufügen, mit denen Sie nach Produkten mit bestimmten Eigenschaften suchen und ähnliche Produkte in der Datenbank finden können.

1. Kehren Sie zur `main.py`-Datei im `api/app`-Ordner zurück und geben Sie eine Methode für die Durchführung von Vektorsuchen im `Products`-Container in Ihrem Azure Cosmos DB-Konto an. Sie können diese Methode unterhalb der vorhandenen Funktionen am Ende der Datei einfügen.

   ```python
   async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
       """Search for similar product vectors in Azure Cosmos DB"""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
           FROM Products p
           WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
           ORDER BY VectorDistance(p.embedding, @query_embedding)
           """,
           parameters = [
               {"name": "@query_embedding", "value": query_embedding},
               {"name": "@num_results", "value": num_results},
               {"name": "@similarity_score", "value": similarity_score}
           ]
       )
       similar_products = []
       async for result in query_results:
           similar_products.append(result)
       formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
       return formatted_results
   ```

2. Als Nächstes erstellen Sie eine Methode mit dem Namen `get_similar_products`, die als Funktion dient, die vom LLM verwendet wird, um Vektorsuchen in Ihrer Datenbank durchzuführen:

   ```python
   async def get_similar_products(message: str, num_results: int):
       """Retrieve similar products based on a user message."""
       # Vectorize the message
       embedding = await generate_embeddings(message)
       # Perform vector search against products in Cosmos DB
       similar_products = await vector_search(embedding, num_results=num_results)
       return similar_products
   ```

    Die `get_similar_products`-Funktion führt asynchrone Aufrufe der `vector_search`-Funktion aus, die Sie oben definiert haben, sowie der `generate_embeddings`-Funktion, die Sie in der vorherigen Übung erstellt haben. In die eingehende Benutzernachricht werden Einbettungen generiert, damit sie mithilfe der integrierten `VectorDistance`-Funktion in Cosmos DB mit den in der Datenbank gespeicherten Vektoren verglichen werden kann.

3. Damit der LLM die neuen Funktionen verwenden kann, müssen Sie das zuvor erstellte `tools`-Array aktualisieren und eine Funktionsdefinition für die `get_similar_products`-Methode hinzufügen:

   ```json
   {
       "type": "function",
       "function": {
           "name": "get_similar_products",
           "description": "Retrieve similar products based on a user message.",
           "parameters": {
               "type": "object",
               "properties": {
                   "message": {"type": "string", "description": "The user's message looking for similar products"},
                   "num_results": {"type": "integer", "description": "The number of similar products to return"}
               },
               "required": ["message"]
           }
       }
   }
   ```

4. Sie müssen auch Code hinzufügen, um die Ausgabe der neuen Funktion zu verarbeiten. Fügen Sie dem Codeblock, der Funktionsaufrufe verarbeitet, die folgende `elif`-Bedingung hinzu:

   ```python
   elif call.function.name == "get_similar_products":
       func_response = await get_similar_products(**json.loads(call.function.arguments))
       messages.append(
           {
               "role": "tool",
               "tool_call_id": call.id,
               "name": call.function.name,
               "content": json.dumps(func_response)
           }
       )
   ```

    Der abgeschlossene Block sieht nun so aus:

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
           elif call.function.name == "get_similar_products":
               func_response = await get_similar_products(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

5. Zuletzt müssen Sie die Definition der Systemansage aktualisieren, um Anweisungen zur Durchführung von Vektorsuchen bereitzustellen. Fügen Sie am Ende des `system_prompt` Folgendes ein:

   ```plaintext
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   ```

    Die aktualisierte Eingabeaufforderung des Systems sieht in etwa so aus:

   ```python
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   """
   ```

6. Speichern Sie die Datei `main.py`.

## Testen der Vektorsuchfunktion

1. Starten Sie die API-App neu, indem Sie Folgendes im geöffneten integrierten Terminalfenster für diese App in Visual Studio Code ausführen:

   ```bash
   uvicorn main:app
   ```

2. Die Benutzeroberfläche sollte noch laufen, aber wenn Sie sie beendet haben, kehren Sie zum integrierten Terminalfenster zurück und führen Sie Folgendes aus:

   ```bash
   python -m streamlit run index.py
   ```

3. Kehren Sie zum Browserfenster zurück, in dem die Benutzeroberfläche ausgeführt wird, und geben Sie bei der Eingabeaufforderung des Chats Folgendes ein:

   ```bash
   Tell me about the mountain bikes in stock
   ```

    Diese Frage wird einige Produkte zurückgeben, die Ihrer Suche entsprechen.

4. Probieren Sie ein paar andere Suchanfragen aus, z. B. „Zeige mir haltbare Pedale“, „Liste mit 5 schicken Trikots“ und „Gib mir Details zu allen Handschuhen, die für das Fahren bei warmem Wetter geeignet sind“.

    Beachten Sie bei den letzten beiden Abfragen, dass die Produkte den zuvor angewendeten Rabatt von 15 % und den Verkaufspreis enthalten.
