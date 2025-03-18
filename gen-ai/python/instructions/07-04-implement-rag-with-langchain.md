---
lab:
  title: 07.4 - RAG mit LangChain und Azure Cosmos DB für die NoSQL-Vektorsuche implementieren
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# Implementieren Sie RAG mit LangChain und Azure Cosmos DB für die NoSQL-Vektorsuche

Die Orchestrierungsfunktionen von LangChain bieten eine Vielzahl von Vorteilen gegenüber der direkten Implementierung der LLM-Integration Ihres Copiloten mit dem Azure OpenKI-Client. LangChain ermöglicht eine nahtlose Integration in verschiedene Datenquellen, einschließlich Azure Cosmos DB, und ermöglicht eine effiziente Vektorsuche, die den Abrufvorgang verbessert. LangChain bietet robuste Tools zum Verwalten und Optimieren von Workflows und erleichtert das Erstellen komplexer Anwendungen mit modularen und wiederverwendbaren Komponenten. Diese Flexibilität vereinfacht nicht nur die Entwicklung, sondern sorgt auch für Skalierbarkeit und Wartung.

In dieser Übung verbessern Sie Ihren Copilot, indem Sie den Endpunkt Ihrer API von der Verwendung des Azure OpenKI-Clients `/chat` auf die leistungsstarken Orchestrierungsfunktionen von LangChain übertragen. Diese Verschiebung ermöglicht einen effizienteren Datenabruf und eine verbesserte Leistung durch die Integration der Vektorsuche-Funktion in Azure Cosmos DB for NoSQL. Ganz gleich, ob Sie den Prozess der Informationsbeschaffung Ihrer App optimieren oder einfach nur das Potenzial von RAG erkunden möchten, dieses Modul führt Sie durch die nahtlose Konvertierung und zeigt Ihnen, wie LangChain die Funktionen Ihrer App optimieren und erweitern kann. Lassen Sie uns diese Reise starten, um neue Effizienz und Einblicke mit LangChain und Azure Cosmos DB zu gewinnen!

> &#128721; Die vorherigen Übungen in diesem Modul sind Voraussetzung für dieses Lab. Wenn Sie noch eine dieser Übungen abschließen müssen, tun Sie dies bitte, bevor Sie fortfahren, da sie die notwendige Infrastruktur und den Startercode für dieses Lab bereitstellen.

## Installieren der LangChain-Bibliotheken

1. Öffnen Sie mit Visual Studio Code den Ordner, in den Sie das Lab-Coderepository für das Lernmodul **Erstellen von Copilots mit Azure Cosmos DB** geklont haben.

2. Navigieren Sie im Bereich **Explorer** in Visual Studio Code zum Ordner **python/07-build-copilot** und öffnen Sie die darin enthaltene Datei `requirements.txt`.

3. Aktualisieren Sie die `requirements.txt` Datei so, dass sie die erforderlichen LangChain-Bibliotheken enthält:

   ```ini
   langchain==0.3.9
   langchain-openai==0.2.11
   ```

4. Starten Sie ein neues integriertes Terminalfenster in Visual Studio Code, und ändern Sie Verzeichnisse in `python/07-build-copilot`.

5. Stellen Sie sicher, dass das integrierte Terminalfenster in Ihrer virtuellen Python-Umgebung ausgeführt wird, indem Sie es mit dem entsprechenden Befehl für Ihr Betriebssystem und Ihre Shell aus der folgenden Tabelle aktivieren:

    | Plattform | Shell | Befehl zum Aktivieren der virtuellen Umgebung |
    | -------- | ----- | --------------------------------------- |
    | POSIX | Bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

6. Aktualisieren Sie Ihre virtuelle Umgebung mit den LangChain-Bibliotheken, indem Sie den folgenden Befehl an der integrierten Terminalaufforderung ausführen:

   ```bash
   pip install -r requirements.txt
   ```

7. Schließen Sie das integrierte Terminal.

## Aktualisieren der Back-End-API

In der vorherigen Übung haben Sie ein RAG-Muster mithilfe des Azure OpenKI-Clients und der Daten von Azure Cosmos DB ausgeführt. Jetzt aktualisieren Sie die Back-End-API, um einen LangChain-Agent mit Tools zu verwenden, um dieselben Aktionen auszuführen.

Die Verwendung von LangChain zur Interaktion mit Sprachmodellen, die in Ihrem Azure OpenAI-Dienst eingesetzt werden, ist vom Code her etwas einfacher ...

