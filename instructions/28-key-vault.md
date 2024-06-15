---
lab:
  title: "Speichern von Azure Cosmos\_DB for NoSQL-Kontoschlüsseln in Azure Key Vault"
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# Speichern von Azure Cosmos DB for NoSQL-Kontoschlüsseln in Azure Key Vault

Für das Hinzufügen eines Azure Cosmos DB-Kontoverbindungscodes zu Ihrer Anwendung müssen Sie nur den URI und den Schlüssel des Kontos angeben. Diese Sicherheitsinformationen werden manchmal in den Anwendungscode hartcodiert. Wenn Ihre Anwendung jedoch in Azure App Service bereitgestellt wird, können Sie die Verschlüsselungsverbindungsinformationen in Azure Key Vault speichern.

In dieser Übung verschlüsseln und speichern wir die Azure Cosmos DB-Kontoverbindungszeichenfolge in Azure Key Vault. Anschließend erstellen wir eine Azure App Service-Web-App, die diese Anmeldeinformationen aus Azure Key Vault abruft. Die Anwendung verwendet diese Anmeldeinformationen und stellt eine Verbindung mit dem Azure Cosmos DB-Konto her. Die Anwendung erstellt dann einige Dokumente in den Azure Cosmos DB-Kontocontainern und gibt ihren Status wieder an eine Webseite zurück.

## Vorbereiten Ihrer Entwicklungsumgebung

Wenn Sie das Labcoderepository **DP-420** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten werden, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie noch nicht mit der Visual Studio Code-Benutzeroberfläche vertraut sind, lesen Sie das Handbuch [Erste Schritte](code.visualstudio.com/docs/getstarted) für Visual Studio Code.

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALT+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, ***schließen*** Sie *Visual Studio Code*. Später öffnen wir es und verweisen direkt auf den Ordner **28-key-vault**.

## Erstellen eines Azure Cosmos DB for NoSQL-Kontos

Azure Cosmos DB ist ein cloudbasierter NoSQL-Datenbankdienst, der mehrere APIs unterstützt. Beim ersten Bereitstellen eines Azure Cosmos DB-Kontos wählen Sie aus, welche APIs das Konto unterstützen soll (z. B. **Mongo-API** oder **NoSQL-API**). Nachdem die Bereitstellung des Azure Cosmos DB for NoSQL-Kontos abgeschlossen ist, können Sie den Endpunkt und den Schlüssel abrufen. Verwenden Sie den Endpunkt und den Schlüssel, um programmgesteuert eine Verbindung mit dem Azure Cosmos DB for NoSQL-Konto herzustellen. Verwenden Sie den Endpunkt und den Schlüssel für die Verbindungszeichenfolgen des Azure SDK for .NET oder eines anderen SDK.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie **+ Ressource erstellen** aus, suchen Sie nach *Cosmos DB*, und erstellen Sie dann eine neue**Azure Cosmos DB for NoSQL**-Kontoressource mit den folgenden Einstellungen, wobei Sie die restlichen Einstellungen auf ihren Standardwerten belassen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe* |
    | **Account Name** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Location** | *Wählen Sie eine verfügbare Region aus.* |
    | **Kapazitätsmodus** | *Bereitgestellter Durchsatz* |
    | **Apply Free Tier Discount** (Free-Tarif anwenden) | *Nicht anwenden* |

    > &#128221; In Ihren Labumgebungen gibt es möglicherweise Einschränkungen, die verhindern, dass Sie eine neue Ressourcengruppe erstellen. Wenn dies der Fall ist, verwenden Sie die vorhandene bereits erstellte Ressourcengruppe.

1. Warten Sie, bis die Bereitstellungsaufgabe abgeschlossen ist, bevor Sie mit dieser Aufgabe fortfahren.

1. Wechseln Sie zur neu erstellten **Azure Cosmos DB**-Kontoressource, und navigieren Sie zum Bereich **Schlüssel**.

