# Antworten Phase 4: Den Workflow verstehen

## Teil A: Aufbau der Workflow-Datei
1. Der Workflow startet bei `push` (auf jeden Branch), bei `pull_request` (mit Ziel `main`) und bei `workflow_dispatch` (manuell per Knopf im Actions-Tab).
2. Er läuft auf `ubuntu-latest`. Das ist eine frische virtuelle Ubuntu-Maschine, die GitHub als Runner bereitstellt und nach dem Lauf wieder verwirft.
3. Der Job heißt `stack-test` (Anzeigename "Compose-Stack testen (Port 8085)") und hat 10 Steps.
4. Nur Step 1 nutzt eine fertige Action (`actions/checkout`), die den Code auf den Runner lädt. Alle anderen Steps führen eigene Shell-Befehle mit `run:` aus.
5. Der Workflow hat nur `contents: read`, darf den Code also nur lesen. Das ist das Prinzip der minimalen Rechte: Wird ein Schritt missbraucht, kann er kaum Schaden anrichten.
6. In der Vorlage (Template-Repository) liegt noch kein Stack, deshalb wird der Job dort übersprungen. In meinem eigenen Repo ist es kein Template, also läuft er immer.

## Teil B: Was wird geprüft?
7. a) Tabulator statt Leerzeichen: Step 3 (YAML-Syntax prüfen)
   b) Ordner `html/` nie committet: Step 7 (Inhalt prüfen)
   c) `8085:8080`: Step 6 (Erreichbarkeit), weil der Webserver im Container auf Port 80 lauscht, nicht auf 8080
   d) `compose.yml` in einem Unterordner: Step 2 (Compose-Datei suchen)
8. Ein Webserver kann auf Port 8085 antworten und trotzdem die falsche Seite ausliefern, zum Beispiel seine Standardseite, wenn das Volume beim Umstieg auf Apache am falschen Pfad eingebunden ist. Der Vergleich mit `diff` prüft das erwartete Ergebnis, nicht nur, ob irgendetwas läuft.
9. Step 10 entfernt Container und Netzwerke sauber. Auf selbst gehosteten Runnern, die wiederverwendet werden, würden sonst Reste liegen bleiben und Ports blockieren. Aufräumen gehört zu jedem sauberen Test.

## Teil C: Einordnen
10. Es ist Continuous Integration: Jede Änderung wird automatisch getestet. Es wird kein fertiges Artefakt veröffentlicht (Delivery) und nichts automatisch auf den Server ausgerollt (Deployment). Den LXC aktualisiere ich selbst mit `git pull`.
11. Der Test läuft bei jeder Änderung und kann nicht vergessen werden. Er läuft immer in derselben sauberen Umgebung, und das Ergebnis ist für alle sichtbar dokumentiert (grüner Haken, Abzeichen). Fehler fallen auf, bevor sie in den Betrieb kommen.

## Teil D: Die DoD-Abzeichen
12. Ein Abzeichen zeigt den letzten Lauf auf genau einem Branch. DoD 3 soll prüfen, ob es `dev` gibt, deshalb läuft der Workflow bei Pushes auf `dev`, und das Abzeichen ist mit `?branch=dev` eingebunden. Die anderen Punkte betreffen den Betriebsstand und damit `main`.
13. Es braucht den zuletzt gemergten Pull Request von `dev` nach `main` mit Merge-Zeitpunkt und letztem Commit. Dazu kommt das Ergebnis des Checks "Compose-Stack testen (Port 8085)" für genau diesen Commit mit seinem Abschlusszeitpunkt. DoD 5 ist nur erfüllt, wenn der Check erfolgreich war und vor dem Merge fertig wurde.
