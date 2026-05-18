---
trigger: always_on
description: > Diese Datei beschreibt das Projekt. Sie ist auf **Deutsch und einfach erklärt**, damit
---

# Zen Robot Garden 🤖🌿

> Diese Datei beschreibt das Projekt. Sie ist auf **Deutsch und einfach erklärt**, damit
> auch ein 10-jähriges Kind beim Planen mitmachen kann. Wo es hilft, gibt es Beispiele.

## Worum geht es?

Wir bauen ein **3D-Spiel im Browser**. Es ist ein kleiner Garten wie ein **Diorama** —
also wie ein Modell in einer Schachtel, auf das man von schräg oben schaut.

Im Garten gibt es:

- einen **Rasen**
- ein kleines **Haus**
- einen **Baum**
- eine **Pflanze**
- ein paar **Gänseblümchen** (Blumen)
- einen **Mähroboter**, der von ganz allein umherfährt und mäht

Es ist eine **ruhige Sandbox** ("zen"): Es gibt keine Punkte, man kann nicht gewinnen
oder verlieren. Man schaut einfach dem Roboter zu, wie er den Garten mäht. Endlos.

## Der Mähroboter

### Wie er fährt

- Der Roboter fährt **ganz allein** (autonom). Niemand steuert ihn.
- Er fährt **geradeaus**, bis er den **Begrenzungsdraht** (siehe unten) erreicht oder
  an ein **Hindernis stößt**. Dann fährt er ein kleines Stück **zurück** und dreht sich
  in eine **zufällige Richtung** — am Draht wie am Hindernis **genau gleich**, so wie
  ein echter Mähroboter. Die Drahtspule spürt nur *innen/draußen*, nicht die
  Richtung des Drahts — darum lenkt der Roboter nicht gezielt ins Feld, sondern
  dreht einfach zufällig. Zeigt er danach wieder zum Draht, kreuzt er ihn eben
  nochmal — an Ecken auch mehrmals kurz hintereinander, wie im echten Garten.
- Er hat **zwei Räder mit je einem Motor** (das nennt man *Differentialantrieb*).
  Der Roboter fährt **nur**, weil sich seine Räder drehen — er wird nicht "gebeamt".
  - Beide Räder gleich schnell → er fährt geradeaus.
  - Ein Rad schneller als das andere → er fährt eine Kurve.
- Er fährt **mit Trägheit**: Er wird langsam schneller (beschleunigt), bremst sanft ab,
  und Drehen braucht einen kurzen Moment. Wie ein echtes Fahrzeug mit Gewicht.

Die Werte (Höchstgeschwindigkeit, Beschleunigung, Drehgeschwindigkeit) sollen
**einstellbar** sein.

## Der Garten (die Welt)

- Der **Rasen** ist **rechteckig** und hat ein sanft gewelltes **3D-Gelände** —
  Hügel und Mulden, über die der Roboter mit **echter Physik** fährt (er
  klettert bergauf, neigt sich in die Hänge, wird am Berg langsamer). Die
  Geländehöhe steckt in einer editierbaren **Höhenkarte** (`terrain.ts`);
  Sicht-Meshes, der Physik-Boden (ein Höhenfeld-Collider) und die Gras-Höhen-
  Textur werden alle daraus abgeleitet.
- **Haus, Baum und Pflanze blockieren** den Roboter — er **stößt** physisch dagegen
  und weicht aus (anstoßen → zurück → wegdrehen).
- Über den Rasen legen wir ein unsichtbares **Gitter aus vielen kleinen Feldern**.
  Jedes Feld merkt sich eine Zahl: *Wie lang ist hier das Gras?*
  - Fährt der Roboter über ein Feld, wird das Gras dort **kurz** ("gemäht").
  - Dieses Gitter ist gleichzeitig die **Mähspur** — man sieht, wo der Roboter war.
- Gemähtes Gras **wächst langsam wieder nach**. Die Nachwachs-Geschwindigkeit ist
  **einstellbar**. So hat der Roboter nie wirklich "fertig" — schön zen und endlos.
- Die Grashöhe zeigen wir mit **echten 3D-Grashalmen**: viele kleine Halme
  über dem Gitter, die der Roboter kurz mäht, beim Drüberfahren platt drückt
  und die langsam nachwachsen. Eine Farb-Ebene scheint zwischen den Halmen
  durch (lang = dunkler, kurz = heller).

### Blumen

Über den Rasen sind ein paar **Gänseblümchen** gestreut — die kleine weiße
Blume mit gelber Mitte. Jedes durchläuft einen kleinen **Lebenszyklus**:
erst ein winziger **Keimling**, dann die fertige **Blüte** (die dann blühen
bleibt).

Die Blumen verhalten sich wie in einem echten Mähroboter-Garten: Fährt der
**mähende** Roboter über eine Blume, **schrumpft** sie kurz zum Keimling
zusammen und wächst von vorne. Auf der oft gemähten Rasenfläche kommen die
Blumen darum kaum zur vollen Blüte — nur im **ungemähten Randstreifen**
(zwischen Begrenzungsdraht und Rasenkante) blühen sie in Ruhe ganz auf.

Die Blumen sind reine **Sicht-Modelle** — keine Hindernisse. Der Roboter
mäht durch sie hindurch, er stößt nicht an ihnen an. Sie stehen an **festen
Plätzen** (in kleinen Gruppen gestreut) und wiegen sich im selben Wind wie
das Gras. (Mehr Blumen-Sorten und ein frei pflanzbarer Blumen-Pinsel könnten
später dazukommen.)

### Der Begrenzungsdraht ("fence wire")

So wie bei einem **echten Mähroboter** liegt ein dünner **Draht** als geschlossene
Schleife im Rasen — ein Stück von der Kante nach innen. Der Roboter hat **zwei
Spulen-Sensoren**, einen vorne und einen hinten. Jede Spule "spürt", ob sie noch
**innerhalb** der Schleife ist oder schon **draußen**.

- Vordere Spule draußen → die Nase hat den Draht überquert → der Roboter setzt
  zurück und dreht in einen **zufälligen** Kurs — dieselbe Reaktion wie nach
  einem Stoß. Er **stößt also nirgends an**, sondern spürt die Grenze. Weil er
  zufällig dreht, kann er gleich wieder zum Draht zeigen und ihn nochmal
  kreuzen — genau dieses „Herantasten" sieht man bei echten Mährobotern auch.
- Beide Spulen draußen → der ganze Roboter ist aus der Schleife heraus. Dann
  **hält er an** — genau wie ein echter Mähroboter, der seine Grenze verliert.
  Man kann ihn dann einfach zurück auf den Rasen ziehen (siehe *Den Roboter
  anfassen*).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robin7331/zen-robot-garden](https://github.com/robin7331/zen-robot-garden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