1. Dieser Bereich enthält die Verbindungsdetails und Anmeldeinformationen, die zum Herstellen einer Verbindung mit dem Konto aus dem SDK erforderlich sind. Insbesondere das Feld **PRIMÄRE VERBINDUNGSZEICHENFOLGE**. Sie verwenden diese **Verbindungszeichenfolge** weiter unten in dieser Übung.

## Erstellen einer Azure Key Vault-Instanz und Speichern der Azure Cosmos DB-Kontoanmeldeinformationen als Geheimnis

Bevor wir unsere Web-App erstellen, sichern wir die Verbindungszeichenfolge des Azure Cosmos DB-Kontos, indem wir sie in ein durch *Azure Key Vault* verschlüsseltes *Geheimnis* kopieren. Das werden wir jetzt tun.

1. Navigieren Sie auf einer neuen Browserregisterkarte zum Azure-Portal, und öffnen Sie die Seite **Schlüsseltresore**.

1. Fügen Sie einen Tresor hinzu, indem Sie die Schaltfläche ***+ Erstellen*** auswählen und den Tresor mit den folgenden Einstellungen konfigurieren. Behalten Sie für *alle verbleibenden Einstellungen die Standardwerte bei*, und wählen Sie dann Folgendes aus, um den Tresor zu erstellen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Abonnement** | *Ihr vorhandenes Azure-Abonnement* |
    | **Ressourcengruppe** | *Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie eine neue Ressourcengruppe*. |
    | **Name des Schlüsseltresors** | *Geben Sie einen global eindeutigen Namen ein.* |
    | **Region** | *Wählen Sie eine verfügbare Region aus.* |
    | **Zugriffskonfiguration/Berechtigungsmodell** | *Tresorzugriffsrichtlinie* |
    | **Zugriffskonfiguration/Zugriffsrichtlinien** | *Kontrollkästchen für den aktuellen Benutzernamen* |

    > &#128221; Beachten Sie, dass Sie in einer Produktionsumgebung wahrscheinlich die RBAC-Steuerung anstelle der Tresorzugriffsrichtlinie auswählen würden, und Ihr Administrator weist Ihnen höchstwahrscheinlich die geeignete RBAC-Rolle zu, um den Key Vault-Zugriff einzuschränken.

1. Nachdem der Tresor erstellt wurde, navigieren Sie zu diesem.

1. Wählen Sie im Abschnitt *Objekte* die Option **Geheimnisse** aus.

1. Wählen Sie die **+ Generieren/Importieren** aus, um unsere Verbindungszeichenfolge für Anmeldeinformationen zu verschlüsseln, und konfigurieren Sie die *Geheimniswerte* mit den folgenden Einstellungen. Behalten Sie für *alle verbleibenden Einstellungen die Standardwerte bei*, und wählen Sie dann Folgendes aus, um das Geheimnis zu erstellen:

    | **Einstellung** | **Wert** |
    | ---: | :--- |
    | **Uploadoptionen** | *Manuell* |
    | **Name** | *Den Namen für Ihr Geheimnis* |
    | **Wert** | *Dieses Feld ist das wichtigste auszufüllende Feld. Kopieren Sie hier den Wert von „PRIMÄRE VERBINDUNGSZEICHENFOLGE“ aus dem Schlüsselabschnitt Ihres Azure Cosmos DB-Kontos. Dieser Wert wird in ein Geheimnis konvertiert.* |
    | **Aktiviert** | *Ja* |
 
1. Unter den Geheimnissen sollte jetzt Ihr neues Geheimnis aufgeführt sein. Wir müssen den *geheimen Bezeichner* abrufen, den wir dem Code unserer Web-App hinzufügen werden. Wählen Sie das erstellte **Geheimnis** aus.

1. Azure Key Vault ermöglicht es Ihnen, mehrere Versionen Ihres Geheimnis zu erstellen, aber für dieses Lab benötigen wir nur eine Version. Wählen Sie die **aktuelle Version** aus.

