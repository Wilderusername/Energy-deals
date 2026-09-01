# PROJECT_STATE.md

Laufendes Änderungsprotokoll für CanSpot. Neuester Eintrag oben. Für dauerhafte Projektregeln/technische Hinweise siehe [CLAUDE.md](CLAUDE.md).

---

## 2026-09-01 — Vollständiger End-to-End-Test des Nutzerablaufs (keine Fehler gefunden)

**Umgesetzt:**
- Kompletten Nutzerablauf über den Browser gegen den laufenden `canspot-preview`-Server (Port 3000) getestet, größtenteils über direkte, aber echte Event-Auslösung (`.click()`, bubbling `change`/`input`-Events) statt reiner Zustandsmanipulation, um die tatsächlichen Event-Handler zu prüfen:
  - **Suche**: Produkt-, Marken- und Händlernamen-Treffer, Leer-Treffer-Zustand, Zurücksetzen.
  - **Markenfilter** (alle 8 Chips) und **weitere Filter**: Umkreis-Chips, Händler-Chips, Volumen-Chips, Verpackungs-Chips, Zeitraum-Auswahl, „Nur verfügbare Angebote“, „Ohne Pfand“, Zurücksetzen-Button, Filter-Zähler-Badge.
  - **Sortierung**: alle 5 Optionen inkl. Stichprobe der tatsächlichen Sortierreihenfolge (Preis, Distanz).
  - **Listen-/Kartenumschalter**: Pin-Klick, Popover, „Ansehen“-Sprung zurück in die Liste mit gesetztem Händlerfilter.
  - **Produktdetail**: Preisverlauf über 7/30/90 Tage, „Weitere Händler“, Favoriten-Herz, Preisalarm setzen (inkl. Validierung: 0 und negative Werte werden korrekt abgelehnt) und entfernen, CTA-Text/-Link, Teilen-Fallback ohne `navigator.share`.
  - **Favoriten**: Hinzufügen/Entfernen (Karte + Detailseite), Favoriten-Tab, Leerzustand mit „Alle Angebote ansehen“.
  - **Preisalarme**: Alarme-Tab, „Zielpreis erreicht“-Kennzeichnung, Aktivieren/Deaktivieren-Schalter, Löschen.
  - **Standort-Sheet**: PLZ/Ort-Eingabe, Radius-Slider, Ganz-Deutschland-Schalter, „Aktualisieren“ inkl. Rückwirkung auf Status-Zeile/Header-Pill/Profil-Zusammenfassung.
  - **Profil-Sheet**: Name/E-Mail bearbeiten, Avatar setzen/entfernen, Theme hell/dunkel/system, Benachrichtigungs-Schalter, Navigation zu Standort-/Benachrichtigungs-Sheet.
  - **Leerzustände**: „Keine Angebote im Umkreis“ mit „Deutschlandweit suchen“, leere Favoritenliste.
  - **Deep-Links** (`#deal=…`) inkl. ungültiger ID (kein Absturz).
  - **Reload-Test**: Favoriten, Preisalarme, Standort/Radius, Theme, Name/E-Mail, Benachrichtigungs-Einstellungen nach echtem Seiten-Reload alle korrekt wiederhergestellt; keine Konsolenfehler zu irgendeinem Zeitpunkt.
- Ergebnis: **keine echten Fehler gefunden** — keine Code-Änderung nötig, Arbeitsbaum bleibt unverändert.

**Wichtige Entscheidungen:**
- Zwei Beobachtungen bewusst NICHT als Fehler gewertet und nicht verändert:
  1. Die Suche matcht nur als reiner Substring (z. B. „redbull“ ohne Leerzeichen findet „Red Bull“ nicht) — das ist konsistentes, vorhersehbares Verhalten der bestehenden Implementierung, keine Fehlfunktion.
  2. Der Verpackungsfilter „Flasche“ liefert immer 0 Treffer, weil `getPackaging()` aktuell absichtlich immer `"Dose"` zurückgibt (siehe Code-Kommentar) — Demo-Datensatz enthält keine Flaschen-Angebote. Filterlogik selbst arbeitet korrekt; das Hinzufügen von Flaschen-Demodaten wäre eine Datensatz-/Produktentscheidung, keine Fehlerbehebung, und wurde daher nicht angetastet.
  3. Der „Nur verfügbare Angebote“-Schalter (abgelaufene Angebote ausblenden) lässt sich aktuell nicht sichtbar testen, weil `DEMO_TODAY` (01.09.2026) exakt dem heutigen Datum entspricht und noch keines der 50 Demo-Angebote abgelaufen ist. Filterlogik wurde durch temporäres Fälschen eines Angebotsdatums verifiziert und funktioniert korrekt.