1. Entfernen Sie die `from openai import AzureOpenAI` Import-Anweisung am Anfang der `main.py` Datei. Diese Clientbibliothek ist nicht mehr erforderlich, da alle Interaktionen mit Azure OpenAK über LangChain bereitgestellte Klassen durchlaufen.

2. Löschen Sie die folgenden Importanweisungen am Anfang der `main.py` Datei, da sie nicht mehr erforderlich sind:

   ```python
   from openai import AsyncAzureOpenAI
   import json
   ```

### Aktualisieren des Einbettungsendpunkts

1. Importieren Sie die `AzureOpenAIEmbeddings` Klasse aus der `langchain_openai` Bibliothek, indem Sie die folgende Importanweisung am Anfang der `main.py` Datei hinzufügen:

   ```python
   from langchain_openai import AzureOpenAIEmbeddings
   ```

2. Suchen Sie die `generate_embeddings` Methode in der Datei, und überschreiben Sie sie mit den folgenden, die die `AzureOpenAIEmbeddings` Klasse verwendet, um Interaktionen mit Azure OpenKI zu verarbeiten:

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Use LangChain's Azure OpenAI Embeddings class
       azure_openai_embeddings = AzureOpenAIEmbeddings(
           azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
       return await azure_openai_embeddings.aembed_query(text)
   ```

    Die `AzureOpenAIEmbeddings` Klasse stellt eine Schnittstelle für die Interaktion mit der Azure OpenKI Embeddings-API bereit und gibt ein vereinfachtes Antwortobjekt zurück, das nur den generierten Vektor enthält.

### Aktualisieren des Chatendpunkts

1. Aktualisieren Sie die `lanchain_openai` Importanweisung, um die `AzureChatOpenAI` Klasse anzufügen:

   ```python
   from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
   ```

1. Importieren Sie die folgenden zusätzlichen LangChain-Objekte, die beim Aufbau des überarbeiteten `/chat` Endpunkts verwendet werden:

   ```python
   from langchain.agents import AgentExecutor, create_openai_functions_agent
   from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
   from langchain_core.tools import StructuredTool
   ```

1. Der Chatverlauf wird mithilfe eines LangChain-Agent auf andere Weise in die Copilot-Konversation eingefügt. Löschen Sie daher die Codezeilen unmittelbar nach der `system_prompt`-Definition. Die Zeilen, die Sie löschen sollten, sind:

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{ "role": "system", "content": system_prompt }]

   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)

   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

1. Definieren Sie anstelle des soeben gelöschten Codes ein `prompt`-Objekt mithilfe der LangChain-Klasse `ChatPromptTemplate`:

   ```python
   prompt = ChatPromptTemplate.from_messages(
       [
           ("system", system_prompt),
           MessagesPlaceholder("chat_history", optional=True),
           ("user", "{input}"),
           MessagesPlaceholder("agent_scratchpad")
       ]
   )
   ```

    `ChatPromptTemplate` wird mit mehreren Komponenten in einer bestimmten Reihenfolge erstellt. Hier erfahren Sie, wie diese Teile zusammenpassen:

    - **Systemmeldung**: Verwendet das `system_prompt`, um dem Copilot eine Persona zu geben, die Anweisungen gibt, wie sich der Assistent verhalten und mit den Benutzenden interagieren soll.
    - **Chatverlauf**: Ermöglicht die Einbindung von `chat_history`, das eine Liste vergangener Nachrichten in der Unterhaltung enthält, in den Kontext, in dem der LLM arbeitet.
    - **Benutzereingabe**: Die aktuelle Benutzernachricht.
    - **Agent Scratchpad**: Ermöglicht Zwischennotizen oder Schritte, die vom Agent ausgeführt werden.

    Die daraus resultierende Eingabeaufforderung stellt eine strukturierte Eingabe für den Konversations-KI-Agent dar und hilft ihm, eine Antwort auf der Grundlage des gegebenen Kontexts zu generieren.

1. Als nächstes ersetzen Sie die `tools`-Array-Definition durch die folgende, die die `StructuredTool`-Klasse von LangChain verwendet, um Funktionsdefinitionen in das richtige Format zu extrahieren:

   ```python
   tools = [
       StructuredTool.from_function(coroutine=apply_discount),
       StructuredTool.from_function(coroutine=get_category_names),
       StructuredTool.from_function(coroutine=get_similar_products)
   ]
   ```

    Die `StructuredTool.from_function`-Methode in LangChain erstellt ein Werkzeug aus einer gegebenen Funktion unter Verwendung der Eingabeparameter und der Docstring-Beschreibung der Funktion. Um sie mit asynchronen Methoden zu verwenden, übergeben Sie den Funktionsnamen an den Eingabeparameter `coroutine`.

    In Python ist ein Docstring (kurz für Dokumentationszeichenfolge) ein spezieller Ziechenfolgetyp, der zur Dokumentation einer Funktion, Methode, Klasse oder eines Moduls verwendet wird. Er bietet eine bequeme Möglichkeit, Dokumentation mit Python-Code zu verknüpfen und wird normalerweise in dreifache Anführungszeichen (""" oder ''') gesetzt. Docstrings werden unmittelbar nach der Definition der Funktion (oder Methode, Klasse oder Modul) platziert, die sie dokumentieren.

    Die Verwendung dieser Funktion automatisiert die Erstellung der JSON-Funktionsdefinitionen, die Sie manuell mit dem Azure OpenAI-Client erstellen mussten, und vereinfacht den Prozess des Funktionsaufrufs.

1. Löschen Sie den gesamten Code zwischen der `tools`-Array-Definition, die Sie oben abgeschlossen haben, und der `return`-Anweisung am Ende der Funktion. Mit dem Azure OpenAI-Client mussten Sie zwei Aufrufe des Sprachmodells durchführen. Den ersten, um festzustellen, welche Funktionsaufrufe gegebenenfalls zur Ergänzung der Eingabeaufforderung erforderlich sind, und den zweiten, um eine RAG-Vervollständigung zu verlangen. Dazwischen mussten Sie Code verwenden, um die Antwort des ersten Aufrufs zu prüfen, um festzustellen, ob Funktionsaufrufe erforderlich waren, und dann Code schreiben, um den Aufruf dieser Funktionen zu „verarbeiten“. Sie mussten dann die Ausgabe dieser Funktionsaufrufe in die an den LLM gesendeten Nachrichten einfügen, damit dieser die angereicherte Eingabeaufforderung als Grundlage für die Formulierung einer Abschlussantwort verwenden konnte. LangChain vereinfacht den Prozess des Aufrufs einer LLM mit Hilfe eines RAG-Musters erheblich, wie Sie weiter unten sehen werden. Der Code, den Sie entfernen sollten, ist:

   ```python
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

   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )

   return final_response.choices[0].message.content
   ```

1. Erstellen Sie direkt unterhalb der `tools`-Array-Definition einen Verweis auf die Azure OpenAI API mit der `AzureChatOpenAI`-Klasse in LangChain:

   ```python
   # Connect to Azure OpenAI API
   azure_openai = AzureChatOpenAI(
       azure_deployment=COMPLETION_DEPLOYMENT_NAME,
       azure_endpoint=AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
       api_version=AZURE_OPENAI_API_VERSION
   )
   ```

1. Damit Ihr LangChain-Agent mit den von Ihnen definierten Funktionen interagieren kann, erstellen Sie einen Agent mit der Methode `create_openai_functions_agent`, dem Sie das Objekt `AzureChatOpenAI`, das Array `tools` und das Objekt `ChatPromptTemplate` übergeben:

   ```python
   agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
   ```

    Die Funktion `create_openai_functions_agent` in LangChain erstellt einen Agent, der externe Funktionen aufrufen kann, um Aufgaben unter Verwendung eines bestimmten Sprachmodells und von Tools auszuführen. Dies ermöglicht die Integration verschiedener Dienste und Funktionen in den Workflow des Agents und bietet Flexibilität und erweiterte Möglichkeiten.

1. In LangChain wird die Klasse `AgentExecutor` verwendet, um den Ausführungsflow der Agents zu verwalten, wie etwa den, den Sie mit der Methode `create_openai_functions_agent` erstellt haben. Sie behandelt die Verarbeitung von Eingaben, den Aufruf von Tools oder Modellen und die Verarbeitung von Ausgaben. Verwenden Sie den folgenden Code, um einen Agent-Executor für Ihren Agent zu erstellen:

   ```python
   agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
   ```

    `AgentExecutor` stellt sicher, dass alle Schritte, die zum Generieren einer Antwort erforderlich sind, in der richtigen Reihenfolge ausgeführt werden. Es abstrahiert die Komplexität der Ausführung für Agents, bietet eine zusätzliche Funktions- und Strukturebene und erleichtert die Erstellung, Verwaltung und Skalierung anspruchsvoller Agents.

1. Sie werden die Methode `invoke` des Agenten-Executors verwenden, um die eingehende Benutzernachricht an den LLM zu senden. Sie schließen auch den Chatverlauf ein. Fügen Sie den folgenden Code unter der Definition von `agent_executor` ein:

   ```python
   completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
   ```

   Die Token `input` und `chat_history` wurden in dem mit `ChatPromptTemplate` erstellte Eingabeaufforderungs-Objekt definiert. Mit der Methode `invoke` werden diese in die Eingabeaufforderung eingefügt, so dass der LLM diese Informationen bei der Erstellung einer Antwort verwenden kann.

1. Aktualisieren Sie schließlich die return-Anweisung, um die `output` des Abschlussobjekts des Agent zu verwenden:

   ```python
   return completion["output"]
   ```

1. Speichern Sie die Datei `main.py`. Die aktualisierte `/chat`-Endpunktfunktion sollte nun wie folgt aussehen:

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
       When asked to provide a list of products, you should:
           - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
       """
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", system_prompt),
               MessagesPlaceholder("chat_history", optional=True),
               ("user", "{input}"),
               MessagesPlaceholder("agent_scratchpad")
           ]
       )
    
       # Define function calling tools
       tools = [
           StructuredTool.from_function(apply_discount),
           StructuredTool.from_function(get_category_names),
           StructuredTool.from_function(get_similar_products)
       ]
    
       # Connect to Azure OpenAI API
       azure_openai = AzureChatOpenAI(
           azure_deployment=COMPLETION_DEPLOYMENT_NAME,
           azure_endpoint=AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
           api_version=AZURE_OPENAI_API_VERSION
       )
    
       agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
       agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
        
       completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
            
       return completion["output"]
   ```