1. Notieren Sie den Wert des Felds **Geheimnisbezeichner**. Dieser Wert wird im Code unserer Anwendung verwendet, um das Geheimnis aus Key Vault abzurufen.  Beachten Sie, dass dieser Wert eine URL ist. Es ist ein weiterer Schritt nötig, damit dieses Geheimnis ordnungsgemäß funktioniert, aber wir erledigen diesen Schritt etwas später.

## Erstellen einer Azure App Service-Web-App

Wir erstellen eine Web-App, die eine Verbindung mit dem Azure Cosmos DB-Konto herstellt und einige Container und Dokumente erstellt. Wir werden nicht die *Azure Cosmos DB-Anmeldeinformationen* in dieser App hartcodieren, sondern den **Geheimnisbezeichner** aus dem Schlüsseltresor. Wir werden sehen, dass dieser Bezeichner ohne die richtigen Rechte nutzlos ist, die der Web-App in der Azure-Schicht zugewiesen sind. Beginnen wir mit dem Programmieren.



1. Öffnen Sie **Visual Studio Code**.  Öffnen Sie den Ordner **28-key-vault**, indem Sie „Datei“ > „Ordner öffnen“ auswählen und nach dem Ordner **28-key-vault** suchen.

    > &#128221; Beachten Sie, dass nur der Ordner **28-key-vault** und die zugehörigen Dateien und Unterordner in der **Explorer**-Struktur angezeigt werden. Wenn Sie das gesamte zuvor geklonte GitHub-Repository sehen können, ***schließen Sie Visual Studio Code***, und öffnen Sie es direkt im Ordner **28-key-vault** erneut.  Die Web-App funktioniert nicht ordnungsgemäß, wenn dieses Verzeichnis nicht um das Stammverzeichnis für Projekte ist. Vergewissern Sie sich daher, dass Sie nur den Ordner **28-key-vault** und die zugehörigen Dateien und Unterordner in der **Explorer**-Struktur sehen können.

1. Öffnen Sie das Kontextmenü für den Ordner **28-key-vault**, und wählen Sie dann die Option **In integriertem Terminal öffnen** aus, um eine neue Terminalinstanz zu öffnen.

    > &#128221; Mit diesem Befehl wird das Terminal geöffnet, wobei das Startverzeichnis bereits auf den Ordner **28-key-vault** festgelegt ist.

1. Erstellen wir nun eine MVC-Web-App-Shell. Wir werden gleich einige der generierten Dateien ersetzen. Führen Sie den folgenden Befehl aus, um die Web-App zu erstellen:

    ```
    dotnet new mvc
    ```


    > &#128221; Mit diesem Befehl wurde die Shell einer Web-App erstellt, wodurch mehrere Dateien und Verzeichnisse hinzugefügt wurden. Wir haben bereits ein paar Dateien mit dem gesamten Code, den wir benötigen. 

1. Ersetzen Sie die Dateien **.\Controllers\HomeController.cs** und **.\Views\Home\Index.cshtml** durch die entsprechenden Dateien aus dem Verzeichnis **.\KeyvaultFiles**.

1. Nachdem Sie die Dateien ersetzt haben, ***löschen*** Sie das Verzeichnis **.\KeyvaultFiles**.

## Importieren der mehreren fehlenden Bibliotheken in das .NET-Skript

Die .NET-CLI enthält den Befehl [Paket hinzufügen](docs.microsoft.com/dotnet/core/tools/dotnet-add-package) zum Importieren von Paketen aus einem vorkonfigurierten Paketfeed. Eine .NET-Installation verwendet NuGet als Standardpaketfeed.