**Offen:**
- Keins.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nächster Schritt: warten auf weitere Änderungswünsche.

---

## 2026-09-01 — Funktionierender Frontend-Prototyp: vollständige localStorage-Persistenz

**Umgesetzt:**
- Bestandsaufnahme ergab: Suche, Markenfilter, weitere Filter, Sortierung, Listen-/Kartenumschalter, Produktdetailseite mit dynamischem Preisverlauf (7/30/90 Tage), Favoriten hinzufügen/entfernen und Preisalarme anlegen/entfernen waren bereits funktional umgesetzt (lokale Testdaten für Produkte/Händler/Preise/Preisverläufe existierten bereits: `products`, `storeLogos`, `storeBranches`, `deals` inkl. 90-Tage-Preisverlauf). Preisalarme, Theme, Name/E-Mail/Avatar und Benachrichtigungs-Einstellungen waren bereits über `localStorage` persistiert.
- **Lücke geschlossen — Favoriten persistieren jetzt**: neuer `localStorage`-Key `canspot-favorites`; wird bei jedem Hinzufügen/Entfernen (Karten-Herz und Produktdetail-Herz) über `saveFavorites()` geschrieben und beim Start geladen (Default beim allerersten Start weiterhin `["p2","p6"]`).
- **Lücke geschlossen — Standort persistiert und ist jetzt tatsächlich funktional**: Der PLZ/Ort-Text im Standort-Sheet war bisher rein kosmetisch (Eingabe wurde nirgends übernommen, überall stand hart codiert „59821 Arnsberg“). Neue Variable `locationLabel` + neuer `localStorage`-Key `canspot-location` (Label, Radius, Ganz-Deutschland-Umschalter) — wird beim Klick auf „Aktualisieren“ übernommen und überall dort verwendet, wo bisher der hart codierte Ort stand (Status-Zeile, Header-Pill, Profil-Zusammenfassung).
- Alle Persistenz-Punkte manuell im Browser verifiziert (Favorit setzen, Preisalarm setzen, Standort/Radius ändern, Theme auf Dunkel wechseln → Seite neu laden → alle vier Zustände korrekt wiederhergestellt; siehe auch Alarme- und Favoriten-Tab).

**Wichtige Entscheidungen:**
- Design bewusst unverändert gelassen (wie angefordert) — ausschließlich JavaScript-Logik ergänzt/verdrahtet, keine CSS-/Markup-Anpassungen außer dem Ersetzen hart codierter Text-Strings durch dieselben, jetzt dynamischen Werte.
- Bewusst NICHT persistiert: Suchtext, Marken-Tab, Sortierung, weitere Filter (Größe/Verpackung/Zeitraum/Pfand), Listen-/Kartenansicht — das sind flüchtiger Anzeigezustand, kein „Nutzereinstellung“, und war nicht Teil der Anforderung. Bei Bedarf leicht ergänzbar nach demselben Muster.
- Die tatsächliche Entfernungsfilterung bleibt weiterhin an die fest hinterlegten Demo-Distanzen (`distanceKm` je Händler) gekoppelt, nicht an eine echte Geocodierung des eingegebenen Orts — das entspricht dem bestehenden Prototyp-Ansatz mit lokalen Testdaten und wurde nicht angetastet.

**Offen:**
- Keins für diesen Meilenstein.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler. Getestet über den lokalen `canspot-preview`-Server (Port-Autozuweisung, aktuell 3000), da die Browser-Vorschau lokale Dateien als `data:`-Snapshot ohne `localStorage` rendert (siehe vorheriger Eintrag).
- Nächster Schritt: warten auf weitere Änderungswünsche.

---

## 2026-09-01 — Produktdetail-Klarheit & Preisalarm für nicht verfügbare Favoriten

