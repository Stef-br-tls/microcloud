# Schritt 04 – Den Workflow verstehen

## Teil A – Aufbau

1. Der Workflow startet bei einem Push auf einen beliebigen Branch, bei einem Pull Request mit Ziel `main` und beim manuellen Start über `workflow_dispatch`.
2. Der Job läuft auf `ubuntu-latest`. GitHub stellt dafür eine virtuelle Ubuntu-Maschine bereit.
3. Die Job-ID lautet `stack-test`. Der Anzeigename ist „Compose-Stack testen (Port 8085)“. Der Job hat zehn definierte Schritte.
4. Schritt 1 lädt den Repository-Code mit `actions/checkout@v7`. Die Schritte 2 bis 10 führen mit `run:` eigene Shell-Befehle aus.
5. `contents: read` gibt dem Workflow-Token Lesezugriff auf den Repository-Inhalt. Das reicht für den Test und begrenzt die Rechte des Tokens.
6. Die Bedingung `!github.event.repository.is_template` sorgt dafür, dass der Job in der Vorlage übersprungen wird. In meinem daraus erstellten Repo läuft er bei den vorgesehenen Ereignissen.

## Teil B – Prüfungen

7. Erster fehlschlagender Schritt je Fehlerfall:
   - Tabulator in der YAML-Einrückung → Schritt 3, „YAML-Syntax prüfen“.
   - `html/` wurde nicht committet → Schritt 7, „Inhalt prüfen“. `html/index.html` fehlt im Checkout.
   - Port-Mapping `8085:8080` → Schritt 6, „Erreichbarkeit auf Port 8085 prüfen“. Docker leitet auf den Container-Port 8080 weiter, Nginx lauscht dort auf Port 80.
   - `compose.yml` liegt in einem Unterordner → Schritt 2, „Compose-Datei suchen“. Der Workflow sucht im Hauptordner.
8. Eine Antwort auf Port 8085 kann auch die Standardseite des Webservers sein. Schritt 7 prüft deshalb HTTP-Status 200 und vergleicht die ausgelieferte Seite mit meiner `html/index.html`.
9. Schritt 10 räumt mit `docker compose down -v` auch nach einem Fehler auf. So werden unter anderem Container und das Compose-Netzwerk entfernt. Auf wiederverwendeten Runnern verhindert das, dass Reste spätere Tests stören.

## Teil C – Einordnung

10. Das ist Continuous Integration (CI). Der Workflow prüft Änderungen automatisch. Er stellt kein Release bereit und rollt nichts auf Kloni aus.
11. Die Tests laufen bei jedem Push automatisch und prüfen unter denselben Bedingungen. Die Ergebnisse bleiben am Commit nachvollziehbar.

## Teil D – DoD-Abzeichen

12. DoD 3 prüft den Entwicklungsbranch `dev` und wird daher beim Push auf `dev` ausgelöst. `?branch=dev` zeigt im README den Status dieses Branches. Die übrigen DoD-Prüfungen beziehen sich auf `main`.
13. Das Skript braucht den letzten gemergten Pull Request von `dev` nach `main`, seinen Merge-Zeitpunkt und den letzten Commit des PRs (`head.sha`). Der CI-Check für genau diesen Commit muss erfolgreich gewesen und bereits vor dem Merge abgeschlossen sein.