1. Fügen Sie das Paket [Microsoft.Azure.Cosmos](nuget.org/packages/microsoft.azure.cosmos/3.22.1) aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. Fügen Sie das Paket [Newtonsoft.Json](nuget.org/packages/Newtonsoft.Json/13.0.1) aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. Fügen Sie das Paket [Microsoft.Azure.KeyVault](nuget.org/packages/Microsoft.Azure.KeyVault) aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. Fügen Sie das Paket [Microsoft.Azure.Services.AppAuthentication](nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) aus NuGet mithilfe des folgenden Befehls hinzu:

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.6.2
    ```

## Hinzufügen des Geheimnisbezeichners zu Ihrer Web-App

1. Öffnen Sie in Visual Studio die Datei `.\Controllers\HomeControler.cs`.

1. Die benutzerdefinierte Funktion **GetKeyVaultSecret** ruft das Azure Cosmos DB-Kontogeheimnis ab. Die Funktion beginnt in *Zeile 98* und sollte wie das folgende Skript aussehen.

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. Sehen wir uns die wichtigen Aufrufe dieser Funktion an.

    - In *Zeile 100* definieren wir das Token der aktuellen Web-App. Dieses Token wird für Azure Key Vault bereitgestellt, um zu ermitteln, welche App versucht, auf den Tresor zuzugreifen. 
    - In *Zeile 104 und 105* bereiten wir den *Key Vault-Client* vor, der eine Verbindung mit Azure Key Vault herstellt. Beachten Sie, dass wir das Web-App-Token als Parameter senden. 
    - In *Zeile 107 und 108*stellen wir den Key Vault-Client mit der URL-Adresse unseres **Geheimnisbezeichners** bereit, der das in diesem Schlüsseltresor gespeicherten Geheimnis zurückgibt. 

1.  Bevor wir unsere Web-App bereitstellen können, müssen wir weiterhin die **Geheimnisbezeichner**-URL senden.  Ersetzen Sie in *Zeile 107* die Zeichenfolge ***<Key Vault Secret Identifier>*** durch die **Geheimnisbezeichner**-URL, die wir im Abschnitt *Geheimnis* notiert haben, und speichern Sie die Datei.

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## (Optional) Installieren der Azure App Service-Erweiterung

Wenn Sie in Visual Studio die Befehlspalette (**STRG+UMSCHALT+P**) aufrufen und beim Suchen nach Azure App-Ressourcenbefehlen nichts zurückgeben wird, müssen wir die Erweiterung installieren.

1. Wählen Sie im linken Menü von Visual Studio Code die Option **Erweiterungen** aus.

1. Suchen Sie in der Suchleiste nach Azure App Service, und wählen Sie die entsprechende Erweiterung aus.

1. Wählen Sie die Schaltfläche „Installieren“ aus, um sie zu installieren.

1. Schließen Sie die Registerkarte **Erweiterungen**, und wechseln Sie zurück zum Code.

## Bereitstellen der Anwendung in Azure App Service

Der restliche Codes ist recht einfach: Er ruft die Verbindungszeichenfolge ab, stellt eine Verbindung mit Azure Cosmos DB her und fügt einige Dokumente hinzu. Die Anwendung sollte uns auch Feedback zu Problemen geben. Nach der Bereitstellung der Anwendung sollten keine weiteren Änderungen vorgenommen werden müssen. Los geht's. 

> &#128221; Die meisten der folgenden Schritte erfolgen in der Befehlspalette (**STRG+UMSCHALT+P**) in der oberen Mitte des Visual Studio-Bildschirms. 

1. Öffnen Sie in Visual Studio Code die Befehlspalette, und suchen Sie nach ***Azure App Service: Neue Web-App erstellen... (erweitert)***.

1. Wählen Sie ***Anmelden bei Azure...*** aus. Diese Option öffnet ein Webbrowserfenster, führt den Anmeldevorgang aus und schließt den Browser, wenn er fertig ist, und kehrt zu Visual Studio Code zurück.

1. (Optional) Wenn nach Ihrem Abonnement gefragt wird, wählen Sie Ihr Abonnement aus.

1. Geben Sie einen global eindeutigen Namen für Ihre Web-App ein.

1. Wählen Sie eine vorhandene Ressourcengruppe aus, oder erstellen Sie bei Bedarf eine neue Ressourcengruppe.

1. Wählen Sie **.NET 6 (LTS)** aus.

1. Wählen Sie **Windows** aus.

1. Wählen Sie einen verfügbaren Standort aus.

1. Wählen Sie **+ Neuen App Service Plan erstellen** aus.

1. Übernehmen Sie den Standardnamen für den App Service-Plan (sollte dem Namen Ihrer Web-App entsprechen), oder wählen Sie einen neuen Namen aus.

1. Wählen Sie **Free (F1) Azure kostenlos testen** aus.

1. Wählen Sie für Application Insights die Option **Vorerst überspringen** aus.

1. Die Bereitstellung sollte jetzt mit einer Statusleiste in der unteren rechten Ecke ausgeführt werden. 

1. Wählen Sie **Bereitstellen**, wenn Sie dazu aufgefordert werden.

1. Wählen Sie **Durchsuchen** aus. Sie sollten sich im Ordner **28-key-vault** befinden. Wählen Sie diesen Ordner aus.

1. Ein Popup mit der Meldung **Erforderliche Konfiguration für die Bereitstellung fehlt in „28-key-vault“** sollte angezeigt werden. Wählen Sie die Schaltfläche „Konfiguration hinzufügen“ aus.  Mit dieser Option wird der fehlende Ordner `.vscode` erstellt.

    > &#128221; Sehr wichtig: Wenn dieses Popup nicht bei Ihrer ersten Bereitstellung der App angezeigt wird, fehlen beim Upload in Azure App Service Dateien. Die Bereitstellung ist erfolgreich, aber die Website gibt immer die Meldung *Sie sind nicht berechtigt, dieses Verzeichnis oder diese Seite anzuzeigen.* zurück. Die wahrscheinlichste Ursache ist, dass Visual Studio Code im geklonten GitHub-Repository geöffnet wurde, anstatt nur im Ordner **28-key-vault**.

1. Wählen Sie **Ja** aus, wenn Sie aufgefordert werden, immer in diesem Arbeitsbereich bereitzustellen.

1. Wählen Sie **Website durchsuchen** aus, wenn Sie dazu aufgefordert werden.  Alternativ können Sie einen Browser öffnen und zu **`https://<yourwebappname>.azurewebsites.net`** navigieren. In beiden Fällen haben wir ein Problem. Auf unserer Webseite wird eine benutzerdefinierte Meldung angezeigt. Die Meldung sollte **Key Vault nicht zugänglich** mit einer erweiterten Fehlermeldung lauten. Das ändern wir gleich.

