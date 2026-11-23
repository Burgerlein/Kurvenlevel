# Spec: Kurvenlevel (BVB-Fanclub Punktesystem)

> Status: Entwurf v2. Punkte mit 🆕 sind Änderungen oder Ergänzungen gegenüber der Excel-Spec, Punkte mit ❓ sind noch zu entscheiden.

## 1. Ziel

Web-App, die die Anwesenheit von Fanclub-Mitgliedern bei BVB-Spielen (Heim und Auswärts) erfasst und daraus eine Rangliste berechnet. Die Rangliste entscheidet später über die Kartenvergabe (in ca. 3 Jahren bekommt der Club Karten über den BVB).

Ziel ist ein kleines, vollständig ausgearbeitetes Tool, das der Club nutzen kann, später mit Login von extern.

## 2. Technische Entscheidungen

| Thema | Entscheidung |
|---|---|
| Basis | Orchard Core (ASP.NET Core) |
| Datenhaltung | Mitglied und Spiel als Content-Types, eigene Entitäten wo nötig |
| Berechnung | Eigener C#-Service ohne Orchard-Abhängigkeit, bei jeder Anfrage neu berechnet, nicht gespeichert |
| Tests | Unit-Tests mit Beispieldaten aus dem Excel als Testfälle |
| Frontend (User) | Eigene Vue-App im BVB-Design, spricht per API mit dem Backend |
| Admin-Oberfläche | Standard-Admin von Orchard Core |
| API | Eigene Endpunkte im Modul (Rangliste, Details, Profil), ggf. ergänzt durch die Content-APIs von Orchard |
| Login | Orchard-Core-Benutzerverwaltung mit Rollen, externe Anmeldung später (OpenID Connect) |
| Hosting | Cloud (Anbieter ❓) |
| Plattform | Web zuerst, Mobile App eventuell später |

## 3. Rollen

- **Admin:** Mitglieder verwalten, Spiele und Einträge erfassen, Punktwerte ändern
- **Mitglied/Viewer:** Rangliste und Details nur ansehen

❓ Soll die Rangliste zunächst ohne Login öffentlich sichtbar sein oder von Anfang an hinter dem Login liegen?

## 4. Datenmodell

### Mitglied
`id`, `vorname`, `nachname`, `zusatz` (optional, bei Namensgleichheit), `anzeigename` (= „Vorname Nachname (Zusatz)“), `mitglied_seit`, `aktiv` (bool)

Inaktive Mitglieder werden nicht gelöscht. Sie behalten ihre Historie, erscheinen aber nicht in Rangliste und Auswahl.

### Spiel 🆕
Neue Entität, damit Spieldaten nicht pro Mitglied doppelt erfasst werden.

`id`, `datum`, `saison` (abgeleitet), `gegner`, `wettbewerb`, `ort`, `km_ab_clubsitz`, `anstoss`

- `wettbewerb`: Bundesliga | DFB-Pokal | Champions League | Europa League ❓ weitere (DFL-Supercup, Klub-WM) 🆕 konfigurierbar statt fest im Code
- `ort`: Heim | Auswärts
- `anstoss`: Normal | Unter der Woche | Freitagabend
- `saison`: ab Juli neue Saison, Format „2026/27“

### Eintrag
`id`, `spiel_id`, `mitglied_id`, `typ`, `anzahl_karten`

- `typ`: Besuch | Karten besorgt | Karte verfallen
- 🆕 Pro Mitglied, Spiel und Typ höchstens ein Eintrag. Wer fährt und zusätzlich Karten besorgt, hat zwei Einträge für dasselbe Spiel.
- `anzahl_karten` gilt nur für „Karten besorgt“

### Einstellungen
Alle Punktwerte aus Abschnitt 5, vom Admin änderbar. Die Rangliste wird danach neu berechnet.

❓ Gelten geänderte Punktwerte für die gesamte Historie oder erst ab einer bestimmten Saison?

### Änderungsprotokoll 🆕
Jede Änderung an Einträgen und Einstellungen wird mit Zeitpunkt und Benutzer protokolliert, weil die Rangliste über Kartenvergabe entscheidet.

## 5. Punktevergabe (Defaults)

### Typ „Besuch“

Grundpunkte:

| Wettbewerb | Heim | Auswärts |
|---|---|---|
| Bundesliga | 3 | 5 |
| DFB-Pokal | 3 | 6 |
| Champions League | 3 | 8 |
| Europa League | 3 | 8 |

