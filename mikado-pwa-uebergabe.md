# Mikado-PWA — Übergabe an Claude Code

## 1. Vorbereitung vor dem Agentenlauf

**Referenzdatei bereitstellen.** Der Agent auf devbox hat keinen Zugriff auf die hier erzeugte HTML-Datei. Vor dem Start:

```bash
mkdir -p ~/Development/Apps/mikado-pwa/reference
# mikado-zaehler.html aus dem Chat herunterladen und ablegen als:
# ~/Development/Apps/mikado-pwa/reference/prototyp.html
```

Der Prototyp enthält die bereits an euer Set angepassten Bandmuster, das Layout und die Rundenlogik. Er ist Referenz, nicht Basis — der Agent baut die Struktur neu auf.

**Session anlegen** (tmux-per-Agent-Konvention):

```bash
tmux new -s devbox-mikado
cd ~/Development/Apps/mikado-pwa
claude --dangerously-skip-permissions
```

**Deployment-Host entscheiden** (siehe Abschnitt 4) — der Prompt braucht diese Angabe nicht, das Deployment erfolgt manuell nach dem Build.

---

## 2. Prompt für Claude Code

> Kopierbar ab hier.

```
Baue eine Progressive Web App als Punktezähler für Mikado. Zielgeräte: iPhone und iPad,
Nutzung am Esstisch, auch von Kindern. Projektverzeichnis ist das aktuelle Verzeichnis.

## Randbedingungen

- Kein Build-Step, kein Bundler, kein Framework. Reines HTML, CSS und ES-Module.
  Auslieferung als statisches Verzeichnis.
- Keine externen Ressourcen zur Laufzeit: keine Web-Fonts von CDNs, keine Bibliotheken
  aus dem Netz, keine Telemetrie. Die App muss vollständig offline funktionieren.
- Persistenz ausschließlich clientseitig über localStorage, JSON unter einem
  versionierten Schlüssel (z. B. `mikado.v1.state`). Kein Backend, keine API.
- Zielbrowser: Safari auf iOS/iPadOS 17+. Keine Polyfills für ältere Engines.
- In `reference/prototyp.html` liegt ein funktionierender Einzeldatei-Prototyp. Nutze ihn
  als Vorlage für Bandmuster, Farbwerte und Layout, aber übernimm die Struktur nicht —
  baue sauber in getrennte Module.

## Datenmodell

SetProfile
  id, name, notiz, istVorlage (bool)
  sticks: StickType[]
  regelvariante: "punkte" | "jedesEins"

StickType
  id, name, punkte, anzahl
  bands: Band[]            // Markierung auf dem Stäbchen
  istHelfer: bool          // darf nach Aufnahme als Werkzeug dienen

Band
  position (0..1, relativ zur Stäbchenlänge), breite (0..1), farbe (Hex),
  art: "voll" | "spirale"

Session
  id, setProfileId, spieler: Spieler[], rundenZiel (default 5),
  runden: Runde[], erstellt, zuletztGeaendert

Runde
  aufnahmen: Aufnahme[]    // Event-Liste, nicht aggregierte Zähler
Aufnahme
  spielerId, stickTypeId, zeitstempel

Punktestände werden immer aus der Event-Liste berechnet, nie separat gespeichert.
Damit sind Undo, Verlauf und spätere Statistik trivial.

## Mitgelieferte Set-Vorlagen

1. "Standard 41" — Mikado 20/1×, Mandarin 10/5×, Bonze 5/5×, Samurai 3/15×, Kuli 2/15×.
   Muster: Mikado blaue Spirale über die ganze Länge; Mandarin rot–blau–rot;
   Bonze rot–blau–rot–blau–rot; Samurai breites gelbes Band mittig, blau oben, rot unten;
   Kuli ein rotes und ein blaues Band mit freiem Holz dazwischen.
   Farbwerte aus dem Prototyp übernehmen.
2. "Unser Set" — Kopie von "Standard 41", aber Stückzahlen frei editierbar und
   anfangs unbekannt (siehe Regel zur Vorratsanzeige unten).
3. "Kinder" — identische Stäbchen, regelvariante "jedesEins".
4. "Pick-Up Sticks 40 (US)" — abweichende Namen und Werte, u. a. ein Stäbchen
   "Grandfather" mit 25 Punkten. Werte als Platzhalter anlegen und im Set-Editor
   korrigierbar machen.

Vorlagen sind schreibgeschützt; "Bearbeiten" erzeugt immer eine Kopie.

## Funktionen

Setup
- Set aus der Bibliothek wählen
- 2 bis 6 Spieler, Namenseingabe, zuletzt genutzte Namen als Vorschlag
- Rundenzahl wählbar, Standard 5

Zählen
- Ein Feld pro Stäbchensorte mit gezeichneter Stäbchengrafik (SVG oder CSS, aus den
  Band-Daten generiert, nicht hartkodiert), Punktwert und Anzahl bisher aufgenommener
  Stäbchen dieser Sorte
- Tippen ordnet der aktiven Person ein Stäbchen zu
- Felder werden nie gesperrt, auch nicht wenn die Sollzahl des Sets überschritten ist.
  Bei Überschreitung nur ein dezenter Hinweis, kein Blockieren.
- Spielerzeile antippen wechselt die aktive Person; separater Button für "Nächster"
- Undo über die Event-Liste, beliebig tief
- Runde manuell auswerten; kein automatisches Ende

Auswertung
- Rangliste der Runde mit Aufschlüsselung nach Stäbchensorte
- Gesamtstand der Serie ab Runde 2: Summe und Anzahl Rundensiege
- Gleichstand explizit ausweisen
- Nach der letzten Runde Serienabschluss mit Endstand

Set-Editor
- Stäbchensorten anlegen, umbenennen, Punktwert und Anzahl setzen
- Bandmuster bearbeiten: Bänder hinzufügen, Position und Breite per Slider, Farbe aus
  einer Palette (rot, blau, gelb, grün, schwarz, gold, natur) plus freier Hex-Eingabe
- Live-Vorschau des Stäbchens
- Set duplizieren, löschen, als JSON exportieren und importieren

Verlauf
- Liste abgeschlossener Sessions mit Datum, Set, Teilnehmenden, Sieger
- Einzelne Session löschen, gesamten Verlauf löschen

## PWA-Anforderungen

- `manifest.webmanifest`: name, short_name "Mikado", display "standalone",
  orientation "any", theme_color und background_color passend zum dunklen Layout,
  Icons 192, 512 und maskable — als SVG erzeugen und zu PNG rastern
  (ImageMagick oder rsvg-convert; falls nicht vorhanden, mit apt installieren)
- Service Worker: Cache-First für alle Assets, Versionskonstante im SW, alte Caches
  beim `activate` löschen. Kein Netzwerkzugriff im Betrieb nötig.
- iOS-Spezifika: `apple-touch-icon`, `apple-mobile-web-app-capable`,
  `apple-mobile-web-app-status-bar-style`, `viewport-fit=cover` plus
  `env(safe-area-inset-*)` im Layout
- Dunkles Layout: Hintergrundfarbe auf `html` UND `body` setzen, zusätzlich
  `color-scheme: dark` — sonst blitzt Weiß durch

## Layout

- Mobile-first, ab 820 px Breite zweispaltig: links Spielerliste, rechts Stäbchenfelder
- Tap-Ziele mindestens 44 px, im Zählbildschirm deutlich größer (Kinderhände)
- Systemschriften (`-apple-system`), keine Web-Fonts
- Dunkler Hintergrund, Bambus- und Lackfarben wie im Prototyp
- `prefers-reduced-motion` respektieren

## Struktur

  index.html
  manifest.webmanifest
  sw.js
  css/app.css
  js/ (app.js, state.js, storage.js, sets.js, render-stick.js, screens/*.js)
  icons/
  data/sets.default.json
  README.md

## Qualitätsanforderungen

- Keine Abhängigkeiten in package.json außer Dev-Tools, die du selbst brauchst
- Alle Nutzertexte auf Deutsch, in einer zentralen Datei gesammelt
- Zustandsänderungen ausschließlich über Funktionen in state.js, kein direktes
  Mutieren aus den Screens
- Kurze JSDoc-Kommentare an den Modulgrenzen
- README mit: Datenmodell, Aufbau der Module, lokaler Start, Deployment-Hinweis,
  Format des Set-Exports

## Verifikation vor Abschluss

1. `python3 -m http.server` im Projektverzeichnis starten, Seite laden, Konsole muss
   fehlerfrei sein
2. Service Worker registriert sich, zweiter Aufruf lädt ohne Netzwerk
   (Server stoppen, Seite neu laden)
3. Vollständige Beispielsession durchspielen: 3 Spieler, 2 Runden, Undo, Set-Editor,
   Export und Reimport eines Sets, Verlauf
4. localStorage-Inhalt nach Reload prüfen, Serie muss vollständig wiederhergestellt sein
5. Layout bei 390 px, 768 px und 1180 px Breite kontrollieren

Arbeite in kleinen Schritten und prüfe nach jedem Modul, dass die Seite noch lädt.
Wenn du eine Designentscheidung treffen musst, die nicht aus diesem Prompt folgt,
dokumentiere sie in der README unter "Entscheidungen" statt zu fragen.

Schicke nach Abschluss eine ntfy-Benachrichtigung an das Topic devbox-baden-ezth mit
einer Zusammenfassung in maximal drei Sätzen.
```