## Zulassen, dass die App eine verwaltete Identität verwendet

Das erste Problem, das wir beheben müssen, besteht darin, dass unsere App eine verwaltete Identität verwendet. Die Verwendung einer verwalteten Identität ermöglicht es unserer App, Azure Services wie Azure Key Vault zu verwenden.

1. Öffnen Sie Ihren Browser, und melden Sie sich beim Azure-Portal an.

1. Öffnen Sie die Seite **App Services**. Ihr Web-App-Name sollte aufgelistet sein. Wählen Sie ihn aus.

1. Wählen Sie im Abschnitt *Einstellungen* die Option **Identität** aus.

1. Wählen Sie unter „Status“ **Ein** und **Speichern** aus.  Wählen Sie **Ja** aus, wenn Sie aufgefordert werden, die *zugewiesene verwaltete Identität* zu aktivieren.

1. Probieren wir unsere Web-App noch einmal aus.  Navigieren Sie im Browser zu **`https://<yourwebappname>.azurewebsites.net`**.

1. Es gibt immer noch ein Problem. Während es sich bei der ersten Meldung um eine benutzerdefinierte Meldung handelt, die unser Programm sendet, ist die zweite eine vom System generierte Meldung. Die zweite Meldung bedeutet, dass uns Zugriff auf die Verbindung mit dem Schlüsseltresor erteilt wurde, aber nicht, um das Geheimnis im Tresor anzuzeigen.  Legen wir eine letzte Einstellung fest, um dieses Problem zu beheben.