Km-Bonus (Entfernung immer ab Clubsitz, nicht ab Wohnort, damit der Wohnort keinen Vorteil bringt):

| ab km | Bonus |
|---|---|
| 0 | 0 |
| 200 | +1 |
| 400 | +2 |
| 600 | +3 (Deckel) |

Anstoß-Bonus:
- „Unter der Woche“ (Di/Mi/Do): +2, Heim und Auswärts
- „Freitagabend“: +1, nur Auswärts

Englische Woche: +2, wenn das Mitglied alle Spiele der Kalenderwoche besucht hat. 🆕 Der Bonus wird vom System aus den Spielen berechnet und einmal pro Woche vergeben (kein manuelles Flag mehr).

### Typ „Karten besorgt“
- +1 pro aktiv besorgter Karte, maximal 5 pro Spiel
- Reines Weiterreichen von Karten zählt nicht

### Typ „Karte verfallen“
- −2 (zugesagte Karte kurzfristig verfallen lassen)

### Treuebonus
- +10 pro Saison mit mindestens 10 Besuchen
- Verfällt nie
- ❓ Vergabe sofort beim 10. Besuch oder erst zum Saisonende?

## 6. Berechnung

Gesamtpunkte = Summe aller Eintragspunkte + Treuebonus

Bei Punktgleichheit erhalten Mitglieder den gleichen Rang (1, 1, 3, …).

## 7. Ansichten

1. **Rangliste:** Rang, Mitglied, Spiele, davon auswärts, Treue (★ pro Treue-Saison), Punkte. Nur aktive Mitglieder, sortiert nach Punkten, Top 3 hervorgehoben.
2. **Details:** pro Mitglied Anzahl Heim, Ausw. Liga, Ausw. Pokal, Ausw. Europa, Spiele gesamt sowie die Punkte aufgeteilt nach Grundpunkten, Km-Bonus, Anstoß-Bonus, Engl. Woche, Karten besorgt, Abzüge, Treuebonus und Gesamt.
3. **Mitglied-Profil:** alle Einzeleinträge eines Mitglieds.
4. **Erfassung (Admin):** schnelles Anlegen eines Spiels und der teilnehmenden Mitglieder auf einmal.

## 8. Design

BVB-Optik: dunkler Hintergrund (#111), Gelb #FDE100 als Akzent, Schwarz/Weiß für Text.

## 9. Datenschutz 🆕

Namen und Anwesenheitsdaten sind personenbezogene Daten. Vor dem Login-Betrieb klären: Wer sieht was, Einwilligung der Mitglieder, Löschkonzept (DSGVO).

## 10. Roadmap

### Phase 1: MVP
- [ ] Projekt-Setup (Orchard Core, Solution, Repo)
- [ ] Content-Types Mitglied und Spiel
- [ ] Einträge erfassen
- [ ] Berechnungs-Service mit Unit-Tests
- [ ] Rangliste, Details, Profil
- [ ] Admin-Einstellungen für Punktwerte
- [ ] BVB-Design

### Phase 2: Login und Betrieb
- [ ] Rollen Admin und Viewer
- [ ] Änderungsprotokoll
- [ ] Deployment in der Cloud
- [ ] Externe Anmeldung (OpenID Connect)

### Phase 3: Integration
- [ ] Discord-Bot „Wer fährt?“, der Einträge automatisch erzeugt (Telegram als Alternative, Signal vermutlich nicht sinnvoll)

### Phase 4: Weiterführend
- [ ] Kartenvergabe-Modul (Bewerbung für Spiel X, Vergabe nach Rangliste)
- [ ] Saison-Gewichtung (aktuell ×1, Vorsaison ×0,5, davor ×0,25, älter ×0)
- [ ] Mindestaktivität für Topspiele (z. B. ≥5 Spiele in den letzten 12 Monaten)
- [ ] Tie-Breaker bei der Kartenvergabe (Los oder wer zuletzt kein Topspiel bekam)
- [ ] Nachweis pro Besuch (Foto-Upload)

## 11. Offene Fragen

1. Rangliste zunächst öffentlich oder direkt hinter Login?
2. Punktwerte: Änderung für die ganze Historie oder ab Saison X?
3. Treuebonus: sofort oder zum Saisonende?
4. Weitere Wettbewerbe (Supercup, Klub-WM)?
5. Cloud-Anbieter und Budget?
6. Projektname: „Kurvenlevel“ ist ein Arbeitstitel.