**Umgesetzt:**
- **CTA im Produktdetail eindeutig gemacht**: Der große Button unten zeigt jetzt konkret Händler + Preis des Angebots, zu dem er führt (z. B. „Zum Angebot bei Kaufland · 0,89 €“) statt des vagen „Zum besten Angebot“. Neue Hilfsfunktion `findBestDealFor(deal)` ermittelt dafür das günstigste nicht abgelaufene Angebot für Produkt+Größe; Button verlinkt jetzt dorthin (vorher verlinkte er immer auf das gerade geöffnete Angebot, auch wenn das nicht das günstigste war).
- **„Bisheriger Tiefstpreis“ präzisiert**: Label zeigt jetzt den Händler, auf den sich der Wert bezieht („Bisheriger Tiefstpreis bei Lidl“ statt nur „Bisheriger Tiefstpreis“).
- **Zurück-Button im Produktdetail ergänzt/repariert**: Der bestehende Schließen-Button (X) lag optisch praktisch deckungsgleich unter den Hero-Buttons (Teilen/Favorit) und war dadurch faktisch unsichtbar/nicht bedienbar. Jetzt eigener Pfeil-zurück-Button oben links (neues Icon `i-arrow-left`, Klasse `.sheet-close-back`), klar getrennt von Teilen/Favorit oben rechts.
- **„Preisalarm setzen“ bei nicht verfügbaren Favoriten**: In den Favoriten-Listen (Mein Bereich & Push-Benachrichtigungen-Sheet) gibt es bei aktuell nicht verfügbaren Favoriten jetzt einen Link „Preisalarm setzen“ mit Inline-Eingabefeld; ist bereits ein Alarm gesetzt, wird stattdessen „Preisalarm bei X €“ angezeigt. Dabei beide bisher fast identischen Render-Funktionen (`renderFavList`, `updateProfileFavList`) auf gemeinsame Helfer (`favRowHtml`, `bindFavRowEvents`) zusammengeführt, um Duplizierung zu vermeiden.

**Wichtige Entscheidungen:**
- Der Produktdetail-CTA verlinkt bewusst immer zum global günstigsten aktuellen Angebot (nicht zum gerade geöffneten Store), da das für eine Preisvergleichs-App die naheliegendste Erwartung ist; die „Weitere Händler“-Liste bleibt unverändert (zeigt weiterhin alle anderen Stores außer dem gerade geöffneten).
- Für den Preisalarm bei nicht verfügbaren Favoriten wird ein editierbarer Default-Wert (0,99 €) vorbelegt, da kein aktueller Preis als Referenz existiert.
- Der bestehende Preisalarm-Mechanismus (`priceAlerts`, Alarme-Tab) wurde wiederverwendet statt einer neuen Datenstruktur — die Alarme-Tab-Logik für „aktuell nicht verfügbar“ existierte bereits und musste nur zugänglich gemacht werden.

**Offen:**
- Keins für diesen Meilenstein.

**Bekannte Fehler / nächste Schritte:**
- Getestet über den lokalen `canspot-preview`-Server (Port 8123), da die Browser-Vorschau lokale Dateien außerhalb des erkannten Projektkontexts als `data:`-Snapshot ohne `localStorage` rendert (führt dort zu stillem Fehlschlagen von `savePriceAlerts()`). Kein Bug im Prototyp selbst, nur eine Einschränkung der Vorschau-Umgebung — bei echtem Aufruf per `file://` oder Server funktioniert `localStorage` normal.
- Nächster Schritt: warten auf weitere Änderungswünsche.

---

## 2026-09-01 — Projekt-Setup: CLAUDE.md, PROJECT_STATE.md, Git-Repo

**Umgesetzt:**
- Gesamten Prototyp (`canspotprototype20260901.html`, ~4000 Zeilen) gesichtet und verstanden: CSS, SVG-Icon-Sprite, Demo-Datenmodell, Rendering-Logik, Sheets/Modals, Theming, Persistenz.
- `CLAUDE.md` erstellt mit Architekturüberblick und Preview-Befehl (`python3 -m http.server 8123`).
- `PROJECT_STATE.md` angelegt (dieses Dokument) als laufendes Änderungsprotokoll.
- Lokales Git-Repository initialisiert (`git init`) — war zuvor kein Git-Repo.

**Wichtige Entscheidungen:**
- Trennung der Dokumentation: `CLAUDE.md` enthält ausschließlich dauerhafte Projektregeln/technische Hinweise, `PROJECT_STATE.md` protokolliert den fortlaufenden Fortschritt (was umgesetzt/entschieden/offen ist).
- Nach jeder abgeschlossenen Änderung wird `PROJECT_STATE.md` kurz aktualisiert; nach jedem sinnvollen Meilenstein folgt ein Git-Commit.

**Offen:**
- Kein `.gitignore` vorhanden — ggf. bei Bedarf ergänzen (aktuell keine Build-Artefakte, die ignoriert werden müssten).
- Git-Identität (Name/E-Mail) wurde von Git automatisch aus Benutzername/Hostname übernommen (`Kevin Walter <kevin@MacBook-Pro.local>`) — ggf. bei Bedarf manuell korrigieren.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler im Prototyp identifiziert (nur gesichtet, keine funktionale Prüfung aller Interaktionen).
- Initial-Commit (`184343e`) erstellt: Prototyp-Baseline + Projekt-Doku. Nächster Schritt: warten auf konkrete Änderungswünsche am Prototyp.
