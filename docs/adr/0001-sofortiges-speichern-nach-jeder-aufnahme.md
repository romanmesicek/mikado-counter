# Sofortiges Speichern nach jeder Aufnahme

iOS Safari kann Hintergrund-Tabs jederzeit beenden, und das Datenmodell hat keinen separaten "laufende Runde"-Zeiger — die laufende Runde ist schlicht das letzte Element in `Session.runden`. Wir speichern deshalb nach jeder Aufnahme und jedem Undo sofort in localStorage, statt erst bei "Runde auswerten" zu batchen. Ohne das würde ein App-Wechsel oder Tab-Verlust mitten in der Runde — am Esstisch der Normalfall — den gesamten Rundenfortschritt kosten.
