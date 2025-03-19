---
lab:
  title: Lab-Umgebung einrichten
  module: Setup
---

# Einrichten einer lokalen Labumgebung

Idealerweise sollten Sie diese Labs in einer gehosteten Labumgebung absolvieren. Wenn Sie sie auf Ihrem eigenen Computer durchführen möchten, können Sie dazu die folgende Software installieren. Bei der Verwendung Ihrer eigenen Umgebung können unerwartete Dialoge und Verhaltensweisen auftreten. Aufgrund der Vielzahl möglicher lokaler Konfigurationen kann das Kursteam keine Unterstützung für Probleme leisten, die in Ihrer eigenen Umgebung auftreten können.

## Azure-Befehlszeilentools

1. [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) oder [Azure Cloud Shell](https://shell.azure.com) - Installieren Sie, wenn Sie Befehle über die CLI statt über das Azure-Portal ausführen möchten.

## Node.js

1. Laden Sie Node.js v18.0.0 oder höher von [nodejs.org/en/download] herunter, und installieren Sie ihn.

1. Laden Sie NPM v10.2.3 oder höher von [npmjs.com/get-npm] herunter und installieren Sie es.

Empfohlene Methode zum Installieren der neuesten Version von NPM und node.js unter Windows:

- Installieren von NVM aus [github.com/coreybutler/nvm-windows]
- Ausführen der neuesten NVM-Installation
- Führen Sie nvm list aus (um die verfügbaren Versionen von NPM/node.js zu sehen)
- Ausführen der zuletzt verwendeten nvm (um die neueste verfügbare Version zu verwenden)

### Git

1. Herunterladen und Installieren von [git-scm.com/downloads].

    - Verwenden Sie die Standardoptionen im Installationsprogramm.

### Visual Studio Code (und Erweiterungen)

1. Laden Sie [code.visualstudio.com/download] herunter und installieren Sie sie.

    - Verwenden Sie die Standardoptionen im Installationsprogramm.

1. Starten Sie nach der Installation Visual Studio Code.

### Azure Cosmos DB-Emulator

1. Herunterladen und Installieren von [docs.microsoft.com/azure/cosmos-db/local-emulator].
    - Verwenden Sie die Standardoptionen im Installationsprogramm.

### Klonen des Lab-Repositorys

Wenn Sie das Labcode-Repository für **Copilots mit Azure Cosmos DB erstellen** noch nicht in die Umgebung geklont haben, in der Sie an diesem Lab arbeiten, führen Sie die folgenden Schritte aus, um dies zu tun. Öffnen Sie andernfalls den zuvor geklonten Ordner in **Visual Studio Code**.

1. Starten Sie **Visual Studio Code**.

    > &#128221; Wenn Sie mit der Visual Studio Code-Schnittstelle noch nicht vertraut sind, lesen Sie das [Handbuch „Erste Schritte“ für Visual Studio Code][code.visualstudio.com/docs/getstarted].

1. Öffnen Sie die Befehlspalette, und führen Sie den Befehl **Git: Clone** aus, um das GitHub-Repository ``https://github.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs`` in einem lokalen Ordner Ihrer Wahl zu klonen.

    > &#128161; Sie können die Tastenkombination **STRG+UMSCHALTTASTE+P** verwenden, um die Befehlspalette zu öffnen.

1. Nachdem das Repository geklont wurde, öffnen Sie den lokalen Ordner, den Sie in **Visual Studio Code** ausgewählt haben.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks

[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[code.visualstudio.com/download]: https://code.visualstudio.com/download
[git-scm.com/downloads]: https://git-scm.com/downloads
[nodejs.org/en/download]: https://nodejs.org/en/download
[npmjs.com/get-npm]: https://npmjs.com/get-npm
[github.com/coreybutler/nvm-windows]: https://github.com/coreybutler/nvm-windows
