# 🏟️ Kurvenlevel

> Wer viel für den Fanclub tut, steigt auf: Auswärtsfahrten, Kartenbeschaffung und Aktionen sammeln Punkte und bestimmen die Rangliste.

![Status](https://img.shields.io/badge/Status-Planung-yellow)
![Backend](https://img.shields.io/badge/Backend-Orchard%20Core-512BD4?logo=dotnet&logoColor=white)
![Frontend](https://img.shields.io/badge/Frontend-Vue.js-4FC08D?logo=vuedotjs&logoColor=white)
![Mobile](https://img.shields.io/badge/Mobile-geplant-lightgrey)

> ⚠️ **Arbeitstitel:** Der Projektname steht noch nicht fest.

## 💡 Idee

In Fanclubs gibt es Mitglieder, die ständig mit zu Auswärtsspielen fahren, anderen Karten besorgen und bei Aktionen mit anpacken. Dieser Einsatz bleibt oft unsichtbar. Kurvenlevel macht ihn sichtbar: Die Anwesenheit bei BVB-Spielen wird erfasst, daraus entsteht eine Rangliste. Diese soll später die Kartenvergabe im Fanclub fair und nachvollziehbar machen.

## ✨ Features

**Geplant für den MVP**

- Mitglieder verwalten (inaktive Mitglieder behalten ihre Historie)
- Spiele und Besuche erfassen, mehrere Mitglieder pro Spiel auf einmal
- Punkteberechnung nach konfigurierbaren Regeln (Wettbewerb, Heim/Auswärts, Entfernung, Anstoßzeit, Englische Woche, Karten besorgt, Treuebonus)
- Rangliste, Detailansicht und Mitglieder-Profil
- Admin-Bereich für Punktwerte

**Später**

- Login mit Rollen (Admin, Mitglied) und externe Anmeldung
- Änderungsprotokoll für mehr Transparenz
- Discord-Bot „Wer fährt?“, der Einträge automatisch erzeugt
- Kartenvergabe-Modul nach Rangliste
- Mobile App

Die vollständige Spezifikation mit Punktetabellen, Datenmodell und offenen Fragen steht in [docs/SPEC.md](docs/SPEC.md).

## 🧱 Architektur

```
┌──────────────┐   API    ┌───────────────────────────────┐
│  Vue-App     │ ───────► │  Orchard Core (ASP.NET Core)  │
│  (Mitglieder)│          │  ├─ Kurvenlevel-Modul         │
└──────────────┘          │  │   (Punkteberechnung, API)  │
                          │  ├─ Content: Mitglied, Spiel  │
┌──────────────┐          │  ├─ Benutzer, Rollen, Login   │
│  Orchard     │ ───────► │  └─ Admin-Oberfläche          │
│  Admin-UI    │          └───────────────────────────────┘
└──────────────┘
```

- **Backend:** Orchard Core liefert Benutzerverwaltung, Rollen, Admin-Oberfläche und Datenverwaltung.
- **Berechnung:** Eigener C#-Service ohne Orchard-Abhängigkeit, bei jeder Anfrage neu berechnet und mit Unit-Tests abgesichert.
- **Mitglieder-Oberfläche:** Eigene Vue-App im BVB-Design (dunkel, Gelb `#FDE100` als Akzent), die per API auf das Backend zugreift.
- **Admin:** Die Standard-Oberfläche von Orchard Core.
- **Hosting:** Cloud (Anbieter noch offen).

## 🗺️ Roadmap

- [ ] Projektname festlegen
- [ ] Offene Fragen der Spec klären
- [ ] Projekt-Setup (Orchard Core, Vue, Repo-Struktur)
- [ ] Content-Types Mitglied und Spiel
- [ ] Punkteberechnung mit Unit-Tests
- [ ] API und Vue-Oberfläche (Rangliste, Details, Profil)
- [ ] Login und Rollen
- [ ] Deployment
- [ ] Discord-Integration
- [ ] Kartenvergabe-Modul
- [ ] Mobile App (optional)

## 🔐 Datenschutz

Die App verarbeitet Namen und Anwesenheitsdaten von Mitgliedern. Vor dem produktiven Einsatz wird geklärt, wer was sehen darf und wie die DSGVO-Anforderungen erfüllt werden.

## 📄 Lizenz

Noch offen.
