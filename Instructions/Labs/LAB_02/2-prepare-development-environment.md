---
lab:
  title: Vorbereiten Ihrer Entwicklungsumgebung
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Vorbereiten Ihrer Entwicklungsumgebung

Zunächst sollten Sie Ihre Entwicklungsumgebung, Ihre Konten und Ihre Software vorbereiten. Bevor Sie beginnen können, müssen Sie die folgenden Aufgaben erledigen.

## Aufgabe 1 - Voraussetzungen installieren

> [!IMPORTANT]
> Um dieses Projekt erfolgreich abzuschließen, benötigen Sie ein Microsoft 365-Konto mit der Berechtigung zum Hochladen von Anwendungen. Um **Übung 2** abzuschließen, muss das Konto auch für Microsoft Copilot für Microsoft 365 lizenziert sein.

Wenn Sie einen neuen Mandanten verwenden, ist es eine gute Idee, sich auf der [Microsoft 365-Seite](https://office.com) unter [https://office.com](https://office.com) anzumelden, bevor Sie beginnen. Je nachdem, wie der Mandant konfiguriert ist, werden Sie möglicherweise aufgefordert, eine Mulit-Faktor-Authentifizierung einzurichten. Stellen Sie sicher, dass Sie auf Microsoft Teams und Microsoft Outlook zugreifen können, bevor Sie fortfahren.

Die folgenden Tools wurden bereits im Lab auf **MS-4010-CLIENT01** installiert. Bitte vergewissern Sie sich, dass sie installiert und funktionsfähig sind:

1. [Visual Studio Code](https://code.visualstudio.com/) (neueste Version)

1. [Azure Storage Explorer](https://azure.microsoft.com/products/storage/storage-explorer/) - Laden Sie diesen herunter, wenn Sie die in diesem Beispiel verwendete Northwind-Datenbank anzeigen und bearbeiten möchten.

<!--## Task 2 - Install nvm-windows

You'll use this tool to install Node.js and optionally switch Node versions as needed for your projects.

1. In a web browser, navigate to [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases).
2. Locate the latest release version and select the **nvm-setup.zip** file to download.  The file will be downloaded to your machine.
3. Open the file folder and **extract** the contents of the zip folder to a folder on your machine.
4. From the new folder, select **nvm-setup.exe** to open the setup file.
5. Follow the prompts in the installer to install the tool using the default options.
6. Nvm for Windows will be installed on your machine.

## Task 3 - Install Node.js

Install Node.js version 18.18.2, which is compatible with all of the solutions in this course.

1. Open the **Command Prompt** application.
2. Enter the command `nvm install 18.18` to install Node.js.
3. The nvm output should confirm that installation is complete.
4. Run the command `nvm use 18.18` to use this version of Node.js.
5. Run the command `node -v` to confirm that you have version 18.18.2 installed.

You have now installed and configured Node.js version 18.18.2-->

## Aufgabe 2 - Herunterladen des Beispielcodes

Sie können das Beispiel-Repository als ZIP-Datei [herunterladen](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/) und es in Ihren **Dokumentenordner** entpacken:

```text
https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/
```

Navigieren Sie in diesem Verzeichnis zum Ordner **samples/msgext-northwind-inventory-ts**. In diesen Labs wird dies als "**Arbeitsordner**" bezeichnet, da Sie dort arbeiten werden.

## Aufgabe 3 - Kopieren von Beispieldokumenten in OneDrive

Die Beispielanwendung enthält einige Dokumente, auf die Copilot während der Labs Bezug nehmen kann. In dieser Aufgabe kopieren Sie diese Dateien in das OneDrive Ihres Benutzenden, damit Copilot sie finden kann. Je nachdem, wie der Mandant eingerichtet ist, werden Sie möglicherweise aufgefordert, im Rahmen dieses Prozesses eine Multi-Faktor-Authentifizierung einzurichten.

1. Öffnen Sie Ihren Browser und gehen Sie zu Microsoft 365 ([https://www.office.com/](https://www.office.com/)). Melden Sie sich mit dem Microsoft 365-Konto an, das Sie während des Labs verwenden werden. Möglicherweise werden Sie aufgefordert, eine Multi-Faktor-Authentifizierung einzurichten.

1. Navigieren Sie über das Apps-Menü in der oberen linken Ecke der Seite 1️⃣ zur Anwendung OneDrive innerhalb von Microsoft 365 2️⃣.

    ![Screenshot der Navigation zur OneDrive-Anwendung in Microsoft 365.](../media/1-02-copy-sample-files-01.png)

1. Navigieren Sie in OneDrive zu **Meine Dateien** 1️⃣. Wenn es einen Dokumentenordner gibt, navigieren Sie auch dorthin. Wenn nicht, können Sie direkt im Verzeichnis **Meine Dateien** arbeiten.

    ![Screenshot des Navigierens zu Ihren Dokumenten in OneDrive.](../media/1-02-copy-sample-files-02.png)

1. Wählen Sie nun **Neu hinzufügen** 1️⃣ und **Ordner** 2️⃣, um einen neuen Ordner zu erstellen.

    ![Screenshot des Hinzufügens eines neuen Ordners in OneDrive.](../media/1-02-copy-sample-files-03.png)

1. Benennen Sie den Ordner **Nordwind Verträge** und wählen Sie **Erstellen**.

    ![Screenshot der Benennung des neuen Ordners „Northwind Verträge“.](../media/1-02-copy-sample-files-03-b.png)

1. Wählen Sie nun in diesem neuen Ordner erneut **Neu hinzufügen** 1️⃣, aber diesmal wählen Sie **Dateien hochladen** 2️⃣.

    ![Screenshot des Hinzufügens neuer Dateien zum neuen Ordner.](../media/1-02-copy-sample-files-04.png)

1. Wechseln Sie nun zum Ordner **sampleDocs** in Ihrem **Arbeitsordner**. Markieren Sie alle Dateien 1️⃣ und wählen Sie **OK** 2️⃣, um sie alle hochzuladen.

    ![Screenshot des Hochladens der Beispieldateien aus diesem Repository in den Ordner.](../media/1-02-copy-sample-files-05.png)

Wenn Sie diese Aufgabe frühzeitig erledigen, ist die Wahrscheinlichkeit groß, dass die Microsoft 365-Suchmaschine sie bereits entdeckt hat, wenn Sie sie brauchen.

## Aufgabe 4: Konfigurieren von Teams-Toolkit für Visual Studio Code

In dieser Aufgabe werden Sie die aktuelle Version von [Teams Toolkit für Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5) installieren. Am einfachsten ist es, dies direkt in Visual Studio Code zu tun.

1. Öffnen Sie Ihren **Arbeitsordner** in Visual Studio Code. Möglicherweise werden Sie gebeten, den Autoren dieses Ordners zu vertrauen; wenn ja, dann tun Sie das bitte. Im **Visual Studio Code Dateimenü** wählen Sie **Ordner öffnen** und öffnen direkt den Ordner **msgext-northwind-inventory-ts**.

1. Wählen Sie nun das Symbol **Teams Toolkit** auf der linken Seite 1️⃣. Wenn die Möglichkeit besteht, ein neues Projekt zu erstellen, befinden Sie sich wahrscheinlich im falschen Ordner.  Sie sollten Abschnitte für Konten, Umgebung usw. sehen, wie unten dargestellt.

1. Wählen Sie unter **Konten** die Option **Anmelden bei Microsoft 365** 2️⃣ und melden Sie sich mit Ihrem Microsoft 365-Konto an.

    ![Screenshot der Anmeldung bei Microsoft 365 über das Teams Toolkit.](../media/1-04-setup-teams-toolkit-01.png)

1. Es öffnet sich ein Browserfenster, in dem Sie sich bei Microsoft 365 anmelden können. Wenn es heißt **Sie sind jetzt angemeldet und schließen diese Seite**, tun Sie dies bitte.

1. Vergewissern Sie sich abschließend, dass ein grünes Häkchen neben **Benutzerdefinierter App-Upload aktiviert** erscheint. Wenn dies nicht der Fall ist, bedeutet dies, dass Ihr Benutzerkonto keine Berechtigung zum Hochladen von Teams-Anwendungen hat. Diese Berechtigung ist standardmäßig ausgeschaltet; hier sind [Anweisungen, wie Benutzende benutzerdefinierte Apps hochladen können](https://learn.microsoft.com/microsoftteams/teams-custom-app-policies-and-settings#allow-users-to-upload-custom-apps)

    ![Screenshot der Überprüfung, ob das Querladen aktiviert ist.](../media/1-04-setup-teams-toolkit-03.png)

## Arbeit überprüfen

Nachdem Sie alle oben genannten Aufgaben ausgeführt haben, sollten Sie die folgenden Dateien installiert und auf Ihren Computer heruntergeladen haben:

- [Visual Studio Code](https://code.visualstudio.com/) (neueste Version)

- [NodeJS version 18.x](https://nodejs.org/download/release/v18.18.2/)

- [Azure Storage Explorer](https://azure.microsoft.com/products/storage/storage-explorer/) (OPTIONAL)

- [Teams Toolkit für Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5)

- Beispiel-Repository: [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/)

Wenn alles korrekt vorbereitet wurde, können Sie die Beispielanwendung als Messaging-Erweiterung ausführen. 

[Fahren Sie mit der nächsten Übung fort... ](./3-exercise-1-run-message-extension.md)