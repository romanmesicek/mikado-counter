# Mikado Zähler

Eine Progressive Web App (PWA) zum Zählen von Mikado-Aufnahmen: für 2 bis 4 Spieler, mit Serien-Tracking. Optimiert für iPhone und iPad, Nutzung am Esstisch, auch von Kindern.

**Live: [https://romanmesicek.github.io/mikado-counter/mikado-zaehler.html](https://romanmesicek.github.io/mikado-counter/mikado-zaehler.html)**

## Features

- **Zwei Zählmodi**: klassische Punkte (Mikado 20 · Mandarin 10 · Bonze 5 · Samurai 3 · Kuli 2) oder einfache Variante (jedes Stäbchen zählt 1)
- **2–4 Spieler** mit frei wählbaren Namen
- **Serien-Tracking**: mehrere Runden, Gesamtstand und Siege pro Spieler
- **Undo** für jede Aufnahme
- **Fortschrittsanzeige**: 41 Stäbchen, Richtwert-Anzeige und pro Sorte, wie viele bereits aufgenommen sind
- **Offline nutzbar**: keine externen Ressourcen, kein Build-Step — eine einzige HTML-Datei
- **Als App installierbar**: Icon und Manifest für den Home-Bildschirm (iOS/Android)

## Nutzung

### Direkt im Browser

Einfach `mikado-zaehler.html` öffnen — fertig. Kein Server, keine Installation nötig.

### Als App auf dem iPad/iPhone

1. Die gehostete Version in Safari öffnen (siehe unten)
2. Teilen → **„Zum Home-Bildschirm"**
3. Die App öffnet sich ab dann ohne Browser-Leiste mit eigenem Icon

## Deployment (GitHub Pages)

Ein GitHub-Actions-Workflow (`.github/workflows/deploy.yml`) veröffentlicht die App bei jedem Push auf `main` automatisch neu auf GitHub Pages.

Die App läuft hier:

```
https://romanmesicek.github.io/mikado-counter/mikado-zaehler.html
```

Einrichtung (bereits erledigt): **Settings → Pages → Source: GitHub Actions**. Wird der Workflow-Status in der README angezeigt, garantiert das grüne Häkchen, dass die Live-Seite dem aktuellen Stand entspricht.

## Projektstruktur

```
mikado-zaehler.html   # Die komplette App (HTML, CSS, JS in einer Datei)
manifest.webmanifest  # PWA-Manifest (Name, Farben, Icons)
icons/                # App-Icons für Home-Bildschirm und Manifest
.github/workflows/    # GitHub-Pages-Deployment
```

## Technische Hinweise

- Reines HTML/CSS/JS, keine Abhängigkeiten, kein Build-Step
- Keine externen Ressourcen zur Laufzeit (keine CDN-Fonts, keine Bibliotheken, keine Telemetrie)
- Dark Mode Design im Bambus/Mikado-Look
- Daten liegen im Speicher der Seite (kein Persistenz-Layer) — nach dem Neuladen beginnt eine neue Serie

## Spielregeln-Kurzreferenz

| Stäbchen | Punkte | Anzahl im Set |
|---|---|---|
| Mikado | 20 | 1 |
| Mandarin | 10 | 5 |
| Bonze | 5 | 5 |
| Samurai | 3 | 15 |
| Kuli | 2 | 15 |

Gesamt: 41 Stäbchen · 170 Punkte

## Lizenz

Privatprojekt — alle Rechte beim Repository-Inhaber.
