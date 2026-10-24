# Abgebrochene Serien bleiben im Verlauf

Der Prototyp verwirft eine per "Serie beenden" abgebrochene Serie stillschweigend. Im Zielmodell erhält sie stattdessen den Session-Status "abgebrochen" und bleibt, davon erkennbar unterschieden, im Verlauf erhalten — statt gelöscht zu werden. Bei rein clientseitiger, localStorage-basierter Persistenz ohne Backup ist verlorener Fortschritt nicht wiederherstellbar; sichtbares Aufbewahren mit klarer Markierung ist die sicherere Vorgabe. Wer die Daten trotzdem los sein will, kann weiterhin "Verlauf löschen" nutzen.
