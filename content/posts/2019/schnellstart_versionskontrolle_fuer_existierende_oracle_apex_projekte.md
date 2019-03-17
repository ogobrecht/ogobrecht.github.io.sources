---
title: Schnellstart - Versionskontrolle für existierende Oracle (APEX) Projekte
description: TBD
tags: [oracle, apex, version-control, doag]
lang: de
publishdate: 2019-12-31
---

*Viele Oracle (APEX) Projekte setzen bis heute keine Versionskontrolle ein. Die Gründe dafür sind vielfältig. Meist geht man wohl davon aus, dass die Datenbank ein sicherer Ort für den Quellcode ist. Das stimmt ja auch, aber was ist mit er Historie desselben? Oft fehlt in laufenden Projekten auch einfach die Zeit, sich zusätzlich noch mit der Einführung einer Quellcodeversionierung zu beschäftigen, weil auf den ersten Blick kein direkter Nutzen zu sehen ist. Wer wagt unter Zeitdruck da den zweiten Blick? Dieser Artikel will die Hürde für die Einführung einer Quellcodeversionierung ein wenig tiefer legen und versucht darüber hinaus den Blick in die Zukunft einer versionierten und versckipteten Quellcodebasis - Stichworte sind hier CI/CD und Schema-Migration.*

Der erte Schritt ist der wichtigste, das wissen alle, die sich schon einmal nach langem Zögern endlich auf den Weg gemacht - worum es auch immer ging. Das gleiche gilt für die Quellcodeversionierung. Wenn man erst einmal auf dem Weg ist fragt man sich, wie man so lange ohne auskommen konnte. Eine versionierte Quellcodebasis vergrößert die Komfortzone bei der Entwicklung und die investierte Zeit zahlt sich mehrfach wieder aus. Doch wie tut man möglichst einfach und schnell den ersten Schritt? Welches Versionskontrollsystem wählt man aus? Braucht man vielleicht zusätzliche, teure Software?

Fragen gibt es viele, wenn man anfängt, sich mit dem Thema auseinenanderzusetzen. Einige davon versucht dieser Artikel zu beantworten.

## Workflowänderung: Dateibasiertes Arbeiten

## Toolvergleich DDL-Export

## Arbeiten mit PLEX

## Welches Versionskontrollsystem

Fangen wir einmal bei der Frage an, welches Versionskontrollsystem auszuwählen ist. Wenn man sich ein wenig umschaut, stellt man fest, das wohl Git mittlerweile das bevorzugte System ist. Git hat tatsächlich ein paar schöne Features - zu den wichtigsten gehören seine verteilte Organisation und die Schnelligkeit. Ersteres sorgt dafür, dass man für das Arbeiten am Code nicht ständig eine Verbindung zum Server braucht. Jeder Entwickler hat immer die gesamte Historie zur Verfügung und man kann mit einem neuen, leeren Repository auf dem lokalen Rechner starten und später den Code in ein zentrales Repository einchecken. Die Geschwindigkeit von Operationen wie z.B. Anschauen von Diffs ist durch die lokale Verfügbarkeit der gesamten Historie sehr schnell. Außerdem funktioniert das Branching und Merging so gut, dass man anfängt, dessen Vorteile ganz selbstverständlich zu nutzen. Aber Git ist trotzdem nicht unbedingt immer die beste Wahl. Wenn man z.B. viele Binärdateien einchecken muß, was per se eigentlich ein Wiederspruch ist bezüglich der Idee Quellcodeversionierung, dann mag z.B. SVN (Subversion) die bessere Wahl sein, weil dort auch Diffs auf Binärdateien angewandt werden und das zu reduziertem Speicherverbrauch führt. Unter Git ohne besondere Vorkehrungen kann die ganze Historie auf den Entwickler-PC's auch mal zu Speicherplatzproblemen führen.

Happy coding and version controlling :-)<br>
Ottmar