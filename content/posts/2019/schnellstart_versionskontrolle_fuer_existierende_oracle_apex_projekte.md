---
title: Schnellstart - Versionskontrolle für existierende Oracle (APEX) Projekte
description: TBD
tags: [oracle, apex, version-control, doag]
lang: de
publishdate: 2019-12-31
---

*Viele Oracle (APEX) Projekte setzen bis heute keine Versionskontrolle ein. Die Gründe dafür sind vielfältig. Meist geht man wohl davon aus, dass die Datenbank ein sicherer Ort für den Quellcode ist. Das stimmt ja auch, aber was ist mit er Historie desselben? Oft fehlt in laufenden Projekten auch einfach die Zeit, sich zusätzlich noch mit der Einführung einer Quellcodeversionierung zu beschäftigen, weil auf den ersten Blick kein direkter Nutzen zu sehen ist. Wer wagt unter Zeitdruck da den zweiten Blick? Dieser Artikel will die Hürde für die Einführung einer Quellcodeversionierung ein wenig tiefer legen und versucht darüber hinaus den Blick in die Zukunft einer versionierten und verskripteten Quellcodebasis - Stichworte sind hier CI/CD und Schema-Migration.*


## Einleitung

Der erte Schritt ist der wichtigste, das wissen alle, die sich schon einmal nach langem Zögern endlich auf den Weg gemacht - worum es auch immer ging. Das gleiche gilt für die Quellcodeversionierung. Wenn man erst einmal auf dem Weg ist fragt man sich, wie man so lange ohne auskommen konnte. Eine versionierte Quellcodebasis vergrößert die Komfortzone bei der Entwicklung und die investierte Zeit zahlt sich mehrfach wieder aus. Doch wie tut man möglichst einfach und schnell den ersten Schritt? Welches Versionskontrollsystem wählt man aus? Braucht man vielleicht zusätzliche, teure Software?

Fragen gibt es viele, wenn man anfängt sich mit dem Thema auseinanderzusetzen. Einige davon versucht dieser Artikel zu beantworten.


## Workflowänderung: Dateibasiertes Arbeiten

Eine der wichtigsten Fragen: Kann ich weiterarbeiten wie bisher? Die Antwort hängt ganz davon ab, wie man bisher gearbeitet hat. Grundsätzlich bedeutet das Arbeiten mit einer Quellcodeversionierung eigentlich auch einen Wechsel von "In der Datenbank entwickeln", also das direkte ändern und kompilieren von Code in der Datenbank oder das ändern von Tabellen mit dem Tool seiner Wahl hin zu "dateibasiertes Arbeiten", also zuerst eine Datei im lokalen Quellcoderepository auf dem PC zu bearbeiten, um diese dann aus der Datei heraus zu kompilieren oder per Skript den enthaltenen DDL Code für Schema-Änderungen auszuführen. Je nach bisher verwendeten Tools ist das beim Bearbeiten von Logik eher nur ein subtiler Unterschied, bei dem Verändern von Datenbankobjekten wie Tabellen dann doch schon eher ein großer Unterschied. Wurde bisher ein Schema-Diff-Tool verwendet, um Änderungen der Entwicklungsumgebung in die Produktion zu transportieren, dann sieht das vielleicht auf den ersten Blick sogar wie ein Rückschritt aus.

Der Punkt ist hier, dass man nur dann eine verwertbare Quellcodehistorie bekommt, wenn man konsequent auf das dateibasierte Arbeiten setzt. Weitere Vorteile dieser Vorgehensweise werden im Laufe des Artikels noch beleuchtet. Es kann auch einen Zwischenschritt hin zu diesem Ansatz geben. Man arbeitet weiter wie bisher und exportiert regelmäßig alle Objekte des Schemas in die Quellcodeverwaltung, um auf diesem Weg wenigstens eine gewisse, geordnete Transparenz der Änderungen zu bekommen. Werden dann im Laufe des Projektes die Vorteile dieses Vorgehens ersichtlich, dann fehlt einem meist noch die Info, wer denn die jeweiligen Änderungen ausgeführt hat - diese Information geht verloren bei diesem Ansatz. Außerdem werden normalerweise durch die gewonnene Transparenz schnell Wünsche geweckt wie "Wenn ich den alten Code ja im Repository habe, dann darf ich ja mal etwas probieren, ich kann ja notfalls zurück". Schon ist die erste Hürde genommen und die Offenheit, vielleicht doch einmal damit loszulegen, da.