## Starten der API- und UI-Apps

1. Öffnen Sie zum Starten der API ein neues integriertes Terminalfenster in Visual Studio Code.

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

1. Öffnen Sie ein neues integriertes Terminalfenster, wechseln Sie in das Verzeichnis `python/07-build-copilot`, um Ihre Python-Umgebung zu aktivieren, wechseln Sie dann in den Ordner `ui` und führen Sie Folgendes aus, um Ihre UI-App zu starten:

   ```bash
   python -m streamlit run index.py
   ```

1. Wenn die Benutzeroberfläche nicht automatisch in einem Browserfenster geöffnet wird, starten Sie einen neuen Browser-Tab oder ein neues Browserfenster und navigieren Sie zu <http://localhost:8501>, um die Benutzeroberfläche zu öffnen.

## Copilot testen

1. Bevor Sie Nachrichten an die Benutzeroberfläche senden, kehren Sie zu Visual Studio Code zurück und wählen Sie das integrierte Terminalfenster aus, das mit der API-App verknüpft ist. In diesem Fenster sehen Sie die „ausführliche“ Ausgabe, die vom LangChain-Agent-Executor generiert wird und Einblicke in die Verarbeitung der von Ihnen gesendeten Anfragen durch LangChain bietet. Achten Sie auf die Ausgabe in diesem Fenster, während Sie die untenstehenden Anfragen senden, und melden Sie sich nach jedem Anruf zurück.