> Kopierbar bis hier.

---

## 3. Was du beim Übergeben zusätzlich klärst

Der Prompt lässt drei Punkte offen, die vom Set abhängen und die du besser selbst entscheidest, bevor der Agent läuft:

| Punkt | Warum offen |
|---|---|
| Tatsächliche Stückzahlen eures Sets | Aus dem Foto nicht zuverlässig auszählen. Ohne Zahl ist die Sollmenge nur Richtwert — genau deshalb sperrt die App nichts. |
| Ob Sync zwischen iPhone und iPad nötig ist | localStorage ist gerätelokal. Sync bräuchte ein Backend und widerspricht dem statischen Ansatz. |
| Ob der Verlauf über Runden hinaus zählt | Bestimmt, ob der Verlauf ein Nebenfeature bleibt oder ein eigener Screen mit Statistik wird. |

---

## 4. Deployment über Tailscale Funnel

Die App ist ein statisches Verzeichnis. `tailscale serve` kann Verzeichnisse direkt ausliefern, ein Webserver ist nicht nötig.

**Host-Wahl.** Auf airbox belegt Umami bereits einen Funnel unter `airbox.minuet-polaris.ts.net`. Funnel erlaubt nur 443, 8443 und 10000; 443 ist dort vergeben. Zwei Wege:

- **airbox, zweiter Port:** URL bekommt `:8443` angehängt. Funktioniert, wirkt aber beim Teilen unsauber.
- **devbox als Host:** eigener Funnel auf 443, saubere URL. Voraussetzung: devbox läuft, wenn gespielt wird.

Empfehlung: devbox, weil dort ohnehin dein Entwicklungsstand liegt und kein Portkonflikt entsteht.

**Ablauf** (Syntax vor Ausführung mit `tailscale serve --help` gegenprüfen — die Flags haben sich zwischen Tailscale-Versionen mehrfach geändert):

```bash
# im Tailnet testen (HTTPS mit gültigem Zertifikat, Service Worker funktioniert)
sudo tailscale serve --bg /home/rm/Development/Apps/mikado-pwa

# öffentlich freigeben
sudo tailscale funnel --bg 443 on

# Status prüfen
tailscale serve status
```

**Wichtig:** Der Service Worker braucht einen Secure Context. Über MagicDNS per HTTP aufgerufen registriert er sich nicht — der Zugriff muss über die `https://…ts.net`-URL laufen. Das gilt auch für die Installation auf den Home-Bildschirm.

**Installation am Gerät:** Safari öffnen, URL aufrufen, Teilen-Menü, "Zum Home-Bildschirm". Danach startet die App im Vollbild ohne Safari-Leiste und funktioniert offline — der Funnel muss dann nur noch für Updates erreichbar sein.

**Update-Weg:** Dateien im Verzeichnis ersetzen, Versionskonstante in `sw.js` erhöhen. Ohne diese Erhöhung liefert der Cache die alte Version weiter.

---

## 5. Nach dem Lauf zu prüfen

- Verhält sich die App im Standalone-Modus anders als im Safari-Tab (Statusleiste, Safe Areas, Scrollverhalten)
- Wird localStorage bei iOS-Speicherdruck geleert — Safari räumt Daten von Sites nach längerer Nichtnutzung auf. Bei häufigem Verlust: Umstieg auf IndexedDB plus regelmäßigem JSON-Export.
- Ob die Stäbchengrafiken bei kleiner Darstellung noch unterscheidbar sind, besonders Mandarin gegen Bonze