Dass so etwas noch viel besser funktionieren kann mit dem dateibasierten, lokalen Arbeiten in einem Repository, wo man selber einen Branch (Entwicklungszweig) lokal anlegen kann, um etwas zu auszuprobieren ohne Gefahr zu laufen, dass man wegen einem unerwartet zu lösenden Bug wieder alles verwerfen muss, darauf kommen die meisten Leute dann selber. Man wechselt jetzt bequem in den Hauptzweig, arbeitet am Bugfix, aktualisiert seinen Branch danach auf den geänderten Hauptzweig und kann dann einfach weiter ausprobieren. Habe ich jetzt irgend jemanden abgehängt? Kein Problem, einfach loslegen - hier hilft nur Praxis. Wenn man einmal die ersten Schritte getan hat, fragt man sich wirklich, wie man nur ohne leben konnte.


## Verzeichnistruktur im Repository

Wie soll man sein Repository strukturieren? Hier ein Vorschlag, der sich im Laufe der Zeit für mich persönlich als vorteilhaft herausgestellt hat. Jedes Projekt mag da andere Vorstellungen haben. Ich verwende hier eine Einfach Auflistung - die ersten Wörter jedes Listpunktes stellt den Verzeichnisnamen dar und etwaige Erklärungen finden sich in Klammern dahinter:

- app_backend (unser Schema DDL)
  - constraints (ein Ordner pro Objekttyp)
  - packages
  - package_bodies
  - ref_constraints
  - sequences
  - tables
  - ...
- app_data (Verfolgung von Masterdaten, optional)
- app_frontend (unsere APEX app)
  - pages
  - shared_components
  - ...
- docs (die Dokumentation)
- reports (optional)
- scripts (alle Masterskripte vereint)
  - logs (Export- und Installations-Logs)
    - fixme: example export log name
    - fixme: example install log name
  - misc (häufig benutzte Skripte)
  - 010_export_app_from_DEV.bat (Master Skript mit parametern für das eigenliche SQL Installationsskript)
  - 020_import_app_into_TEST.bat
  - 030_import_app_into_PROD.bat
  - install_app_custom_code.sql
  - install_backend_generated_by_plex.sql
  - install_frontend_generated_by_apex.sql
- README.md (generelle Informationen zum Projekt und Links in die Dokumentation)
- ...

Das Repository sollte logisch aufgebaut sein und keine extrem tiefen Verzeichnisstrukturen beinhalten, um den Umgang damit zu erleichtern. Alle Masterskripte sind vereint in einem Skript-Ordner, auch das von APEX generierte Installationsskript für die Anwendung. Alle Masterskripte sollten während der Verwendung die jeweiligen Rückmeldungen in entsprechende Log-Dateien im Unterordner logs ablegen - damit sind Exporte von Quellcode und Installation in Zielsysteme nachvollziehbar.


## Toolvergleich: DDL-Export

Wie bewerkstelligt man den ersten Schritt? Also, wie bekommt man die Schema-Objekte seiner Anwendung in ein wohl strukturiertes Quellcoderepository? Die meisten der verwendeten grafischen Entwicklungstools bringen dafür eine Export-Funktion mit. Diese ist je nach Tool mehr oder weniger gut dafür geignet, ein Quellcoderepository aufzubauen. Deshalb schauen wir uns jetzt an, wie die am häufigsten verwendeten Tools SQL-Developer, PL/SQL Developer und Toad das können. Ein Wort vorweg: Die Intention des Schema-Exports dieser Tools ist eigentlich, die Objekte in ein anderes Schema, vielleicht sogar in eine andere Datenbank zu bringen und nicht ein Quellcoderepository zu erstellen. Hier die Fragen, die uns bewegen, wenn es darum geht ein Quellcoderepository aufzubauen:

- Ist eine Script-Datei pro Objekt möglich?
- Sind Unterverzeichnisse pro Objekttyp möglich?
- Sind eigene Dateien für Fremdschlüssel möglich? (nützlich für einfachere Masterskripte)
- Können "Object already exist"-Fehler verhindert werden? (anders gefragt: sind die Skripte wiederanlauffähig?)
- Können Daten exportiert werden? (möglichst im CSV-Format zur Verfolgung von Stammdatenänderungen)
- Ist eine APEX App exportierbar? (womöglich auch zerlegt in die Einzelteile wie Pages, Shared Components usw.)


Das Ergebnis bezogen auf die Fragestellung:

| Kriterium             | SQL Dev.    | PL/SQL Dev.    | Toad       |
|-----------------------|-------------|----------------|------------|
| Datei pro Objekt      | Ja          | Ja             | Ja         |
| Unterverz. pro Typ    | Ja          | Nein           | Ja         |
| FK Constr. extra      | Ja          | Nein           | Ja         |
| Verhi. "object exist" | Nein        | Nein           | Nein       |
| Export Daten          | Ja          | Nein           | ***Jein*** |
| Export APEX App       | ***Jein***  | Nein           | Nein       |