1. Geben Sie in der Eingabeaufforderung des Chats in der Benutzeroberfläche „Rabatt anwenden“ ein und senden Sie die Nachricht.

    Sie sollten eine Antwort erhalten, in der Sie gefragt werden, welchen Rabattprozentsatz Sie anwenden möchten und für welche Produktkategorie.

1. Antwort: „Handschuhe“

    Sie erhalten eine Antwort, in der Sie gefragt werden, welchen Rabattprozentsatz Sie auf die Kategorie „Handschuhe“ anwenden möchten.

1. Senden einer Nachricht von „25 %“.

    Sie sollten die Antwort „Ein Rabatt von 25 % wurde erfolgreich auf alle Produkte in der Kategorie ‚Handschuhe‘ angewendet“ erhalten.

1. Bitten Sie Copilot, „alle Handschuhe zu zeigen“.

    In der Antwort sollten Sie eine Liste aller Handschuhe in der Datenbank sehen, die den Rabattpreis von 25 % enthält.

1. Zum Schluss fragen Sie: „Welche Handschuhe eignen sich am besten für Fahrten bei kaltem Wetter?“ Durchführen einer Vektorsuche Dies umfasst einen Funktionsaufruf der `get_similar_items`-Methode, die dann sowohl die `generate_embeddings`-Methode, die Sie aktualisiert haben, um eine LangChain-Implementierung zu verwenden, als auch die `vector_search`-Funktion aufruft.

1. Schließen Sie das integrierte Terminal.

1. Schließen Sie **Visual Studio Code**.
