# Mikado-Zähler

Domänenmodell für die Mikado-Punktezähler-PWA: Sets von Stäbchen, Spielsessions, Runden und die Zählweisen darauf.

## Language

**Regelvariante**:
Die Zählweise, mit der Punkte aus aufgenommenen Stäbchen berechnet werden ("punkte" = Punktwert je Sorte, "jedesEins" = jedes Stäbchen zählt 1). Eigenschaft des **Set**, nicht der Session — ein Wechsel der Zählweise ist ein Set-Wechsel, kein Umschalter im laufenden Spiel.
_Avoid_: Spielmodus, Modus, Zählmodus (als Session-Parameter verstanden)

**Helferstäbchen** (`istHelfer`):
Markierung an einer Stäbchensorte für Stäbchen, die nach dem Aufnehmen als Werkzeug für weitere Züge dienen dürfen (Tischregel). Reine Metainformation — hat keine Auswirkung auf Zählung, Anzeige oder Auswertung.
_Avoid_: Werkzeugstäbchen, Hebel

**Aktive Person**:
Die Spielerin oder der Spieler, der/dem als Nächstes aufgenommene Stäbchen zugeordnet werden. Wird pro Runde verfolgt und persistiert, damit ein Neuladen nicht mitten in der Runde die falsche Person aktiv zeigt.
_Avoid_: aktiver Spieler, Zug

**Rundenziel**:
Die geplante Anzahl Runden einer Serie. Ein weicher Richtwert — sein Erreichen betont die Serienabschluss-Darstellung in der Auswertung, blockiert aber nie "Nächste Runde".
_Avoid_: Rundenlimit, Maximalrunden

**Session-Status**:
Der Lebenszyklus einer Serie: laufend, abgeschlossen (letzte Runde regulär ausgewertet) oder abgebrochen (per "Serie beenden" vorzeitig verlassen). Ein Abbruch verwirft die Serie nicht — sie bleibt, als abgebrochen markiert, im Verlauf.
_Avoid_: beendet (uneindeutig zwischen abgeschlossen und abgebrochen)

**Sollmenge**:
Der erwartete Bestand einer Stäbchensorte in einem Set, gegen den beim Zählen verglichen wird (Fortschrittsanzeige, Hinweis bei Überschreitung). Kann fehlen — dann gibt es keinen Richtwert und keine Fortschrittsanzeige für diese Sorte, unabhängig von der Anzahl bereits aufgenommener Stäbchen.
_Avoid_: Anzahl (wenn "Sollwert" gemeint ist), Stückzahl (wenn "kein Wert bekannt" gemeint ist)

## Relationships

- Ein **Set** legt genau eine **Regelvariante** fest.
- Eine **Stäbchensorte** hat eine **Sollmenge**, die fehlen kann (kein Richtwert).
- Eine **Stäbchensorte** kann als **Helferstäbchen** markiert sein.
- Eine **Session** hat einen **Session-Status** und ein **Rundenziel**.
- Eine **Runde** trackt die **aktive Person**.

## Example dialogue

> **Dev:** "Können wir während der Runde von Punkten auf 'jedes Stäbchen zählt 1' umschalten?"
> **Domain expert:** "Nein — das ist keine Einstellung im Spiel, das ist ein anderes Set. 'Kinder' *ist* die einfache Zählweise."

## Flagged ambiguities

- Der Prototyp hatte einen Punkte/Stäbchen-Umschalter pro Spiel — im Zielmodell ist das kein Session-Parameter mehr, sondern über das gewählte Set festgelegt. Bewusste Abweichung, bestätigt.
- "Anzahl" wurde in der Übergabe-Doku sowohl für den festen Bestand als auch für "anfangs unbekannt" verwendet — aufgelöst: das ist die **Sollmenge**, die explizit fehlen kann.
- "beendet" wurde für regulären Abschluss und für Abbruch gleichermaßen verwendet — aufgelöst als zwei getrennte **Session-Status**-Werte (abgeschlossen/abgebrochen).
- "Kein automatisches Ende" (Zählen) und "Serienabschluss nach der letzten Runde" (Auswertung) wirkten widersprüchlich — aufgelöst: **Rundenziel** ist weich, sein Erreichen ändert nur die Darstellung, nie die Bedienbarkeit.