## Erteilen einer Zugriffsrichtlinie für die Schlüsseltresorschlüssel für die Web-App

Das ursprüngliche Ziel dieses Labs bestand darin, zu verhindern, dass unsere Azure Cosmos DB-Konten in unseren Anwendungen hartcodiert werden. Aber wir haben unsere **Geheimnisbezeichner**-URL hartcodiert, die jeder sehen kann. Wie können wir unsere Anmeldeinformationen schützen? Die gute Nachricht ist, dass der Geheimnisbezeichner allein nutzlos ist. Der **Geheimnisbezeichner** bringt Sie nur zur Tür von Azure Key Vault, aber der Tresor entscheidet, wer eintreten darf und wer vor der Tür bleibt. Das bedeutet, dass wir eine Key Vault-Zugriffsrichtlinie für unsere Anwendung erstellen müssen, damit sie die Geheimnisse in diesem Tresor sehen kann. Sehen wir uns diese Lösung an.

1. (Optional) Bevor wir die Richtlinie erstellen, überprüfen wir den aktuellen Inhalt unserer Azure Cosmos DB-Datenbank.  Wechseln Sie im Azure-Portal zu Ihrem Azure Cosmos DB-Konto. Ist die **GlobalCustomers**-Datenbank vorhanden? Wenn sie nicht vorhanden ist, wird sie von der erfolgreichen Ausführung der Web-App erstellt. Wenn sie vorhanden ist, überprüfen Sie die Anzahl der Elemente in der Datenbank, und die erfolgreiche Ausführung der Web-App fügt weitere Elemente hinzu.

1. Wechseln Sie im Azure-Portal zum zuvor erstellten Schlüsseltresor.

1. Wählen Sie im Abschnitt *Einstellungen* die Option **Zugriffskonfiguration** aus.

1. Vergewissern Sie sich, dass **Tresorzugriffsrichtlinie** ausgewählt ist, und wählen Sie dann **Zu Zugriffsrichtlinien wechseln** aus.

1. Wählen Sie **+ Erstellen** aus.

1. Aktivieren Sie auf der Registerkarte **Berechtigungen** das Kontrollkästchen **Abrufen** für **Schlüsselberechtigungen** und **Geheimnisberechtigungen**, und wählen Sie dann **Weiter** aus.

1. Geben Sie auf der Registerkarte **Prinzipal** im Suchfeld den Namen ein, den Sie Ihrer App Service-Instanz gegeben haben, wählen Sie ihn aus der Liste aus, und wählen Sie dann **Weiter** aus.

1. Wählen Sie auf der Registerkarte **Anwendung (optional)** **Weiter** aus.
    
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.

1. Probieren wir unsere Web-App noch einmal aus.  Navigieren Sie im Browser zu **`https://<yourwebappname>.azurewebsites.net`**.

1. Erfolg! Unsere Webseite sollte angeben, dass wir neue Elemente in den Kundencontainer eingefügt haben. Wir sehen auch, dass das tatsächliche Geheimnis angezeigt wird.

    > &#128221; In einer Produktionsumgebung dürfen Sie **nie** das Geheimnis anzeigen, hier dient das nur zur Veranschaulichung.


1. Wechseln Sie zu Ihrem Azure Cosmos DB-Konto, und stellen Sie sicher, dass entweder eine neue **GlobalCustomers**-Datenbank mit Daten vorhanden ist, oder, falls die Datenbank bereits vorhanden war, jetzt weitere Elemente in der Datenbank vorhanden sind.

Wir haben Azure Key Vault jetzt erfolgreich verwendet, um die Schlüssel Ihres Azure Cosmos DB-Kontos zu schützen.