Anmerkungen SQL Developer:

- Ist am übersichtlichsten
- Viele Formate für Datenexport (auch CSV)
- Umfangreich konfigurierbar
- Export APEX App nur mit Commandline-Version SQLcl 
- Sieha auch Blog Post Blain Carter: [CI/CD for Database Developers – Export Database Objects into Version Control](https://learncodeshare.net/2018/07/16/ci-cd-for-database-developers-export-database-objects-into-version-control/)


Anmerkungen PL/SQL Developer:

- Wenig konfigurierbar
- Enttäuscht für den Aufbau eines Quellcode-Repository


Anmerkungen Toad:

- Zwei Exportmöglichkeiten (mindestens)
  - Entweder Unterverzeichnisse pro Objekttyp...
  - ... oder Daten
- Daten nur als Insert Statements
- Umfangreich konfigurierbar, relativ unübersichtlich

Keines der Tools liefert uns ein fertiges, gut strukturiertes Quellcoderepository. Man muss mehr oder weniger viel nacharbeiten. Das heißt nicht, dass es nicht gehen würde, zumal das Einsortieren der Objekte in eine vernünftige Struktur eine einmalige Arbeit sein sollte. Lediglich bei dem Export der APEX-Anwendung wünscht man sich eine automatische Lösung, weil dieser Export in Zukunft häufiger gemacht werden muss. Das liegt in der Natur von APEX als deklarativem Low Code Framework. Wir entwickeln die APEX Anwendung ja im Browser und die Meta-Daten der Anwendung liegen im APEX Repository der Datenbank. Ein Export der Anwendung kann auch mit dem Browser gemacht werden, dabei erhält man aber nur ein eeinzige großes SQL-Datei mit der gesamten Applikation. Wünschenswert für ein Quellcode-Repository wäre aber ein Export der Einzelteile wie z.B. Pages, Shared Components, Plugins usw.. Damit lässt sich dann für die Anwendung verfolgen, wie sie sich entwickelt hat und auch eine Suche im Repository nach z.B. einem bestimmten Package-Aufruf macht bei einer zerlegten APEX-Anwendung wesentlich mehr Sinn. In der Vergangeheit konnte man dafür den APEX Export-Splitter verwenden, im Grunde eine Java-Klasse, die im Lieferumfang jeder APEX-Version bis heute enthalten ist. Der Nachteil ist dann, dass man die so erstellte Verzeichnisstruktur so nehmen muss, wie sie vom Splitter erstellt wurde, oder man verändert sie im Nachgang mit lokalen Skripten auf dem PC. Schöner wäre allerdings, die Verzeichnisstruktur wäre schon während des Exports anpassbar um unser Ziel zu erreichen, dass alle Master-Skripte in einem zentralen Ordner des Repositories liegen sollten.

## Schnellstart: Package PLEX

## Geschwindigkeit: Immer Skripte

## Ausblick: CI/CD & Schema-Migration



## Welches Versionskontrollsystem

Fangen wir einmal bei der Frage an, welches Versionskontrollsystem auszuwählen ist. Wenn man sich ein wenig umschaut, stellt man fest, das wohl Git mittlerweile das bevorzugte System ist. Git hat tatsächlich ein paar schöne Features - zu den wichtigsten gehören seine verteilte Organisation und die Schnelligkeit. Ersteres sorgt dafür, dass man für das Arbeiten am Code nicht ständig eine Verbindung zum Server braucht. Jeder Entwickler hat immer die gesamte Historie zur Verfügung und man kann mit einem neuen, leeren Repository auf dem lokalen Rechner starten und später den Code in ein zentrales Repository einchecken. Die Geschwindigkeit von Operationen wie z.B. Anschauen von Diffs ist durch die lokale Verfügbarkeit der gesamten Historie sehr schnell. Außerdem funktioniert das Branching und Merging so gut, dass man anfängt, dessen Vorteile ganz selbstverständlich zu nutzen. Aber Git ist trotzdem nicht unbedingt immer die beste Wahl. Wenn man z.B. viele Binärdateien einchecken muß, was per se eigentlich ein Wiederspruch ist bezüglich der Idee Quellcodeversionierung, dann mag z.B. SVN (Subversion) die bessere Wahl sein, weil dort auch Diffs auf Binärdateien angewandt werden und das zu reduziertem Speicherverbrauch führt. Unter Git ohne besondere Vorkehrungen kann die ganze Historie auf den Entwickler-PC's auch mal zu Speicherplatzproblemen führen.

Happy coding and version controlling :-)<br>
Ottmar