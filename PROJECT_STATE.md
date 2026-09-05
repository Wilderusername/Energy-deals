# PROJECT_STATE.md

Laufendes Änderungsprotokoll für CanSpot. Neuester Eintrag oben. Für dauerhafte Projektregeln/technische Hinweise siehe [CLAUDE.md](CLAUDE.md).

---

## 2026-09-05 (33) — Abstand "Sicherheit" → "Konto löschen" vergrößert

**Umgesetzt:** Ausschließlich `margin-top` des `.field-label` "Konto löschen" (das gleichzeitig die Trennlinie trägt) von `32px` auf `44px` erhöht — die Trennlinie wandert dadurch entsprechend weiter nach unten mit. Nichts sonst an diesem Bereich angefasst: `padding-top:22px` (Abstand Linie → Text "Konto löschen"), Beschreibungstext, Button (Größe/Position/Farbe/Radius) und alle anderen Bereiche der Seite unverändert.

**Verifiziert** über einen temporären lokalen Server (Mobile + Desktop): Abstand zwischen Passwort-Feld und Trennlinie per `getBoundingClientRect()` nachgemessen (jetzt 44px statt zuvor 32px), Innenabstände des Löschbereichs selbst unverändert (`padding-top:22px`, Beschreibungstext-`margin-bottom:10px` identisch); keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v73` erhöht (Pflichtregel).

---

## 2026-09-05 (32) — Bugfix Passwort-Feld + "Konto löschen" zweistufig mit Bestätigung neu gestaltet

**Bugfix (gemeldet): Passwort-Bearbeitungsfeld klappte nie wieder ein.**
- Ursache: Die Felder Name/E-Mail/Geburtsdatum/Adresse/Passwort (`.field-edit-row`) hatten noch nie ein "Klick außerhalb schließt"-Verhalten - fiel bisher nur beim neuen Passwort-Feld auf, da die anderen meist direkt über "Speichern" wieder geschlossen wurden.
- Fix: Ein Klick-Handler auf `#accountManageOverlay` prüft bei jedem Klick, ob ein `.field-block` gerade ein offenes Bearbeiten-Feld hat UND der Klick außerhalb dieses Blocks lag - falls ja, wird es eingeklappt. Klicks auf den Stift, in die Eingabefelder selbst oder auf "Speichern" bleiben davon unberührt (lösen weiterhin ihre eigenen, unveränderten Handler aus). Gilt jetzt einheitlich für alle fünf Felder, nicht nur Passwort.

**"Konto löschen" neu gestaltet (zweistufig, gemäß Vorgabe):**
- Überschrift "Gefahrenzone" → "Konto löschen"; Beschreibungstext ersetzt durch den vorgegebenen Wortlaut; Abschnitt zusätzlich per Trennlinie + größerem Abstand (`border-top`/`margin-top:32px`) sichtbar vom übrigen Kontobereich abgesetzt.
- Erster Button (`#deleteAccountBtn`) bleibt unverändert der bereits vorhandene `.profile-action-danger`-Stil (dunkler Grund, rote Outline/Schrift/Icon über `currentColor`) - dieser existierte bereits exakt in der gewünschten "nicht vollflächig rot"-Optik, wurde nur inhaltlich (Text) angepasst, nicht das Styling.
- Klick darauf öffnet jetzt statt direkt eines Toasts eine neue Bestätigungs-Sheet (`#deleteAccountConfirmOverlay`, wiederverwendet die bestehende generische Sheet-Infrastruktur `openSheet`/`closeSheet`/`[data-close]` - keine neue Dialog-Komponente): Überschrift "Konto wirklich löschen?", vorgegebener Text, "Abbrechen" (neuer Einsatz der bereits vorhandenen, bisher ungenutzten `.sheet-btn-secondary`-Klasse) und "Konto endgültig löschen" (neue Klasse `.sheet-btn-danger`, direkt auf `.sheet-btn` aufgesetzt wie `.sheet-btn-secondary` - einzige vollflächig rote Aktion der App, `#ef4444` als modernes Rot gemäß Vorgabe, da die gedeckten `--status-error-*`-Tokens als Flächenfarbe blass gewirkt hätten). Erst ein Klick auf "Konto endgültig löschen" löst den bisherigen (unveränderten) Demo-Hinweis-Toast aus.
- **Bug gefunden und behoben, bevor er auffiel**: Ein HTML-Kommentar wurde versehentlich mit JS-Syntax `*/` statt `-->` geschlossen - dadurch verschluckte der nicht beendete Kommentar sämtliches nachfolgendes Markup bis zum nächsten zufälligen "-->" im Dokument, wodurch u.a. `#deleteAccountBtn` aus dem DOM verschwand und ein `Cannot read properties of null`-Fehler beim Skriptstart auftrat. Beim ersten Testdurchlauf in der Konsole bemerkt und sofort korrigiert.
- Verifiziert über einen temporären lokalen Server (Mobile + Desktop): Passwort-Feld (und alle anderen Felder) klappen jetzt beim Klick woanders zuverlässig ein, Tippen in Eingabefeldern und Wechsel zu einem anderen Feld funktionieren weiterhin korrekt; "Konto löschen" öffnet die neue Bestätigung, "Abbrechen" kehrt sauber zu "Konto verwalten" zurück, "Konto endgültig löschen" zeigt den bekannten Demo-Toast; keine doppelten IDs, keine Konsolenfehler nach dem Fix.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v72` erhöht (Pflichtregel).

---

## 2026-09-05 (31) — "Adresse" in "Konto verwalten" optisch ausgeblendet, "Passwort ändern" ergänzt

**Umgesetzt:**
- **"Adresse" nur optisch entfernt, Funktion vollständig erhalten**: Feld-Label und `#addressBlock` bekommen `style="display:none"`, Markup/JS (`profileStreet`/`profileZip`/`profileCity`/`profileCountry`, `[data-save="address"]`-Zweig, `localStorage`-Schlüssel `canspot-address`) bleiben unverändert bestehen — per Test bestätigt: Speichern über die (aktuell unsichtbaren) Adressfelder schreibt weiterhin korrekt nach `localStorage`. Lässt sich später durch einfaches Entfernen der beiden `display:none` wieder einblenden, ohne etwas neu bauen zu müssen.
- **Neuer Punkt "Passwort" unter "Sicherheit"** (ersetzt an derselben Stelle die bisherige "Adresse"-Gruppe): folgt exakt demselben Bearbeiten-Muster wie Name/E-Mail/Geburtsdatum (`.field-block`/`.field-view`/`.field-edit-row`, generischer `[data-edit]`/`[data-save]`-Mechanismus). Zeigt im Ruhezustand eine maskierte Platzhalterzeile ("••••••••") statt eines echten Werts (wie in vergleichbaren Apps üblich, da ein Passwort nie im Klartext angezeigt wird); Antippen des Stifts öffnet drei Felder (Aktuelles Passwort, Neues Passwort, Neues Passwort bestätigen) plus "Speichern".
- **Bewusste Sicherheits-/Demo-Entscheidung**: Anders als Name/E-Mail/Geburtsdatum wird das neue Passwort NICHT in `localStorage` gespeichert (dort gäbe es ohnehin nichts, das es anzeigen oder damit vergleichen könnte, da kein echtes Backend existiert) — "Speichern" validiert nur, dass alle drei Felder ausgefüllt sind und die neuen Passwörter übereinstimmen, zeigt bei Erfolg einen Bestätigungs-Toast und leert die Felder wieder, bei Fehlern bleibt die Zeile offen (gleiches Verhalten wie bei leerem Namen/E-Mail).
- Verifiziert über einen temporären lokalen Server (Mobile + Desktop): "Adresse" ist nicht mehr sichtbar, "Passwort" erscheint an ihrer Stelle unter neuer Gruppe "Sicherheit"; Validierung (leere Felder, nicht übereinstimmende Passwörter) funktioniert und hält die Zeile offen; erfolgreicher Speichervorgang zeigt Toast, leert Felder, schließt die Zeile, legt nachweislich keinen Wert in `localStorage` ab; Adress-Logik im Hintergrund weiterhin nachweislich funktionsfähig; keine doppelten IDs, keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v71` erhöht (Pflichtregel).

---

## 2026-09-05 (30) — Geburtsdatum in "Konto verwalten" ergänzt, "Favoriten" aus "Mein Bereich" entfernt

**Umgesetzt:**
- **Neues Feld "Geburtsdatum"** unter "Konto verwalten" → "Persönliche Daten", direkt zwischen E-Mail-Adresse und Adresse eingefügt. Nutzt exakt dasselbe bereits vorhandene Bearbeiten-Muster wie Name/E-Mail (`.field-block`/`.field-view`/`.field-edit-row`, generischer `[data-edit]`/`[data-save]`-Mechanismus samt `FIELD_FOCUS_TARGET`-Eintrag) - keine neue UI-Logik, nur ein weiteres Feld im bestehenden System. `<input type="date">`, Anzeige über die bereits vorhandene `formatSingleDate()` (liefert "TT.MM.JJJJ", dieselbe Funktion, die auch beim "Preis geprüft"-Hinweis auf den Angebotskarten verwendet wird). Persistiert unter dem neuen `localStorage`-Schlüssel `canspot-birthdate`, wird beim Laden wie Name/E-Mail/Adresse übernommen.
- **"Favoriten" aus "Mein Bereich" entfernt** (war erst in der vorherigen Änderung hinzugekommen): Zeile unter "Meine Aktivität" samt Klick-Handler restlos entfernt. "Meine Aktivität" enthält jetzt nur noch "Zuletzt angesehen" (Platzhalter). Favoriten bleiben unverändert über den bestehenden Favoriten-Tab der Bottom Navigation erreichbar - daran hat sich nichts geändert, es ist lediglich der zusätzliche Verweis darauf in "Mein Bereich" wieder verschwunden.
- Verifiziert über einen temporären lokalen Server (Mobile + Desktop): Geburtsdatum lässt sich eingeben, speichern, wird korrekt formatiert angezeigt und übersteht einen Seiten-Reload; "Favoriten"-Zeile ist weg, "Zuletzt angesehen" weiterhin vorhanden und funktionsfähig; keine doppelten IDs, keine Konsolenfehler auf Mobile und Desktop.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v70` erhöht (Pflichtregel).

---

## 2026-09-05 (29) — "Mein Bereich" und "Einstellungen" neu strukturiert (persönlich vs. App-Verhalten)

**Analyse vor der Umsetzung**: Geprüft, welche der elf betroffenen Funktionen bereits existieren, um nichts doppelt zu bauen: Favoriten-Navigation (`#navFav`-Klick-Handler), Alarme-Verwaltung inkl. Löschen/Toggle (`renderAlertsView()`, Alarme-Tab), Standortverwaltung (`locOverlay`, Startseite), Theme-Umschalter (`.theme-option`, bereits global synchron), Push-Benachrichtigungen (`notifOverlay`, seit letzter Änderung im Burger-Menü), Konto verwalten/Abmelden (`accountManageOverlay`/`logoutBtn`), Rechtstexte (`.profile-legal`) — "Zuletzt angesehen" und echte Datenschutz-/Sortier-/Ansichts-Einstellungen existieren dagegen technisch nicht.

**1. Aus "Mein Bereich" entfernt:**
- Erscheinungsbild (Theme-Umschalter) — lebt jetzt ausschließlich in Einstellungen (war zuvor an beiden Stellen synchron vorhanden).
- Standort & Umkreis (`profileLocRow`) — Standort bleibt ausschließlich über die unveränderte Startseiten-Funktion (`locOverlay`, "Ändern"-Link) verwaltbar, keine zweite Standortverwaltung.
- Preisalarme-Liste unter "Meine Aktivität" (`profileAlertList`/`profileAlertCount`, samt `updateProfileAlertList()`) — bereits vollständig über den Alarme-Tab der Bottom Navigation abgedeckt (dort inkl. Löschen/Pausieren, hier war es nur eine zweite, rein lesende Ansicht).
- (Push-Benachrichtigungen war bereits in der vorherigen Änderung entfernt worden.)

**2. Nach "Einstellungen" verschoben/dort gebündelt** (Burger-Menü, Gruppe "Weitere" enthält jetzt nur noch den einen Punkt "Einstellungen" statt vormals drei separater Punkte):
- Erscheinungsbild (Theme-Umschalter, unverändert Hell/Dunkel/System) — jetzt Unterabschnitt "Erscheinungsbild" ganz oben in Einstellungen.
- Push-Benachrichtigungen — bleibt an der gleichen Stelle wie seit der letzten Änderung, jetzt unter Unterabschnitt "Benachrichtigungen".
- "Hilfe & Feedback" und "Über CanSpot" waren zuvor eigene Top-Level-Accordions neben "Einstellungen" — sind jetzt als verschachtelte Unter-Accordions INNERHALB von Einstellungen untergebracht (Inhalt 1:1 unverändert übernommen).
- Neu als Platzhalter-Unterabschnitt "Datenschutz" ergänzt (siehe Punkt 4).
- **Bewusst NICHT hinzugefügt**: "Suche & Angebote" (bevorzugte Händler/Standard-Sortierung/Listen-Kartenansicht als Einstellung) — dafür gibt es aktuell keine entsprechende speicherbare Einstellung, nur die per-Sitzung-UI auf der Startseite (Sortierung/Ansicht-Umschalter), das wäre etwas anderes. Laut Anfrage bewusst nicht ohne echte Funktion dahinter angelegt.
- **Technischer Nebeneffekt behoben**: Da "Einstellungen" jetzt selbst verschachtelte Accordions enthält, hätte das bestehende Einzel-Öffnen-Verhalten (`toggleMenuAccordion()`) beim Öffnen z.B. von "Datenschutz" das übergeordnete "Einstellungen" gleich wieder geschlossen (die Funktion schloss bisher ALLE offenen Accordions im gesamten Menü, nicht nur Geschwister). Behoben, indem die Funktion jetzt nur noch Geschwister auf derselben Verschachtelungsebene schließt (`:scope`-Query) — vor dem Testen als Bug erkannt und behoben, nicht erst danach aufgefallen.

**3. Wiederverwendete bestehende Funktionen (keine neue Logik gebaut):**
- "Favoriten" unter "Meine Aktivität" löst per `.click()` denselben `#navFav`-Handler aus wie der Bottom-Nav-Tab — keine zweite Favoriten-Logik.
- "Konto verwalten"/"Abmelden" unverändert (`accountManageOverlay`/`logoutBtn`).
- Theme-Umschalter, Push-Benachrichtigungen, Rechtstexte/Version: 1:1 dieselbe Logik/Sheets wie zuvor, nur an neuer Stelle.

**4. Nur als Platzhalter vorbereitet (keine echte Funktion dahinter):**
- "Zuletzt angesehen" unter "Meine Aktivität" — Zeile vorhanden, Tap zeigt einen Demo-Hinweis (gleiches Muster wie `#logoutBtn`/`#deleteAccountBtn`: `showToast(...)`), da es aktuell keinerlei Tracking angesehener Produkte gibt.
- "Datenschutz" unter Einstellungen — aufklappbarer Unterabschnitt mit Hinweistext, keine echten Datenschutz-Einstellungen, bewusst ohne komplexen neuen Bereich.

**Wichtiger Hinweis / Nebenwirkung für die Desktop-/Webapp-Ansicht:** Wie schon bei der letzten Änderung (Push-Benachrichtigungen) gilt: Das Burger-Menü existiert nur in der Mobile-Ansicht. Da jetzt auch das Erscheinungsbild (Theme-Umschalter) ausschließlich dort geführt wird und nicht mehr in "Mein Bereich" (welches auch am Desktop erreichbar ist), können Desktop-/Webapp-Nutzer:innen aktuell weder das Theme noch die Push-Benachrichtigungen ändern — beide Funktionen sind dort komplett unerreichbar geworden. War explizit so angefragt, aber als Konsequenz hier festgehalten.

**Verifiziert** über einen temporären lokalen Server (375px + Desktop-Breite): "Mein Bereich" zeigt exakt die angefragte reduzierte Struktur (Profil, Meine Aktivität mit Favoriten/Zuletzt angesehen, Konto, Rechtstexte); "Favoriten"-Zeile navigiert nachweislich zum bestehenden Favoriten-Tab; "Einstellungen" zeigt die gebündelte Struktur mit funktionierenden verschachtelten Accordions (Öffnen eines Unterabschnitts lässt "Einstellungen" selbst offen, öffnen eines anderen TOP-LEVEL-Punkts wie "Händler" schließt "Einstellungen" weiterhin korrekt); Push-Benachrichtigungen öffnet weiterhin dieselbe Sheet; Schließen/Wiederöffnen des Menüs setzt auch die neuen verschachtelten Accordions zurück; keine doppelten IDs, keine Konsolenfehler auf Mobile und Desktop.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v69` erhöht (Pflichtregel).

---

## 2026-09-05 (28) — "NEU"-Badge-Farbe korrigiert + Push-Benachrichtigungen von "Mein Bereich" ins Burger-Menü verschoben

**Umgesetzt:**
- **Bug behoben: "NEU"-Badge kaum lesbar in beiden Themes.** Ursache war ein CSS-Spezifitätskonflikt: `.fav-row .fav-txt span{color:var(--text-muted)}` (2 Klassen + 1 Element) ist spezifischer als `.new-badge{color:#fff}` (1 Klasse) und hat die weiße Textfarbe deshalb in BEIDEN Themes überschrieben — sichtbares Ergebnis war ein kontrastarmes, gräulich wirkendes "NEU" auf blauem Grund (in der Anfrage als "im Dark Mode ungünstig" beschrieben, betraf aber ebenso den hellen Modus, per `getComputedStyle()` nachgewiesen: tatsächliche Farbe vorher `rgb(105,108,128)` statt des beabsichtigten Weiß). Behoben mit einer eindeutig höher spezifischen Regel `.fav-row .fav-txt .new-badge{color:var(--brand-primary-contrast)}` — dieselbe Lösung, die die App bereits für Text auf `--brand-primary-blue`-Flächen vorsieht (Token war bereits definiert, aber bisher ungenutzt). In beiden Themes verifiziert: jetzt durchgehend `rgb(255,255,255)`.
- **"Push-Benachrichtigungen" von "Mein Bereich" ins Burger-Menü verschoben**: Die Zeile (`profileNotifRow`/`profileNotifSummary`) wurde aus `profileOverlay` entfernt und lebt jetzt als `#menuNotifRow` im "Einstellungen"-Accordion des Burger-Menüs, direkt unter dem Theme-Umschalter. Öffnet weiterhin exakt dieselbe `notifOverlay`-Sheet/-Logik wie zuvor (über den bereits bestehenden generischen `.profile-row[data-open]`-Mechanismus, keine neue Logik) — das Menü bleibt dabei im Hintergrund offen, analog zum bisherigen Verhalten von "Mein Bereich" hinter seinen Untermenüs. Die Zusammenfassungszeile ("X von 4 Benachrichtigungen an") wurde von `refreshProfileSummaries()` in eine neue, kleinere Funktion `refreshMenuNotifSummary()` verschoben (nicht dupliziert) und aktualisiert sich beim Öffnen des Accordions sowie beim Ändern/Speichern der Einstellungen live.
- Verifiziert über einen temporären lokalen Server: "Push-Benachrichtigungen" erscheint korrekt unter Burger-Menü → Einstellungen mit live-aktualisierter Zusammenfassung, öffnet dieselbe Sheet wie zuvor, Menü bleibt dahinter offen; "Mein Bereich" zeigt die Zeile nicht mehr, "Standort & Umkreis" dort unverändert vorhanden; keine doppelten IDs, keine Konsolenfehler auf Mobile und Desktop.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v68` erhöht (Pflichtregel).

**Wichtiger Hinweis / mögliche Nebenwirkung:** Das Burger-Menü existiert ausschließlich in der Mobile-Ansicht (≤540px, siehe `.iconbtn-menu`-Regel in `index.html`). Da "Push-Benachrichtigungen" jetzt AUSSCHLIESSLICH dort geführt wird und nicht mehr in "Mein Bereich" (das sowohl mobil als auch in der Webapp/Desktop-Ansicht erreichbar ist), sind die Push-Einstellungen in der Desktop-/Webapp-Ansicht damit aktuell für Nutzer nicht mehr erreichbar — das war explizit so angefragt ("aus Mein Bereich verschwinden"), aber als Konsequenz festgehalten, falls das nicht beabsichtigt war.

---

## 2026-09-05 (27) — Mehrfachauswahl im Postfach entfernt, Wisch-Farben differenziert, hängenbleibendes Accordion-Grau behoben

**Umgesetzt:**
- **Haken-Symbol/Mehrfachauswahl komplett entfernt**: `#newsSelectToggleBtn`, `#newsSelectBar` (Zähler + Sammel-Löschen-Button), die Checkbox-Optik (`.news-inbox-check`) sowie der gesamte zugehörige State/Code (`newsSelectionMode`, `selectedNewsIds`, `resetNewsSelectionMode()`, `updateNewsSelectBar()`, `toggleNewsSelection()`) restlos aus Markup/CSS/JS entfernt — per Grep verifiziert, keine toten Referenzen übrig. Die Kopfzeile der Postfach-Ansicht zeigt jetzt wieder nur den "← Neuigkeiten"-Zurücklink wie vor der Mehrfachauswahl-Funktion; Wisch-Löschen und Wisch-Gelesen/Ungelesen pro Nachricht bleiben unverändert bestehen (waren nicht Teil der jetzt entfernten Auswahl-Funktion).
- **Farbliche Unterscheidung beim Wisch-nach-rechts**: Die aufgedeckte Fläche zeigt jetzt zwei unterschiedliche Farben je nachdem, was beim Loslassen passiert — Blau/Info-Ton (`--status-info-bg/-text`), wenn die Nachricht dadurch als GELESEN markiert wird, Gold/Highlight-Ton (`--highlight-bg/-text`, dieselbe Farbe wie die "Demnächst"-Hinweise), wenn sie dadurch wieder als UNGELESEN markiert wird — beide Male mit demselben Umschlag-Symbol, nur die Farbe ändert sich mit dem tatsächlichen Ziel-Status.
- **Bug behoben: Accordion-Reiter blieben nach dem Schließen angegraut.** Ursache: `.profile-row:hover` (Hintergrundfarbe beim Hover) bleibt auf reinen Touch-Geräten nach einem Tap "hängen", weil kein `mouseout`-Ereignis folgt, das den Hover-Zustand zurücksetzt — ein bekanntes Verhalten von WebKit/Chrome auf Touchscreens. Behoben an der Wurzel, indem die Hover-Regel in `@media (hover:hover)` gekapselt wurde: Geräte ohne echten Zeiger-Hover (Touch) wenden die Regel nun gar nicht erst an, sodass nichts "hängen bleiben" kann; Geräte mit echter Maus (Webapp/Desktop-Nutzung von "Mein Bereich" o.ä.) behalten den Hover-Effekt unverändert. Betrifft dieselbe `.profile-row`-Klasse, die auch die Menü-Accordions nutzen — bewusst an der Wurzel statt nur für die Burger-Menü-Reiter gefixt, da derselbe Fehler auf Touch-Geräten grundsätzlich überall dort aufgetreten wäre, wo `.profile-row` verwendet wird (z.B. auch in "Mein Bereich").
- Verifiziert über einen temporären lokalen Server: In der emulierten Mobile-Ansicht (`matchMedia('(hover:hover)')` liefert dort korrekt `false`, wie auf einem echten Touchscreen) bleibt der Hintergrund eines Accordion-Reiters vor und nach dem Öffnen/Schließen nachweislich identisch (`rgb(8,10,13)`, keine Verfärbung); in der emulierten Desktop-Ansicht liefert dieselbe Abfrage `true`, Hover-Verhalten dort unverändert. Wisch nach rechts zeigt nachweislich Blau bei ungelesen→gelesen und Gold bei gelesen→ungelesen (Screenshots). Kein Auswahl-Icon mehr sichtbar, keine doppelten IDs im Dokument, Detail-Navigation (öffnen/zurück) weiterhin funktionsfähig, keine Konsolenfehler auf Mobile und Desktop.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v67` erhöht (Pflichtregel).

---

## 2026-09-05 (26) — X-Position im Burger-Menü korrigiert + Wisch nach rechts markiert Gelesen/Ungelesen

**Umgesetzt (Burger-Menü, nur Mobile):**
- **Schließen-X ausgerichtet**: `#menuOverlay .sheet-close` (scoped, alle anderen Sheets behalten unverändert `top:14px`) sitzt jetzt auf einer Linie mit der "Menü"-Überschrift (per `getBoundingClientRect()` nachgemessen und nachjustiert, vertikale Mitten stimmen jetzt bis auf 0,25px überein). Da es sich um denselben, immer sichtbaren Button handelt, der über alle drei internen Ansichten (Hauptmenü/Neuigkeiten/Detail) hinweg bestehen bleibt, gilt die neue Position automatisch überall im Burger-Menü, ohne zusätzlichen Code je Ansicht.
- **Wisch nach rechts markiert Gelesen/Ungelesen**: Jede Nachricht im Postfach lässt sich jetzt in beide Richtungen wischen — links weiterhin Löschen (roter Papierkorb), rechts neu Gelesen/Ungelesen umschalten (blaues Umschlag-Symbol `i-mail`, neu im SVG-Sprite). Die Beschriftung der aufgedeckten Fläche passt sich automatisch der aktuellen Leserichtung an ("Als gelesen markieren" bzw. "Als ungelesen markieren"). Neue Funktion `toggleNotificationRead()` (schaltet in `NEWS_READ_KEY` in beide Richtungen um, im Unterschied zu `markNotificationRead()`, das beim Öffnen einer Nachricht weiterhin nur "gelesen setzen" kann) — Badge an Burger-Icon/Menüzeile aktualisiert sich sofort mit.
- **Bug gefunden und behoben, bevor er auffiel**: Die beiden Wisch-Flächen (Löschen rechts, Gelesen/Ungelesen links) waren zunächst beide mit `inset:0` über die komplette Zeile gelegt — dadurch lag die später im Markup stehende rote Löschfläche auch beim Wischen nach rechts über der blauen Fläche und verdeckte deren Symbol (per Screenshot bemerkt: rote Fläche ohne sichtbares Symbol beim Rechts-Wisch). Behoben, indem beide Flächen nur noch ihren eigenen 76px breiten Rand einnehmen (`left:0`/`right:0` statt `inset:0`), sodass sie sich nicht mehr überlappen.
- Verifiziert über einen temporären lokalen Server (375px, Touch-Events simuliert): X sitzt sichtbar auf Höhe von "Menü"; Wisch nach rechts deckt korrekt die blaue Umschlag-Fläche auf, Tippen schaltet den Gelesen-Status nachweislich in beide Richtungen um (`loadReadNewsIds()` geprüft), Wisch nach links funktioniert weiterhin unverändert (Löschen); andere Sheets (`locOverlay` exemplarisch) behalten unverändert `top:14px` für ihr X; Desktop-/Webapp-Ansicht unverändert, keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v66` erhöht (Pflichtregel).

---

## 2026-09-05 (25) — "Standort & Entfernung" durch "Neuheiten" ersetzt + Postfach: Wisch-Löschen und Mehrfachauswahl

**Umgesetzt (Burger-Menü, nur Mobile):**
- **"Standort & Entfernung" komplett entfernt**: Zeile, Panel, Zusammenfassungstext (`menuLocRow`/`menuLocPanel`/`menuLocSummary`) sowie die dafür zuletzt eingeführte scoped-CSS (`.menu-acc-loc`, `.menu-acc-info`) restlos aus Markup/CSS/JS entfernt — per Grep verifiziert, keine toten Referenzen übrig. Betraf ausschließlich diesen Menüpunkt; die eigentliche Standortlogik (`locationLabel`/`radiusKm`/`searchWide`, `locOverlay`, "Ändern" auf der Startseite, "Mein Bereich") ist davon nicht betroffen und bleibt unverändert erreichbar.
- **Neuer Accordion "Neuheiten" an derselben Stelle** (Gruppe "Mein CanSpot", erste Position): listet alle Produkte mit `products[].isNew === true` aus `deals.json` — rein informativ (Bild, Name, "NEU"-Badge, Marke/Größe), keine eigene Angebots-/Klicklogik nötig, wiederverwendet die bestehenden `.fav-row/-emoji/-txt`-Klassen aus "Mein Bereich"/Favoriten. Neues Icon `i-sparkle` im SVG-Sprite ergänzt.
- **Demo-Datensatz ergänzt**: `deals.json` bekommt ein neues Produkt `p29` "Red Bull The Glacier Edition" (`isNew: true`, echtes hotlinked Produktbild von redbull.com, wie alle anderen Produktbilder). `_meta.fields` in `deals.json` sowie die `products[]`-Doku in `CLAUDE.md` um das neue optionale Feld `isNew` ergänzt (Konvention: Datenform bleibt an einer Stelle dokumentiert).
- **Neuigkeiten-Postfach: Wisch-Löschen** — jede Nachricht steckt jetzt in einem Wrapper mit dezenter Löschfläche dahinter (rotes Papierkorb-Symbol, `status-error`-Farbtöne wie beim bereits vorhandenen `.profile-action-danger`); Wisch nach links deckt sie auf (dieselbe "dominante Richtung"-Erkennung wie beim bestehenden `initSwipeToClose()`, nur horizontal), ein Tap auf den Papierkorb löscht sofort, ein Tap auf die noch offene Zeile schließt sie nur wieder (kein versehentliches Öffnen der Detailansicht). Nur eine Zeile gleichzeitig aufgedeckt.
- **Mehrfachauswahl über Haken-Symbol** (an typischen Mail-/Notification-Apps orientiert: "Auswählen"-Umschalter oben, der zwischen Haken- und X-Icon wechselt): im Auswahl-Modus ersetzt eine antippbare Kreis-Checkbox den Gelesen-Punkt je Zeile, eine Leiste unter der Liste zeigt die Anzahl Ausgewählter und einen "Löschen"-Button (deaktiviert bei 0 Auswahl). Wisch-Geste ist während der Auswahl deaktiviert, um Konflikte zu vermeiden.
- **Datenmodell für Löschungen**: neuer, zu `NEWS_READ_KEY` paralleler `localStorage`-Schlüssel `canspot-news-deleted` (`loadDeletedNewsIds()`/`saveDeletedNewsIds()`/`deleteNotifications()`), gefiltert direkt in `getRelevantNotifications()` — dadurch wirkt eine Löschung automatisch überall dort, wo diese Funktion als Quelle dient (Postfach im Burger-Menü UND die unveränderte Desktop-Glocke/`newsOverlay`/Badge), ohne zusätzliche Sonderfälle.
- **Bug gefunden und behoben**: Der neue "Auswählen"-Button lag anfangs unter dem immer sichtbaren X-Schließen-Button des Drawers (beide oben rechts, sich überlappende Klickflächen laut `getBoundingClientRect()`-Check). Behoben mit `padding-right:46px` auf der neuen `.menu-view-topbar`, damit für den fest positionierten X-Button Platz bleibt — vor dem Ausliefern erkannt und korrigiert, nicht erst vom Nutzer gemeldet.
- Verifiziert über einen temporären lokalen Server (375px): "Neuheiten" zeigt Red Bull The Glacier Edition korrekt mit Badge und Bild; alle übrigen Accordions (Händler/Einstellungen/Hilfe/Über CanSpot) weiterhin unverändert funktionsfähig; Wisch-Geste (per simulierten Touch-Events getestet) deckt die Löschfläche korrekt auf, Tap auf Papierkorb löscht sofort, Tap auf offene Zeile schließt nur; Mehrfachauswahl (2 Einträge wählen → Löschen) funktioniert und beendet den Auswahl-Modus danach automatisch; gelöschte/gelesene Zustände bleiben nach Schließen/Wiederöffnen des Menüs erhalten; leerer Zustand korrekt; Desktop-/Webapp-Glocke (`newsOverlay`) unverändert funktionsfähig; keine doppelten IDs im gesamten Dokument (Grep-Check); keine Konsolenfehler während der gesamten Testreihe.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v65` erhöht (Pflichtregel).

**Bekannte Einschränkung:** Wie in den vorherigen Änderungen wurden reale Touch-Gesten teils über synthetische `TouchEvent`s statt über das (in dieser Sitzung wiederholt instabile) Browser-Automatisierungstool verifiziert — ein kurzer manueller Gegentest der Wisch-Geste auf einem echten Touchscreen wird empfohlen.

---

## 2026-09-04 (24) — Neuigkeiten im Burger-Menü: eigenes Postfach statt separatem Overlay

**Analyse vor der Umsetzung** (wie angefordert): Geprüft, dass es bereits eine vollständige Notification-Logik gibt (`getRelevantNotifications()`, `NEWS_READ_KEY`/`loadReadNewsIds()`/`saveReadNewsIds()`, `updateNotifDot()`), bislang aber nur text-basierte Hinweise ohne strukturierte Produkt-/Angebotsreferenz lieferte, sowie eine zweite, ähnliche aber unabhängige Funktion (`computeFavoriteEvents()`/`newsCardHtml()`) für die große Karten-Darstellung in der "Alarme"-Ansicht — bewusst NICHT angefasst, da außerhalb des Auftrags. Für "Zum Angebot" wiederverwendet: dieselbe `openStoreDetail(dealId)`-Funktion/Ansicht, die auch die "Zum Angebot"-Buttons auf den Angebotskarten überall sonst in der App öffnen.

**Umgesetzt (nur Mobile Burger-Menü — die Desktop-/Webapp-Glocke bleibt unverändert):**
- **Kein separates Overlay mehr für Mobile**: "Neuigkeiten" im Burger-Menü öffnet nicht länger `newsOverlay` (das war zuvor ein eigenes Bottom-Sheet über der App). Stattdessen verwandelt sich der Inhalt des bereits vorhandenen rechten Side-Drawers in eine "Postfach"-Ansicht — der Drawer selbst (Breite, Position, Backdrop, Animation) ist unverändert derselbe wie zuvor.
- **Mini-Navigation innerhalb des Drawers**: Drei Ansichten (`#menuViewMain`, `#menuViewNews`, `#menuViewNewsDetail`) liegen im selben `.sheet-inner`, immer ist genau eine sichtbar (`showMenuView()`), der Wechsel bekommt eine kurze (0.22s), dezente horizontale Slide-+Fade-Animation je nach Richtung (vorwärts = von rechts, zurück = von links) über CSS-`@keyframes` — kein neues Layoutsystem, keine übertriebene Animation. Navigation: Menü → "Neuigkeiten" antippen → Postfach → Nachricht antippen → Detailansicht → "← Neuigkeiten" → Postfach → "← Neuigkeiten" → Menü. Schließen des gesamten Menüs (X/Tippen außerhalb/ESC) setzt die Navigation auf die Hauptansicht zurück (Erweiterung derselben Stelle in `closeSheet()`, die das bereits für die Accordion-Abschnitte tut).
- **Postfach-Liste** (`renderNewsInboxList()`): kompakte Nachrichtenzeilen statt großer Karten — Punkt links (gefüllt = ungelesen, nur umrandet + gedämpfter Text = gelesen), Nachrichtentext 1:1 aus `getRelevantNotifications()` (`n.text`, unverändert), darunter optional eine Metazeile "Händler · Preis · Größe" wenn ein konkretes Angebot dahintersteht. Dezente Trennlinie zwischen Zeilen, kein Rahmen/Schatten. Leerer Zustand ("Keine neuen Neuigkeiten.") wenn keine Favoriten-Neuigkeiten vorliegen.
- **Detailansicht** (`renderNewsDetail()`): Produktname, Bild (`deal.img || product.img`, dieselbe Logik wie an anderen Stellen im Projekt), Status-Pill (wiederverwendet `.news-card-status`/`.green`/`.gold`, bereits vorhandene Klassen aus der Alarme-Kartenansicht), Händler, Preis + Packungsgröße (`formatSize(deal)`, bestehende Funktion), Normalpreis, Ersparnis, Gültigkeitszeitraum (`formatDateRange()`, bestehende Funktion) und "Zum Angebot"-Button. Der Button ruft `openStoreDetail(deal.id)` auf — exakt dieselbe Funktion/Ansicht wie überall sonst in der App, keine zweite Angebotslogik. Der Demo-Platzhalter-Eintrag (kein echtes Angebot vorhanden) zeigt nur Produktbild + Text + "(Beispiel)", ohne Preisblock/Button.
- **Gelesen-Status pro Nachricht statt pauschal beim Öffnen**: Anders als die Desktop-Glocke (`openNewsSheet()` markiert weiterhin beim Öffnen alles auf einen Schlag als gelesen, unverändert) markiert das Postfach jede Nachricht erst einzeln beim Antippen als gelesen (`markNotificationRead()`), damit der Punkt tatsächlich zwischen "gelesen"/"ungelesen" pro Eintrag unterscheidet. Beide Wege schreiben in denselben `NEWS_READ_KEY` in `localStorage` und rufen danach `updateNotifDot()` auf, das weiterhin einheitlich Glocken-Punkt (Desktop), Burger-Icon-Punkt und den Punkt neben "Neuigkeiten" im Menü synchron hält.
- **Datenstruktur zukunftsfest erweitert**: `getRelevantNotifications()` liefert je Eintrag jetzt zusätzlich `dealId`/`productId` (rein additiv, bestehende Konsumenten wie `renderNewsList()` unverändert kompatibel) — damit lassen sich später echte News-Daten anschließen, ohne die Postfach-/Detail-Logik anzufassen, solange sie dieselbe Objektform (`id`, `text`, `icon`, `cls`, optional `dealId`/`productId`) liefern.
- Verifiziert über einen temporären lokalen Server (375px): Postfach zeigt alle vier Demo-Neuigkeiten mit korrektem Gelesen/Ungelesen-Zustand, Antippen öffnet die Detailansicht mit korrekten Angebotsdaten (Preis, Normalpreis, Ersparnis, Gültigkeit) und markiert die Nachricht als gelesen (Punkt an Burger-Icon und Menüzeile aktualisiert sich sofort, verschwindet vollständig nach dem Lesen aller vier), "Zum Angebot" öffnet `storeDetailOverlay` mit den richtigen Händler-/Preisdaten während das Menü im Hintergrund offen bleibt, Zurück-Navigation (Detail→Postfach→Menü) funktioniert, X/ESC setzen die Navigation zurück auf die Hauptansicht, leerer Zustand korrekt gerendert, Desktop-/Webapp-Glocke (`#notifBtn`/`newsOverlay`) unverändert funktionsfähig getestet — keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v64` erhöht (Pflichtregel).

**Bekannte Einschränkung:** Wie in den vorherigen Änderungen war der reale Maus-/Touch-Klick über das Browser-Automatisierungstool in dieser Sitzung wiederholt instabil; alle Interaktionen wurden zusätzlich über direkte `.click()`-Aufrufe auf denselben Elementen verifiziert, die ein echter Tap ebenfalls träfe.

---

## 2026-09-04 (23) — "Standort & Entfernung" im Burger-Menü: ein zusammenhängender Container statt zwei Karten

**Umgesetzt (ausschließlich dieser eine Menüpunkt, alle anderen Accordions unverändert):**
- **Problem**: Titel-Zeile ("Standort & Entfernung") und die darunter aufklappende Standort-Zusammenfassung ("59821 Arnsberg · 10 km") hatten jeweils ihren eigenen Hintergrund/Rand/Eckenradius (geteilte `.menu-acc-toggle`/`.menu-acc-panel`-Basisregeln aus der vorherigen Änderung) — dadurch wirkten sie wie zwei getrennte, hart aneinandergesetzte Flächen statt einem Menüpunkt.
- **Fix, streng auf diesen einen Eintrag begrenzt**: Neue Wrapper-Klasse `.menu-acc-loc` (nur auf dem `<div class="menu-acc ...">` von "Standort & Entfernung" ergänzt) trägt jetzt allein Hintergrund/Rand/Eckenradius; Titel-Zeile (`#menuLocRow`) und Panel (`#menuLocPanel`) selbst sind per ID-Selektor auf transparent/randlos/radius:0 gesetzt und übernehmen optisch keine eigene Kartenform mehr. Eine sehr dezente 1px-Trennlinie (`border-top`, nur im aufgeklappten Zustand sichtbar) trennt Titel und Standort-Info, statt einer harten zweiten Fläche. Alle anderen Accordions (Händler, Einstellungen, Hilfe & Feedback, Über CanSpot) nutzen weiterhin unverändert die geteilten `.menu-acc`/`.menu-acc-toggle`/`.menu-acc-panel`-Basisregeln von vorher.
- Bestehende Funktionalität vollständig erhalten: Klick klappt weiterhin über `toggleMenuAccordion()` auf/zu, Standort-/Umkreis-Text kommt weiter aus derselben Formel wie `refreshProfileSummaries()`, Icon/Chevron/Farben/Schrift/Abstände unverändert übernommen.
- Verifiziert über einen temporären lokalen Server (375px): Standort-Container ist jetzt eine einzige zusammenhängende Fläche mit außen sichtbarem `border-radius:16px` (Wrapper) und `border-radius:0` an Zeile/Panel, Trennlinie nur bei geöffnetem Zustand sichtbar, Zuklappen reduziert die Panel-Höhe sauber auf 0; "Händler" (und die übrigen Accordions) exemplarisch geprüft und unverändert (eigener Zeilen-Hintergrund, oben abgerundet, Wrapper transparent, wie zuvor) — keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v63` erhöht (Pflichtregel).

---

## 2026-09-04 (22) — Burger-Menü: Einträge klappen jetzt inline auf statt neue Fenster zu öffnen

**Umgesetzt (ausschließlich innerhalb von #menuOverlay):**
- **Recherche**: Vor der Umsetzung nach dem Accordion-/Expand-Muster in verbreiteten Mobile-Apps mit Burger-/Side-Menüs geschaut (z.B. verschachtelte, aufklappbare Kategorien statt tiefer Navigation, rotierender Chevron als Auf/Zu-Indikator, "wachsende Karte"-Optik beim Aufklappen) und daraus einen eigenen, zum bestehenden CanSpot-Design passenden Stil abgeleitet (siehe unten).
- **Alle Menüpunkte außer "Neuigkeiten" sind jetzt Accordions**: Ein Tap auf "Standort & Entfernung", "Händler", "Einstellungen", "Hilfe & Feedback" oder "Über CanSpot" klappt den Inhalt direkt im Menü auf/zu (`toggleMenuAccordion()`), statt wie zuvor ein eigenes Sheet zu öffnen. "Neuigkeiten" bleibt bewusst die einzige Ausnahme und öffnet weiterhin `openNewsSheet()` wie zuvor — als klares visuelles Signal dafür behält nur diese Zeile den navigierenden "›"-Pfeil, alle Accordion-Zeilen zeigen stattdessen einen Chevron, der beim Aufklappen um 180° rotiert und sich blau (CanSpot-Akzent) einfärbt.
- **Eigener Stil "wachsende Karte"**: Die aufgeklappte Zeile verliert ihre untere Eckrundung, das Panel darunter setzt nahtlos mit gleichem Hintergrund/Rand fort (`border-radius` nur unten) — wirkt wie eine einzige Karte, die sich ausdehnt, statt wie ein separat wirkendes, zweites Element. Sanfte `max-height`-Transition (0.3s) für ein "dezentes, flüssiges" Auf-/Zuklappen ohne JS-Höhenberechnung.
- **Nur ein Abschnitt gleichzeitig offen**: Öffnen eines neuen Accordions klappt ein zuvor offenes automatisch zu (klassisches Akkordeon-Verhalten), damit das Menü nicht unbegrenzt wächst.
- **Inhalte pro Accordion, alles wiederverwendet statt neu erfunden**:
  - *Standort & Entfernung*: zeigt die aktuelle Standort-/Umkreis-Zusammenfassung read-only an, mit exakt derselben Formel wie `refreshProfileSummaries()` für `#profileLocSummary` (`${locationLabel} · ${radiusKm} km` bzw. "Ganz Deutschland") — keine zweite unabhängige Standortlogik. Echtes Ändern des Standorts bleibt unverändert über "Ändern" auf der Startseite bzw. "Mein Bereich" möglich (dortige Sheets/Funktionen wurden nicht angefasst).
  - *Händler*: identischer Inhalt wie zuvor im separaten Sheet (`renderStoreListSheet()`, unverändert übernommen), jetzt inline im Panel statt in einem eigenen Fenster — Öffnungsstatus/Route weiterhin über `isStoreOpenNow()`/`formatHours()`.
  - *Einstellungen*: zeigt direkt den Theme-Umschalter (Hell/Dunkel/System) — dieselben `.theme-option`-Buttons/Klasse wie in "Mein Bereich", dank bereits klassenbasierter (nicht ID-gebundener) Bindung in `applyTheme()` automatisch synchron mit den Buttons dort (verifiziert: Klick im Menü ändert auch den Zustand in "Mein Bereich" und umgekehrt). Die volle "Mein Bereich"-Ansicht (Konto, Push-Benachrichtigungen, Abmelden) bleibt bewusst dort und wird hier nicht dupliziert.
  - *Hilfe & Feedback* / *Über CanSpot*: derselbe Platzhaltertext bzw. dieselben Rechtstexte/Version wie zuvor im jeweiligen Sheet, jetzt direkt inline.
- **Aufräumen**: Die zuvor extra angelegten Sheets `storeListOverlay`, `helpOverlay`, `aboutOverlay` (nur für diese eine Burger-Menü-Funktion gebaut, sonst nirgends verlinkt) wurden entfernt, ihr Inhalt lebt jetzt direkt in den jeweiligen Accordion-Panels — dadurch auch aus `initSwipeToClose`/`initHandleTapToClose` entfernt (sonst hätten diese beim Start `getElementById(...)` von nicht mehr existierenden IDs erhalten und wären mit einem TypeError abgebrochen).
- **Zurücksetzen beim Schließen**: `closeSheet("menuOverlay")` klappt alle Accordions automatisch wieder ein (wie der bereits bestehende Sonderfall für `profileOverlay` in derselben Funktion), damit das Menü beim nächsten Öffnen wieder im Ausgangszustand startet.
- **Bugfix im Zuge der Prüfung**: `initSwipeToClose`/`initHandleTapToClose` liefen zunächst noch mit den (aus dem letzten Schritt stammenden) IDs `storeListOverlay`/`helpOverlay`/`aboutOverlay` in ihren Arrays — nach deren Entfernung aus dem Markup hätte das beim Laden zu einem JS-Fehler geführt (Zugriff auf `null.querySelector(...)`). Vor dem Testen bereits erkannt und behoben, indem die drei IDs aus beiden Arrays entfernt wurden.
- Verifiziert über einen temporären lokalen Server (375px): alle fünf Accordion-Zeilen klappen einzeln korrekt auf/zu (nur je eine gleichzeitig offen), Menü bleibt beim Klick auf eine Accordion-Zeile offen (schließt sich nicht mehr wie zuvor), "Neuigkeiten" öffnet weiterhin das Sheet und schließt dabei das Menü, Theme-Wechsel im Menü synchronisiert sich sofort mit "Mein Bereich", X-Button setzt aufgeklappte Panels beim Schließen zurück, Desktop/Webapp-Ansicht (≥541px) unverändert (Glocke sichtbar, Burger weiterhin `display:none`) — keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v62` erhöht (Pflichtregel).

**Bekannte Einschränkung:** Wie in der vorherigen Änderung war der reale Maus-/Touch-Klick über das Browser-Automatisierungstool in dieser Sitzung wiederholt instabil (Timeouts); alle Interaktionen wurden zusätzlich über direkte `.click()`-Aufrufe auf denselben Elementen verifiziert, die ein echter Tap ebenfalls träfe. Ein kurzer manueller Gegentest auf einem echten Gerät wird weiterhin empfohlen.

---

## 2026-09-04 (21) — Burger-Menü von Bottom-Sheet zu Side-Drawer (rechts) umgebaut

**Umgesetzt (ausschließlich #menuOverlay, alle anderen Sheets unverändert):**
- **Position/Animation**: `#menuOverlay .sheet` schiebt sich jetzt horizontal von rechts ein/aus (`transform:translateX(100%)` → `translateX(0)` beim Öffnen, `transition .28s cubic-bezier(.32,.72,0,1)`) statt wie zuvor von unten (`translateY`). Der Overlay-Backdrop richtet den Sheet-Container jetzt rechtsbündig aus (`justify-content:flex-end;align-items:stretch`) statt unten zentriert.
- **Breite/Höhe**: `width:82%` (max. 340px) und `height:100%` statt voller Bildschirmbreite/`max-height:86vh` — das Menü reicht sauber bis zum unteren Bildschirmrand, links bleibt der abgedunkelte Hintergrund der Startseite sichtbar und anklickbar (der bestehende generische Overlay-Klick-Handler `if(e.target===ov) closeSheet(...)` deckt das Schließen beim Tippen außerhalb bereits ab, ohne neuen Code).
- **Ecken**: `border-radius` nur noch links (`var(--radius-2xl) 0 0 var(--radius-2xl)`) statt oben beidseitig, passend zur neuen Andockung am rechten Rand.
- **Drag-Handle entfernt**: Das `.handle`-Element wurde aus dem Markup von `#menuOverlay` entfernt (ergibt bei einem seitlichen Drawer keinen Sinn) — Kopfbereich zeigt jetzt direkt "Menü" + × nebeneinander, "Neuigkeiten" folgt sofort darunter als erste Zeile, genau wie angefordert.
- **Wisch-/Handle-Geste entfernt**: `menuOverlay` aus den Arrays `initSwipeToClose`/`initHandleTapToClose` herausgenommen (vertikales Wisch-nach-unten passt nicht zu einem Drawer, der sich horizontal nach rechts ausschiebt, und hätte mit der neuen `translateX()`-Logik kollidiert). Schließen läuft jetzt über das bestehende X, Tippen auf den abgedunkelten Bereich und ESC (unverändert von der letzten Änderung).
- **Unverändert wie gewünscht**: Farben, Icons, Menüstruktur (Neuigkeiten → Mein CanSpot → Weitere, keine neuen Einträge), `.profile-row`-Zeilenstyling, Abstände zwischen den Gruppen, News-Badge-Logik (`updateNotifDot()` steuert weiterhin Burger-Dot + Menü-Zeilen-Dot aus derselben Quelle) — keine dieser Regeln wurde angefasst, nur Position/Maße/Animation des Sheet-Containers selbst.
- Verifiziert über einen temporären lokalen Server (375px): Menü schiebt sich sichtbar von rechts ein (Screenshot bestätigt ~82% Breite/307px bei 375px Viewport, linke Bildhälfte bleibt sichtbar/abgedunkelt), Badge an Burger-Icon und bei "Neuigkeiten" erscheint/verschwindet weiterhin synchron, X-Button/Tippen-außerhalb (per dispatchtem Klick auf das Overlay-Element)/ESC schließen zuverlässig, andere Sheets (`locOverlay` exemplarisch geprüft) sind unverändert Bottom-Sheets mit Handle und voller Breite — keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v61` erhöht (Pflichtregel).

**Bekannte Einschränkung:** Der reale Maus-/Touch-Klick über das Browser-Automatisierungstool war in dieser Sitzung wiederholt instabil (Timeouts) — die Tap-außerhalb-schließt-Funktion wurde daher über ein dispatchtes Klick-Event auf dasselbe Overlay-Element verifiziert, das ein echter Tap ebenfalls träfe (identischer, unveränderter Code-Pfad wie bei allen anderen Sheets). Empfehlung: bei Gelegenheit einmal manuell auf einem echten Gerät/Browser gegentesten.

---

## 2026-09-04 (20) — Glocke in Mobile-Header durch Burger-Menü ersetzt (nur Mobile)

**Umgesetzt (ausschließlich Mobile, Webapp unverändert):**
- **Abgrenzung Mobile/Webapp ermittelt**: Es gibt keine separate Desktop-Komponente/-Datei — `index.html` ist eine einzige responsive Ansicht (`.app{max-width:560px}`), bei der die bestehende Regel `@media (max-width:540px){...}` (Kommentar dort: "Desktop >540px … Mobile ≤540px") schon vorher als die einzige Mobile/Webapp-Grenze im Projekt diente. Die Änderung nutzt exakt dieselbe Grenze wieder, statt einen neuen Breakpoint einzuführen.
- **Neuer Burger-Button** (`#menuBtn`, `.iconbtn-menu`) zusätzlich zum bestehenden `#notifBtn` (Glocke) im Header eingefügt; per CSS ausschließlich unterhalb 540px sichtbar (`#notifBtn{display:none}` / `.iconbtn-menu{display:flex}` innerhalb der Mobile-Media-Query) — oberhalb 540px (Webapp) bleibt exakt die bisherige Glocke mit ihrem bisherigen Klick-Verhalten (`openNewsSheet()`) erhalten, keine einzige bestehende Desktop-Regel wurde verändert.
- **News-Badge**: `updateNotifDot()` steuert jetzt drei Punkte statt einem (`#notifDot` Glocke/Webapp, `#menuDot` Burger-Icon, `#menuNewsDot` im Menüpunkt "Neuigkeiten") aus derselben Quelle (`getRelevantNotifications()` + `NEWS_READ_KEY`) — bereits auf der geschlossenen Startseite sichtbar, da `updateNotifDot()` schon beim Init (`finishInit → renderFavList`) läuft.
- **Neues Burger-Menü-Sheet** (`#menuOverlay`, neues Icon `i-menu`) nutzt das bestehende generische Sheet-System (`openSheet`/`closeSheet`, Tap-außerhalb-schließt, Wisch-Geste, Handle-Tap — alle in die jeweiligen Arrays ergänzt) plus neu: ESC-Taste schließt gezielt nur `menuOverlay` (kein globaler Eingriff in andere Sheets). Struktur: "Neuigkeiten" (immer zuerst) → Gruppe "Mein CanSpot" (Standort & Entfernung, Händler) → Gruppe "Weitere" (Einstellungen, Hilfe & Feedback, Über CanSpot). Zeilen nutzen die bestehenden `.profile-row/-icon/-txt/-arrow`-Klassen aus "Mein Bereich" wieder (kein neues Row-Design nötig).
- **Wiederverwendete bestehende Logik/Sheets**: "Neuigkeiten" → `openNewsSheet()` (identisch zur bisherigen Glocke); "Standort & Entfernung" → bestehendes `locOverlay`; "Einstellungen" → bestehendes `profileOverlay` ("Mein Bereich", enthält bereits Erscheinungsbild/Standort/Push/Konto); "Über CanSpot" → neues Sheet, aber mit denselben Rechtstexten/Versionsangabe wie bisher in `profileOverlay` (`.profile-legal`/`.profile-version`, dort unverändert weiterhin vorhanden).
- **Neu gebaut (kein Vorgänger vorhanden)**: "Händler"-Übersicht (`#storeListOverlay`/`renderStoreListSheet()`) — listet alle Händler aus den bereits geladenen `storeBranches`/`storeLogos` (aus `deals.json`), Öffnungsstatus/Adresse/Öffnungszeiten über die bestehenden Funktionen `isStoreOpenNow()`/`formatHours()` (keine zweite unabhängige Logik), Route-Button nutzt denselben Google-Maps-Link-Aufbau wie die Händler-Detailansicht. "Hilfe & Feedback" (`#helpOverlay`) ist eine reine Platzhalter-Ansicht ohne echte Datenquelle.
- **Neue Icons** im SVG-Sprite ergänzt (`i-menu`, `i-store`, `i-settings`, `i-help`, `i-info`) im bestehenden Linien-Icon-Stil (`stroke`, `stroke-width:2`), keine Emojis verwendet.
- Verifiziert über einen temporären lokalen Server: bei 375px Viewport zeigt der Header den Burger (nicht die Glocke), Badge erscheint bereits vor dem Öffnen bei ungelesenen Neuigkeiten und verschwindet nach dem Öffnen (Burger- und Glocken-Badge synchron); alle sechs Menüpunkte öffnen die jeweils korrekte Ansicht und schließen zuvor das Menü; ESC und Tippen außerhalb schließen das Menü; bei ≥541px Viewport (Webapp) zeigt der Header unverändert die Glocke, der Burger bleibt `display:none`, Klick auf die Glocke öffnet weiterhin direkt "Neuigkeiten" wie vor der Änderung — keine Konsolenfehler in beiden Ansichten. Hinweis: Der Service Worker cached `index.html` aggressiv (cache-first) — beim lokalen Testen war ein Unregister/Cache-Clear nötig, um die Änderung zu sehen; für echte Besucher übernimmt das der `CACHE_NAME`-Bump automatisch.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v60` erhöht (Pflichtregel).

**Offen / Platzhalter:**
- "Hilfe & Feedback": reine Platzhalter-Ansicht, keine echte FAQ-/Feedback-Anbindung.
- "Neuigkeiten": Datenquelle ist weiterhin die bestehende `getRelevantNotifications()`-Logik (echte Favoriten-Angebote + ein klar markiertes "demnächst"-Demo-Beispiel, siehe Kommentar dort) — unverändert, keine neue Mock-Struktur nötig, da bereits vorhanden.
- "Über CanSpot": Kontakt-Mailadresse (`kontakt@canspot.example`) und Impressum/Datenschutz-Links (`href="#"`) sind Platzhalter, identisch zum bisherigen Stand in "Mein Bereich".
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (19) — Favoriten-Animation bei Mehrfach-Angeboten behoben + Monster/Rockstar-Bundle-Bilder korrigiert

**Umgesetzt (Mobile + Webapp):**

**1. Favoriten-"Pop"-Animation lief bei Produkten mit mehreren Angeboten auf der falschen Karte:**
- **Ursache**: Beim Klick auf ein Herz rief der Handler `render()` auf (baut `dealsContainer.innerHTML` komplett neu), danach wurde die Animation über `dealsContainer.querySelector('[data-fav="productId"] .icon')` ausgelöst — bei Produkten mit mehreren Angeboten (verschiedene Händler/Packungsgrößen, z.B. Red Bull mit 8 Karten) liefert `querySelector` aber IMMER das erste Vorkommen im DOM, unabhängig davon, welche Karte tatsächlich angeklickt wurde. Die Animation lief dadurch fast immer auf der falschen (oft nicht sichtbaren) Karte — genau die vom Nutzer bemerkten "manche Sorten und Marken".
- **Fix**: Vor dem `render()`-Aufruf wird jetzt zusätzlich die eindeutige Angebots-ID (`data-id` der `.card`) der tatsächlich angeklickten Karte gemerkt; nach dem Neu-Rendern wird gezielt genau diese Karte wiedergefunden und nur deren Herz animiert. Betraf ausschließlich die Hauptliste (`dealsContainer`) — die Herz-Buttons in Preisverlauf-Sheet (`#detailFavBtn`, eindeutige ID) und Favoriten-Tab waren nicht betroffen.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): systematischer Test aller 26 Produkte — jeweils das LETZTE Angebot (ungünstigster Fall für den alten Bug) angeklickt und geprüft, ob die Animation auf genau dieser Karte lief (nicht auf einer anderen) — alle 26 bestehen jetzt; vorher lief die Animation bei Produkten mit >1 Angebot nachweislich auf der ersten Karte statt der angeklickten (per Vorher/Nachher-Test bestätigt); keine Konsolenfehler.

**2. Monster/Rockstar-Bundle-Bilder: Einzeldose verdeckte Favoriten-Herz:**
- **Ursache**: Die bisherigen Amazon-Bundle-Bilder für 5 Monster-/Rockstar-Angebote waren Amazon-Composite-Aufnahmen mit einer zusätzlich freigestellten Einzeldose oben rechts im Bild — exakt dort, wo das Herz-Symbol (`.thumb-frame .btn.fav`, `top:-3px;right:-3px`) positioniert ist, wodurch es auf der Dose zu liegen kam.
- **Fix**: 5 der 6 betroffenen Bundle-Bilder durch andere, real recherchierte Produktfotos ersetzt, die keine freigestellte Einzeldose in der rechten Bildhälfte zeigen (reine Karton-/Tray-Aufnahmen ohne Corner-Dose, jeweils einzeln visuell verifiziert): Monster Energy Ultra 4×500ml (REWE-Kartonfoto), Monster Energy Classic 12×500ml, Monster Mango Loco 12×500ml, Monster Pipeline Punch 12×500ml, Rockstar Energy Original 12×500ml (je ein alternatives Amazon.de-Produktbild von derselben Angebotsseite). Eine echte "Dose links statt rechts"-Variante existierte bei keinem der geprüften Bilder (Amazons Composite-Generator platziert die Akzent-Dose grundsätzlich immer rechts) — die tray-only-Aufnahmen ohne jede freigestellte Dose lösen das eigentliche Problem (Herz-Überlappung) genauso zuverlässig, ohne eine Bildseite künstlich zu spiegeln (was Text/Logos auf der Verpackung seitenverkehrt und unlesbar gemacht hätte).
- **1 Ausnahme unverändert gelassen**: Monster Energy Ultra 24×500ml (`d59`) — für dieses Angebot war trotz Suche bei REWE (keine 24er-Packung im Sortiment) und mehreren Amazon-/idealo-Bildquellen keine Alternative ohne rechtsseitige Einzeldose auffindbar; bewusst nicht verändert, um keine Bildseite zu spiegeln oder ein nicht-reales Bild zu verwenden.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): alle 5 neuen Bilder laden fehlerfrei (`img.complete`, `naturalWidth`>0, kein `thumb-fallback`); Screenshots aller 5 betroffenen Karten (4er/12er-Filter) bestätigen, dass keine Dose mehr mit dem Herz überlappt; 24er-Pack (unverändert) zur Kontrolle ebenfalls geprüft; `git diff` bestätigt, dass ausschließlich die 5 `bundleImg`-Werte geändert wurden, sonst nichts; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v59` erhöht (Pflichtregel).

**Offen:**
- Monster Energy Ultra 24×500ml (`d59`) zeigt weiterhin die Einzeldose rechts im Bild — falls der Nutzer eine Lösung wünscht, käme z.B. eine andere Packungsgröße/ein anderer Händler für dieses Angebot infrage, oder eine bewusste Ausnahme in der Herz-Positionierung nur für dieses eine Bild.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (18) — Filter "Ohne Pfand"/"Verpackung" und "Favoriten" in Mein Bereich entfernt

**Umgesetzt (Mobile + Webapp):**
- **Filter "Ohne Pfand" komplett entfernt**: Switch-Row im Filter-Sheet, State-Variable `noPfandOnly`, Filterlogik in `render()` und in der Leer-Zustand-Radius-Erweiterungs-Zählung, `updateFilterCount()`, `resetExtraFilters()` und der zugehörige `change`-Event-Listener — alles restlos entfernt, keine toten Referenzen übrig (per Grep bestätigt).
- **Filter "Verpackung" (Dose/Flasche) komplett entfernt**: Chip-Reihe im Filter-Sheet, State-Variable `packagingFilter`, dieselben Anknüpfungspunkte wie oben (Filterlogik, Leer-Zustand-Zählung, `updateFilterCount()`, `resetExtraFilters()`, Klick-Listener für `.chip[data-pack]`) — restlos entfernt. Das zugrunde liegende `packaging`-Datenfeld in `deals.json`/`buildDeals()` bleibt unangetastet (ist weiterhin Teil des Datenmodells, nur nicht mehr als Filteroption nutzbar — wie gewünscht nur die Filteroption entfernt, keine Daten gelöscht).
- **"Favoriten" aus "Mein Bereich" entfernt**: Das eigenständige Favoriten-Listing unter "Meine Aktivität" (`#profileFavCount`/`#profileFavList`, mit eigener `updateProfileFavList()`-Funktion) entfernt — "Preisalarme" bleibt als einziger Punkt unter "Meine Aktivität" erhalten. Der reguläre "Favoriten"-Tab in der unteren Navigation (eigene Ansicht, `renderFavList()`) ist davon **nicht** betroffen und funktioniert unverändert — war explizit nicht Teil der Anfrage.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Filter-Sheet zeigt "Verpackung" und "Ohne Pfand" nicht mehr (Screenshot bestätigt, sauberer Übergang von "Packungsgröße" zu "Angebotszeitraum" bzw. "Nur verfügbare Angebote" bleibt als letzter Punkt); "Mein Bereich" zeigt "Preisalarme (0)" direkt unter "Meine Aktivität", kein "Favoriten"-Eintrag mehr; bestehender Favoriten-Tab (untere Navigation) weiterhin funktionsfähig; Größen-/Packungsgrößen-Filter funktionieren weiterhin korrekt (Test mit Volumen-Filter "500 ml" → 27 Treffer, Filterzähler zeigt korrekt "1"); keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v58` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (17) — Echter 6er-Pack gefunden (Red Bull bei Lidl) und ergänzt

**Kontext**: Nutzer bemerkte, dass "6er Pack" im Packungsgrößen-Filter komplett ausgegraut war (0 Angebote mit `units:6` in den Daten, da die drei einzigen 6er-Angebote in einer früheren Sitzung als nicht real verifizierbar entfernt worden waren). Nutzer bat um erneute, auf die 6 in der App abgebildeten Händler (Kaufland, Lidl, EDEKA, Penny, REWE, Netto) beschränkte Prüfung, mit der Auflage, den Filter-Reiter zu entfernen, falls sich nirgendwo etwas findet.

**Ergebnis der Prüfung**: Bei **Lidl gibt es tatsächlich einen echten 6er-Pack** — Red Bull Energy Drink 6×0,25l ist dort laut supermarktcheck.de (aktuell getrackter Preis) das EINZIGE gelistete Red-Bull-Produkt überhaupt, für 7,74 € (5,16 €/L); zusätzlich bereits früher über prospektangebote.de als historisches Lidl-Flyer-Angebot bestätigt. Bei den anderen 5 Händlern und für die übrigen 6 Marken (Monster, Rockstar u.a.) wurde gezielt bei REWE/Kaufland/EDEKA/Lidl/Netto nachgeprüft — kein weiterer 6er-Pack gefunden.

**Umgesetzt**: Da ein echter 6er-Pack gefunden wurde, bleibt der Filter-Reiter erhalten (die Entfernungs-Bedingung galt nur für den Fall "nirgendwo etwas gefunden"). Stattdessen 1 neues, real recherchiertes Bundle-Angebot ergänzt (`d67`): Red Bull Energy Drink 6×250ml, Lidl, 7,74 €, Bundle-Bild von der REWE-Produktseite für dieselbe Packungsgröße (`img.rewe-static.de/0116447/...`, zeigt "6 PACK 250ml", visuell verifiziert). `git diff` bestätigt rein additive Änderung an `deals.json` (nur 1 neuer Offer, nichts Bestehendes verändert).
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Packungsgrößen-Filter zeigt "6er Pack" wieder als aktiv/anklickbar; Filtern danach liefert genau 1 Treffer (Red Bull, Lidl, 7,74 €) mit korrektem Bundle-Bild und Preis pro Liter; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v57` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (16) — Scroll-Chaining behoben (Hintergrund scrollte in offenen Sheets mit)

**Umgesetzt (Mobile + Webapp — reine CSS-Ergänzung, keine Design-/Layoutänderung):**
- **Ursache**: Die scrollenden Sheet-Container (`.sheet`, `#locOverlay`/`#filterOverlay .sheet-scroll`) hatten kein `overscroll-behavior` gesetzt. Erreichte man beim Scrollen innerhalb eines Sheets (z.B. Produktdetail, Filter, Preisverlauf) den oberen oder unteren Rand und scrollte weiter, "kettete" der Browser die restliche Scroll-Bewegung an das nächste scrollende Element dahinter durch — das war die Hintergrundseite (`body`, kein eigenes `overflow:hidden`), die dadurch sichtbar mitscrollte.
- **Fix**: `overscroll-behavior: contain` auf `.sheet`, `#locOverlay .sheet-scroll`/`#filterOverlay .sheet-scroll` sowie `.search-suggestions-list` (Such-Autocomplete-Dropdown, dieselbe Bug-Klasse) ergänzt — je 1 CSS-Zeile pro Selektor, sonst nichts geändert. Das ist die moderne Standardlösung genau für dieses Problem (von allen aktuellen Browsern inkl. Safari 16+ unterstützt) und stoppt das Durchreichen der Scroll-Bewegung exakt am Rand des jeweiligen Containers, ohne Body-Scroll-Lock oder sonstige JS-Änderungen zu benötigen.
- Bestehende Wischgeste zum Schließen (`initSwipeToClose()`) unangetastet — die arbeitet bereits mit einer eigenen "ist am oberen Rand?"-Prüfung und ist von dieser Änderung unabhängig.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Hintergrundseite auf `scrollY:300` positioniert, Produktdetail-Sheet geöffnet, im Sheet über den oberen UND unteren Rand hinaus gescrollt (simulierte Wheel-Events) — Hintergrund blieb in beiden Fällen exakt bei `scrollY:300`, kein Mitscrollen mehr; `overscroll-behavior:contain` computed-style-geprüft auf allen 3 betroffenen Selektoren; Regressionstest (Preisverlauf-Sheet öffnen/schließen, Filter-Sheet mit Größenfilter) läuft weiterhin fehlerfrei; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v56` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (15) — Nährwerte-Kacheln umgebaut (Kohlenhydrate statt Taurin) + aufklappbare Vollansicht

**Umgesetzt (Mobile + Webapp):**
- **4 Übersichts-Kacheln geändert**: „Taurin" durch „Kohlenhydrate" ersetzt — zeigt jetzt Kalorien, Zucker, Koffein, Kohlenhydrate. Taurin wurde NICHT aus den Daten entfernt (weiterhin in `NUTRITION_MOCK_BY_BRAND`/`NUTRITION_MOCK_BY_PRODUCT`), nur aus den vier großen Kacheln in die neue Vollansicht verschoben.
- **Neuer aufklappbarer Bereich** unterhalb der Kacheln: Button „⌄ Alle Nährwerte anzeigen" (Chevron-Icon aus dem bestehenden Icon-Sprite statt eines rohen Unicode-Zeichens, damit es zum bestehenden Icon-System passt) öffnet eine kompakte Tabelle mit Energie (kJ/kcal), Fett, davon gesättigte Fettsäuren, Kohlenhydrate, davon Zucker, Eiweiß, Salz, Koffein, Taurin — schließt sich bei erneutem Klick wieder (Chevron dreht sich, Label wechselt zu „Nährwerte ausblenden"/zurück). Zustand wird bei jedem Öffnen eines (auch desselben) Produkts zurückgesetzt, damit nie ein fälschlich offener Zustand vom vorherigen Produkt hängen bleibt.
- **Datenmodell erweitert**: `NUTRITION_FALLBACK_PER_100ML`/`NUTRITION_MOCK_BY_BRAND`/`NUTRITION_MOCK_BY_PRODUCT` um `carbsG`, `fatG`, `satFatG`, `proteinG`, `saltG` ergänzt (bisher nur `kcal`/`sugarG`/`caffeineMg`/`taurineMg`) — alle Werte real recherchiert (das-ist-drin.de, fddb.info, openfoodfacts.org, teils Amazon.de-Produktseiten für Zero-Varianten wie Monster Ultra), `saltG` wo nötig per EU-Standardformel Salz = Natrium×2,5 aus recherchierten Natrium-Werten umgerechnet (keine erfundenen Werte). Energie in kJ wird aus kcal berechnet (Standardumrechnung 1 kcal = 4,184 kJ), nicht separat recherchiert.
- Zero-/Sugarfree-Varianten ohne eigene Quelle übernehmen Fett/Eiweiß/Salz der jeweiligen regulären Sorte derselben Marke (nachvollziehbar, keine Erfindung) — nur Monster Ultra (White) und Carabao Sugar Free hatten eigene abweichende Quellenwerte für Kohlenhydrate/Eiweiß, entsprechend übernommen.
- `getNutritionPer100ml()`/Fallback-Kette unverändert — nur die Datensätze selbst wurden erweitert, keine strukturelle Änderung an der Auflösungslogik.
- Design der 4 Kacheln (Größe, Grid, Farben) unverändert; neue Vollansicht nutzt bestehende Farbvariablen (`--bg-surface-sunken`, `--border-subtle`, `--text-muted`/`--text-primary`) für konsistente Optik in Hell-/Dunkelmodus.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Kacheln zeigen korrekt Kalorien/Zucker/Koffein/Kohlenhydrate (Taurin nicht mehr sichtbar dort); Aufklappen zeigt vollständige, korrekt berechnete Tabelle inkl. Taurin (Screenshot bestätigt); erneuter Klick schließt wieder korrekt; Zustand setzt sich beim Wechsel zu einem anderen Produkt zuverlässig zurück; Test mit Zero-Variante (Monster Ultra, u.a. abweichende Kohlenhydrate 0,9g statt 0g) und mit Nullwert (28 Black Taurin 0mg wird korrekt als "0 mg" angezeigt, nicht weggelassen, da es ein echter recherchierter Wert ist) erfolgreich; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v55` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (14) — Taurin-Werte nochmals gegen unabhängige Laboranalyse geprüft und präzisiert

**Kontext**: Nutzer bestand darauf, dass die Taurin-Angabe (1000mg bei 250ml, 2000mg bei 500ml) weiterhin falsch sei, trotz der Korrektur in der Vorsitzung. Vollständige Neuprüfung aller 26 Produkte durchgeführt.

**Ergebnis der Neuprüfung**: Das gleichförmige "400mg/100ml → 1000mg/250ml bzw. 2000mg/500ml"-Muster ist für Red Bull, Monster, Rockstar, effect und Carabao tatsächlich **real und korrekt** — zusätzlich zu den bereits in der Vorsitzung herangezogenen Quellen (das-ist-drin.de, fddb.info) jetzt auch anhand der unabhängigen Laboranalyse **"Energy Drinks 07/2013" der Stiftung Warentest** (test.de, chemisch gemessene Werte, nicht nur deklarierte Herstellerangaben) einzeln für Red Bull (3900mg/l), Monster (4300mg/l), Rockstar (4100mg/l), effect (4000mg/l) und 28 Black ("n.n." = nicht nachweisbar, bestätigt 0mg) gegengeprüft — alle lagen nah an den bereits hinterlegten Werten. Das auffällig runde Muster war also kein Bug, sondern spiegelt real wider, dass sich die meisten Marken am deutschen Markt auf ca. 0,4% Taurin eingependelt haben.

**Trotzdem umgesetzt — Präzisierung anhand der jetzt vorliegenden Labordaten** (`NUTRITION_MOCK_BY_BRAND`/`NUTRITION_MOCK_BY_PRODUCT` in `index.html`), um pro Marke echte, differenzierte Werte statt eines einheitlichen Platzhalters zu zeigen:
- Red Bull: Taurin 400→390 mg/100ml (250ml-Dose zeigt jetzt 975 statt 1000mg), Koffein 32→31mg
- Monster: Taurin 400→430 mg/100ml (500ml-Dose zeigt jetzt 2150 statt 2000mg), Koffein 32→30mg
- Rockstar: Taurin 400→410 mg/100ml (500ml-Dose zeigt jetzt 2050mg), Kalorien 59→57 kcal/100ml (Warentest-Messwert)
- 28 Black: Kalorien 59→57 kcal/100ml (Warentest-Messwert), Taurin bleibt 0 (jetzt zusätzlich Warentest-bestätigt)
- effect: Koffein 32→31mg (Warentest-Messwert), Taurin bleibt 400 (exakte Übereinstimmung mit Warentest)
- GÖNRGY/Carabao: unverändert (keine Warentest-Daten verfügbar für diese Marken, bleiben bei den bereits in der Vorsitzung recherchierten Werten aus fddb.info/openfoodfacts.org)
- Alle produktgenauen Ausnahmen (`NUTRITION_MOCK_BY_PRODUCT`, Zero-Varianten, Monster-Ultra-Linie, Rockstar Punched Guava) entsprechend an die neuen Markenwerte angeglichen.
- Kommentarblock über den Konstanten erklärt jetzt explizit, warum das 1000/2000mg-Muster bei den betroffenen Marken real ist (industrieweite Konvergenz auf ~0,4% Taurin), damit das bei künftiger Bearbeitung nicht wieder als Bug missverstanden wird.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): alle 26 Produkte einzeln durchgerechnet, Red Bull zeigt jetzt 975mg (statt 1000mg) in der Detailansicht (Screenshot bestätigt), keine Konsolenfehler, kein Produkt fällt auf `NUTRITION_FALLBACK_PER_100ML` zurück (alle 7 Marken korrekt in `NUTRITION_MOCK_BY_BRAND` hinterlegt, geprüft).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v54` erhöht (Pflichtregel).

**Offen:**
- GÖNRGY und Carabao sind zu klein/neu, um in der 2013er-Stiftung-Warentest-Studie vertreten zu sein — deren Werte stützen sich weiterhin auf Sekundärquellen (fddb.info, openfoodfacts.org), nicht auf eine unabhängige Laboranalyse.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (13) — Nicht-verfügbare Produkte/Bundles entfernt, Nährwerte real recherchiert und korrigiert

**Umgesetzt (Mobile + Webapp — reine Datenkorrektur, keine Design-/Funktionsänderung):**

**1. Entfernte Produkte/Bundles (real recherchiert, nicht mehr im deutschen Handel):**
- **Produkt `p9` "Red Bull Red Edition (Wassermelone)" komplett entfernt** (inkl. aller 3 Angebote d13/d14/d53): Auf der offiziellen `redbull.com/de-de`-Editions-Seite taucht "Wassermelone"/"Red Edition" nirgends mehr in der aktuellen Editions-Liste auf (Summer/Glacier/Peach/Sea Blue/Apricot/Green/White/Blue Edition), REWEs komplette Red-Bull-Sortimentsseite (`rewe.de/shop/productList?brand=Red%20Bull`, 41+ Artikel) enthält den Suchbegriff "Wassermelone" ebenfalls nicht — nur noch Reseller-Restbestände auf eBay/Amazon Marketplace zu deutlich erhöhten Preisen, kein regulärer Handel mehr.
- **3 Bundle-Angebote entfernt** (Produkt bleibt jeweils erhalten, da als Einzeldose weiterhin real erhältlich, nur das konkrete Mehrpack-Angebot existiert nicht): `d54` Monster Ultra Zero 6×500ml (am deutschen Markt nur 4er/12er/24er auffindbar), `d55` effect Classic Energy 6×250ml (REWE führt effect nur einzeln, als 4×0,33l oder 6×1l-PET, keine 6×0,25l-Dose), `d57` Carabao Green Apple Sugar Free 12×330ml (keine verlässliche deutsche 12er-Quelle gefunden, Carabao bei REWE gar nicht gelistet).
- Ergebnis: 66→60 Angebote, 27→26 Produkte. `git diff` zeigt für `deals.json` ausschließlich Löschungen der betroffenen Einträge, keine Änderung an übrigen Daten. Alle verbleibenden 12 Bundle-Angebote haben jetzt ein reales, verifiziertes `bundleImg` — keine Bundle-Lücke mehr offen.

**2. Nährwerte real recherchiert und korrigiert** (`NUTRITION_MOCK_BY_BRAND`/`NUTRITION_MOCK_BY_PRODUCT` in `index.html`), Quellen: offizielle Marken-Websites, das-ist-drin.de, fddb.info, openfoodfacts.org (Nährwerttabellen pro 100 ml):
- **GÖNRGY war komplett falsch**: bisher wie eine reguläre Vollzucker-Marke behandelt (48 kcal/12 g Zucker/100ml) — real ist die komplette GÖNRGY-Linie zuckerfrei (~3 kcal/0 g Zucker/100ml) UND hat mit 30 mg/100ml deutlich weniger Taurin als angenommen (statt 400 mg) — z.B. GÖNRGY Raspberry Cheesecake (500ml) zeigte vorher 240 kcal/60 g Zucker/2000 mg Taurin, jetzt korrekt 15 kcal/0 g Zucker/150 mg Taurin.
- **28 Black führt laut eigener Zutatenliste GAR KEIN Taurin** (0 mg) — vorher fälschlich wie alle anderen Marken mit 400 mg/100ml geführt (z.B. 28 Black Açaí 250ml zeigte vorher 1000 mg Taurin, jetzt korrekt 0 mg — das ist der vom Nutzer als Beispiel genannte Fehler).
- **Monster "Ultra"-Linie (p3 Monster Energy Ultra, p14 Monster Ultra Watermelon) bekam erstmals eine eigene Zuckerfrei-Ausnahme** in `NUTRITION_MOCK_BY_PRODUCT` — bisher liefen beide fälschlich über den regulären Monster-Markendefault (47 kcal/11g Zucker), obwohl die komplette Ultra-Linie real zuckerfrei/kalorienfrei ist (2 kcal/0g Zucker/100ml, offiziell "Zero Zucker und Zero Kalorien" beworben) — nur `p11` Monster Ultra Zero hatte vorher schon eine (leicht ungenaue) Ausnahme.
- **Carabao Green Apple Sugar Free (`p17`)** bekam ebenfalls erstmals eine Zuckerfrei-Ausnahme (trotz "Sugar Free" im Namen lief es vorher über den vollzuckrigen Markendefault).
- **Rockstar/Carabao/28 Black/effect-Markendefaults kcal & Zucker korrigiert**: Rockstar 45→59 kcal, 11→14 g Zucker; Carabao 45→56 kcal, 11→14 g; 28 Black 45→59 kcal, 11→14 g; effect 48→46 kcal (alle anhand realer Nährwerttabellen). Red Bull und Monster (Classic) waren bereits korrekt und blieben unverändert.
- **Rockstar Energy Punched Guava (`p16`)** bekam eine eigene Ausnahme (67 kcal/16g Zucker statt Rockstar-Original-Werten 59/14 — reale Nährwerttabelle weicht spürbar vom Original ab).
- Caffeine-Werte weitgehend bestätigt (~32mg/100ml herstellerübergreifend üblich), Monster-Markendefault-Koffein von 36 auf die real bestätigten 32mg/100ml korrigiert.
- Weiterhin als Demo-Werte gekennzeichnet (`#pdNutritionCaption`: "Demo-Werte, keine geprüften Herstellerangaben") — nicht jede einzelne Geschmacksrichtung wurde individuell nachgeprüft, aber alle Markendefaults und die auffälligsten Abweichungen (GÖNRGY, 28 Black, Monster Ultra) sind jetzt real recherchiert statt geraten.
- Keine Änderung an der bestehenden Fallback-Kette/Anzeige-Logik (`getNutritionPer100ml()`, `renderNutrition()`) — nur die Zahlenwerte wurden korrigiert.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): `deals.length`/`products.length` korrekt auf 60/26; Nährwerte für 13 Stichproben-Produkte per Berechnung geprüft (Red Bull, Monster Ultra/Classic, Rockstar Original/Punched Guava, GÖNRGY, Carabao/Sugar Free, effect, 28 Black/Zero) — alle Werte stimmen mit den recherchierten Markendefaults/Ausnahmen überein; Detailansicht für 28 Black Açaí (0mg Taurin) und GÖNRGY Raspberry Cheesecake (15kcal/0g Zucker) per Screenshot bestätigt; Regressionstest über Markenfilter/Karte/Favoriten/Alarme/Packungsgrößen-Ausgrauen/Bundle-Sortierung — alles funktioniert weiterhin fehlerfrei; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v53` erhöht (Pflichtregel).

**Offen:**
- Nicht jede einzelne Geschmacksrichtung (insbesondere kleinere Monster-"Juiced"-Sorten wie Mango Loco/Pipeline Punch sowie effect-/28-Black-Geschmacksvarianten) wurde einzeln nachrecherchiert — sie laufen weiterhin über den (jetzt korrigierten) Markendefault ihrer Marke, was für ein Demo-Prototyp als ausreichend genau eingeschätzt wurde.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (12) — Fehlende Bundle-Bilder bei den ursprünglichen 10 Multipack-Angeboten nachgetragen

**Umgesetzt (Mobile + Webapp — reine Datenergänzung, keine Code-Änderung nötig):**
- **Ursache**: `bundleImg` wurde in der vorletzten Sitzung nur für die 6 neu hinzugefügten Bundle-Angebote (d61–d66) recherchiert und gesetzt — die 10 bereits vorher vorhandenen Multipack-Angebote (d51–d60, u.a. das vom Nutzer genannte Beispiel "Red Bull 24×250ml bei EDEKA" = d58) hatten das Feld nie bekommen und zeigten deshalb weiterhin die Einzeldose.
- **6 der 10 fehlenden Bundle-Bilder real recherchiert und ergänzt** (jeweils einzeln visuell verifiziert, von REWE- bzw. Amazon.de-Produktseiten):
  - d51 Red Bull 4×250ml (Kaufland) — REWE (identisches Bild wie d61, da exakt dieselbe Packungsgröße)
  - d52 Monster Energy Ultra 4×500ml (Lidl) — Amazon.de
  - d56 Red Bull 12×250ml (Netto) — REWE (`rewe.de/shop/p/red-bull-energy-drink-12x0-25l/6392199`)
  - d58 Red Bull 24×250ml (EDEKA) — Amazon.de (identisches Bild wie d62)
  - d59 Monster Energy Ultra 24×500ml (Kaufland) — Amazon.de
  - d60 28 Black Açaí 24×250ml (Penny) — Amazon.de
- **4 Angebote bewusst OHNE Bundle-Bild gelassen** (Fallback bleibt die Einzeldose, wie von Anfang an vorgesehen für den Fall "kein eindeutig passendes Bild gefunden") — für keines dieser vier existiert nachweislich ein reales Mehrpack-Produkt in dieser exakten Kombination aus Geschmack/Größe/Packungsgröße im deutschen Handel (jeweils per REWE-Sortimentsseite bzw. gezielter Suche gegengeprüft, keine Vermutung):
  - d53 Red Bull Red Edition (Wassermelone) 6×250ml — REWE führt diese Geschmacksrichtung nachweislich nur als Einzeldose, kein 6er-Pack im Sortiment
  - d54 Monster Ultra Zero 6×500ml — für Monster (Ultra) 500ml-Dosen sind am deutschen Markt nur 4er/12er/24er-Packungsgrößen auffindbar, kein 6er
  - d55 effect Classic Energy 6×250ml — REWE führt effect nur als Einzeldose (0,25l), 4×0,33l oder 6×1l-PET-Flaschen, keine 6×0,25l-Dosenpackung
  - d57 Carabao Green Apple Sugar Free 12×330ml — Carabao ist bei den geprüften deutschen Händlern (REWE) gar nicht gelistet; keine verlässliche deutsche 12er-Pack-Quelle für exakt diese Geschmacksrichtung gefunden
  - Diese vier Angebote sind rein fiktive Demo-Kombinationen ohne reales Vorbild in dieser Packungsgröße — bewusst nicht mit einem unpassenden Ersatzbild (z.B. falsche Geschmacksrichtung oder falsche Packungsgröße) verfälscht.
- Keine Code-Änderung nötig — der bestehende `bundleImg`-Mechanismus aus der vorletzten Sitzung greift automatisch, sobald das Feld in `deals.json` gesetzt ist.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): alle 6 neuen Bilder laden fehlerfrei (`img.complete`, `naturalWidth`>0, kein `thumb-fallback`); d58 (EDEKA-Beispiel des Nutzers) zeigt jetzt sowohl in der Kartenliste als auch in der Detailansicht (`pdImg.src`) korrekt die 24er-Palette; d53/d54/d55/d57 zeigen weiterhin korrekt (und bewusst) die Einzeldose; keine Konsolenfehler; `git diff --stat` bestätigt rein additive Änderung (6 Insertions, 0 Deletions).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v52` erhöht (Pflichtregel).

**Offen:**
- 4 Bundle-Angebote (d53/d54/d55/d57) haben weiterhin kein eigenes Bundle-Bild (bewusst, siehe oben) — falls der Nutzer möchte, könnten diese vier stattdessen durch real existierende Packungsgrößen/Geschmacksrichtungen ersetzt werden, für die es tatsächlich Handelsware gibt.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (11) — Bundles ans Listenende, Packungsgröße pro Marke ausgrauen, Bundle-Bild in Detailansicht

**Umgesetzt (Mobile + Webapp — reine Verhaltens-Erweiterung, kein Design geändert):**
- **Bundles immer am Ende der Liste**: Nach dem bestehenden Sortier-Schritt in `render()` sortiert ein zweiter, stabiler `sort()`-Aufruf (`(a,b)=>(a.units>1?1:0)-(b.units>1?1:0)`) alle Angebote mit `units>1` ans Ende — die Reihenfolge innerhalb "Einzelartikel" bzw. innerhalb "Bundles" bleibt dabei exakt wie von der gewählten Sortierung (Preis/Entfernung/Ersparnis/Endet bald) vorgegeben, da `Array.prototype.sort` seit ES2019 spezifiziert stabil ist. Gilt für alle Sortieroptionen gleichermaßen.
- **Packungsgrößen-Filter markenabhängig ausgegraut**: Neue Funktion `syncUnitsChipsAvailability()` prüft für jede Packungsgrößen-Chip (4er/6er/12er/24er) im Filter-Sheet, ob es für die aktuell gewählte Marke (`selectedBrand`) überhaupt ein Angebot mit dieser `units`-Zahl gibt (geprüft gegen das volle `deals`-Array, nicht das bereits gefilterte) — falls nicht, wird die Chip-Schaltfläche per nativem `disabled`-Attribut ausgegraut und nicht mehr anklickbar (neue CSS-Regel `.chip:disabled{opacity:.4;cursor:default}`). Beispiel: Marke "Rockstar" (aktuell nur als 12er-Pack im Sortiment) graut "4er/6er/24er Pack" aus, "12er Pack" bleibt aktiv. War der zuvor aktive `unitsFilter` für die neu gewählte Marke nicht mehr verfügbar, wird er automatisch auf "Alle" zurückgesetzt (sonst bliebe eine ausgegraute Chip fälschlich als "aktiv" markiert, während die Liste bereits leer wäre) — per Test mit Markenwechsel Rockstar→GÖNRGY (keine Mehrpacks) bestätigt: `unitsFilter` sprang automatisch von "12" auf "Alle", Liste zeigte korrekt alle 7 GÖNRGY-Angebote statt leer zu bleiben. Wird aufgerufen beim Marken-Wechsel, beim Öffnen des Filter-Sheets, beim "Zurücksetzen" der Filter und einmalig nach dem initialen Laden der Angebote.
- **Bundle-Bild jetzt auch in der Produktdetailansicht**: Bisher öffnete ein Klick auf das Bild/den Namen einer Bundle-Karte die Produktdetailansicht immer mit der Einzeldose (`product.img`), weil dort nur die `productId` bekannt war. Die Angebotskarten (Hauptliste UND die Bundle-fähigen "Alarme"-Karten in `newsCardHtml()`) tragen jetzt zusätzlich `data-deal-id="${deal.id}"` auf Bild und Name; `openProductDetail(productId, dealId)` nimmt das optionale zweite Argument entgegen und zeigt als Hero-Bild `deal.img` (das bereits bestehende `bundleImg`-Fallback aus `buildDeals()`) statt `product.img`, wenn ein `dealId` übergeben wurde. Ohne `dealId` (Ähnliche Produkte, Favoriten — unverändert) bleibt exakt das bisherige Verhalten (`product.img`). Nährwerte und "Ähnliche Produkte" bleiben unverändert produktbezogen (nicht angebotsbezogen), wie vom Nutzer nicht angefordert.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Bundle-Reihenfolge per `deals.find`+Index-Check bestätigt (erstes Bundle bei Index 50 von 66, danach kein Einzelartikel mehr); Filter-Sheet bei Marke "Rockstar" zeigt 4er/6er/24er ausgegraut, 12er aktiv (Screenshot + `.disabled`-Property-Check); Klick auf Rockstar-12er-Pack-Bild (d66) öffnet Detailansicht mit dem Amazon-Bundle-Bild statt der Einzeldose (Screenshot + `pdImg.src`-Check); Klick auf ein normales Einzelprodukt (d1) zeigt weiterhin korrekt die Einzeldose (Regressionstest); Alarme-Ansicht mit neuem `data-deal-id` funktioniert ebenfalls fehlerfrei; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v51` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (10) — Echte Mehrpack-Produktbilder für die 6 neuen Bundles

**Umgesetzt (Mobile + Webapp — reine Datenergänzung + eine minimale, generische Code-Änderung):**
- **Neues optionales Offer-Feld `bundleImg`** in `deals.json`: überschreibt für GENAU dieses eine Angebot das Standard-Produktbild (`products[].img`, Einzeldose). In `buildDeals()` (`index.html`): `img: (typeof d.bundleImg === "string" && d.bundleImg) ? d.bundleImg : product.img` — exakt dieselbe Fallback-Kette wie bei `pfand`/`packaging`/`units`/`lastCheckedAt`. Fehlt das Feld (der Fall bei **allen** bisherigen Angeboten, inkl. der 10 alten Multipack-Angebote d51–d60), bleibt das Verhalten 1:1 wie zuvor — nur die 6 neuen Bundle-Angebote (d61–d66) haben das Feld gesetzt.
- **Produktkatalog bleibt unangetastet**: Das Produktdetail-Sheet und die "Ähnliche Produkte"-Leiste lesen weiterhin ausschließlich `product.img` (Einzeldose) direkt aus `products[]`, komplett unabhängig von `bundleImg` — per Test bestätigt (`pdImg.src` bei p1 weiterhin die Einzeldose, trotz zweier Red-Bull-Bundle-Angebote mit eigenem Bild).
- **6 real recherchierte Mehrpack-Packshots** (jeweils von der offiziellen Händler-/Retailer-Seite, wie bei den bisherigen Produktbildern):
  - Red Bull 4×250ml (d61) — Bild von REWE (`img.rewe-static.de`, von der REWE-Produktseite `rewe.de/shop/p/red-bull-energy-drink-4x0-25l/2843250`)
  - Red Bull 24×250ml (d62) — Bild von Amazon.de (`m.media-amazon.com`, von der Amazon.de-Produktseite für den 24er-Karton)
  - Monster Energy Classic 12×500ml (d63) — Amazon.de
  - Monster Mango Loco 12×500ml (d64) — Amazon.de
  - Monster Pipeline Punch 12×500ml (d65) — Amazon.de
  - Rockstar Energy Original 12×500ml (d66) — Amazon.de
  - Jedes Bild wurde vor der Übernahme einzeln visuell im Browser verifiziert (zeigt tatsächlich Tray/Verpackungseinheit mit korrekter Stückzahl/Größe, nicht nur eine Einzeldose) — keines wurde ungeprüft übernommen.
- `_meta.fields.offers[]` in `deals.json` um die Beschreibung von `bundleImg` ergänzt; `CLAUDE.md` (Abschnitt **Images**) entsprechend aktualisiert.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Für alle 6 neuen Angebote zeigt die Karte jetzt das jeweilige Mehrpack-Bild (per Screenshot + `deal.img`-Check bestätigt); alle bisherigen Multipack-Angebote (u.a. d51/d56/d58 Red Bull, d52/d59 Monster Ultra, d57 Carabao, d60 28 Black) zeigen unverändert weiterhin die Einzeldose; alle 6 neuen Bilder laden fehlerfrei (`img.complete`, `naturalWidth` > 0, kein `thumb-fallback`); keine Konsolenfehler; `git diff` bestätigt rein additive Änderung an `deals.json`.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v50` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (9) — Echte deutsche Energy-Drink-Bundles/Mehrpacks ergänzt

**Umgesetzt (Mobile + Webapp — reine Datenergänzung, betrifft beide gleich):**
- **Analyse zuerst**: Das bestehende Datenmodell unterstützte Mehrpacks bereits vollständig über das optionale `units`-Feld in `offers[]` (10 Bestandsangebote d51–d60 nutzten es schon, z.B. Red Bull 4/12/24er). `formatSize()`, `totalMl`, Preis-pro-Liter, der Packungsgrößen-Filter ("1/4/6/12/24er Pack") und alle Preisverlaufs-/Badge-Funktionen greifen bereits generisch auf `units` zu — **es waren daher keine Code-Änderungen an `index.html` nötig**, nur neue Datensätze in `deals.json`.
- **6 neue, real recherchierte Bundle-Angebote** (`d61`–`d66`) für bestehende Produkte ohne bisherige Mehrpack-Angebote bzw. mit Lücken in der Packungsgrößen-Abdeckung, jeweils einem der 6 bestehenden Händler zugeordnet:
  - Red Bull Energy Drink 4×250ml — REWE — 5,49 € (5,49 €/L)
  - Red Bull Energy Drink 24×250ml — Lidl — 25,55 € (4,26 €/L)
  - Monster Energy Classic 12×500ml — EDEKA — 14,39 € (2,40 €/L)
  - Monster Mango Loco 12×500ml — Netto — 12,99 € (2,16 €/L)
  - Monster Pipeline Punch 12×500ml — Penny — 17,88 € (2,98 €/L)
  - Rockstar Energy Original 12×500ml — Kaufland — 17,88 € (2,98 €/L) — Rockstar hatte zuvor **kein einziges** Mehrpack-Angebot.
- **Quelle für alle 6 Preise**: idealo.de-Preisvergleich (aggregiert reale, aktuelle Angebote deutscher Händler/Verkäufer je exakter Packungsgröße), per Live-Browser-Recherche abgerufen. Die Packungsgröße 6×0,25l für Red Bull Original wurde zusätzlich über die REWE-Produktseite (rewe.de) als real im Sortiment bestätigt, aber mangels belastbarem aktuellem Preisdatenpunkt bewusst NICHT hinzugefügt (keine erfundenen Preise). Ausschließlich Kernvarianten ohne Sondereditionen verwendet, um zuverlässig dauerhaft im deutschen Handel erhältliche Produkte abzubilden.
- **Kein Hochrechnen aus Einzelpreisen**: Für jedes Bundle wurde der tatsächlich bei idealo.de gelistete Bundle-Gesamtpreis übernommen (nicht Einzeldosenpreis × Anzahl).
- **`regularPrice` = `offerPrice`** bei allen 6 neuen Angeboten (bewusst kein erfundener "UVP"/Vorher-Preis, da dafür keine belastbare Quelle vorlag) — dadurch zeigt die Karte für diese Angebote transparent "0% GÜNSTIGER" statt eines erfundenen Rabatts; visuell/technisch läuft das über exakt dieselbe bestehende Badge-Logik wie bei allen anderen Angeboten, keine Sonderbehandlung im Code.
- `pfand` konsistent zur bestehenden Konvention (0,25 €/Dose × units), `packaging: "Dose"`, `distanceKm`/Öffnungszeiten identisch zur jeweils bestehenden Filiale des Händlers übernommen, `link: "#"` (Platzhalter wie bei allen bisherigen Demo-Angeboten), `lastCheckedAt: "2026-09-01"` (konsistent zu `DEMO_TODAY`, wie alle bisherigen Angebote).
- Verifiziert über einen temporären lokalen Server (Mobile 375px): `deals.length`/Angebotszähler steigt korrekt von 60 auf 66; Packungsgrößen-Filter "12er Pack" liefert korrekt 6 Treffer (2 bestehende + 4 neue); alle 6 neuen Karten rendern mit korrektem Bild, Name, Packungsgröße ("12 × 500 ml" etc.), Preis, Preis/Liter, Pfand, Händler-Logo, Distanz und Öffnungsstatus — identisch zum bestehenden Kartendesign; „Zuletzt geprüft"-Sprechblase und Preisverlauf-Sheet funktionieren für ein neues Bundle (d66) korrekt; keine Konsolenfehler; `git diff --stat` bestätigt rein additive Änderung an `deals.json` (0 gelöschte Zeilen, keine bestehenden Einträge verändert).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v49` erhöht (Pflichtregel, da `deals.json` in `APP_SHELL`).

**Offen:**
- Weitere, nicht hinzugefügte Bundle-Kandidaten (z.B. Red Bull Sugarfree/Zero-Mehrpacks, Monster Ultra-Varianten, GÖNRGY-Mehrpacks) mangels ausreichend belastbarer aktueller Preisdaten bei der Recherche bewusst ausgelassen — bei Bedarf in einer Folge-Session gezielt nachrecherchieren.
- Nutzer könnte optional gezielt einen realistischen `regularPrice` (Vorher-Preis) für die 6 neuen Angebote ergänzen, falls das "0% GÜNSTIGER" optisch stören sollte — aktuell bewusst nicht erfunden.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (8) — Preis-Sprechblase: Emoji entfernt, immer unterhalb positioniert

**Umgesetzt (Mobile + Webapp, wie bei der ursprünglichen Sprechblase):**
- **Kalender-Emoji entfernt**: Text der Bubble ist jetzt schlicht „Zuletzt geprüft · 01.09.2026" statt „🗓 Zuletzt geprüft · …".
- **Immer unterhalb des Preises**: Die bisherige Logik (oberhalb bevorzugt, nur bei Platzmangel unterhalb) wurde entfernt — `showPricePopover()` positioniert die Bubble jetzt ausnahmslos unterhalb des angeklickten Preises (`top = btnRect.bottom + gap`), damit sie nie Produktbild oder Produktname verdeckt, die in der Karte oberhalb des Preises liegen. Die jetzt überflüssige `data-placement`-Logik (JS und CSS-Selektoren für `[data-placement="top"]`/`"bottom"`) wurde entfernt, `.price-popover::after` (Pfeilspitze) zeigt nur noch die „unten"-Variante (Pfeil oben an der Bubble, zeigt nach oben zum Preis).
- Horizontale Positionierung/Clamping an den Viewport-Rand sowie sämtliches Schließverhalten (erneuter Klick, Klick außerhalb, Scroll/Resize, Wechsel zu anderem Preis) unverändert wie zuvor.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Bubble-Text ohne Emoji bestätigt; Bubble-Top liegt zuverlässig unterhalb von `btn.getBoundingClientRect().bottom` (8px Abstand); Screenshot bestätigt, dass Produktbild und -name frei bleiben; Toggle/Außerhalb-Klick/Wechsel-zu-anderem-Preis funktionieren weiterhin wie zuvor; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v48` erhöht (Pflichtregel).
- `CLAUDE.md` entsprechend aktualisiert (kein Emoji mehr, "immer unterhalb" statt "flips above/below").

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (7) — "Zuletzt geprüft"-Fenster durch kleine Sprechblase am Preis ersetzt

**Umgesetzt (Mobile + Webapp — Nutzer hat für diese Änderung ausdrücklich "auf Mobile sowie Desktop" verlangt):**
- **Großes zentriertes Popup entfernt**: Das in der vorherigen Sitzung eingeführte `#checkedOverlay`/`.mini-modal`-Fenster ("Zuletzt geprüft am ...") wurde komplett entfernt (HTML, CSS, `openLastChecked()`).
- **Neu: kleine, dezente Sprechblase direkt am Preis**: Klick auf den Preis (`button.price`, `data-checked`) öffnet jetzt eine kompakte Popover-Sprechblase (`#pricePopover`) mit dem Text „🗓 Zuletzt geprüft · 01.09.2026" (Datum aus `deal.lastCheckedAt`, wie zuvor). Eine einzelne, wiederverwendete `#pricePopover`-Box wird pro Klick neu positioniert statt für jede Karte eine eigene Instanz vorzuhalten (`showPricePopover()`/`hidePricePopover()`/`togglePricePopover()` in `index.html`).
- **Positionierung**: `position:fixed`, berechnet aus `getBoundingClientRect()` des angeklickten Preis-Buttons — horizontal zentriert über/unter dem Preis, an den Viewport-Rand geklemmt (8px Mindestabstand), Pfeilspitze (`::after`, per CSS-Variable `--price-popover-arrow` positioniert) zeigt weiterhin exakt auf den Preis, auch wenn die Box selbst an den Rand geklemmt wurde. Passt sich automatisch an, ob über oder unter dem Preis genug Platz ist (`data-placement="top"|"bottom"`, Pfeilrichtung dreht sich entsprechend). Funktioniert unverändert bei jeder Fensterbreite (Mobile 375px und Desktop 1280px getestet).
- **Schließen**: erneuter Klick auf denselben Preis (Toggle), Klick außerhalb (globaler `document`-Click-Listener, ignoriert Klicks auf die Bubble selbst oder auf einen `[data-checked]`-Button), sowie zusätzlich bei Scroll/Resize (sonst würde die `position:fixed`-Box beim Scrollen optisch vom Preis abdriften — kein expliziter Nutzerwunsch, aber notwendig für sauberes Verhalten einer fixed-positionierten Box). Klick auf einen ANDEREN Preis, während die Bubble schon offen ist, bewegt sie direkt zum neuen Preis (kein zusätzlicher Klick zum Schließen nötig).
- **Optik**: nutzt bestehende Design-Tokens (`--bg-surface-elevated`, `--border-subtle`, `--shadow-md`, `--radius-sm`), passt sich damit automatisch an Hell-/Dunkelmodus an, ohne eigene Farbwerte zu definieren — bewusst dezent (kleine Schrift, dünner Rand, kein knalliger Toast-Stil).
- Sonst keine Änderungen am bestehenden Design; `lastCheckedAt`-Datenfeld in `deals.json` und der Fallback (`LAST_CHECKED_FALLBACK`) aus der vorherigen Sitzung unverändert übernommen.
- Verifiziert über einen temporären lokalen Server: Mobile 375px UND Desktop 1280px — Bubble öffnet mit korrektem Text/Datum, exakt horizontal über dem Preis zentriert (per `getBoundingClientRect()`-Vergleich bestätigt); Klick auf denselben Preis schließt sie wieder; Klick außerhalb schließt sie; Klick auf einen anderen Preis bewegt sie dorthin, ohne vorher schließen zu müssen; Scroll schließt sie; bei wenig Platz oberhalb (Preis nah am oberen Bildschirmrand) klappt sie automatisch nach unten (`data-placement="bottom"`) und der Pfeil dreht sich mit; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v47` erhöht (Pflichtregel).
- `CLAUDE.md` aktualisiert: Beschreibung von `lastCheckedAt`/Klick-auf-Preis-Verhalten an die neue Popover-Komponente angepasst (`#pricePopover` statt `#checkedOverlay`/`.mini-modal`).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (6) — "Zuletzt geprüft am ..."-Fenster beim Antippen des Preises

**Umgesetzt (nur Mobile):**
- **Neues manuell gepflegtes Datenfeld `lastCheckedAt`**: Jedes `offers[]`-Objekt in `deals.json` hat jetzt ein Feld `lastCheckedAt` (`"YYYY-MM-DD"`), aktuell einheitlich auf `"2026-09-01"` gesetzt (Demo-Default). Anders als das bestehende, rein clientseitig aus der offer-id abgeleitete `checkedMinutesAgo` ("Preis geprüft vor ...", aktuell in der UI ausgeblendet) ist `lastCheckedAt` ein echtes, händisch pflegbares Redaktionsfeld in der Datenquelle — direkt in `deals.json` bearbeitbar, um jederzeit ein aktuelles Datum einzutragen. `_meta.fields.offers[]` in `deals.json` dokumentiert das neue Feld. In `buildDeals()` (`index.html`) gibt es einen Fallback (`LAST_CHECKED_FALLBACK` = `DEMO_TODAY_STR`) für den Fall, dass ein Angebot das Feld nicht setzt.
- **Preis ist jetzt antippbar**: Der Preis auf jeder Angebotskarte (`.price` in `.price-line`) ist von einem `<span>` zu einem `<button class="price" data-checked="...">` geworden. Klick/Tap öffnet ein kleines, zentriertes Fenster (`#checkedOverlay`, neue `.mini-modal`-Komponente statt der bestehenden Bottom-Sheets) mit „Zuletzt geprüft am [Datum]" (`openLastChecked()`, Datum formatiert über neue Hilfsfunktion `formatSingleDate()`). Das Fenster nutzt dieselbe Overlay-Backdrop-/Schließen-Mechanik (`openSheet`/`closeSheet`, Klick auf X oder auf den abgedunkelten Hintergrund) wie die bestehenden Sheets, ist aber bewusst klein und zentriert statt als Bottom-Sheet umgesetzt (`.overlay.center` + `.mini-modal`, eigene Scale-/Fade-Animation statt Slide-up).
- **Nur die Preisangabe ist klickbar, kein leerer Bereich daneben**: `button.price` bekommt einen Reset (`background:none;border:0;padding:0;` usw.), damit der Button exakt so breit wie der Preistext selbst bleibt (kein Full-Width-Button durch Browser-Standardstile) — die leere Fläche rechts daneben in `.price-line` (z.B. neben "+ 0,25 € Pfand") bleibt nicht klickbar.
- Sonst keine Änderungen an Design oder anderen Bereichen; die bereits ausgeblendete "Preis geprüft vor ..."-Logik (`checkedMinutesAgo`) bleibt unverändert und weiterhin unsichtbar.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Klick auf den Preis öffnet das Fenster mit dem korrekten, aus `deal.lastCheckedAt` gelesenen Datum (per DOM-Test für mehrere Angebote bestätigt, inkl. Test mit manuell auf ein abweichendes Datum gesetztem `lastCheckedAt` — Anzeige folgt korrekt); Klick auf X sowie auf den abgedunkelten Hintergrund schließt das Fenster; `elementFromPoint` am rechten Rand von `.price-line` (neben dem Pfand-Hinweis) trifft weiterhin den umgebenden `<div>`, nicht den Preis-Button; Breite des Preis-Buttons entspricht der Textbreite (kein Full-Width-Button); keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v46` erhöht (Pflichtregel) — betrifft sowohl `index.html` als auch `deals.json` (beide in `APP_SHELL`).
- `CLAUDE.md` aktualisiert: `lastCheckedAt` im `offers[]`-Feldschema ergänzt, alte Notiz "deliberately deferred extension, not wired up" entfernt/ersetzt, da jetzt umgesetzt.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (5) — Produktname-Höhe bei "Ähnliche Produkte" vereinheitlicht

**Umgesetzt (nur Mobile):**
- **Feste Höhe für den Produktnamen**: `.similar-name` (Produktname in der `.similar-card`-Leiste im Produktdetail-Sheet) bekommt jetzt `min-height:2.5em` (= 2 Zeilen bei `line-height:1.25`). Dadurch reserviert der Namensbereich bei jeder Karte immer den Platz für 2 Textzeilen, auch wenn der eigentliche Produktname (z.B. "Red Bull Zero") nur eine Zeile braucht — die darunterliegenden Zeilen (ml-Angabe/Händler-Meta und Preis) stehen dadurch bei allen Karten exakt auf derselben Höhe, unabhängig von 1- oder 2-zeiligem Namen. Bild-Ausrichtung oben (aus dem vorherigen Fix) bleibt unverändert bestehen.
- Sonst keine Änderungen an Design oder anderen Bereichen.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Bild-, Meta- und Preis-Offset innerhalb der Karte per `getBoundingClientRect()` für alle 6 „Ähnliche Produkte"-Karten gemessen — Bild bei allen 11px, ml-Angabe bei allen 126px, Preis bei allen 143px (unabhängig davon, ob der Name 1 oder 2 Zeilen braucht); `.similar-name` wird nur an dieser einen Stelle im Code verwendet, keine Seiteneffekte auf andere Bereiche; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v45` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (4) — Bilder bei "Ähnliche Produkte" ausgerichtet + Sortieroption "Günstigster €/Liter" entfernt

**Umgesetzt (nur Mobile):**
- **Bildausrichtung bei "Ähnliche Produkte"**: Die Produktbilder in der horizontalen `.similar-card`-Leiste (im Produktdetail-Sheet) standen bei Produkten mit einzeiligem Namen (z.B. "Red Bull Zero") sichtbar tiefer als bei zweizeiligen Namen. Ursache: `.similar-card` ist ein `<button>` ohne explizites `display`; Chrome zentriert den Inhalt von Buttons in solchen Fällen vertikal innerhalb der (durch `align-items:stretch` im Elternflex `.similar-scroll` gestreckten) Button-Höhe, statt ihn oben auszurichten — bei kürzerem Textinhalt blieb dadurch mehr Leerraum, der zur Hälfte oberhalb des Bildes landete. Behoben, indem `.similar-card` selbst `display:flex;flex-direction:column;align-items:stretch;justify-content:flex-start` bekommt, wodurch der Button-Inhalt wieder zuverlässig oben ausgerichtet wird, unabhängig von 1- oder 2-zeiligem Produktnamen. (Ein einfaches `display:block` allein reichte nicht aus, um Chromes interne Button-Zentrierung zu überschreiben — das wurde zuerst versucht und im Browser verifiziert, dass es nicht ausreicht.)
- **Sortieroption entfernt**: „Günstigster €/Liter" komplett aus der Sortierung entfernt — sowohl aus dem (versteckten) `<select id="sort">` als auch aus dem Sortierungs-Sheet (`.sort-row[data-sort="liter"]`), inklusive der zugehörigen Sortierlogik (`sort.value==="liter"`-Zweig) und des Labels in `sortLabels`. Die übrigen Optionen (Günstigster Preis, Nächster Händler, Beste Ersparnis, Endet bald) sind unverändert. Die €/L-Anzeige auf den einzelnen Angebotskarten selbst (`literPrice`-Berechnung für "3,56 €/L") ist davon nicht betroffen und bleibt bestehen — nur die Sortier-*option* wurde entfernt, nicht die Anzeige.
- Bei der Verifizierung im Vorschau-Server fiel auf, dass eine zwischengeschaltete Proxy-/Cache-Schicht der Browser-Vorschau alte `index.html`-Inhalte auslieferte, obwohl der Server selbst (per `curl` bestätigt) bereits den aktuellen Stand hatte — ein Cache-Busting-Query-Parameter beim Navigieren hat das für die Testsitzung umgangen; kein Hinweis, dass dies die echte Auslieferung an Endnutzer (GitHub Pages / Service Worker) betrifft.
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Bild-Offset innerhalb der Karte per `getBoundingClientRect()` für alle 6 „Ähnliche Produkte"-Karten gemessen — vorher 11px vs. 18.5px je nach Textlänge, nachher einheitlich 11px bei allen; Sortierungs-Sheet zeigt nur noch die vier verbleibenden Optionen, Sortieren nach „Nächster Händler" funktioniert weiterhin korrekt; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v44` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 (3) — Herz-Schatten im hellen Modus entfernt + leere Klickflächen bei Produktname/Händlerzeile entfernt

**Umgesetzt (nur Mobile, ab jetzt Standard-Arbeitsumfang):**
- **Herz-Icon-Schatten**: `.thumb-frame .btn.fav` hatte einen festen `filter:drop-shadow(...)`, der unabhängig vom Theme immer angewendet wurde und im hellen Modus als unschöner leichter schwarzer Schatten auffiel. Der Schatten gilt jetzt nur noch im dunklen Theme (`:root[data-theme="dark"] .thumb-frame .btn.fav{filter:...}`), im hellen Modus ist er komplett entfernt. Da dies eine reine Theme-Eigenschaft ist (nicht Viewport-abhängig), wirkt sich die Änderung zwangsläufig gleich auf Mobile und Webapp aus — es gibt kein sinnvolles Mobile-only-CSS dafür.
- **Leere anklickbare Flächen entfernt**: `.product` (Produktname auf der Karte, wiederverwendet auch in der Alarme-Ansicht) und die neue Klasse `.store-name` (Händlername + Entfernung, z.B. "Kaufland · 1.4 km") waren als normale Block-`<div>`s mit `data-product-detail`/`data-store-detail` immer so breit wie ihr Elternelement (bzw. wie die breitere Nachbarzeile) — dadurch war der leere Bereich rechts neben kurzem Text mit anklickbar und öffnete fälschlich das Produkt- bzw. Händlerfenster. Beide Elemente haben jetzt `width:fit-content`, sodass die Klickfläche exakt dem sichtbaren Text entspricht. Layout/Umbruchverhalten bei langen Namen unverändert (fit-content bleibt innerhalb der verfügbaren Breite).
- Verifiziert über einen temporären lokalen Server (Mobile 375px): Herz-Schatten computed `filter` ist im hellen Theme `none`, im dunklen Theme weiterhin `drop-shadow(...)`; Klick auf die leere Fläche neben einem kurzen Produktnamen bzw. neben "1.4 km" öffnet nichts mehr (per `elementFromPoint` + Klick-Test bestätigt), Klick auf den eigentlichen Text öffnet weiterhin zuverlässig Produkt- bzw. Händlerfenster; keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v43` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.
- Ab dieser Sitzung: Arbeitsumfang standardmäßig nur noch Mobile, außer der Nutzer nennt explizit "beide Versionen" (siehe Nutzeranweisung).

---

## 2026-09-04 (2) — Suche bleibt im Hintergrund + "Mein Bereich" bleibt beim Öffnen von Untermenüs offen

**Umgesetzt (Mobile + Webapp):**
- **Suche springt nicht mehr sofort**: Tippen in der Suchleiste filtert die Startseite/Liste im Hintergrund jetzt nicht mehr live mit — sie bleibt unverändert sichtbar, während die Vorschlagsliste (Autocomplete) darüber Treffer anzeigt. Erst ein Klick auf einen konkreten Vorschlag übernimmt den Produktnamen in die Suchleiste und filtert die Liste auf genau dieses Produkt ("springt zum Produkt"). Technisch: `render()` filtert jetzt nach einem separaten `committedSearch`-Wert statt direkt nach `search.value`; der `input`-Listener aktualisiert nur noch die Vorschläge (und setzt `committedSearch` nur zurück, wenn das Feld komplett geleert wird); beide bestehenden "Filter zurücksetzen"-Stellen (leere Suche über den Reset-Button im Empty-State bzw. über "Start" in der unteren Navigation) setzen `committedSearch` konsistent mit zurück.
- Platzhaltertext der Suchleiste geändert von „Energy Drink, Marke oder Händler..." zu „Energy Drink oder Marke eingeben".
- Beschreibungstext „Deine Daten, Einstellungen und dein Konto." direkt unter der Überschrift „Mein Bereich" entfernt (Überschrift bleibt, Rest des Bereichs unverändert).
- **"Mein Bereich" bleibt beim Öffnen eines Untermenüs (Standort & Umkreis, Push-Benachrichtigungen, Konto verwalten) jetzt im Hintergrund geöffnet**, statt geschlossen zu werden — es schließt sich nur noch, wenn der Nutzer es aktiv wegklickt/wegwischt (X-Button, Klick auf den abgedunkelten Hintergrund oder Wischgeste). Speichern/Schließen eines Untermenüs (z.B. „Speichern" bei Push-Benachrichtigungen, „Aktualisieren" bei Standort & Umkreis) schließt weiterhin nur dieses Untermenü, „Mein Bereich" bleibt darunter sichtbar offen. Technisch: `openSheet()` verschiebt das zu öffnende Overlay-Element jetzt an das Ende seines Elternelements, bevor es sichtbar geschaltet wird — dadurch liegt es bei gleichem z-index immer über bereits offenen Overlays, unabhängig von der ursprünglichen Reihenfolge im HTML (relevant, weil „Standort & Umkreis" und „Push-Benachrichtigungen" im Markup eigentlich vor „Mein Bereich" stehen). Der bisherige `closeSheet("profileOverlay")`-Aufruf beim Klick auf eine Profil-Zeile wurde entfernt.
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 1280px): Suche bleibt beim Tippen im Hintergrund unverändert (Zähler bleibt bei 60), Vorschlagsauswahl filtert korrekt auf 1 Treffer, Leeren der Suche stellt wieder alle 60 Angebote her; „Mein Bereich" bleibt beim Öffnen von „Standort & Umkreis"/„Push-Benachrichtigungen"/„Konto verwalten" sichtbar im Hintergrund offen (per DOM-Check bestätigt), das jeweilige Untermenü schließt sich beim Speichern eigenständig, „Mein Bereich" bleibt offen bis zum expliziten Schließen; keine echten Konsolenfehler (nur bekanntes Test-Tool-Rauschen durch simulierte Touch-Events, kein App-Code betroffen).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v42` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-04 — "Konto verwalten" und "Abmelden" vertauscht

**Umgesetzt (Mobile + Webapp):**
- Reihenfolge im „Konto"-Abschnitt von „Mein Bereich" getauscht: „Konto verwalten" steht jetzt vor „Abmelden" (zuvor umgekehrt). Reine Markup-Umstellung, IDs/Klick-Verhalten beider Buttons unverändert.
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px): neue Reihenfolge sichtbar, beide Buttons weiterhin funktionsfähig (Konto verwalten öffnet Untermenü, Abmelden zeigt Demo-Toast), keine Konsolenfehler.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v41` erhöht (Pflichtregel).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Name-/E-Mail-Bearbeitung + neue Adressverwaltung nach "Konto verwalten" verschoben

**Umgesetzt (Mobile + Webapp):**
- **"Mein Bereich"-Hauptbildschirm vereinfacht**: Avatar und Name bleiben wie gefordert direkt sichtbar (unverändert an derselben Stelle), aber der Name ist dort jetzt reiner Anzeigetext (`.profile-name-display`, kein Stift-Icon mehr) — das komplette E-Mail-Feld ist von dort entfernt.
- **Neuer Abschnitt „Persönliche Daten" unter „Konto verwalten"**: enthält jetzt Name (Bearbeiten-Mechanik 1:1 hierher verschoben, identisches Verhalten) und E-Mail-Adresse (komplettes Feld verschoben). Der Name wird intern an zwei Stellen synchron gehalten (Anzeige in „Mein Bereich" + Eingabefeld unter „Konto verwalten") — beim Speichern aktualisieren sich beide sofort.
- **Neu: Adressverwaltung** („Adresse"-Feld unter „Persönliche Daten", nach Vorbild typischer Shopping-Apps wie Amazon/Zalando): Straße & Hausnummer, PLZ, Ort, Land (Land vorbelegt mit „Deutschland"). Zeigt im Ansichtsmodus eine Kurzfassung („Musterstraße 12, 59821 Arnsberg, Deutschland") oder „Noch keine Adresse hinterlegt", falls nichts gespeichert ist; im Bearbeiten-Modus ein gestapeltes Formular (PLZ+Ort nebeneinander, restliche Felder je eigene Zeile, Speichern-Button über volle Breite) — neue CSS-Variante `.field-edit-row.stacked` für mehrfeldige Formulare, ergänzt die bestehende einzeilige Variante (Name/E-Mail), ohne diese zu verändern. Persistiert als JSON unter dem neuen localStorage-Key `canspot-address`, nach demselben Muster wie `canspot-name`/`canspot-email`.
- Bearbeiten-/Speichern-Mechanik (`[data-edit]`/`[data-save]`-Handler, `setFieldEditing()`) generisch erweitert (Fokus-Ziel-Zuordnung jetzt als kleine Map statt hartcodierter Zwei-Wege-Verzweigung), damit sie ohne Sonderfall auch für „address" funktioniert — Verhalten für „name"/„email" dabei unverändert.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v40` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px, Light + Dark Mode): Hauptbildschirm zeigt Avatar+Name ohne Bearbeiten-Möglichkeit und ohne E-Mail-Feld; „Konto verwalten" zeigt Name/E-Mail/Adresse mit funktionierendem Bearbeiten+Speichern; Namensänderung dort aktualisiert nachweislich sofort auch die Anzeige in „Mein Bereich"; Adresse speichert korrekt in `localStorage`, Kurzfassung wird korrekt gebildet, bleibt nach Neuladen erhalten, zeigt „Noch keine Adresse hinterlegt" wenn leer; Profilbild-Bearbeitung/-Entfernen, „Abmelden", „Konto löschen" weiterhin unverändert funktionsfähig; Filter/Suche unverändert fehlerfrei; keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — "Bester Preis" öffnet Händler-Fenster, Banner-Profil-Button entfernt, Suche fokussiert zuverlässiger, "Mein Bereich" neu sortiert

**Umgesetzt (alles für Mobile + Webapp, wo nicht anders vermerkt):**
- **"BESTER PREIS"-Badge**: öffnete bisher per `window.open(deal.link, "_blank")` direkt einen externen Tab. Jetzt `data-store-detail="${deal.id}"` statt `data-link` — nutzt denselben, bereits vorhandenen In-App-Mechanismus wie „Zum Angebot"/Händler-Logo (öffnet das Händler-Fenster; von dort aus weiterhin über „Angebot online öffnen" erreichbar, falls gewünscht). Alter, jetzt toter `.badge-best`-Klick-Handler entfernt.
- **„Mein Bereich"-Button oben rechts im Banner entfernt** — war redundant zum „Mein Bereich"-Tab in der unteren Navigation, der unverändert funktioniert. Zugehörigen (sonst verwaisten) Event-Listener mit entfernt, um einen `null`-Fehler beim Laden zu vermeiden.
- **„Suche"-Tab**: `search.focus()` wird jetzt VOR dem `window.scrollTo({top:0})` aufgerufen statt danach — mobile Browser scrollen ein neu fokussiertes Feld automatisch mit ins Bild, sobald die virtuelle Tastatur erscheint, was einen vorher gestarteten Scroll-zum-Seitenanfang sonst wieder überschreiben könnte; `focus()` bleibt weiterhin synchron im Klick-Handler (Voraussetzung dafür, dass mobile Browser die Tastatur überhaupt automatisch öffnen). **Nebenbei gefundenen Bug behoben**: der Tab hat bisher `setActiveNav("home")` statt `setActiveNav("search")` aufgerufen — beim Antippen von „Suche" wurde also fälschlich „Start" hervorgehoben statt „Suche" selbst.
- **„Mein Bereich" neu sortiert** (an gängigen Shopping-App-Kontoseiten orientiert, z.B. Amazon/Zalando-Stil: Kern-Einstellungen direkt sichtbar, riskante Kontoaktionen einen Tipp tiefer):
  - „Erscheinungsbild" bleibt wie gefordert direkt sichtbar, unverändert an derselben Stelle.
  - Neue Gruppenüberschrift „Einstellungen" über „Standort & Umkreis" und „Push-Benachrichtigungen" (beide inhaltlich unverändert, nur jetzt klar gruppiert statt ohne Überschrift zu schweben).
  - Neue Gruppenüberschrift „Meine Aktivität" über „Favoriten (N)"/„Preisalarme (N)" (jetzt als kleinere `.field-sublabel`-Unterüberschriften innerhalb der Gruppe statt zwei gleichrangige Überschriften direkt hintereinander) — Inhalte/Listen selbst unverändert.
  - „Konto löschen" aus der Hauptansicht entfernt und in ein neues Untermenü „Konto verwalten" verschoben (erreichbar über eine neue Zeile unter „Abmelden", das sichtbar bleibt) — dort unter einer „Gefahrenzone"-Überschrift mit Warnhinweis, wie bei den meisten Shopping-Apps üblich für destruktive Kontoaktionen. Button/ID/Klick-Verhalten (Demo-Toast) unverändert, nur der Ort hat sich geändert.
  - Neues Untermenü folgt exakt demselben Sheet-Muster wie alle anderen Fenster (`.overlay > .sheet > .sheet-inner`, X-Button, Handle) und ist in beide bereits bestehenden Listen für „Wisch nach unten zum Schließen" und „Tippen auf Handle zum Schließen" eingetragen — dieselbe Erwartung, die für alle anderen Fenster bereits gilt.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v39` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px, Light + Dark Mode): „BESTER PREIS" öffnet nachweislich das Händler-Fenster statt eines neuen Tabs; Banner zeigt nur noch die Glocke; „Suche" scrollt zuverlässig nach oben, fokussiert das Suchfeld UND markiert jetzt korrekt „Suche" (nicht mehr „Start") als aktiven Tab; „Mein Bereich" zeigt „Konto löschen" nicht mehr direkt, „Konto verwalten" öffnet das neue Untermenü korrekt, Löschen-Button dort weiterhin funktionsfähig (Demo-Toast), Untermenü schließt per X, Wisch-Geste und Handle-Tipp; Name-/E-Mail-Bearbeitung, Theme-Umschalter, Favoriten-/Preisalarm-Listen weiterhin unverändert funktionsfähig; Filter/Suche/Karte-Tab unverändert fehlerfrei; keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Klickziele auf der Angebotskarte präzisiert (Mobile + Webapp): kein Klick auf leere Fläche mehr, Händler öffnet Karte, Herz nur exakt getroffen

**Umgesetzt:**
- **"Klick auf leere Fläche öffnet Produktansicht" entfernt**: Der `dealsContainer`-Klick-Handler hatte am Ende einen Catch-all-Fallback (`const card = e.target.closest(".card"); if(card...) openHistory(...)`), der auf JEDEN Klick irgendwo auf der Karte reagierte, der nicht bereits von einem spezifischeren Handler abgefangen wurde. Ersatzlos entfernt — ein Klick auf Preis, Rabatt-Badges, Zeitraum, Ersparnis usw. löst jetzt bewusst keine Aktion mehr aus.
- **Händler-Logo/-Name öffnet jetzt die Händler-Karte statt (fälschlich) die Produktansicht**: `data-store-detail="${deal.id}"` auf das Händler-Logo (`.store-logo`) und den Händlernamen-Text ergänzt — nutzt den bereits vorhandenen `openStoreDetail()`-Mechanismus (identisch zum "Zum Angebot"-Button), keine neue Klick-Logik nötig.
- **Neu: Mini-Karte im Händler-Fenster** ("Standort"-Abschnitt, oberhalb von Adresse/Öffnungszeiten): zeigt denselben radialen "Du in der Mitte, Pin nach Winkel+Distanz positioniert"-Stil wie der bestehende Karte-Tab (`renderMap()`), hier mit genau einem Pin für den jeweiligen Händler (`renderStoreDetailMap()`, neue Funktion) — dieselben `.map-canvas`/`.map-pin`/`.map-ring`/`.map-user`-CSS-Klassen wiederverwendet, keine neue Kartendarstellung/Bibliothek nötig, dadurch optisch konsistent zum Rest der App.
- **Produktname jetzt zusätzlich anklickbar**: `data-product-detail="${deal.productId}"` auf den Produktnamen-Text ergänzt (das Produktbild hatte dieses Attribut bereits) — öffnet die Produktseite mit den Inhaltsstoffen, wie beim Bild.
- **Herz favorisiert jetzt wirklich nur bei direktem Treffer**: Die Klickfläche des Herz-Buttons (`.thumb-frame .btn.fav`) war deutlich größer als das sichtbare Icon (Webapp: 44px Button um ein 26px Icon, 9px unsichtbarer Rand je Seite; Mobile: 36px um 20px, 8px Rand) — ein Klick "in der Nähe" des Herzens, aber eigentlich auf dem Produktbild, favorisierte trotzdem. Klickfläche auf 30px (Webapp) bzw. 24px (Mobile) verkleinert, nur noch 2px Rand um das weiterhin gleich große, gleich positionierte Icon — Position/Optik des Herzens dadurch unverändert (nachgerechnet: exakt dieselbe Pixel-Position wie zuvor), nur der unsichtbare Klickbereich ist jetzt eng genug, dass ein Klick daneben ans darunterliegende Bild durchgereicht wird (öffnet dort korrekt die Produktseite statt zu favorisieren).
- Cursor:pointer für die neuen Klickziele ergänzt (`[data-product-detail]`, `[data-store-detail]`) als dezente Hover-Rückmeldung, sonst keine optischen Änderungen.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v38` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px, Light + Dark Mode): Klick auf leere Kartenbereiche (Preiszeile, Ersparnis-Zeile, Karte selbst) öffnet nachweislich nichts mehr; Produktname UND -bild öffnen die Produktseite; Händler-Logo UND -Name öffnen das Händler-Fenster mit korrekt gerenderter Mini-Karte; Herz-Klick favorisiert weiterhin zuverlässig bei direktem Treffer, ein Klick auf das Bild (aber nicht mehr auf das jetzt kleinere Herz-Ziel) favorisiert nicht mehr und öffnet stattdessen korrekt die Produktseite; „Zum Angebot"-Button, „Preisverlauf"-Button sowie der Produktlink im Alarme-Tab weiterhin funktionsfähig; Filter/Suche/Karte-Tab unverändert fehlerfrei; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Für die Händler-Mini-Karte bewusst die exakt gleiche Positionierungsformel (Winkel aus `hashStr(store)`, Radius aus `distanceKm`/`radiusKm`) wie im bestehenden Karte-Tab wiederverwendet statt einer eigenen Logik — dadurch bei nahen Händlern (verglichen mit dem Standard-Suchradius von 10 km) ein Pin nah am Zentrum, exakt wie im Haupt-Kartentab schon der Fall (kein neues/inkonsistentes Verhalten, nur an einer zweiten Stelle sichtbar gemacht).
- Herz-Klickfläche nicht auf exakte Icon-Größe (0px Rand) reduziert, sondern auf ca. 2px Rand — verhindert das beschriebene Fehlverhalten zuverlässig, ohne die Fläche so klein zu machen, dass ein exakt gemeinter Klick versehentlich knapp danebengeht.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Wisch-nach-unten-zum-Schließen für ALLE Sheets (nur Mobile), fehlte bei 7 von 9

**Umgesetzt:**
- Nutzerhinweis: Nicht jedes Sheet ließ sich per Wisch nach unten schließen. Geprüft: von 9 Overlays (`locOverlay`/Standort, `notifOverlay`/Benachrichtigungen, `newsOverlay`/Neuigkeiten, `filterOverlay`/Weitere Filter, `sortOverlay`/Sortierung, `profileOverlay`/Mein Bereich, `histOverlay`/Preisverlauf, `productDetailOverlay`/Produktdetail, `storeDetailOverlay`/Händler-Detail) hatten nur `histOverlay` und `productDetailOverlay` die echte Wisch-Geste (`initSwipeToClose`) — die übrigen 7 hatten nur `initHandleTapToClose` (Tippen auf den kleinen Handle-Balken oben, keine Wischgeste). `initSwipeToClose` jetzt für alle 9 Overlays aktiviert.
- **Bug gefunden und behoben, der beim naiven Übertragen aufgetreten wäre**: `initSwipeToClose` prüfte bisher nur `sheet.scrollTop`, um zu erkennen "ist der Nutzer noch ganz oben, oder scrollt er gerade im Inhalt?". Manche Sheets scrollen aber nicht direkt über `.sheet`, sondern über ein inneres `.sheet-scroll` (verwendet bei Sheets mit fest stehendem Footer-Button, z. B. `locOverlay` mit "Aktualisieren" und `filterOverlay` mit "Anwenden" — siehe CLAUDE.md-Markup-Konvention). Ohne Fix hätte ein Wisch nach unten das Sheet geschlossen, obwohl der Nutzer nur innerhalb des gescrollten Inhalts nach oben wischen wollte. Behoben: `initSwipeToClose` ermittelt jetzt `overlay.querySelector(".sheet-scroll") || sheet` und prüft dessen `scrollTop` statt immer `.sheet` selbst — für die beiden bereits vorher funktionierenden Sheets (kein `.sheet-scroll`) exakt dasselbe Verhalten wie zuvor (Regressionsrisiko minimiert), für die neu hinzugekommenen mit `.sheet-scroll` (`locOverlay`, `filterOverlay`) jetzt korrekt.
- Betrifft nur echte Touch-Geräte (Mobile) — auf Maus-basierten Desktop-/Webapp-Bildschirmen feuern `touchstart`/`touchmove`/`touchend` ohnehin nie, daher wie gewünscht keine Auswirkung auf die Webapp-Version.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v37` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server mit simulierten Touch-Events (375px Breite): alle 7 zuvor fehlenden Sheets schließen jetzt korrekt per Wisch-Geste; `histOverlay`/`productDetailOverlay` weiterhin unverändert funktionsfähig (kein Regressions-Bruch); horizontales Ziehen am Umkreis-Regler (`locOverlay`) löst weiterhin korrekt NICHT das Schließen aus (Geste wird anhand Vertikal-/Horizontal-Dominanz unterschieden); Wisch nach unten bei bereits heruntergescrolltem Inhalt (`filterOverlay`, `scrollTop=200`) schließt korrekt NICHT (bestätigt den `.sheet-scroll`-Fix); Filter-Chips, Sortier-Zeilen und Benachrichtigungs-Toggle innerhalb der betroffenen Sheets weiterhin normal klickbar; Suche, Karte, Favoriten-Tab weiterhin fehlerfrei; keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Preisverlauf-Chart (nur Mobile): Punkt bleibt nach Antippen eines Tages stehen statt zurückzuspringen

**Umgesetzt:**
- Nutzerwunsch, explizit nur für die Mobile Version: Der Auswahl-Punkt im Preisverlauf-Chart soll durch Antippen eines Datums dorthin verschoben werden, den Preis dort anzeigen — und dort **stehen bleiben**, statt beim Loslassen zur „Heute"-Position zurückzuspringen.
- In `initChartScrub()` die Touch-`pointerup`-Behandlung angepasst: ruft nicht mehr `hide()` auf, sondern setzt nur noch das interne `dragging`-Flag zurück — der zuletzt angetippte/gezogene Tag (gestrichelte Linie, hervorgehobener Punkt, Tooltip mit Datum+Preis+Händler) bleibt sichtbar, bis der nächste Tipp auf einen anderen Tag erfolgt. `pointercancel` verhält sich für Touch identisch (kein automatisches Ausblenden). Die Unterscheidung läuft über `pointerType === "touch"` — die Maus-Hover-Vorschau der Webapp/Desktop (verschwindet beim Verlassen der Chart-Fläche) ist dadurch selektiv unangetastet geblieben; Design, Chart-Optik, Legende, Statistik-Boxen und alle sonstigen Funktionen unverändert.
- **Bug gefunden und behoben** (beim Testen aufgefallen, betraf potenziell auch schon die vorherige Scrubbing-Version): `svg.setPointerCapture(e.pointerId)` kann in bestimmten Fällen eine Exception werfen (`NotFoundError`, z. B. wenn kein vom Browser als „aktiv" erkannter Zeiger mit dieser ID vorliegt) — das brach die komplette `pointerdown`-Behandlung ab, sodass der Tooltip beim Antippen gar nicht erst erschien. `setPointerCapture` jetzt in try/catch gekapselt: Punkt-Erfassung ist nur eine Verbesserung (Zeigerbewegung weiterverfolgen, auch wenn der Finger die Chart-Fläche verlässt), ein Fehlschlag darf die Kernfunktion (Tag antippen → Preis anzeigen) nicht mehr verhindern.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v36` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Simuliertes Antippen (Touch-Pointer-Events) bei 375px Breite zeigt Tooltip mit Datum/Preis/Händler des angetippten Tages und dieser bleibt nach `pointerup` sichtbar stehen; zweites Antippen an anderer Stelle bewegt Punkt+Tooltip dorthin; Maus-Hover auf breiterem Viewport (Webapp) weiterhin unverändert — erscheint bei Hovern, verschwindet beim Verlassen der Chart-Fläche. Regressionstest bestehender Funktionen (Packungsgrößen-Filter, Suche, Karte, Zeitraum-Wechsel) weiterhin fehlerfrei, keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Preisverlauf zurück zum Chart (statt Tages-Karussell), jetzt mit antippbarem/ziehbarem Scrubbing

**Umgesetzt:**
- Nutzerwunsch: Preisverlauf soll wieder ein **Diagramm** sein (wie vor der Karussell-Version), nur optisch moderner — orientiert an Preisvergleichs-/Finanz-Apps. Das Tages-Karussell (horizontal durchblätterbare Einzeltag-Karten, letzte Runde) ist entfernt; die Linien-Chart mit Gradient-Fläche, geglätteter Kurve und den zwei farbigen Punkten (Blau = aktueller Preis, Grün = günstigster Preis) samt Legende ist zurück — technisch identisch zur bereits zuvor modernisierten Chart-Version (Catmull-Rom-Glättung, viewBox exakt auf Pixelgröße gesetzt, damit die Punkte echte Kreise bleiben statt Ovale).
- **Neu, über die vorherige Chart-Version hinaus** (nutzt die in der Karussell-Runde eingeführte datierte Preis-Datenstruktur `deal.priceHistoryPoints`, die bewusst NICHT wieder entfernt wurde): Der Chart ist jetzt interaktiv **scrubbar** — Antippen/Ziehen auf Mobile bzw. Hovern mit der Maus auf der Webapp zeigt eine gestrichelte Linie + hervorgehobenen Punkt + Tooltip-Blase mit Datum, Preis und Händler des jeweils nächstgelegenen Tages (z. B. „0,87 € · Di, 25.08. · Kaufland"), exakt wie bei modernen Aktien-/Preisvergleichs-Apps (Robinhood, Apple Aktien u. ä.). Desktop reagiert auf reines Hovern (kein Klick nötig), Touch erfordert ein aktives Ziehen (`pointerdown`→`pointermove`), damit nicht jede zufällige Berührung beim Scrollen den Tooltip auslöst. Beim Loslassen/Verlassen kehrt der Chart in die Normalansicht zurück.
- `.day-carousel`/`.day-card`/`.day-nav-*`/`.day-progress`-CSS sowie `buildDayCardHtml()`, `renderDayCarousel()`, `goToDay()` und die zugehörigen Button-/Tastatur-/Scroll-Listener vollständig entfernt (kein toter Code). Das nur für die Karussell-Navigation ergänzte `i-arrow-right`-Icon-Symbol ebenfalls wieder entfernt, da nicht mehr verwendet.
- `deal.priceHistoryPoints` (datierte Punkte) UND `deal.history` (abgeleitetes Zahlen-Array) bleiben unverändert bestehen — die austauschbare, datierte Datenquelle aus der letzten Runde ist also nicht verloren gegangen, nur die Visualisierung hat sich (auf Nutzerwunsch) wieder geändert. Alle anderen Funktionen unangetastet: Zeitraum-Chips, Tiefstpreis-/Höchstpreis-/Ø-Preis-Boxen, Preisalarm, „Weitere Händler", Favoriten-Button, Teilen.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v35` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px, Light + Dark Mode): Chart zeigt bei 7/30/90 Tagen korrekt jeweils 3 Kreis-Marker (Low/Current/Scrub-Platzhalter) mit exakt pixelgroßer `viewBox`; Hover (Maus) zeigt sofort Tooltip mit korrektem Datum/Preis/Händler des nächstgelegenen Punkts; Touch-Move ohne vorheriges Antippen wird korrekt ignoriert, Touch-Down zeigt Tooltip, Touch-Up blendet ihn wieder aus; Fenster-Resize zeichnet den Chart korrekt in neuer Pixelgröße neu. Regressionstest bestehender Funktionen (Packungsgrößen-Filter, Such-Vorschläge inkl. Schreibweisen-Toleranz, Karte, Favoriten-Tab) weiterhin fehlerfrei, keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Preisverlauf komplett überarbeitet: Tages-Karussell statt Linien-Chart, datierte Preisverlaufs-Datenstruktur

**Umgesetzt:**
- **Kernänderung**: Die Linien-Chart im Preisverlauf-Sheet (zeigte alle Tage des gewählten Zeitraums gleichzeitig gequetscht auf einem Bildschirm) ist ersetzt durch ein **horizontal durchwischbares Tages-Karussell** — ein Tag pro Ansicht, volle verfügbare Breite, kein Zusammenpressen von Schrift/Statistik. Umgesetzt mit nativem CSS `scroll-snap` (fühlt sich auf Mobile wie eine native Wisch-Geste an, funktioniert auf der Webapp zusätzlich mit Maus-Rad/Trackpad) plus Prev-/Next-Buttons und Tastatur-Pfeiltasten (◀/▶) für Nicht-Touch-Bedienung. Kartenreihenfolge chronologisch (ältester Tag links, „Heute" rechts) — dadurch entspricht Wischen nach rechts (Finger bewegt sich nach rechts, Inhalt rutscht nach rechts, Vortag wird sichtbar) exakt „zum vorherigen Tag", wie in der Anforderung beschrieben.
- **Neue Datenstruktur**: Jeder Preis-Datenpunkt ist jetzt ein eigenständiges, datiertes Objekt statt einer nackten Zahl: `{ date, price, retailer, deposit, unitPrice, available }`. Das Datum ist damit fest mit dem Preis verknüpft (nicht dekorativ) — bein Wischen zu einem anderen Tag werden dessen tatsächliche Daten geladen und angezeigt (Preis, Preisänderung ggü. Vortag, Literpreis, Pfand, Preisvergleich zum Ø, „Gesamt an der Kasse", ggf. Tiefstpreis-/Nicht-verfügbar-Badge — alles pro Tag live berechnet, nicht fest verdrahtet).
- **Sauber austauschbare Datenquelle** (Hauptanliegen der Anfrage): `genPriceHistory()` generiert weiterhin 90 Tage synthetische Demo-Daten als Fallback, aber `normalizePriceHistory()` übernimmt ein von `deals.json` geliefertes `priceHistory`-Array 1:1 und ergänzt nur fehlende optionale Felder (`retailer`→`store` des Angebots, `deposit`→`pfand`, `unitPrice`→aus `price` berechnet, `available`→`true`) — eine künftige echte Datenquelle muss also nur minimale `{date, price}`-Paare liefern, keine Preisverlaufs-Logik neu bauen. Das alte, undatierte `number[]`-Format wird weiterhin akzeptiert (Rückwärtskompatibilität; Daten werden dann vom heutigen Tag rückwärts gezählt), ist aber nicht mehr die empfohlene Form. `deals.json`'s `_meta.fields` und `CLAUDE.md` entsprechend aktualisiert (neuer Eintrag `priceHistory[].fields`).
- **Bestehende Logik bewusst NICHT verändert**: `deal.history` (reines Zahlen-Array) bleibt als abgeleitetes Array (`priceHistoryPoints.map(p => p.price)`) exakt bestehen — alle bisherigen Verbraucher (Tiefstpreis-Badge auf der Karte, Ø-Preis-Vergleichsbadge, die drei „Tiefstpreis/Höchstpreis/Ø Preis"-Boxen im Sheet, `findBestDealFor()`, `renderOtherStores()`) laufen unverändert weiter, ohne selbst etwas von der neuen Datenstruktur zu wissen. Zeitraum-Chips (7/30/90 Tage), Preisalarm, Favoriten-Button, „Weitere Händler"-Liste, Teilen-Button — alles unangetastet.
- **Bug gefunden und behoben**: Die erste Implementierung von `dateStrAddDays()` (Datum ± N Tage) nutzte `Date.prototype.toISOString()` auf einem lokal konstruierten `Date`-Objekt — das rechnet intern nach UTC um und kippt dadurch in jeder Zeitzone östlich von UTC (z. B. UTC+2, wie dieser Test-Server) den letzten Tag eines Zeitraums fälschlich auf den Vortag zurück (aus „Heute" wurde "Gestern" im Datensatz). Behoben durch ausschließliche Rechnung in UTC-Millisekunden (`Date.UTC(...)`), unabhängig von der Zeitzone des aufrufenden Geräts.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v34` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px, Light + Dark Mode): Karussell zeigt bei „Heute" tatsächlich das korrekte heutige Datum (nach dem Zeitzonen-Fix); Wischen/Buttons/Tastaturpfeile navigieren korrekt und zeigen dabei jeweils die echten Daten des angezeigten Tages (Preis, Preisänderung, Store — an einem Beispieltag im Vergleich zu „Heute" unterschiedlich und korrekt neu berechnet); 7-Tage-Zeitraum zeigt anklickbare Punkte, 30/90-Tage zeigen stattdessen einen schlanken Fortschrittsbalken „Tag X/Y" (vermeidet 90 winzige Punkte); Wechsel des Zeitraums bzw. Öffnen eines anderen Angebots springt korrekt zurück auf „Heute"; Fenster-Resize (Mobile↔Webapp-Breite) bei offenem Sheet bleibt auf demselben Tag stehen statt zurückzuspringen; `normalizePriceHistory()` mit synthetischem Test sowohl im neuen datierten Format als auch im alten `number[]`-Format erfolgreich geprüft. Regressionstest bestehender Funktionen (Filter inkl. Packungsgröße, Suche/Autocomplete, Karte, Favoriten-/Alarme-Tab) weiterhin fehlerfrei, keine Konsolenfehler.

**Wichtige Entscheidungen:**
- `deal.history` bewusst als abgeleitetes Array beibehalten statt alle bisherigen Verbraucher auf die neue Punkt-Struktur umzustellen — deutlich risikoärmer („bestehende Funktionen dürfen nicht beschädigt werden") und die einzig wirklich nötige Änderung ist ohnehin die zeitliche Zuordnung fürs Karussell, nicht die bereits funktionierenden Aggregat-Statistiken.
- Retailer/Pfand nicht auf jedem einzelnen Preis-Datenpunkt zur Pflicht gemacht, sondern als optionale, vom Angebot geerbte Defaults behandelt — ein `priceHistory`-Array gehört immer zu genau einem `offers[]`-Eintrag (= ein Produkt+Händler+Gebinde), Wiederholung auf jedem Punkt wäre redundant.
- Bei langen Zeiträumen (30/90 Tage) bewusst weiterhin JEDEN Tag als eigene Karte durchblätterbar gelassen (nicht zu Wochen/Monaten aggregiert) — für die aktuelle Datenmenge performant genug, und die Anforderung erlaubte Aggregation nur optional, nicht zwingend. Nur die Positionsanzeige wechselt bei >10 Tagen von Punkten auf einen Fortschrittsbalken, um visuelles Chaos zu vermeiden.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Preisverlauf-Chart: ovaler Punkt behoben, moderneres Kurven-Design (Mobile + Webapp)

**Umgesetzt:**
- **Bug behoben:** Der Preis-Punkt im Preisverlauf-Chart (`#histChart`) war oval statt rund. Ursache: das SVG hatte eine feste `viewBox="0 0 300 140"` mit `preserveAspectRatio="none"`, während die tatsächliche gerenderte Breite je nach Sheet-/Bildschirmbreite stark abweicht (z.B. 313px mobil, 483px auf breiteren Screens) — dadurch wurde die interne 300er-Koordinatenbreite non-uniform auf die echte Pixelbreite gestreckt, was die `<circle>`-Marker (gleicher Radius in beide Richtungen) zu Ellipsen verzerrt hat.
- **Fix:** `drawChart()` misst jetzt bei jedem Zeichnen die tatsächliche gerenderte Pixelgröße des SVG (`getBoundingClientRect()`) und setzt die `viewBox` exakt darauf — 1 Koordinaten-Einheit entspricht dadurch immer exakt 1 CSS-Pixel in beide Richtungen, wodurch Kreis-Marker unabhängig von der Container-Breite immer echte Kreise bleiben. `preserveAspectRatio="none"` aus der statischen HTML-Markup entfernt (nicht mehr nötig). Neuer `resize`-Listener zeichnet das Chart neu, während das Preisverlauf-Sheet offen ist (z.B. bei Fenster-Resize zwischen Mobile- und Webapp-Breite).
- **Chart-Darstellung modernisiert** (an gängige Kurs-/Preis-Charts wie in Aktien-/Finanz-Apps angelehnt), ohne sonst etwas an der Sheet-Struktur zu verändern: Linie läuft jetzt als sanft geglättete Kurve durch die Datenpunkte (Catmull-Rom → kubische Bezierkurven, Standardtechnik moderner Chart-Bibliotheken) statt als kantige Geradenstücke; y-Achse bekommt zusätzlich ca. 12% Puffer über/unter dem tatsächlichen Tiefst-/Höchstpreis, damit Linie und Punkte nicht mehr am oberen/unteren Rand der Chart-Box kleben; der aktuelle-Preis-Punkt am Linienende etwas größer (r 4.5→5) und mit dickerem Rand für mehr Präsenz, passend zum bereits vorhandenen Farbschema (Blau = aktueller Preis, Grün = günstigster Preis) und zur bestehenden Legende — Farben, Gradient-Füllung, Legende, Zeitraum-Chips (7/30/90 Tage) und alles andere in der Preisverlauf-Ansicht bewusst unverändert gelassen.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v33` erhöht (Pflichtregel).
- **Regressionstest über alle Kernfunktionen** durchgeführt (Mobile 375px + Webapp/Desktop 577px, Light + Dark Mode): Preisverlauf-Chart bei 7/30/90 Tagen sowie bei Fenster-Resize (Chart bleibt kreisrund, `viewBox` folgt exakt der Pixelgröße); Tiefstpreis/Höchstpreis/Ø-Preis-Zahlen weiterhin unverändert aus den Rohdaten berechnet (nur die visuelle Skalierung im Chart bekam den Puffer, keine Auswirkung auf angezeigte Werte); Favoriten-Toggle und Preisalarm-Setzen im Preisverlauf-Sheet; „Weitere Händler"-Liste; Filter-Kombination (Packungsgröße + Volumen gemeinsam); Such-Vorschläge (inkl. Schreibweisen-Toleranz aus vorheriger Runde); Kartenansicht; Favoriten-/Alarme-Tab — alles fehlerfrei, keine Konsolenfehler, kein horizontaler Overflow.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Neuer Filter „Packungsgröße" (4er/6er/12er/24er-Multipacks), für Mobile und Webapp

**Umgesetzt:**
- **Neues Datenfeld `units`** in `deals.json`-Angeboten (Anzahl Dosen/Flaschen im Angebot; optional, Fallback `UNITS_FALLBACK = 1` in `index.html`, exakt nach dem bereits bestehenden Muster von `PFAND_FALLBACK`/`PACKAGING_FALLBACK`). Bei Mehrfachpackungen sind `regularPrice`/`offerPrice`/`pfand` in deals.json der **Gesamtpreis für die ganze Packung** (nicht pro Dose) — `_meta.fields.offers[]` entsprechend dokumentiert.
- **10 neue Demo-Angebote** (`d51`–`d60`, bestehende 50 Angebote unverändert gelassen) als Multipacks für einen Querschnitt der bestehenden Produkte/Händler ergänzt: je 2× 4er-Pack, 3× 6er-Pack, 2× 12er-Pack, 3× 24er-Pack, mit realistischer Mengenrabatt-Bepreisung.
- **Neuer Filter „Packungsgröße"** im „Weitere Filter"-Sheet (Chips: Alle/Einzeln/4er Pack/6er Pack/12er Pack/24er Pack — Benennung wie bei Amazon & Co. üblich), als eigener `unitsFilter`-State parallel zu `sizeFilter`/`packagingFilter` aufgebaut (gleiches Muster: Filterlogik in `render()` inkl. der "0 Treffer, aber X deutschlandweit"-Zählung, `updateFilterCount()`, `resetExtraFilters()`, Klick-Handler). Wirkt identisch auf Mobile und Webapp (gemeinsame Logik).
- **Preisberechnungen um `units` erweitert**, damit Multipack-Angebote korrekte statt grob falsche Zahlen zeigen: `deal.totalMl` (= `sizeMl × units`) neu in `buildDeals()`, und **alle** Stellen, die bisher Literpreis aus `sizeMl` berechnet haben (Kartenanzeige, Sortierung „Bester Preis pro Liter", Favoriten-Vergleichsbadge an zwei Stellen), nutzen jetzt `totalMl`. Bei `units=1` (alle bisherigen 50 Angebote) ist `totalMl === sizeMl` — Ergebnis dort also bit-für-bit unverändert.
- **Zwei Vergleichsstellen um `units` ergänzt**, um eine reale Verwechslungsgefahr zu vermeiden: `findBestDealFor()` und `renderOtherStores()` (Preisverlauf-Sheet → „Andere Händler") verglichen bisher nur Produkt+Volumen — ohne den Zusatz hätte z.B. eine einzelne Dose für 0,89 € neben einem 24er-Pack für 16,99 € als "günstigerer Preis" erschienen, obwohl der 24er-Pack pro Liter tatsächlich günstiger ist. Jetzt werden nur Angebote mit identischer Packungsgröße verglichen.
- **Neue Anzeige-Hilfsfunktion `formatSize(deal)`**: zeigt bei Multipacks „N × Xml" (z.B. „6 × 250 ml") statt nur „Xml", eingesetzt auf der Angebotskarte, im Preisverlauf-Sheet (Untertitel) und im Händler-Detail-Sheet — überall dort, wo ein *konkretes Angebot* (nicht nur der Produktkatalog) angezeigt wird. Produktkatalog-Anzeigen ohne Bezug zu einem einzelnen Angebot (Produktdetail-Sheet, Ähnliche-Produkte, News-Karten, Such-Vorschläge) bewusst unverändert gelassen, da dort kein einzelnes Angebot/keine Packungsgröße gemeint ist.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v32` erhöht (Pflichtregel, da `deals.json` zu `APP_SHELL` gehört und sich inhaltlich geändert hat).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px): 60 statt 50 Angebote geladen; Literpreis/Gesamtpreis für alle vier neuen Packungsgrößen stichprobenartig von Hand nachgerechnet und korrekt (z.B. 24er-Pack 16,99 €/24×250ml → 2,83 €/L); Sortierung „Bester Preis pro Liter" mischt Einzeldosen und Multipacks jetzt korrekt nach echtem Literpreis; Filter-Chips (Klick, aktiver Zustand, Zähl-Badge, Zurücksetzen) über echte Klick-Events getestet; Preisverlauf-Sheet zeigt bei einem Multipack ohne weitere gleich große Angebote korrekt "Aktuell nur bei X im Angebot" statt fälschlich Einzeldosen als Alternative; Kartenlayout hält auch bei zweistelligen Preisen (z.B. „16,99 €") ohne Umbruch/Overflow; Karte/Favoriten/Alarme-Tab weiterhin fehlerfrei; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Multipack-Preise als Packungs-Gesamtpreis (nicht Preis pro Dose) modelliert, weil das der einzige Weg ist, mit dem "nach Packungsgröße filtern" tatsächlich sinnvolle/realistische Preise zeigt — eine reine Filter-Notiz ohne Preisanpassung hätte z.B. bei einem 24er-Pack einen absurd niedrigen "Dosenpreis" suggeriert.
- Bewusst 10 neue Angebote HINZUGEFÜGT statt bestehende umzuwandeln, damit alle 50 bisherigen Angebote (Preise, IDs, Inhalte) unangetastet bleiben.
- Preisalarme (`renderAlertsView()`, `Math.min` über alle Angebote eines Produkts) bewusst NICHT auf `units` umgestellt: Einzeldosen-Preise sind zahlenmäßig immer kleiner als Multipack-Gesamtpreise, wirken sich auf `Math.min` also ohnehin nie aus — Alarme bleiben dadurch unverändert auf Einzelpreise bezogen, was der bisherigen Bedeutung des selbst gesetzten Wunschpreises entspricht.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Suche: Schreibweisen-tolerant (Leerzeichen/Bindestrich-unabhängig), für alle Produkte generisch

**Umgesetzt:**
- Neue Hilfsfunktion `normalizeSearchText(str)` (klein schreiben + alle Leerzeichen/Bindestriche entfernen). Damit gelten z. B. „Redbull", „Red Bull" und „Red-Bull" als identisch, ebenso „Rockstar" und „Rock Star" — generisch für alle Produkte/Marken im Katalog, kein Wörterbuch mit fest hinterlegten Einzelfällen nötig, da bei jedem Vergleich beide Seiten (Sucheingabe UND Produkt-/Marken-/Händlername) gleich normalisiert werden.
- In `render()` (Haupt-Filterlogik der Angebotsliste, inkl. der Zähl-Logik für den „aber X Angebote deutschlandweit"-Hinweis bei 0 Treffern) sowie in `getSearchSuggestions()` (Autocomplete-Vorschläge aus der letzten Runde) auf `normalizeSearchText()` umgestellt statt nur `.toLowerCase()`.
- `highlightMatch()` (Hervorhebung des Treffers in den Vorschlägen) entsprechend angepasst: der Abgleich läuft normalisiert, die Hervorhebung erscheint aber weiterhin an der korrekten Stelle in der Original-Schreibweise (inkl. enthaltenem Leerzeichen) — z. B. markiert Eingabe „redbull" den kompletten Abschnitt „Red Bull" in einem Stück, nicht nur ein Teilwort.
- Betrifft Mobile und Webapp gleichermaßen (gemeinsame Logik, kein Media Query/keine Verzweigung).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v31` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile 375px + Webapp/Desktop 577px): „Redbull" → 5 Red-Bull-Vorschläge + 10 passende Angebote in der Liste; „Rock Star" → 2 Rockstar-Vorschläge + 3 passende Angebote; Hervorhebung zeigt korrekt „Red Bull"/„Rockstar" als zusammenhängenden Block; bereits vorher funktionierende Fälle (Händlersuche „kaufland", leere Suche → volle Liste + geschlossenes Dropdown) weiterhin unverändert korrekt. Keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Suchfeld: Textfarbe korrigiert + Live-Vorschläge (Autocomplete) für Mobile und Webapp

**Umgesetzt:**
- **Bug behoben:** `.search input{color:var(--text-primary)}` verwendete die themenabhängige Textfarbe. Die Suchbox selbst ist aber bewusst immer hell (`rgba(255,255,255,.96)`, unabhängig vom Theme). Im Dark Mode ist `--text-primary` fast weiß → eingegebener Text war praktisch unsichtbar (weißer Text auf weißem Feld). Fix: Eingabetext und Platzhalter nutzen jetzt die themenunabhängigen Palette-Variablen `var(--n700)` (dunkles Navy) bzw. `var(--n500)` (mittleres Grau) statt der Theme-Variablen — dadurch immer gut lesbar, unabhängig von Light/Dark Mode. Betrifft Mobile und Webapp gleichermaßen (eine gemeinsame Regel, kein Media Query).
- **Neu: Live-Suchvorschläge** unterhalb des Suchfelds, wie bei gängigen Shopping-/Preisvergleichsseiten: Bei jeder Eingabe (`input`-Event) werden bis zu 6 passende Produkte aus dem Produktkatalog gesucht (Treffer nach Produktname/Marke, `startsWith` vor `includes`, alphabetisch sortiert) und als Dropdown-Karte mit Bild, Name (Treffer farblich hervorgehoben via `<mark>`) und Marke/Menge angezeigt.
  - Klick auf einen Vorschlag übernimmt den vollen Produktnamen ins Suchfeld, schließt das Dropdown und filtert die bestehende Angebotsliste sofort auf dieses eine Produkt (nutzt die bereits vorhandene Such-/Filterlogik in `render()` — keine neue Navigation, kein neuer View, damit der eigentliche Zweck der App — Preisvergleich — im Fokus bleibt statt z. B. zu einem reinen Produktinfo-Sheet zu springen).
  - Kein Treffer → dezenter Hinweis „Keine passenden Produkte gefunden" statt leerem/verwirrendem Dropdown.
  - Dropdown schließt bei Klick außerhalb, bei Escape und nach Auswahl; öffnet sich erneut beim Fokussieren des Feldes, falls noch Text drinsteht. Wird zusätzlich an den beiden bestehenden Stellen geschlossen, an denen `search.value` programmatisch geleert wird (Filter zurücksetzen, „Start"-Tab).
  - Neue Funktionen `getSearchSuggestions()`, `highlightMatch()`, `renderSearchSuggestions()`, `hideSearchSuggestions()` sowie zugehörige Event-Listener ergänzt; bestehende Fav-/Card-/Sheet-Logik nicht verändert.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v30` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Mobile (375px) und Webapp/Desktop (577px), Light + Dark Mode — Eingabetext/Platzhalter in beiden Themes gut lesbar; Vorschläge korrekt sortiert/gefiltert (getestet u. a. mit „Re", „Mo", „Ca"); Klick auf Vorschlag füllt Suchfeld und filtert Liste korrekt (z. B. „Re" → Auswahl „Red Bull Blue Edition (Heidelbeere)" → Liste zeigt nur noch die 2 Angebote dieses Produkts); Schließen per Klick außerhalb und per Escape bestätigt; kein horizontaler Overflow; keine Konsolenfehler.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Mobile Feintuning: „BESTER PREIS"-Badge nicht mehr gestreckt, Herz ragt nicht mehr ins Produktbild

**Umgesetzt:**
- `.card-head .badge-best{margin-left:0;flex-basis:100%}` aus dem mobilen Media Query entfernt — diese Regel hat das Badge auf `flex-basis:100%` gezwungen, wodurch es (obwohl der Text „BESTER PREIS" nur einen Bruchteil davon braucht) die komplette Zeilenbreite eingenommen hat und wie ein langgezogener, leerer orangener Balken wirkte. Ohne diese Regel erbt Mobile jetzt dieselbe Basisregel wie die Webapp-Version (`display:inline-flex;margin-left:auto`): Badge ist nur noch so breit wie „BESTER PREIS" selbst und sitzt direkt neben dem Produktnamen (bzw. bricht bei langen Namen automatisch in eine eigene, aber nicht gestreckte Zeile um).
- Herz-Button auf dem Produktbild: Bild (`.card-top .thumb`) im mobilen Media Query von 64px auf 72px vergrößert (etwas Luft war noch vorhanden), gleichzeitig Herz-Button/-Icon nur für diesen mobilen Kontext (`.thumb-frame .btn.fav`, wirkt nicht auf andere `.btn.fav`-Vorkommen wie den Fav-Button im Preisverlauf-Sheet) von 44px/26px auf 36px/20px verkleinert und der Versatz von `-10px` auf `-7px` reduziert — dadurch sitzt das Herz jetzt sauber in der oberen rechten Bildecke, ohne spürbar auf das eigentliche Produktfoto zu ragen. Keine Logik/Funktion angefasst, nur Größen/Position per CSS.
- Webapp-/Desktop-Version (>540px) bewusst komplett unangetastet gelassen — beide Änderungen stehen ausschließlich im `@media(max-width:540px)`-Block; dort diente die Desktop-Darstellung nur als optische Orientierung („Badge sitzt kompakt neben dem Namen"), ohne dass an der Desktop-Regel selbst etwas geändert wurde.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v29` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server bei 375px (mehrere Karten, mit und ohne BESTER-PREIS-Badge, lange und kurze Produktnamen): kein horizontaler Overflow, „Zum Angebot"-Zeile weiterhin ohne Umbruch/Overflow trotz des größeren Produktbilds. Desktop (577px) per Screenshot bestätigt unverändert.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Mobile Kartenlayout an Referenz „mobil-neu" angeglichen (Zum-Angebot-Button neben Rabattzeile, Händler/Zeitraum nebeneinander)

**Umgesetzt:**
- Auf Wunsch des Nutzers (zwei Referenz-Screenshots „mobil-alt"/„mobil-neu") das Karten-Layout erneut angepasst — diesmal weg vom komplett gestapelten mobilen Layout aus der vorherigen Runde, hin zu einer Anordnung näher am ursprünglichen Zwei-Spalten-Gefühl, aber ohne das alte Grid-Modell wiederherzustellen:
  - `.actions` (der „Zum Angebot"-Button) aus `.card-bottom` herausgenommen und in einen neuen Wrapper `.price-cta-row` (Flex-Zeile, `justify-content:space-between`, `align-items:center`) direkt unter `.unit` in `.card-top-info` verschoben — er sitzt jetzt rechts neben `.price-cta-info` (Rabattzeile `.saving` + optional `.avg-context`), vertikal zentriert zu diesem Zwei-Zeilen-Block, statt wie zuvor ganz unten in eigener Zeile/Spalte.
  - `.card-bottom` enthält jetzt nur noch zwei Kinder (`.store` links, `.meta-row` rechts) und bleibt auf allen Breiten eine Flex-Zeile (`justify-content:space-between`, `align-items:flex-start`) — `.meta-row` (Zeitraum + „Preisverlauf") ist jetzt rechtsbündig (`align-items:flex-end`). Die bisherige `@media(max-width:540px){.card-bottom{flex-direction:column}}`-Regel (stapelte Händler/Zeitraum/Button untereinander) wurde entfernt, da das neue Referenzdesign explizit nebeneinander verlangt.
- Da bei ~375px Breite nur noch ~230px für Preistext+Button bzw. Händler+Meta zur Verfügung stehen (Produktbild+Gap nehmen den Rest), wurden auf schmalen Screens gezielt Schrift-, Icon- und Innenabstandsgrößen reduziert (Produktbild 74→64px, Preis 24→19px, Rabatt-Pill/„Zum Angebot"-Button/Preisverlauf-Pill kompakter), statt die Anordnung zu verändern — wie vom Nutzer explizit gefordert („intelligent skalieren statt umstrukturieren").
- **Bug beim Umsetzen gefunden und behoben:** die `@media(max-width:540px)`-Regel stand im Stylesheet vor den Basis-Regeln für `.saving`, `.price`, `.unit` etc. (weiter unten im „BUTTONS"-Abschnitt definiert) — bei gleicher Selektor-Spezifität gewinnt die später im Stylesheet stehende Regel, wodurch die Media-Query-Overrides für genau diese Properties trotz zutreffender Bedingung ignoriert wurden (nur die zufällig vor der Media Query stehenden Basis-Regeln, z. B. `.card-top .thumb`, griffen). Behoben, indem der gesamte Mobile-Media-Query-Block ans Ende des `<style>`-Blocks verschoben wurde (steht jetzt direkt vor `</style>`) — dadurch gewinnt er zuverlässig, unabhängig von der Position der jeweiligen Basis-Regel im Stylesheet.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v28` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server bei 375px (Start- und Favoriten-Tab, mehrere Karten inkl. Karten mit Badges-Zeile „Nur noch 2 Tage gültig" und ohne Ø-Preis-Vergleich): kein horizontaler Overflow, kein Textumbruch mitten im Wort (`white-space:nowrap` auf Preis/Rabatt-Pill/Button/Datum), Preisformat weiterhin korrekt mit Komma. Desktop/Tablet (>540px, App-Shell weiterhin bei 560px gedeckelt) ungestört geprüft — Button sitzt dort ebenfalls neben der Rabattzeile statt wie zuvor ganz unten, was angesichts der ohnehin auf ~560px gedeckelten App-Breite konsistent wirkt und nicht „kaputt".

**Wichtige Entscheidungen:**
- Bewusst kein Duplizieren von HTML-Templates für Mobile/Desktop — die neue Anordnung (Button neben Rabattzeile, Händler/Meta nebeneinander) gilt jetzt einheitlich für alle Breiten, da die `.app`-Shell ohnehin durchgehend auf 560px gedeckelt ist und „Desktop" hier de facto dieselbe schmale Kartenbreite wie ein Smartphone hat.
- „0,20 € unter Ø Preis (…)" darf bei langen Beträgen weiterhin auf zwei Zeilen umbrechen (kein `nowrap` erzwungen) — laut Vorgabe des Nutzers niedrigste Priorität („keine unnötigen Umbrüche" kommt nach Layout-Treue und Lesbarkeit), bewusst nicht durch noch kleinere Schrift erzwungen.

**Offen:**
- Exakte Pixelwerte (Abstände/Schriftgrößen) der Referenz „mobil-neu" konnten nur visuell abgeschätzt werden, nicht pixelgenau vermessen (Referenzbild ohne Metadaten/Maßangaben) — die Anordnung/Struktur wurde exakt übernommen, Feintuning einzelner Abstände ggf. nach Rückmeldung des Nutzers nötig.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Favoriten-Herz als Pin auf dem Produktbild (statt in der Kopfzeile neben dem Namen)

**Umgesetzt:**
- Herz-Button aus `.card-head` (der Zeile mit Produktname/Marke) herausgenommen und stattdessen direkt auf dem Produktbild platziert: neuer Wrapper `.thumb-frame` (`position:relative`) um `.thumb`, das Herz sitzt darin `position:absolute;top:-10px;right:-10px` — pinnt es an die obere rechte Ecke des Bildes, leicht überlappend, exakt wie in der vom Nutzer bereitgestellten Referenz. Dadurch steht der Produktname/Marke-Text jetzt direkt neben dem Bild, ohne dass das Herz optisch dazwischensteht.
- Herz-Icon bekommt zusätzlich `filter:drop-shadow(0 1px 3px rgba(0,0,0,.5))` (nur an dieser Stelle, scoped über `.thumb-frame .btn.fav`), damit die Outline-Variante (nicht favorisiert) auf unterschiedlich hellen/dunklen Produktbildern lesbar bleibt — weiterhin ohne sichtbaren Kasten/Hintergrund, nur ein dezenter Schlagschatten. Größe (44×44px Touch-Ziel, 26px Icon), Farblogik (Outline in Light/Dark, rot gefüllt wenn aktiv) und die Fav-Funktion selbst (Event-Delegation, `saveFavorites()`, Pop-Animation) komplett unverändert übernommen.
- `.card-head` dadurch jetzt nur noch Produktname-Spalte + optional BESTER-PREIS-Badge (`justify-content:space-between` statt `margin-left:auto` am Badge, da nur noch 2 statt 3 Elemente in der Zeile).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v27` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Desktop bei 541px (kritische untere Breakpoint-Grenze, weiterhin alle drei Elemente der unteren Zeile in einer Reihe), 577px; Mobile bei 320/375px (Herz sitzt genauso auf dem kleineren 74px-Bild); Light+Dark Mode. Kein horizontaler Overflow, keine Konsolenfehler. Funktional erneut vollständig durchgetestet: Favorisieren/Entfavorisieren (inkl. Pop-Animation) über das neu positionierte Herz, „Preisverlauf"-Sheet, „Zum Angebot" → Händler-Sheet, Produktdetail-Sheet (Klick aufs Bild — funktioniert weiterhin trotz des überlappenden Herz-Buttons, da beide als separate Elemente mit eigenen Klick-Handlern koexistieren), Fav-Button im Preisverlauf-Sheet (`#detailFavBtn`, unverändert). Alarme-Tab („Hinweise für dich"-Karten, nutzen weiterhin `.thumb-price`, nicht `.thumb-frame`) unverändert bestätigt.

**Wichtige Entscheidungen:**
- Herz per `position:absolute` auf das Bild gepinnt statt als Flex-Geschwister in der Kopfzeile — einzige Möglichkeit, gleichzeitig „Herz exakt wie im Bild platziert" UND „Text direkt neben dem Bild" zu erfüllen, da beide Vorgaben ein Element zwischen Bild und Text ausschließen.
- `drop-shadow`-Filter statt eines sichtbaren Hintergrund-Kastens hinter dem Herz gewählt, um die aus einer früheren Runde stammende Vorgabe „kein sichtbarer Kasten" zu respektieren, während die Lesbarkeit auf wechselnden Produktbild-Hintergründen trotzdem sichergestellt ist.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Angebotskarte komplett neu aufgebaut: oben Bild+Info, unten eine Händler-/Aktionszeile (nach Design-Vorlage)

**Umgesetzt:**
- Karten-Template grundlegend umstrukturiert, nach vom Nutzer bereitgestelltem Design-Bild ("neues design.png"): weg vom bisherigen Zwei-Spalten-Modell (`.card-columns` Grid mit unabhängiger linker/rechter Spalte, oft viel Leerraum unten rechts), hin zu einem einfacheren Aufbau mit nur zwei vertikal gestapelten Blöcken:
  - `.card-top` (Flex-Zeile): Produktbild links (jetzt 100px statt 58px, deutlich prominenter wie im Design) + `.card-top-info` rechts, darin `.card-head` (Herz → Produktname/Marke → optional BESTER-PREIS-Badge rechtsbündig per `margin-left:auto`) gefolgt von Preis, Literpreis/Gesamtpreis, Ersparnis, Ø-Preis-Vergleich und Badges-Zeile.
  - `.card-bottom` (Flex-Zeile, `justify-content:space-between`): Händlerblock (Logo+Name+Entfernung+Route/Status) — Zeitraum+Preisverlauf (Mitte) — „Zum Angebot" (rechts), alle in einer Zeile statt in einer eigenen, oft halbleeren rechten Spalte.
  - Vorher wurde per Klärungsfrage abgesichert, dass der Produktname (im ersten Entwurfsbild des Nutzers nicht sichtbar) trotzdem erhalten bleiben soll — der Nutzer hat daraufhin ein zweites, präziseres Referenzbild mit sichtbarem Produktnamen geliefert, das genau diese Struktur zeigt.
- **Mobile** dadurch stark vereinfacht: die bisherige `display:contents`+`order`-Trickserei (nötig für das alte Grid-Layout) entfällt komplett, da `.card-top` und `.card-bottom` als eigenständige Blöcke schon in der richtigen Reihenfolge im DOM stehen. Media Query (`max-width:540px`) macht nur noch: Produktbild zurück auf 74px, BESTER-PREIS-Badge bekommt `flex-basis:100%` (eigene Zeile statt neben Herz+Name zu quetschen) und `.card-bottom` wechselt von Zeile auf Spalte (Händler → Zeitraum/Preisverlauf → Zum Angebot untereinander) — Reihenfolge entspricht weiterhin der bereits zuvor bestätigten mobilen Priorität.
- Beim Testen an der kritischen unteren Breakpoint-Grenze (541–545px, direkt oberhalb der 540px-Mobile-Grenze) ist die neue `.card-bottom`-Zeile (Händler + Zeitraum/Preisverlauf + Zum-Angebot-Button nebeneinander) hauchdünn umgebrochen (Inhalt minimal breiter als verfügbarer Platz) — behoben durch Reduzieren von `gap:12px` auf `gap:6px`; ab da bei 541px, 545px, 560px programmatisch bestätigt: alle drei Elemente bleiben in einer Zeile, kein horizontaler Overflow.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v26` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Desktop bei 541/545/560/577px (Layout exakt wie im Referenzbild: großes Bild links, Herz+Name+BESTER-PREIS oben, Preis/Ersparnis darunter, Händler/Zeitraum/Preisverlauf/Zum-Angebot als eine kompakte untere Zeile ohne Leerraum), Mobile bei 320/375px (Bild+Herz+Name nebeneinander, BESTER PREIS eigene Zeile, Händler/Zeitraum/Preisverlauf/Zum Angebot untereinander, bereits zuvor bestätigte Reihenfolge unverändert), Light+Dark Mode beide geprüft. Funktional erneut bestätigt: Favorisieren/Entfavorisieren (inkl. Pop-Animation), „Preisverlauf"-Sheet (weiterhin ohne „Preis geprüft"), „Zum Angebot" → Händler-Sheet, sowie der Fav-Button im Preisverlauf-Sheet (`#detailFavBtn`, weiterhin unverändert 24px, von der neuen `.card-head .btn.fav .icon`-Regel nicht betroffen). Alarme-Tab („Hinweise für dich"-Karten, nutzen weiterhin unverändert `.thumb-price`) stichprobenartig unverändert bestätigt. Kein horizontaler Overflow und keine Konsolenfehler in allen getesteten Zuständen.

**Wichtige Entscheidungen:**
- Vor der Umsetzung gezielt nachgefragt, ob der im ersten Mockup fehlende Produktname wirklich entfernt werden soll, statt das stillschweigend zu übernehmen — Produktname/Marke sind zentrale Unterscheidungsmerkmale zwischen sehr ähnlich aussehenden Varianten (z. B. Red Bull Zero vs. Sugarfree) und ihr Wegfall wäre eine funktionale Verschlechterung, nicht nur eine optische Änderung gewesen. Der Nutzer hat daraufhin ein zweites Bild mit Produktname geliefert, das die Grundlage für die Umsetzung ist.
- Bei der Breakpoint-Kante bewusst den `gap` verkleinert statt z. B. Schriftgrößen oder Button-Padding zu ändern — kleinster, am wenigsten sichtbarer Eingriff, der das exakte Umbruchproblem behebt, ohne die vom Referenzbild vorgegebene Optik sonst zu verändern.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Angebotskarte neu strukturiert: links Produkt/Preis/Favorit, rechts Händler/Angebot

**Umgesetzt:**
- Karten-Template inhaltlich neu gruppiert (Desktop, `.card-columns` 60/40-Grid unverändert bei `1.5fr 1fr`): LINKS (`.card-product`) enthält jetzt Produktkopf (Bild, Name, Marke/Volumen, Favoriten-Herz), BESTER-PREIS-Badge, Preis/Pfand, Literpreis/Gesamtpreis, Ersparnis (alter Preis, %, Ø-Preis-Vergleich) und „Zum Angebot" — RECHTS (`.card-store`) enthält nur noch Händlerblock (Logo, Name, Entfernung, Route/Öffnungsstatus), Angebotszeitraum und „Preisverlauf". Vorher stand BESTER PREIS und die Aktionszeile (Herz+CTA) in der rechten Spalte; beides wandert jetzt vollständig auf die linke Produktseite.
- **Favoriten-Herz neu positioniert**: sitzt jetzt direkt rechts neben dem Produktnamen (`.product-name-row`, neue Wrapper-Klasse nur für die Angebotskarte) statt unten neben „Zum Angebot". Weiterhin kein sichtbarer Kasten (`.btn.fav` Basisstil unverändert: `background:none;border:none`), Touch-Ziel weiterhin 44×44px; Icon-Größe nur für diese Instanz per scoped Regel (`.product-name-row .btn.fav .icon{width:26px;height:26px}`) von 24px auf 26px angehoben ("etwas größer"), ohne den Favoriten-Button im Preisverlauf-Sheet (`#detailFavBtn`, teilt sich die Basisklasse `.btn.fav`) zu beeinflussen — dort weiterhin 24px, per Stichprobe erneut bestätigt unverändert. Fav-Funktion selbst (Event-Delegation über `closest("[data-fav]")`, `saveFavorites()`, `render()`, Pop-Animation) komplett unangetastet, da rein DOM-Positionsänderung.
- **BESTER PREIS neu positioniert**: Badge steht jetzt zwischen Produktkopf und Preis auf der linken Seite (`margin:10px 0 8px`), bleibt kompakt (`display:inline-flex`) statt wie ein Block auf volle Spaltenbreite zu strecken — dieser Stretch-Bug wurde beim ersten Testdurchlauf entdeckt (Grund: `.badge-best` hat selbst `display:flex`, was im normalen Blockfluss ohne Weiteres auf 100% Spaltenbreite expandiert) und mit der `inline-flex`-Regel behoben.
- **„Zum Angebot"** bleibt CTA, aber jetzt alleiniger Inhalt von `.actions` (Fav-Button wurde herausgenommen) und linksbündig statt rechtsbündig ausgerichtet (`.actions{justify-content:flex-start}`, einzige Verwendung dieser Klasse im Code, daher direkt an der Basisregel geändert); die vorherige schmale Kompakt-Größe (`.card-store .actions .btn.primary{padding:9px 10px;font-size:12px}`), die nur für die enge rechte Spalte nötig war, wurde entfernt — der Button nutzt jetzt die normale `.btn.primary`-Standardgröße, da er in der breiteren linken Spalte genug Platz hat.
- **Mobile Reihenfolge** (Produkt → BESTER PREIS → Preis → Ersparnis → Händler → Zeitraum → Preisverlauf → Zum Angebot) weiterhin über die bestehende `@media (max-width:540px)`-Grenze gelöst, aber mit neuer Technik: `.card-product`/`.card-store` bekommen `display:contents`, wodurch ihre direkten Kinder zu Grid-Items von `.card-columns` werden und sich per `order` frei in die gewünschte Reihenfolge bringen lassen, ohne das DOM zu duplizieren oder eine zweite Kartenvorlage zu pflegen. Dabei zweiten Stretch-Bug entdeckt und behoben: als direktes Grid-Item erbte `.badge-best` zusätzlich `justify-self:stretch` (Grid-Default) und wurde dadurch trotz `inline-flex` wieder volle Breite — behoben mit `.card-product .badge-best{justify-self:start}`, nur im Mobile-Media-Query.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v25` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Desktop bei 577px, 1280px (App-Shell deckelt ohnehin bei 560px) und an der bisher kritischen Grenze 545px — sauberes 60/40-Layout, Herz exakt neben Produktname (per `getBoundingClientRect()` programmatisch verifiziert: Fav-Button liegt am rechten Rand der linken Spalte, vertikal auf Höhe des Produktnamens), BESTER PREIS kompakt, „Zum Angebot" einzeilig und linksbündig, kein horizontaler Overflow, keine Konsolenfehler; Light+Dark Mode beide geprüft (Herz-Outline hell/dunkel wie zuvor, rot wenn favorisiert). Mobile bei 375px und 320px: einspaltige Reihenfolge exakt wie gefordert, BESTER PREIS nach Bugfix ebenfalls kompakt, kein Overflow. Funktional erneut bestätigt: Favorisieren/Entfavorisieren, „Preisverlauf"-Sheet (inkl. „Preis geprüft" weiterhin nicht vorhanden), „Zum Angebot" → Händler-Sheet, Produktdetail-Sheet (Klick aufs Bild) — alle unverändert funktionsfähig. Alarme-Tab („Hinweise für dich"-Karten, `.thumb-price` dort unverändert ohne `.product-name-row`) stichprobenartig unverändert bestätigt.

**Wichtige Entscheidungen:**
- Spaltenverhältnis `1.5fr 1fr` (60/40) unverändert gelassen statt auf z. B. `1.6fr 1fr` erhöht — liegt bereits am unteren Rand des gewünschten Bereichs „ca. 60–65% links", zusätzliche Änderung ohne konkreten Anlass hätte nur unnötiges Risiko für die Desktop-Optik bedeutet.
- Für die neue Mobile-Reihenfolge bewusst `display:contents` + `order` statt einer zweiten, dupliziertenKarten-Vorlage gewählt — hält die Datenlogik/JS unangetastet (ein einziges `card.innerHTML`-Template) und macht künftige Reihenfolge-Anpassungen zu reinen Ein-Zeilen-CSS-Änderungen.
- Beide beim Testen gefundenen „volle Breite"-Bugs bei `.badge-best` (Block-Stretch auf Desktop, Grid-Stretch auf Mobile) sind zwei unterschiedliche CSS-Mechanismen und wurden deshalb mit zwei getrennten, kontextspezifischen Regeln behoben (`display:inline-flex` global für die Karte, `justify-self:start` zusätzlich nur im Mobile-Media-Query).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Eigene, einspaltige Mobile-Anordnung der Angebotskarte (löst „immer nebeneinander" ab)

**Umgesetzt:**
- Nutzer hat das erzwungene Nebeneinander-Layout beim echten Testen auf dem Smartphone doch wieder als „zu stark gequetscht" empfunden und explizit eine eigene, für Mobile optimierte Anordnung angefordert — bei ausdrücklicher, mehrfach betonter Auflage, die Desktop-Darstellung dabei über reine Media-Query-Scoping vollständig unangetastet zu lassen.
- `@media (max-width:540px)`-Block wieder eingeführt (diesmal dauerhaft, nicht als Testzustand): schaltet `.card-columns` von `grid-template-columns:1.5fr 1fr` zurück auf `1fr` — auf Mobile stehen Produkt/Preis (bisherige linke Spalte) und Händler/Angebot (bisherige rechte Spalte) wieder untereinander, in genau der vom Nutzer vorgegebenen Reihenfolge (Produkt → Preis → Preisvorteil → Händler → Zeitraum → Preisverlauf → Aktionen), da diese Reihenfolge bereits so im HTML-Template steht und nur die Grid-Spalte umgebrochen wird.
- Zusätzliche mobile-only Feinschliffe im selben Media-Query-Block: Produktbild in der Angebotskarte von 58px auf 74px vergrößert (`.card-product .thumb`, nur mobil, nur in der Angebotskarte — Preisalarm-/Hinweise-Karten bleiben bei 58px); Aktionszeile von zusammengruppiert-rechtsbündig auf `justify-content:space-between` umgestellt, damit Herz links und „Zum Angebot" rechts mit sichtbarem Abstand auseinanderstehen; Header-Innenabstände reduziert (`.hero`, `.headline`, `.search`), damit Logo/Slogan/Suche auf Mobile weniger vertikalen Platz beanspruchen (Logo bleibt optisch größer als Slogan, da nur Innenabstand/Außenabstand verändert wurde, keine Schriftgrößen).
- „Preis geprüft" bewusst nicht wieder eingeführt — war bereits in einer früheren Runde komplett aus dem Karten-Template entfernt worden, bleibt es auch hier.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v24` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Mobile bei 320px und 375px (Light Mode) — einspaltige Reihenfolge wie gefordert, kein horizontaler Overflow (`scrollWidth === clientWidth` programmatisch bestätigt), Herz/„Zum Angebot" sichtbar auseinandergezogen, Marken-Chips/Filter-Sort-Leiste/Standortzeile/Liste-Karte-Umschalter/Bottom-Navigation weiterhin frei bedienbar und nicht von Karten/Toasts verdeckt. Desktop-Preset (≥560px) im Anschluss separat gegengeprüft: Screenshot zeigt unverändert das ursprüngliche zweispaltige Layout (BESTER-PREIS-Badge + Händlerblock rechts, Herz+„Zum Angebot" weiterhin gruppiert rechtsbündig statt auseinandergezogen), `bodyScrollWidth === bodyClientWidth` (kein Overflow), keine Konsolenfehler. Alarme-Tab (Preisalarm-/„Hinweise für dich"-Karten) stichprobenartig auf Mobile erneut geprüft — optisch unverändert, da `.card-product`/`.card-store` dort nicht verwendet werden.

**Wichtige Entscheidungen:**
- Löst die vorherige „immer nebeneinander, final bestätigt"-Entscheidung vom selben Tag ab. Grund: Der Nutzer hatte diese zunächst nach kurzem Test bestätigt, beim intensiveren realen Gebrauch auf dem Smartphone aber doch als zu gequetscht empfunden und wollte stattdessen eine für Mobile eigens optimierte, einspaltige Struktur — mit Desktop strikt über Media Query isoliert, damit frühere Entscheidungen zur Desktop-Ansicht (Zweispaltigkeit, gruppierte Aktionen) nicht angetastet werden.
- Die in der vorherigen Runde eingeführten Overflow-Fixes (`min-width:0` auf `.card-store .store`/`.meta-row`, `flex-wrap:wrap` auf `.store-sub`, `overflow-wrap:break-word` auf `.card-store`) bewusst nicht zurückgebaut — sie schaden nicht und die einspaltige Mobile-Breite hat ohnehin mehr Platz, sodass sie kaum noch greifen, aber als Absicherung für sehr lange Händlernamen sinnvoll bleiben.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Zweispaltige Angebotskarte auf Smartphone erzwungen (bestätigt, final)

**Umgesetzt:**
- Nutzer hat nach eigenem Test auf dem iPhone bestätigt: Die erzwungene Nebeneinander-Darstellung bleibt dauerhaft so — kein Testzustand mehr, sondern finales Verhalten.
- Die Media Query `@media (max-width:540px){.card-columns{grid-template-columns:1fr}}` entfernt — die Angebotskarte zeigt Produkt/Preis (links) und Händler/Angebot (rechts) jetzt **immer** nebeneinander, auch auf schmalen Smartphone-Bildschirmen.
- Beim Testen bei 375px echten horizontalen Overflow entdeckt (Karte war ~45px breiter als ihr Container) — verursacht durch `.store` und `.meta-row`, die als Flex-Kinder von `.card-store` standardmäßig `min-width:auto` hatten und sich dadurch nicht unter ihre Inhaltsbreite verkleinern ließen. Behoben mit gezielten, rein defensiven Ergänzungen (keine Größen-/Abstandsänderung): `min-width:0` auf `.card-store .store`, `.card-store .store > div`, `.meta-row` sowie `flex-wrap:wrap` + `min-width:0` auf `.store-sub` (nur hier verwendet, unbedenklich) und `overflow-wrap:break-word` auf `.card-store`. Bei 375–560px danach kein Overflow mehr (`scrollWidth === clientWidth`, mehrfach programmatisch verifiziert); bei sehr schmalen 320px (praktisch nur noch iPhone SE 1. Generation, seit 2018 nicht mehr verkauft) bleibt ein kleiner Rest-Überlauf bei sehr langen Händlernamen wie "Kaufland" bestehen — nicht weiter verfolgt, da das reale Zielgerät des Nutzers (aktuelles iPhone) bereits bei 375px sauber ist.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v23` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server bei 320/375/414/560px sowie Light+Dark Mode: Karte bei 375px zweispaltig ohne Überlappung/Abschneiden (Textzeilen brechen dafür etwas mehr um, u. a. "Zum Angebot" teils zweizeilig); Favorisieren und „Preisverlauf" erneut funktionsfähig bestätigt; Preisalarm-/Hinweise-Karten im Alarme-Tab optisch unverändert (nur `.store-sub`/`.meta-row` betroffen, beide exklusiv in der Angebotskarte verwendet); keine Konsolenfehler.

**Wichtige Entscheidungen:**
- War zunächst ein Testzustand zum Ausprobieren auf dem eigenen Smartphone; nach Rückmeldung „sieht gut aus" jetzt final bestätigt. Es wurde nur das Minimum geändert (Breakpoint entfernt + notwendige Overflow-Fixes), keine weiteren Anpassungen an Schriftgrößen/Abständen vorgenommen.

**Offen:**
- Keine offenen Rückfragen — Entscheidung gefallen, Nebeneinander-Layout bleibt.

**Bekannte Fehler / nächste Schritte:**
- Sehr schmale Geräte (~320px, z. B. iPhone SE 1. Gen.) zeigen noch einen kleinen Rest-Overflow bei langen Händlernamen — nur relevant, falls das Nebeneinander-Layout dauerhaft beibehalten wird und dieses spezielle, mittlerweile seltene Gerät unterstützt werden soll.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-03 — Produktbilder größer, Angebotspreis etwas kleiner

**Umgesetzt:**
- `.thumb` (gemeinsame Produktbild-Klasse) von `46px` auf `58px` vergrößert — bewusst an der Basis-Regel geändert statt nur für die Angebotskarte, da der Nutzer „allgemein" größere Produktbilder wollte: wirkt dadurch einheitlich überall dort, wo `.thumb` bereits verwendet wird (Angebotskarte, Preisalarm-Karte im Alarme-Tab, „Hinweise für dich"-Karten).
- `.card .price` (Angebotspreis, nur in der Angebotskarte) von `28px` auf `24px` reduziert — die auf die Karte beschränkte Zusatzregel aus der vorherigen Runde blieb bestehen, nur der Wert wurde angepasst; die Basis-Regel `.price{}` (u. a. von der Händler-Detailansicht genutzt) weiterhin unangetastet bei `25px`.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v22` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile + Desktop, Light + Dark Mode): Produktbilder in Angebotskarte, Preisalarm-Karte und „Hinweise für dich"-Karten sichtbar größer, keine Überlappung/kein Umbruch-Problem; Preis in der Angebotskarte spürbar kleiner, bleibt aber weiterhin klar prominent; Favorisieren-Funktion stichprobenartig erneut bestätigt; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- `.thumb`-Basis-Regel bewusst direkt geändert (nicht wie sonst über eine kartenspezifische Zusatzregel) — einzige sinnvolle Umsetzung von „allgemein größer", ohne die Größe an drei Stellen einzeln duplizieren zu müssen. Vorab geprüft, dass alle drei Verwendungsorte (Angebotskarte, Preisalarm-Karte, Hinweise-Karten) die größere Größe unfallfrei vertragen.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Angebotskarte: Zweispaltiges Layout (Produkt/Preis links, Händler/Angebot rechts)

**Umgesetzt:**
- Karten-Template komplett neu strukturiert: `.card-columns` (CSS Grid, `1.5fr 1fr`, `gap:16px`) teilt die Karte in `.card-product` (links: Bild+Name/Marke, Preis groß, Pfand/Literpreis/Gesamtpreis, durchgestrichener Alt-Preis+Ersparnis-%, Ø-Preis-Vergleich, sonstige Badges) und `.card-store` (rechts: BESTER-PREIS-Badge, Händlerlogo+Name+Distanz, Route/Öffnungsstatus, Zeitraum, Preisverlauf-Button, Herz+„Zum Angebot" als Aktionszeile). Keine Trennlinie — Trennung ausschließlich über Spalten/Abstand, wie gefordert.
- **Responsive**: Unter 540px Viewportbreite wechselt `.card-columns` per Media Query auf eine Spalte (rechte Seite rutscht vollständig unter die linke) — bei den meisten echten Smartphone-Breiten (≤430px) ohnehin schon der Fall. Breakpoint bewusst höher als ursprünglich geplant (480px) gesetzt, nachdem sich im Test zeigte, dass der „Zum Angebot"-Button im schmalen Bereich 480–540px sonst zweizeilig umgebrochen wäre; ab 545px passt er nachweislich einzeilig.
- `.meta-row` (Zeitraum + Preisverlauf) von nebeneinander (`justify-content:space-between`) auf untereinander (`flex-direction:column`) umgestellt — passt zur schmaleren rechten Spalte; Klasse wird ausschließlich hier verwendet (geprüft), keine Seiteneffekte.
- `.badge-best` bekam `align-self:flex-start`, da er als direktes Kind der neuen Flex-Spalte `.card-store` sonst auf volle Spaltenbreite gestreckt worden wäre (im Test entdeckt und behoben) — Farbe/Position/Funktion sonst unverändert (Verkleinerung war bereits aus der vorherigen Runde vorhanden).
- Kleiner, auf die Angebotskarte beschränkter Override `.card-store .actions .btn.primary{padding:9px 10px;font-size:12px}`, um „Zum Angebot" im schmaleren rechten Bereich zuverlässig einzeilig darzustellen — Basis-`.btn.primary` (an vielen anderen Stellen der App verwendet) bewusst unangetastet.
- Das bisherige `<div class="trust-line">…Preis geprüft…</div>` wurde diesmal komplett aus dem Karten-Template entfernt (nicht nur per CSS versteckt wie zuvor) — passend zur diesmal expliziten Formulierung „vollständig entfernen". Die zugrunde liegenden Daten (`deal.checkedMinutesAgo`, `timeAgoLabel()`) bleiben unangetastet; die gleichnamige `.trust-line`-Klasse bleibt unverändert `display:none` und wird weiterhin von `#detailTrust` im Preisverlauf-Sheet verwendet (davon unberührt).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v21` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server bei 320px, 375px, 414px (gestapelt), 500–540px (gestapelt, Grenzbereich), 545px, 560px (nebeneinander) sowie Light+Dark Mode: zweispaltiges Layout wie in der Skizze, Preis links klar dominant, Händlerinfos/Zeitraum/Preisverlauf/Aktionen rechts sauber gruppiert, „BESTER PREIS" kompakt (nicht mehr gestreckt), „Zum Angebot" durchgehend einzeilig; alle Interaktionen erneut einzeln nachgetestet (Favorisieren, „Zum Angebot" → Händlerdetail, „Preisverlauf", Klick auf Produktbild → Produktdetail) funktionieren unverändert; Preisalarm-Karte im Alarme-Tab optisch komplett unverändert (erneut per Stichprobe geprüft, u. a. weil `.badge-best`/`.meta-row` diesmal mitgeändert wurden); „Preis geprüft" nirgends mehr vorhanden; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- `.thumb-price` (Bild+Name-Kopfzeile) unverändert weiterverwendet, jetzt aber innerhalb von `.card-product` statt neben dem Aktionsbereich — spart eine neue Klasse, da sich am eigentlichen Flex-Verhalten (Bild links, Text rechts, zentriert) nichts ändern musste.
- Responsive Breakpoint anhand tatsächlichem Testergebnis (nicht Bauchgefühl) auf 540px statt der ursprünglich angedachten 480px gesetzt — vermeidet zweizeiligen Button-Text im kritischen Zwischenbereich, ohne die Seitenverhältnis-Vorgabe (60–65 % / 35–40 %) zu verletzen.
- Bei jeder wiederverwendeten, aber auch anderswo genutzten Klasse (`.badge-best`, `.meta-row`, `.btn.primary`, `.trust-line`) einzeln geprüft, ob sie exklusiv für die Angebotskarte ist, bevor sie verändert wurde — Muster aus den vorherigen beiden Karten-Layout-Runden konsequent fortgeführt.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Angebotskarte: Preis/Aktionen getrennt, Preis prominenter

**Umgesetzt:**
- Analyse ergab: Kopfbereich (Produktname/Marke/BESTER-PREIS), Händlerbereich (Logo/Name/Distanz/Route/Status) und Zeitraum+Preisverlauf-Zeile entsprachen inhaltlich/strukturell bereits exakt der gewünschten Zielgliederung — dort nur Feinschliff (siehe unten), keine Strukturänderung nötig.
- **Größte Änderung (Abschnitt 5/8 der Anfrage)**: Preisbereich und Aktionsbereich (Herz + „Zum Angebot") teilten sich bisher eine Zeile (`.bottom{justify-content:space-between;align-items:end}`), wodurch die Aktionen unten rechts neben einem bis zu 5-zeiligen Preistext "klebten". `.bottom` ist jetzt `flex-direction:column` mit Abstand (`gap:12px`) — Preisblock (Bild+Preis/Pfand/Literpreis/Ersparnis/Ø-Vergleich) und Aktionsblock stehen jetzt klar als zwei getrennte, übereinanderliegende Gruppen. `.actions` bekam `justify-content:flex-end`, damit Herz+Button weiterhin (wie zuvor) rechtsbündig zusammenstehen, jetzt aber auf eigener, voller Zeilenbreite statt seitlich gequetscht.
- **Preis deutlich größer**: `.card .price` (neue, auf die Angebotskarte beschränkte Regel) von 25px auf 28px — bewusst NICHT die Basis-Regel `.price{}` direkt geändert, da diese auch von `#storeDetailPrice` (Händler-Detailansicht, außerhalb des angefragten Bereichs) verwendet wird und dort unverändert bleiben sollte.
- **„BESTER PREIS"-Badge dezenter**: Padding `7px 9px`→`5px 8px`, Schrift `10px`→`9px`, zusätzlich `opacity:.9` (volle Deckkraft erst bei Hover) — bleibt in Farbe/Position/Funktion unverändert, tritt aber weniger stark gegen den jetzt größeren Preis an.
- `.thumb-price` (Bild+Preistext-Zeile) bewusst NICHT verändert — dieselbe Klasse wird auch von der Preisalarm-Karte im Alarme-Tab verwendet (`.alert-card .thumb-price`), die laut Aufgabe unangetastet bleiben soll; die gewünschte Trennung wurde ausschließlich über den umgebenden `.bottom`/`.actions`-Umbau erreicht, ohne diese gemeinsame Klasse anzufassen.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v20` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile, Desktop, Light + Dark Mode): Preisblock und Aktionsblock jetzt klar getrennte Zeilen, Preis deutlich prominenter, BESTER-PREIS-Badge dezenter; alle Interaktionen einzeln nachgetestet — Favorisieren/Entfavorisieren, „Zum Angebot" (öffnet Händlerdetail), „Preisverlauf", Klick auf Produktbild (öffnet Produktdetail) — funktionieren alle unverändert; Preisalarm-Karte im Alarme-Tab optisch komplett unverändert (Stichprobe durchgeführt); keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Scoping-Prinzip konsequent angewendet: Wo eine CSS-Klasse (`.price`, `.thumb-price`) auch außerhalb der Angebotskarte verwendet wird, wurde entweder eine spezifischere Zusatzregel ergänzt (`.card .price`) oder die Klasse komplett unangetastet gelassen, statt sie direkt zu verändern — verhindert unbeabsichtigte Layoutänderungen an der Händler-Detailansicht bzw. den Preisalarm-Karten.
- Kopf-/Händler-/Zeitraum-Bereiche nicht neu gruppiert, da deren bestehende Struktur bereits der gewünschten Zielgliederung entsprach — nur dort geändert, wo tatsächlich eine Abweichung zur Anfrage bestand (Preis-/Aktionstrennung, Preisgröße, Badge-Gewicht).

**Offen:**
- Rückfrage an Nutzer: Punkt 9 der Anfrage beschreibt die „Preis geprüft"-Fußzeile als weiterhin sichtbar/dezent vorhanden — sie ist aber seit der vorherigen, expliziten Anfrage bewusst per `display:none` ausgeblendet (nicht rückgängig gemacht, da diese Anweisung neuer und eindeutig war). Rückmeldung nötig, ob sie im Rahmen dieses Layout-Umbaus wieder eingeblendet werden soll.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — "Preis geprüft" ausgeblendet, Produkte in Alarme anklickbar

**Umgesetzt:**
- **"Preis geprüft vor ..." ausgeblendet**: `.trust-line{display:flex}` → `display:none`. Betrifft beide Stellen, die diese Klasse nutzen (Angebotskarte + `#detailTrust` im Preisverlauf-Sheet), ohne deren HTML/JS anzufassen — `checkedMinutesAgo`/`timeAgoLabel()` und das Befüllen der Zeile laufen unverändert im Hintergrund weiter (51 `.trust-line`-Elemente existieren weiterhin im DOM, sind nur nicht sichtbar), damit die Funktion laut Nutzerwunsch später ohne Aufwand wieder eingeblendet werden kann (einfach `display` zurück auf `flex`).
- **Produkte im "Alarme"-Tab anklickbar**: Sowohl die "Hinweise für dich"-Karten (`newsCardHtml()`, alle drei Varianten: aktuell/demnächst/nicht verfügbar) als auch die bestehenden Preisalarm-Karten (`.alert-card`) haben jetzt `data-product-detail="{productId}"` auf Produktbild UND Produktname. Ein neuer Zweig ganz oben im bestehenden `alertsView`-Klick-Delegate ruft bei Treffer `openProductDetail(productId)` auf — identisch zur Funktion, die auf der Startseite beim Klick auf das Produktfoto bereits existiert. Bestehende Buttons in denselben Karten ("Angebote ansehen", "Löschen", Enable/Disable-Toggle) unverändert und weiterhin einzeln funktionsfähig geprüft.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v19` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: "Preis geprüft" nirgends mehr sichtbar; Klick auf Produktbild UND auf Produktname öffnet in allen drei "Hinweise für dich"-Kartentypen sowie in der Preisalarm-Karte zuverlässig die Produktdetailansicht (Nährwerte/ähnliche Produkte) mit korrektem Titel; "Angebote ansehen" (öffnet weiterhin Preisverlauf) und "Löschen" (entfernt weiterhin den Preisalarm) unverändert funktionsfähig; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Bei der ersten Umsetzung wurde `data-product-detail` per `replace_all` nur auf 2 von 3 `newsCardHtml()`-Varianten angewendet, weil die dritte (Standardfall "Jetzt im Angebot") aus einer anderen Einrückungstiefe stammt und dadurch nicht exakt beim automatischen Ersetzen traf — im Test bemerkt (Klick auf Namen bei einem "Jetzt im Angebot"-Eintrag reagierte nicht) und gezielt nachgezogen; alle drei Varianten jetzt einzeln verifiziert.
- Ausblenden statt Löschen für "Preis geprüft" gewählt, exakt wie vom Nutzer verlangt ("wird ggf. später wieder eingepflegt") — keine Logik/Daten entfernt, nur die visuelle Ausgabe unterdrückt.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Toast-System: oben mit Safe-Area statt unten über der Navigation

**Umgesetzt:**
- Analyse ergab: Das zentrale Toast-System existierte bereits vollständig — `showToast(msg)` ist die **einzige** Stelle, die je eine Kurzmeldung anzeigt (22 Aufrufstellen: Favoriten, Preisalarm, Profil, Standort, Teilen, Konto-Demo-Hinweise, erreichte Preisalarme etc.), alle über dasselbe `#toast`-Element. Punkt 9 ("zentrale Lösung für alle Meldungen") war damit bereits erfüllt — hier ausschließlich CSS/Timing der bestehenden Komponente überarbeitet, keine der 22 Aufrufstellen verändert.
- **Position von unten nach oben verschoben**: `.toast` stand bisher bei `bottom:24px` (kollidierte optisch mit der Bottom-Navigation). Jetzt `top:calc(env(safe-area-inset-top, 0px) + 14px)` — verwendet die tatsächliche Geräte-Safe-Area (Dynamic Island/Notch/Statusleiste) plus 14px zusätzlichen visuellen Abstand, mit `0px`-Fallback für Geräte/Browser ohne Safe-Area-Konzept. Bleibt weiterhin `position:fixed`, damit die Meldung unabhängig von der Scrollposition sichtbar ist (mit `scrollTo`-Test bei 1500px Scroll-Offset verifiziert: Toast bleibt oben im sichtbaren Viewport).
- **Animation** von "Einblenden von unten" (`translateY(20px)→0`) auf "Einblenden von oben" (`translateY(-10px)→0`) gedreht, passend zur neuen Position; Transition-Dauer leicht beschleunigt (0,25s → 0,22s) für ein knackigeres Gefühl.
- **Breite**: `max-width` von `80%` (auf Desktop potenziell sehr breit) auf `min(360px, calc(100% - 32px))` geändert — kompakt auf Desktop, mit garantiertem 16px-Seitenabstand auf schmalen Displays; Text bricht bei Bedarf mehrzeilig um (`line-height:1.4` ergänzt für saubere Lesbarkeit bei Zeilenumbruch).
- **Anzeigedauer dynamisch**: `showToast()` berechnet jetzt `Math.min(2500, Math.max(1500, 1500 + msg.length * 12))` statt einer festen 2400ms für alle Meldungen — kurze Texte (~20 Zeichen) landen bei ~1,7s, lange Texte (~75+ Zeichen) werden bei 2,5s gedeckelt.
- `pointer-events:none` (verhindert Blockieren von Klicks/Touches auf darunterliegende Elemente wie die Bottom-Navigation) war bereits vorhanden und wurde unverändert beibehalten — genau wie das bestehende Stapel-Verhalten (`clearTimeout` + Text-/Timer-Reset statt neuer DOM-Elemente), das mehrere schnelle Meldungen bereits korrekt nacheinander/ersetzend statt überlappend behandelt.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v18` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server (Mobile + Desktop, Light + Dark): Toast erscheint zuverlässig oben im sichtbaren Bereich, auch nach Scrollen um 1500px; Klick auf Bottom-Nav-Items funktioniert sofort auch während eine Meldung sichtbar ist (`navFav` wurde correctly aktiv, während `toast.show` noch `true` war); mehrere schnelle `showToast()`-Aufrufe hintereinander erzeugen nur ein einziges `.toast`-Element mit ersetztem Text, kein Stapeln; Anzeigedauer skaliert nachweislich mit der Textlänge (kurze/lange Meldung unterschiedlich lang sichtbar); guter Kontrast in Light UND Dark Mode (Toast sitzt auf dem durchgehend dunklen Header-Hintergrund, der in beiden Themes gleich bleibt); keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Keine der 22 `showToast(...)`-Aufrufstellen angefasst — die zentrale Architektur war bereits korrekt, es fehlte nur an Positionierung/Timing der gemeinsamen Komponente selbst.
- Hintergrundfarbe/Textfarbe des Toasts bewusst unverändert gelassen (`var(--brand-primary-dark)` + Weiß) — dieselbe feste dunkle Farbe wie der App-Header in beiden Themes, dadurch automatisch gutem Kontrast in Light und Dark Mode, ohne neue Farb-Logik einführen zu müssen.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Produktbild-Hintergrund im Dark Mode auf #12151C

**Umgesetzt:**
- Alle drei Stellen, an denen ein Produktbild in einem eigenen Hintergrund-Container sitzt, nutzten bisher `background:var(--bg-surface-sunken)` (Dark-Mode-Wert `#080a0d`, wirkt nahezu schwarz): `.thumb` (Produktbild in Angebotskarten, Alarm-/Neuigkeiten-Karten, Favoriten-Zeilen — überall dieselbe Klasse), `.detail-hero` (großes Produktbild im Preisverlauf- und Produktdetail-Sheet) und `.fav-row .fav-emoji` (Produktbild in den Favoritenlisten von Profil und Push-Einstellungen).
- `--bg-surface-sunken` selbst bewusst **nicht** angefasst, da diese Variable an 27 weiteren, produktbild-fremden Stellen verwendet wird (u. a. Status-Boxen, Chart-Hintergrund, Store-Detail-Logo) — eine Änderung dort hätte weit über die Produktbilder hinausgewirkt. Stattdessen drei gezielte, auf Dark Mode beschränkte Zusatzregeln ergänzt: `:root[data-theme="dark"] .thumb`, `:root[data-theme="dark"] .detail-hero`, `:root[data-theme="dark"] .fav-row .fav-emoji`, jeweils direkt `background:#12151C`. Light Mode (`:root` ohne `data-theme="dark"`) bleibt dadurch unverändert bei `var(--n100)`.
- Store-/Händler-Logos (`.store-logo`, `.store-row-logo`, `.store-detail-logo`, Kartenpins) bewusst nicht verändert — der Nutzer bezog sich ausdrücklich auf Produktbilder, nicht auf Händlerlogos.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v17` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server: Im Dark Mode zeigen `.thumb` (Kartenliste), `.detail-hero` (Preisverlauf-Sheet) und `.fav-emoji` (Profil-Favoritenliste) übereinstimmend `rgb(18, 21, 28)` (= `#12151C`); im Light Mode unverändert `rgb(239, 239, 242)` (alter Wert); keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Drei einzelne, eng zielgerichtete Selektor-Overrides statt einer Änderung an der gemeinsamen Variable — minimiert das Risiko, versehentlich andere (nicht produktbildbezogene) UI-Bereiche im Dark Mode mitzufärben.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Handle-Tap überall, Herz-Glow entfernt, Händler A–Z, zwei Rockstar-Bilder ersetzt

**Umgesetzt:**
- **Handle-Tap zum Schließen jetzt einheitlich für alle Sheets**: `initHandleTapToClose()` (bisher nur für `histOverlay`/`productDetailOverlay`) wird jetzt für alle neun Overlays mit `.handle` aufgerufen: `locOverlay`, `notifOverlay`, `newsOverlay`, `filterOverlay`, `sortOverlay`, `profileOverlay`, `histOverlay`, `productDetailOverlay`, `storeDetailOverlay`. Tippen auf den Balken ruft überall dasselbe, bereits bestehende `closeSheet(id)` auf — keine sonstige Funktion innerhalb dieser Sheets (Formulare, Toggles, Sub-Buttons) wurde angefasst. Wischgeste bleibt bewusst weiterhin nur bei den beiden Detail-Sheets (war nicht Teil dieser Anfrage).
- **"Schein" am favorisierten Herz entfernt**: `filter:drop-shadow(...)`-Zeile aus `.btn.fav.active .icon` gestrichen. Die Pop-Animation (`@keyframes fav-pop`, `triggerFavPop()`) ist komplett unverändert und funktioniert weiterhin identisch — nur der Leucht-/Glow-Effekt um das gefüllte Herz ist weg, Farbe (kräftiges Rot) bleibt.
- **Händler-Filterchips alphabetisch sortiert**: "Alle" bleibt als erste (Sonder-)Option, danach EDEKA, Kaufland, Lidl, Netto, Penny, REWE (vorher: Kaufland, Lidl, EDEKA, Penny, REWE, Netto). Reine Markup-Reihenfolge geändert, keine Logik.
- **Rockstar Energy Original – neues Bild**: Altes Bild (`rockstarenergy.com`-CDN) war bereits in der Originaldatei so beschnitten, dass der Schriftzug links/rechts abgeschnitten war (verifiziert auch in der unverkleinerten Originalauflösung). `rockstarenergy.de` (offizielle DACH-Seite) ist durch Bot-Schutz (Incapsula) nicht abrufbar. Stattdessen ein aktuelles, professionelles "Packshot"-Bild der 500-ml-Dose von der offiziellen UK-Website (`rockstarenergy.co.uk`, gleiche 500-ml-EU-Dosenform wie in Deutschland verkauft) verwendet: `rockstar_packshot_500ml_Front_Original_NoRTB_157x420px.png`.
- **Rockstar Punched Guava → "Rockstar Energy Punched Guava" + neues Bild**: Name in `deals.json` angepasst. Bisheriges Bild (openfoodfacts.org, Nutzer-Foto) durch dasselbe professionelle Packshot-Format von `rockstarenergy.co.uk` ersetzt (`rockstar_packshot_500ml_Front_TropicalGuava_NoRTB_157x420px.png`) — das aktuelle Verpackungsdesign dieser Guave-Sorte trägt dort die Aufschrift "Tropical Guava" statt "Punched" (bereits auf dem alten, ersetzten Foto so beschriftet — keine neue Diskrepanz, nur bessere Bildqualität). "Punched" existiert bei Rockstar aktuell nur noch als eigene US-Produktlinie mit anderen Sorten (u. a. Lime Freeze), nicht mehr als Guave-Geschmack.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v16` erhöht (Pflichtregel, `index.html` und `deals.json` geändert).
- Verifiziert über einen temporären lokalen Server: alle 9 Overlays schließen per Handle-Tap zuverlässig (systematisch durchgetestet); Herz-Glow weg (`filter: none`), Pop-Animation weiterhin aktiv; Händler-Chips in korrekter A–Z-Reihenfolge; beide neuen Rockstar-Bilder laden erfolgreich (200, korrekte Maße 157×420) und werden in Karte, Produktdetail sowie "Ähnliche Produkte" korrekt und scharf angezeigt; neuer Produktname überall (Karte, Produktdetail-Titel, ähnliche Produkte) konsistent; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Für beide Rockstar-Bilder bewusst die offizielle UK-Website statt eines Drittanbieters/Stockfoto-Portals gewählt, um im Rahmen der bestehenden "nur offizielle Hersteller-/Händler-Quellen"-Konvention zu bleiben und das vom Nutzer explizit genannte rechtliche Risiko zu vermeiden — UK statt DE, weil die offizielle DACH-Seite technisch (Bot-Schutz) nicht erreichbar war, das Dosenformat (500 ml, EU-Design) aber identisch zum deutschen Markt ist.
- Wischgeste bewusst NICHT auf weitere Sheets ausgeweitet — die Anfrage bezog sich explizit nur auf den Handle-Tap ("so wie z. B. bei dem Preisverlauf" bezog sich erkennbar auf das Antippen, nicht auf das Wischen).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Sortierung/Filter: zwei eigenständige Pills statt einer gemeinsamen Gruppe

**Umgesetzt:**
- Vorherige gemeinsame `.toolbar-group`-Pille (Sortierung + Trenner + reines Filter-Icon) durch zwei eigenständige, klar unterscheidbare Pills ersetzt: `#sortBtn` ("Günstigster Preis ⌄") und `#filterBtn` ("Filter" + Trichter-Icon + Zähler-Badge), beide mit dezentem Rahmen/Hintergrund im bestehenden `.chip`-Look (`var(--bg-surface)`, `var(--border-subtle)`, `radius-pill`), nebeneinander in `.toolbar-actions` rechts ausgerichtet (`justify-content:space-between` auf `.controls`, kein `margin-left:auto`-Hack mehr nötig). „Filter" zeigt jetzt erstmals ein Text-Label (vorher nur ein reines Icon ohne Beschriftung).
- Trennpunkt „·" zwischen „50 Angebote" und den Pills entfernt — passend zur neuen links/rechts-Aufteilung ohne verbindendes Satzzeichen (Layout jetzt „50 Angebote“ ... „Günstigster Preis ⌄“ „Filter ⚱“, exakt wie vom Nutzer vorgegeben).
- „50 Angebote" (`.toolbar-count`) von `12.5px/700` auf `14.5px/600` angepasst — laut Vorgabe "ca. 14–15px, gut lesbar aber dezent".
- Aktive-Filter-Anzeige unverändert über das bereits bestehende `#filterCount`/`updateFilterCount()` gelöst (kein neuer Zählmechanismus nötig) — Badge sitzt jetzt oben rechts an der breiteren "Filter"-Pille statt am kleinen quadratischen Icon-Button.
- Nicht mehr benötigte `.toolbar-group`/`.toolbar-divider`-Regeln entfernt (nur noch für diese eine, jetzt ersetzte Struktur verwendet).
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v15` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server auf 320px, Desktop und mit aktiven Filtern: beide Pills öffnen weiterhin ihr jeweiliges bestehendes Sheet (Sortierung/Filter unverändert); Filter-Badge zeigt korrekt "2" bei zwei aktiven Filtern (Pfand-frei + Volumen) und verschwindet nach Zurücksetzen; Trefferzahl links aktualisiert sich dynamisch (0 bei den Testfiltern, 50 nach Reset); bei sehr langem Sortier-Label bricht die Zeile auf 320px sauber um (Count auf eigener Zeile, Pills darunter zusammen), nichts abgeschnitten/überlappend; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Für die Anzeige aktiver Filter bewusst beim bereits vorhandenen, funktionierenden Badge-Mechanismus geblieben (kleine Zahl oben rechts an der Pille) statt einer neuen "Filter · 2"-Text-Variante — beide waren laut Aufgabenstellung zulässig, die bestehende Lösung war Zero-Risk, da weder `updateFilterCount()` noch die Zählweise selbst angefasst werden mussten.
- Beide Pills bewusst optisch identisch behandelt (gleicher Rahmen/Hintergrund/Radius/Padding) für ein einheitliches, ruhiges Erscheinungsbild, wie es der Nutzer als "modernes Shopping-App-Gefühl" beschrieben hat.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Feintuning: Sortierung/Filter als eine Gruppe, engere Abstände

**Umgesetzt:**
- **Sortierung + Filter visuell zusammengefasst**: Neuer `.toolbar-group`-Container (dezente `--bg-surface-sunken`-Füllung, `radius-pill`, dünner 1px-Rahmen) umschließt jetzt `#sortBtn` und `#filterBtn` gemeinsam, mit `margin-left:auto` an den rechten Rand der Toolbar-Zeile gerückt; ein schmaler `.toolbar-divider` (1px-Linie) trennt Sortierlabel und Filter-Icon innerhalb der Gruppe optisch dezent voneinander. Das Filter-Icon "schwebt" dadurch nicht mehr allein am Rand, sondern wirkt als Teil einer einzigen kleinen Steuerungseinheit. `#filterBtn` dabei von 30px auf 34px Klick-/Touch-Fläche leicht vergrößert (weiterhin unsichtbarer Hintergrund/kein Rahmen am Button selbst — nur die gemeinsame Gruppe hat die dezente Füllung). „50 Angebote" bleibt links außerhalb der Gruppe, durch „·" getrennt, gut lesbar.
- **Vertikale Abstände reduziert**: `.controls`-Padding von `8px…4px` auf `6px…3px`, `.status`-Außenabstand von `6px 0` auf `4px 0`, `.view-toggle`-Abstand von `2px 0 4px` auf `2px 0 3px` — die vier Bereiche (Marken-Tabs, Toolbar, Standort, Liste/Karte) rücken dadurch dezent näher zusammen, bleiben aber klar als eigene Zeilen erkennbar.
- **Liste/Karte ~14 % kompakter**: `.view-btn`-Padding von `7px` auf `6px` reduziert (Ausgangswert vor der vorherigen Änderung war `10px` — Nutzer bezog sich in dieser Runde auf eine weitere leichte Reduktion des bereits kompakteren Zwischenstands). Aktiver blauer Zustand, Farben und Funktion unverändert.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v14` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server auf 320px, 375px und Desktop-Breite: Gruppierung wirkt visuell zusammenhängend, kein "loses" Filter-Icon mehr; bei sehr langem Sortier-Label ("Günstigster €/Liter") bricht auf 320px nur die Gruppe sauber in eine zweite Zeile um (kein Abschneiden/Überlappen); Sortier- und Filter-Sheet öffnen weiterhin korrekt, Filter-Badge erscheint weiterhin korrekt in der Gruppe; erste Produktkarte erscheint noch etwas früher im Viewport als zuvor; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Bewusst nur eine KLEINE, eng am Inhalt anliegende Pille um Sortierung+Filter gelegt (kein flächiger Kasten über die ganze Zeile) — erfüllt den expliziten Wunsch nach visueller Gruppierung, ohne zur ursprünglichen großen Filter-Zeile zurückzukehren.
- Alle Struktur-/Funktionsentscheidungen aus der vorherigen Kompaktierungs-Runde (Container-Klasse `.controls` unverändert für `showDealsView()`/`showAlertsView()`, `#count`-Span dauerhaft in der Toolbar, PLZ-Kürzung in `updateStatus()`) unangetastet gelassen — diese Runde war ausschließlich Spacing/Gruppierung, keine erneute Strukturänderung.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Bereich über den Produktkarten kompaktiert (Toolbar, Standort, Liste/Karte)

**Umgesetzt:**
- **Kompakte Toolbar** statt großer Sortier-/Filter-Zeile: `.controls` (Container-Klasse und JS-Selektor `controlsEl` bewusst unverändert gelassen, nur Inhalt/Optik geändert) zeigt jetzt „50 Angebote · Günstigster Preis ⌄ · [Filter-Icon]“ in einer schlanken Zeile. `#sortBtn` verloren Rahmen/Hintergrund/großes Padding (jetzt reiner Text-Button mit Chevron, führendes Sortier-Icon entfernt), `#filterBtn` von 44px großer umrandeter Box auf 30px reines Icon ohne Rahmen reduziert (Zähler-Badge bleibt). Neuer `<span id="count">` (aus der Standortzeile hierher verschoben) zeigt die Trefferanzahl live an — `render()`s bestehende Zeile `count.textContent = filtered.length` musste dafür nicht angepasst werden.
- **Kompakte Standortzeile**: `.status` von großer umrandeter Box (Hintergrund, Border, 13px Padding) auf schlanke Textzeile reduziert. `updateStatus()` zeigt jetzt „📍 Arnsberg · 10 km“ statt „📍 59821 Arnsberg · Radius 10 km · 50 Angebote“ — Postleitzahl wird per Regex (`^\d{4,5}\s+`) abgeschnitten, sofern vorhanden (Fallback: voller Text, falls kein führendes PLZ-Muster erkannt wird, z. B. bei "Aktueller Standort" oder frei getippten Ortsnamen ohne PLZ). Angebots-Anzahl entfernt (jetzt in der Toolbar). „Ändern“-Button (`#statusEdit`) unverändert.
- **Liste/Karte kompakter**: `.view-btn`-Padding von `10px` auf `7px` reduziert — Umschaltung insgesamt niedriger, Klickfläche bleibt ausreichend groß, Farben/aktiver Zustand unverändert.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v13` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server auf 320px, 375px und Desktop-Breite: Toolbar bleibt in allen Fällen einzeilig (auch mit dem längsten Sortier-Label „Günstigster €/Liter“), keine Überlappungen/Abschneidungen; Sortier- und Filter-Sheet öffnen weiterhin korrekt per Klick; Filter setzen aktualisiert Badge UND Toolbar-Trefferzahl live und korrekt (getestet: Pfand-frei-Filter → 0 Treffer, zurückgesetzt → 50); Standortzeile getestet mit PLZ-Label, PLZ-losem Label und "Ganz Deutschland"-Modus — alle drei Fälle sauber ohne Fehler; „Ändern“-Button öffnet weiterhin das Standort-Sheet; Alarme-Tab blendet Toolbar/Standortzeile weiterhin korrekt aus (unverändertes `showAlertsView()`); erste Produktkarte ist jetzt ohne Scrollen sichtbar (vorher musste gescrollt werden); keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Container-Klasse `.controls` bewusst NICHT umbenannt, obwohl sie jetzt eine "Toolbar" darstellt — die Klasse wird von `showDealsView()`/`showAlertsView()` per `document.querySelector(".controls")` referenziert; eine Umbenennung hätte diese (nicht angefragte) Anzeige-Logik mit anfassen müssen.
- Die globale `select{...}`-CSS-Regel (für `#filterPeriod` im Filter-Sheet) bewusst nicht berührt, obwohl im selben CSS-Bereich — sie wird an einer anderen, nicht zum Aufgabenbereich gehörenden Stelle gebraucht.
- Trefferzahl-`<span id="count">` lebt jetzt dauerhaft in der Toolbar statt (wie vorher) bei jedem `updateStatus()`-Aufruf in der Standortzeile neu erzeugt zu werden — behebt nebenbei eine vorbestehende Fragilität (die alte Node wurde bei jedem `innerHTML`-Replace der Standortzeile durch eine neue mit gleicher ID ersetzt, während die JS-Variable `count` noch auf die alte, losgelöste Node zeigte). Rein eine notwendige Konsequenz des Verschiebens, keine eigenständige Funktionsänderung.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Kaufland-/REWE-Logo vergrößert + „Start" setzt Filter zurück

**Umgesetzt:**
- **Kaufland-Logo**: `deals.json` → `stores.Kaufland.logo` von `kaufland-logo-social.png` (1200×600, das "K"-Signet nimmt darin nur die mittlere Hälfte ein, Rest ist weißer Leerraum links/rechts — dadurch wirkte es in der quadratischen 44×44-Box mit `object-fit:contain` kleiner als Lidl/Penny) auf die offizielle, quadratisch zugeschnittene Wikimedia-Commons-Datei `Kaufland_201x_logo.svg` (500×500, Vektor, nur das rote "K"-Signet mit "Kaufland"-Schriftzug, kein Fremd-Branding) geändert.
- **REWE-Logo**: von `rewe.de/icons/apple-touch-icon.png` (180×180, Sprechblasen-Form mit "Schwänzchen" unten rechts, lässt dadurch an den Ecken mehr Leerraum als ein füllendes Icon) auf `rewe.de/icons/icon-512.png` (512×512, offizielles PWA-App-Icon: abgerundetes, randfüllendes Rot-Quadrat mit "REWE DEIN MARKT") geändert — höher aufgelöst und füllt die Logo-Box genauso satt wie Lidl/Penny.
- Beide neuen Quellen offiziell (Wikimedia-Commons-Spiegelung der aktuellen Kaufland-Marke bzw. direkt von rewe.de) und höher aufgelöst/vektoriell als vorher, also keine Unschärfe beim Skalieren; keine Fremdlogos/Zusatz-Branding enthalten.
- **„Start" setzt Filter zurück**: `#navHome`-Click-Handler um dieselben Reset-Schritte ergänzt, die bereits der bestehende "Filter zurücksetzen"-Button im Empty-State nutzt (`search.value=""`, `selectedBrand="Alle"` + Tab-Chip-UI, `sort.value="price"`, `resetExtraFilters()` — setzt Radius, Störer-Filter, Größe/Verpackung, Zeitraum, Pfand-Only, "Abgelaufene ausblenden" zurück und synct alle zugehörigen Checkboxen/Chips/Slider). Bestehender `data-reset="filters"`-Handler (Empty-State-Button) dafür unverändert gelassen — Logik bewusst dupliziert statt in eine gemeinsame Funktion ausgelagert, um keinen bestehenden, funktionierenden Code anzufassen.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v12` erhöht (Pflichtregel, `index.html` und `deals.json` geändert).
- Verifiziert über einen temporären lokalen Server: Kaufland- und REWE-Logo jetzt sichtbar genauso groß/satt wie Lidl/Penny in Angebotskarte, keine Ladefehler; Filter (Suche, Marke, Sortierung, Radius, Pfand-Only, Größe, "Abgelaufene ausblenden") gezielt gesetzt und per Klick auf „Start" vollständig auf den Ausgangszustand zurückgesetzt (inkl. UI-Zustand von Sortierbutton-Label und Filter-Zähler-Badge, alle 50 Demo-Angebote wieder sichtbar); Favoriten-Ansicht wird beim Klick auf „Start" ebenfalls verlassen; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Für Kaufland/REWE wurden bewusst offizielle, aber jeweils andere Asset-Varianten der gleichen Händler gewählt (Wikimedia-Spiegel für Kaufland, PWA-Icon-Variante direkt von rewe.de) statt eines pauschalen CSS-Fixes (z. B. Zoom/Skalierung) — Letzteres hätte bei den anderen, bereits korrekt aussehenden Logos (Lidl, EDEKA, Penny, Netto) zu Verzerrungen geführt, da das eigentliche Problem im Bild-Zuschnitt lag, nicht in der CSS-Box.
- „Start"-Reset dupliziert bewusst den bestehenden Reset-Code statt ihn in eine gemeinsame Hilfsfunktion zu extrahieren — laut bisherigem Muster in diesem Projekt werden bestehende, funktionierende Codepfade nicht angefasst, wenn eine rein additive Ergänzung genügt.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Handle-Tap zum Schließen (Detail/Preisverlauf) + korrigiertes Penny-Logo

**Umgesetzt:**
- **Handle-Tap schließt Detail-/Preisverlauf-Sheet**: Neue Funktion `initHandleTapToClose(overlayId)` hängt einen Klick-Listener an den `.handle`-Balken eines Sheets, der `closeSheet(overlayId)` aufruft — aufgerufen ausschließlich für `histOverlay` (Preisverlauf) und `productDetailOverlay` (Produktdetail), direkt neben den bestehenden `initSwipeToClose(...)`-Aufrufen. Bewusst **nicht** global für alle Sheets mit `.handle` (loc-/notif-/filter-/sort-/profile-/storeDetail-/news-Overlay bleiben unverändert) — per Test bestätigt: Klick auf deren Handle schließt sie weiterhin nicht. Zurück-Pfeil und Wischgeste bei den beiden betroffenen Sheets funktionieren unverändert weiter, kommen alle drei Wege jetzt nebeneinander vor.
- **Penny-Logo korrigiert**: `deals.json` → `stores.Penny.logo` von `https://cdn.penny.de/.../PENNY_DEL_DEB_Logo.svg` auf `https://upload.wikimedia.org/wikipedia/commons/8/8e/Penny-Logo.svg` geändert. Ursache des bisherigen Fehlers: Die alte Datei ist ein zusammengesetztes Footer-Sujet mit **drei** Logos nebeneinander (rotes "PENNY DEL"-Co-Branding-Logo, das eigentliche schlichte "PENNY."-Wortmarke sowie ein Payback-Logo) in einem extrem breiten Viewbox (160×31) — dadurch erschien im App-Kontext (44×44-Box, `object-fit:contain`) alles verkleinert samt "DEL"-Zusatz. Recherchiert: Penny selbst verlinkt auf der eigenen Website (auch im Presse-Bereich) exakt dieselbe zusammengesetzte Datei; ihr `apple-touch-icon.png`/Favicon zeigen nur ein reduziertes "P."-Icon statt eines lesbaren Schriftzugs (anders als bei REWE, dessen `apple-touch-icon.png` das volle "REWE Dein Markt"-Logo zeigt). Als sauberste verfügbare Quelle für einen alleinstehenden, lesbaren "PENNY."-Schriftzug (quadratisches Viewbox, direkt vergleichbar zur REWE-Größe) wurde stattdessen die aktuelle offizielle Wikimedia-Commons-Datei verwendet — einzige Ausnahme vom sonst in diesem Projekt durchgängigen "nur offizielle Händler-Website"-Hotlink-Muster, da Penny selbst keinen sauberen Einzelschriftzug öffentlich hostet.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v11` erhöht (Pflichtregel, `index.html` und `deals.json` geändert — Letzteres ist ebenfalls Teil des precached App-Shells).
- Verifiziert über einen temporären lokalen Server: Handle-Tap schließt `histOverlay`/`productDetailOverlay` zuverlässig, andere Sheets unbeeinflusst, Pfeil-/Wisch-Wege weiterhin funktionsfähig; neues Penny-Logo erscheint in Angebotskarte, Karten-Popover und Händler-Detailansicht überall korrekt und gleich groß wie z. B. Lidl/EDEKA; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Handle-Tap-Funktion bewusst als eigene, kleine Funktion mit expliziter Overlay-ID-Liste umgesetzt statt eines globalen `document.querySelectorAll(".handle")`-Listeners — verhindert versehentliche Verhaltensänderung an den sechs anderen, nicht genannten Sheets.
- Wikimedia Commons als Ausnahme-Quelle für das Penny-Logo transparent dokumentiert (siehe oben) — falls gewünscht, könnte alternativ langfristig ein lokal eingebettetes SVG-Snippet (nur der "PENNY_DEB_neg_CMYK_hor"-Teilpfad aus der Originaldatei) genutzt werden, das wurde hier aber nicht umgesetzt, um von der bestehenden Hotlink-only-Konvention nicht noch weiter abzuweichen.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Favoriten-Herz: kräftigeres Rot + Leucht-Effekt + Pop-Animation

**Umgesetzt:**
- Nutzer-Feedback zur vorherigen Herz-Überarbeitung: Rot im Light Mode zu dunkel/matt, im Dark Mode zu "rosa". Ursache: beide nutzten den geteilten Token `--status-error-text` (Light `#ae352d`, Dark `#de8e89` — für Fehlermeldungen bewusst gedämpft). `.btn.fav.active` nutzt jetzt stattdessen einen fest hinterlegten, kräftigen Rot-Ton `#ef4444` in **beiden** Themes (bewusst keine Wiederverwendung/Änderung von `--status-error-text`, um andere Fehler-/Badge-Stellen im Rest der App nicht zu beeinflussen). Zusätzlich ein dezenter Leucht-Effekt via `filter:drop-shadow(0 0 4px rgba(239,68,68,.6))` auf dem gefüllten Herz, damit es speziell im Dark Mode sichtbar "leuchtet".
- **Pop-Animation beim Favorisieren**: neue `@keyframes fav-pop` (kurzer Scale-Bounce 1 → 1.35 → 0.92 → 1, 380ms) plus Hilfsfunktion `triggerFavPop(iconEl)`, die die Animationsklasse auf das Herz-Icon setzt und nach Ablauf automatisch wieder entfernt. Wird ausschließlich beim **Hinzufügen** zu Favoriten ausgelöst (nicht beim Entfernen), an beiden bestehenden Fav-Toggle-Stellen (Angebotskarte `[data-fav]` sowie `#detailFavBtn` im Preisverlauf-Sheet) — dezent, kein neues visuelles Element, exakt wie vom Nutzer gewünscht ("nicht zu auffällig, wie bei einem normalen Shoppingportal").
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v10` erhöht (Pflichtregel).
- Verifiziert über einen temporären lokalen Server in Light- und Dark-Mode: Rot deutlich kräftiger/gesättigter in beiden Themes (kein "Rosa" mehr im Dark Mode), Leucht-Effekt sichtbar; Pop-Animation läuft beim Favorisieren an beiden Stellen (Karte + Produktdetail), bleibt beim Entfernen aus, Klasse entfernt sich zuverlässig nach Animationsende; bestehende Favoritenfunktion unverändert getestet; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Bewusst ein neuer, fest codierter Rotton statt eines geteilten Design-Tokens, weil `--status-error-text` an mehreren anderen, unveränderten Stellen der App (Fehler-/Ablauf-Badges) verwendet wird — eine Änderung dort hätte über den angefragten Scope (nur das Herz) hinausgewirkt.
- Gleicher Rotton für Light UND Dark Mode (statt zweier abweichender Werte) — entspricht dem Wunsch nach einem kräftigen, "leuchtenden" Rot in beiden Modi und ist der in Shopping-/Social-Apps übliche Ansatz (Herz-Rot bleibt themenunabhängig knallig).
- Animation nur auf das Icon angewendet (nicht auf den 44×44-Klickbereich), damit der unsichtbare Touch-Bereich aus der vorherigen Änderung stabil bleibt und nicht mitspringt.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Favoriten-Herz: sichtbarer Kasten entfernt, freistehendes Icon

**Umgesetzt:**
- `.btn.fav` (Herz-Button auf Angebotskarten `[data-fav]` sowie `#detailFavBtn` im Preisverlauf-Sheet) neu gestylt: `background:none; border:none; padding:0` entfernt den bisherigen sichtbaren Kasten (inkl. des roten Kastens im favorisierten Zustand, vorher `.btn.fav.active{background:var(--status-error-bg);border-color:...}`). Button bleibt `width:44px;height:44px;flex-shrink:0` groß als **unsichtbarer** Klickbereich (44×44 CSS-Px, wie gefordert), das Herz-Icon selbst wurde über `.btn.fav .icon{width:24px;height:24px}` von 16px auf 24px vergrößert (Richtwert 22–26px).
- Farbe bewusst über bestehende, bereits theme-abhängige Design-Tokens gelöst statt neuer Werte: nicht favorisiert nutzt weiterhin `color:var(--text-primary)` (Light: `#1b213f` dunkles Navy, Dark: `#f2f3f6` Off-White — automatisch korrekt je Theme, keine Änderung nötig, war bereits so vererbt). Favorisiert nutzt `color:var(--status-error-text)` (Light: `#ae352d`, Dark: `#de8e89` — dieselben Rot-Töne, die im Rest der App bereits für Fehler-/Favoriten-Kontexte verwendet werden, z. B. das statische Herz in der Favoritenliste). Icon-Fill-Umschaltung (Outline ↔ ausgefüllt) war bereits vorhanden (`.btn.fav.active .icon{fill:currentColor;stroke:none}`) und wurde unverändert beibehalten.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v9` erhöht (Pflichtregel, da `index.html` geändert wurde).
- Verifiziert über einen temporären lokalen Server in Light- UND Dark-Mode (via `localStorage["canspot-theme"]`): kein sichtbarer Kasten/Hintergrund/Rahmen in beiden Zuständen und beiden Themes; nicht favorisiert = dunkles Outline-Herz (Light) bzw. helles Outline-Herz (Dark); favorisiert = rotes ausgefülltes Herz in beiden Themes; Klickbereich exakt 44×44px gemessen (`getBoundingClientRect`); Favorisieren/Entfavorisieren per Klick funktioniert weiterhin unverändert (getestet über `[data-fav]` und `#detailFavBtn`); andere Buttons (`#detailShareBtn`, `.btn.primary`) unverändert mit sichtbarem Kasten; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Keine neuen Farbwerte eingeführt — ausschließlich bereits vorhandene, längst theme-adaptive Tokens (`--text-primary`, `--status-error-text`) wiederverwendet, die im Rest der App schon für exakt diese Bedeutung (Haupttext bzw. Favoriten-/Fehlerrot) stehen. Das erfüllt "orientiere dich am bestehenden Design" ohne jede Farb-Neuerfindung.
- `flex-shrink:0` beim ersten Testlauf ergänzt, weil der Button in der Karten-Actions-Zeile (`display:flex`) sonst unter 44px zusammengedrückt wurde — notwendige Korrektur, um den geforderten 44×44px-Klickbereich zuverlässig einzuhalten.
- Die statische, nicht klickbare Herz-Deko in der Favoritenliste (`favRowHtml`, immer rot gefüllt, kein `.btn`) bewusst nicht angefasst — sie hatte nie einen Kasten und ist kein Favoriten-Button im Sinne der Aufgabe.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Logo im Banner nochmals vergrößert, „Alarme"-Hinweise als Karten

**Umgesetzt:**
- **Banner**: `.brand svg` (Logo) von `36px` auf `50px` Höhe erhöht ("noch einmal deutlich größer"); `.headline`-Slogan von `20px` auf `17px` reduziert und der Abstand darüber (`margin-top`) von `18px` auf `24px` vergrößert, für klare Trennung/Hierarchie. Logo bleibt eindeutig das dominante Element, passt auf Mobile (375px) wie Desktop ohne Überlappung in die bestehende `.top`-Zeile (geprüft: Logo + Profil-/Glocken-Icons zusammen bleiben deutlich unter der verfügbaren Breite).
- **„Alarme"-Bereich, Abschnitt „Hinweise für dich"**: Bisher eine einfache, einzeilige Text-/Icon-Liste (`.event-row`, 12,5px). Ersetzt durch kompakte Karten (`.news-card`, neue CSS-Klassen `.news-card*`) mit Produktbild, fett hervorgehobenem Produktnamen, Marke/Menge, groß hervorgehobenem Preis, Händler (+ Entfernung, falls vorhanden), farbigem Status-Badge oben ("Jetzt im Angebot" grün / "Demnächst im Angebot" gold / "Aktuell nicht verfügbar" grau — Farben reuse der bereits bestehenden `.info-badge`-Palette) sowie optionalen Zusatz-Badges ("Läuft {wann} ab", "{X} € unter Ø-Preis"). `computeFavoriteEvents()` liefert dafür jetzt EIN Status-Objekt pro Favorit statt bis zu drei loser Sätze — die zugrunde liegenden Regeln (läuft in ≤3 Tagen ab / >0,15 € günstiger als Ø / kein Angebot) sind unverändert, nur als Flags an das jeweilige Status-Objekt gehängt statt als eigene Zeilen. Neu (echt, nicht gemockt) ist die Erkennung eines "demnächst" startenden Angebots (`validFrom` in der Zukunft, analog zur Logik der Glocken-Neuigkeiten) — liefert mit den aktuellen Demo-Daten nichts, da `deals.json` keine zukünftig startenden Angebote enthält, kein Platzhalter ergänzt (Nutzer wollte explizit keine Dummy-Daten).
- Bestehende Preisalarm-Karten (`.alert-card`, Zielpreis/Aktueller-Preis-Vergleich, Toggle, Löschen/Angebote-ansehen) sowie die Glocke/„Neuigkeiten"-Sheet (aus der vorherigen Änderung) **vollständig unangetastet** — beide nutzen andere, eigene Funktionen (`renderAlertsView`s zweiter Teil bzw. `getRelevantNotifications()`), die nicht verändert wurden.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v8` erhöht (Pflichtregel, da `index.html` geändert wurde).
- Verifiziert über einen temporären lokalen Server (Mobile + Desktop): Logo sichtbar dominant, Slogan spürbar zurückhaltender, keine Überlappungen; Alarme-Karten mit echten Favoriten-Daten (Bild, Name, Preis, Händler, Entfernung, "unter Ø-Preis"-Badge) korrekt gerendert; alle drei Status-Varianten (aktuell/demnächst/nicht verfügbar) einzeln durchgetestet und sehen sauber aus, auch ohne Preis-/Store-Zeile bei "nicht verfügbar"; bestehende Preisalarm-Karten und Empty-State unverändert funktionsfähig; Glocke öffnet weiterhin unverändert das Neuigkeiten-Sheet; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Für die Produktbilder in den neuen Karten wurde bewusst die bestehende `.thumb`-Klasse wiederverwendet (statt einer neuen Bildklasse) — dadurch greift der bereits vorhandene Fehler-Fallback-Handler (`alertsView.querySelectorAll(".thumb")` → `FALLBACK_IMG`) automatisch mit, ohne zusätzlichen Code.
- Kein Mock/Platzhalter für "demnächst im Angebot" im Alarme-Bereich ergänzt (anders als zuvor bewusst bei der Glocke) — Nutzer hat für diese Änderung explizit "keine Dummy-Daten, wenn keine benötigt werden" verlangt; die echte Erkennungslogik ist vorhanden und greift automatisch, sobald `deals.json` ein Angebot mit zukünftigem `validFrom` enthält.
- `.alert-card` (Ziel-/Aktueller-Preis-Feature) bewusst nicht angefasst — war nicht der explizit kritisierte Teil ("einfache Textliste") und birgt bei Änderung unnötiges Regressionsrisiko für ein bereits gut funktionierendes Feature.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Header-Banner (Logo/Slogan/Standort) und Glocke → Neuigkeiten-Bereich

**Umgesetzt:**
- **Banner**: `.brand svg` (CanSpot-Logo) von `height:22px` auf `36px` vergrößert; `.headline` ("Dein Energy. Dein bester Preis.") von `var(--fs-display)` (26px) auf `20px` verkleinert (lokal auf der Regel überschrieben, die geteilte CSS-Variable `--fs-display` bleibt unangetastet, da sie sonst nirgends verwendet wird). Logo jetzt eindeutig größer/prominenter als der Slogan-Text, passt weiterhin sauber in die bestehende `.top`-Zeile (Höhe dort ohnehin von den 38px hohen Icon-Buttons vorgegeben).
- **Standort-Pill im Banner entfernt**: `#locBtn` (die "59821 · 10 km"-Pille neben Profil/Glocke) per `style="display:none"` ausgeblendet. Element, `#locLabel`-Kind-Span und der bestehende Klick-Handler (`openSheet("locOverlay")`) bleiben unverändert im DOM/JS erhalten, da `updateStatus()` weiterhin `locLabel.textContent` setzt (hätte sonst bei komplettem Entfernen des Elements einen Fehler geworfen). Die Standortanzeige/-bearbeitung bleibt über die `.status`-Zeile unterhalb des Banners ("Ändern"-Button, `#statusEdit`, ebenfalls unverändert) voll erreichbar.
- **Glocke öffnet jetzt einen Neuigkeiten-Bereich** (`#newsOverlay`, neu, folgt dem bestehenden Sheet-Muster) statt direkt der Push-Einstellungen. Neue Funktionen (bei `renderFavList()`/den Notifications-Funktionen einsortiert): `getRelevantNotifications()` berechnet für jeden Favoriten (a) ein "ist aktuell bei X für Y € im Angebot"-Hinweis aus echten Daten (`favorites` × `deals`, kein Mock nötig) und (b) prüft bereits echt auf zukünftig startende Angebote (`validFrom` in der Zukunft) — liefert damit aktuell nichts, da `deals.json` keine solchen Einträge enthält, ergänzt deshalb genau **ein** klar mit `demo:true` markiertes Platzhalter-Beispiel ("… wird demnächst irgendwo im Angebot sein.", in der UI mit "(Beispiel)"-Hinweis), das automatisch entfällt, sobald echte "demnächst"-Angebotsdaten vorhanden sind. `renderNewsList()` rendert die Liste (reine Wiederverwendung bestehender `.event-row`/`.info-badge`-Stile, keine neue CSS nötig). `openNewsSheet()` rendert, markiert alle aktuell gezeigten Benachrichtigungs-IDs als gelesen (neuer, eigener localStorage-Key `canspot-news-read`, unabhängig von den bestehenden `canspot-notif-*`-Push-Einstellungs-Keys) und öffnet das Sheet.
- **Punkt an der Glocke** (`#notifDot`) neu verdrahtet: `updateNotifDot()` zeigt ihn nur, wenn `getRelevantNotifications()` mindestens eine ID liefert, die noch nicht im `canspot-news-read`-Set steht; verschwindet beim Öffnen des Neuigkeiten-Sheets. Vorher war die Sichtbarkeit einfach an "hat der Nutzer überhaupt Favoriten" gekoppelt (in `renderFavList()`) — diese eine Zeile wurde durch den Aufruf von `updateNotifDot()` ersetzt, der Rest von `renderFavList()` (Rendering von „Deine Favoriten" innerhalb der Push-Einstellungen) ist unverändert.
- Push-Benachrichtigungseinstellungen (`#notifOverlay`, alle Switches, `#profileNotifRow` in Profil) **vollständig unverändert** und weiterhin erreichbar — nur der direkte Klick-Pfad von der Glocke wurde umgehängt.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v7` erhöht (Pflichtregel, da `index.html` geändert wurde).
- Verifiziert über einen temporären lokalen Server (mobiler Viewport): Logo (36px) sichtbar größer als Slogan (20px); Standort-Pille im Banner weg, `.status`-Zeile mit „Ändern" funktioniert weiterhin; Glocke öffnet Neuigkeiten-Sheet (nicht mehr Push-Einstellungen); Push-Einstellungen weiterhin über Profil erreichbar; Punkt erscheint bei ungelesenen Neuigkeiten und verschwindet nach dem Öffnen; Alarme-Tab (`renderAlertsView`/`computeFavoriteEvents`, eigener `navAlertsDot`) unverändert und funktionsfähig; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- `computeFavoriteEvents()`/`renderAlertsView()` (bestehende "Alarme"-Tab-Logik) bewusst **nicht** wiederverwendet oder verändert — konzeptionell ähnlich, aber ein separates bestehendes Feature; eine eigene, neue Funktion für die Glocke vermeidet jedes Regressionsrisiko dort.
- Für "demnächst im Angebot" wurde die echte Erkennungslogik (zukünftiges `validFrom`) implementiert, nicht nur eine reine Mock-Liste — sobald `deals.json` einen entsprechenden Eintrag bekommt, greift automatisch der echte Zweig statt des Platzhalters, ohne Codeänderung.
- `#locBtn` wurde ausgeblendet statt aus dem Markup entfernt, um `updateStatus()` (Standortlogik) exakt unangetastet zu lassen, wie vom Nutzer gefordert.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Preis-Eingabe: Punkt-Tastendruck jetzt schon vor Anzeige abgefangen

**Umgesetzt:**
- Nutzer berichtete, dass beim Löschen und manuellen Neueintippen eines Preises weiterhin ein Punkt statt Komma erscheint, insbesondere beim Antippen der Dezimaltaste auf einer (virtuellen) Zahlentastatur — trotz des vorherigen Fixes (`type="text"`, `sanitizePriceInputValue()` im `input`-Handler). Vermutete Ursache: Der bisherige Fix korrigierte den Punkt erst *nachdem* er im Feld sichtbar geworden war (reaktiv im `input`-Event); auf manchen Geräten/Tastaturen (v. a. virtuelle Zahlentastatur mit `inputmode="decimal"`, die spec-bedingt oft grundsätzlich einen Punkt statt Komma liefert) kann das kurz aufblitzen oder durch Eigenheiten der jeweiligen Tastatur-Implementierung nicht zuverlässig greifen.
- Härtung in `attachGermanPriceInput()` (`index.html`, direkt bei `euro()`): neuer `beforeinput`-Listener fängt ein getipptes „.“ bereits *vor* dem eigentlichen Einfügen ab (`e.preventDefault()`), fügt stattdessen direkt ein „,“ an der Cursorposition ein (nur falls noch kein Komma im Feld steht) und stößt danach synthetisch das bestehende `input`-Event an. Der bisherige `input`-Sanitizer bleibt als zweites Sicherheitsnetz bestehen (für Einfügen/Autofill/Browser ohne `beforeinput`-Unterstützung).
- Per direkt dispatchten `beforeinput`-/`InputEvent`s (Chromiums `execCommand`-Testpfad feuert kein `beforeinput`, echte Tastatur-/Tap-Eingaben in echten Browsern aber schon) für beide betroffenen Felder (`#alertPriceInput`, `[data-quick-alert-input]`) verifiziert: ein getipptes „.“ wird abgefangen (`preventDefault` greift, `dispatchEvent`-Rückgabewert `false`) und durch „,“ ersetzt, Cursor korrekt danach positioniert.
- `CACHE_NAME` in `service-worker.js` auf `canspot-cache-v6` erhöht (neue Pflichtregel aus CLAUDE.md), da `index.html` erneut geändert wurde.

**Wichtige Entscheidungen:**
- Zusätzliche `beforeinput`-Absicherung bewusst *ergänzend*, nicht ersetzend zum bestehenden `input`-Sanitizer eingebaut — deckt so sowohl das direkte Tippen/Antippen als auch Einfügen/Autofill/ältere Browser ab, ohne bestehende Logik zu entfernen.
- Der exakte, vom Nutzer beschriebene Fehlerfall ließ sich mit den in dieser Umgebung verfügbaren Test-Simulationsmethoden (`execCommand insertText` Zeichen für Zeichen) nicht reproduzieren — beide Ebenen wurden daher separat durch direkt konstruierte `beforeinput`-Events verifiziert, statt blind weitere Änderungen zu raten. Falls der Fehler nach diesem Fix weiterhin auftritt, sind Gerät/Browser sowie ob physische oder virtuelle Tastatur benutzt wurde, für die weitere Diagnose hilfreich.

**Offen:**
- Rückmeldung des Nutzers abwarten, ob das Problem nach diesem Fix behoben ist; falls nicht, genaue Geräte-/Browser-Angabe erbitten.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler in den automatisierten Tests dieser Session.
- Nicht committet/gepusht — Nutzer committet/pusht selbst nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Preis-Eingabefelder auf deutsches Komma-Format umgestellt

**Umgesetzt:**
- Bug: Die beiden Preisalarm-Eingabefelder (`#alertPriceInput` im Preisverlauf-Sheet, `[data-quick-alert-input]` in der Favoriten-Schnell-Alarm-Form) waren `<input type="number" step="0.01">`. Native Zahlenfelder normalisieren ihren Wert intern immer mit Punkt als Dezimaltrennzeichen und zeigen keine unwesentlichen Nachkommastellen an — dadurch wurde ein per JS gesetzter Wert wie `0.60` als `0.6` dargestellt, und je nach Browser/Locale wurde ein getipptes Komma zu einem Punkt.
- Fix: Beide Felder auf `type="text" inputmode="decimal"` umgestellt (gleiches numerisches Tastaturlayout auf Mobilgeräten, aber volle Kontrolle über Anzeige/Format statt Browser-nativer Zahlennormalisierung). Neue Hilfsfunktionen in `index.html` (direkt bei `euro()`): `parseGermanPrice()` liest getippten Text unabhängig von Komma/Punkt zu einer Zahl ein; `sanitizePriceInputValue()` normalisiert während des Tippens jeden Punkt zu einem Komma, entfernt ungültige Zeichen, erlaubt nur ein Komma und max. 2 Nachkommastellen; `attachGermanPriceInput()` hängt beides als `input`-/`blur`-Handler an ein Feld (bei Blur wird über `euro()` auf exakt 2 Nachkommastellen normalisiert, z. B. „0,6“ → „0,60“). Angewendet auf `#alertPriceInput` (einmalig bei Init) sowie auf alle `[data-quick-alert-input]`-Felder (in `bindFavRowEvents()`, da diese bei jedem Favoriten-Rerender neu erzeugt werden).
- Die beiden Stellen, die den bisherigen Wert per JS in ein Feld schrieben, nutzen jetzt `euro(...)` statt der rohen Zahl (Preisverlauf-Sheet-Öffnen sowie der hartcodierte Default `value="0,99"` im Favoriten-Template); die beiden Stellen, die den Feldwert lasen (`alertSetBtn`-Click, Quick-Alert-Submit), nutzen jetzt `parseGermanPrice(...)` statt `+input.value`.
- `CACHE_NAME` in `service-worker.js` gemäß der neuen Pflichtregel (siehe CLAUDE.md) auf `canspot-cache-v5` erhöht, da `index.html` geändert wurde.
- Verifiziert über einen temporären lokalen Server: alle 7 vom Nutzer genannten Testfälle (0,60 / 0,99 / 1,40 / 1,49 / 2,50 / 10,00 / 10,99 €) geprüft — live beim Tippen, nach Blur und nach „Alarm setzen“ jeweils korrektes Komma-Format mit exakt 2 Nachkommastellen; Sanitizing-Kantenfälle (getippter Punkt, mehrere Kommas, >2 Nachkommastellen, Buchstaben) geprüft; keine Konsolenfehler; Layout/Design der Eingabezeile optisch unverändert (generische `.alert-input-row input`-CSS-Regel ist nicht typ-spezifisch, betrifft weiterhin beide Feld-Varianten identisch).

**Wichtige Entscheidungen:**
- Ausschließlich die zwei Preisalarm-Eingabefelder und deren Lese-/Schreibstellen angefasst — alle anderen Preis-*Anzeigen* liefen bereits ausnahmslos über `euro()` (korrekt) und wurden nicht verändert; keine sonstigen UI-, Layout-, Text- oder Logikänderungen.
- `type="text"` + `inputmode="decimal"` statt eines Versuchs, `type="number"` per `lang`/`step` zum Komma zu zwingen — Letzteres ist browserübergreifend nicht zuverlässig steuerbar, ersteres gibt volle Kontrolle über Anzeige und Parsing.

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht — Nutzer committet/pusht laut vereinbartem Workflow selbst, nach eigenem Test in der Vorschau.

---

## 2026-09-02 — Wischgeste zum Schließen + Preisverlauf/Produktdetail getrennt

**Umgesetzt:**
- **Wischgeste zum Schließen** (`initSwipeToClose()` in `index.html`): Zusätzlich zum bestehenden Zurück-Pfeil lässt sich die Detailansicht (`#histOverlay`) sowie die neue Produktdetailansicht (`#productDetailOverlay`) per Herunterwischen schließen. Touch-Handler auf `.sheet` zieht nur, wenn `sheet.scrollTop<=0` UND die Bewegung eindeutig vertikal nach unten geht (Schwelle 8px, sonst gewinnt horizontales/normales Scrollen); unter 110px Zugdistanz federt die Sheet per CSS-Transition zurück, darüber schließt sie animiert. `.sheet.dragging{transition:none}` neu in CSS für ruckelfreies 1:1-Ziehen während der Geste.
- **Preisverlauf und Produktbild-Klick getrennt**: Bisher öffneten beide denselben `#histOverlay`. Der Produktbild-Klick (`.thumb` in der Angebotskarte, jetzt mit `data-product-detail="{productId}"`) öffnet neu `openProductDetail(productId)` → eigenes Sheet `#productDetailOverlay` mit Nährwerten + „Ähnliche Produkte“; „Preisverlauf“ (`data-hist`) öffnet weiterhin ausschließlich `openHistory(dealId)` (Preisverlauf-Chart, Kontext bleibt preisbezogen: Bester-Preis-Badges, weitere Händler, Preisalarm — unverändert, nur zusätzlich mit klar sichtbarer Händler-Zeile `#histStoreRow`/`#histStoreLogo`/`#histStoreName` direkt unter dem Produktnamen ergänzt, damit Produkt + Shop auf einen Blick erkennbar sind).
- **`openProductDetail(productId)`** (Produktkatalog-basiert, nicht angebots-/händlerspezifisch): zeigt Produktbild, Name, Marke/Menge, Nährwerte-Grid (Kalorien/Zucker/Koffein/Taurin, hochgerechnet auf die Dosengröße) sowie eine horizontal scrollende „Ähnliche Produkte“-Reihe (gleiche Marke zuerst, sonst andere Marken, max. 6) mit optionalem „ab X €“-Preis aus aktiven `deals`. Klick auf ein ähnliches Produkt öffnet rekursiv dessen eigene Produktdetailansicht.
- **Nährwerte als klar markierte Mock-Daten**: `NUTRITION_MOCK_BY_PRODUCT` → `NUTRITION_MOCK_BY_BRAND` → `NUTRITION_FALLBACK_PER_100ML` (dieselbe Fallback-Ketten-Struktur wie `PFAND_FALLBACK`/`PACKAGING_FALLBACK`), da `deals.json` keine echten Nährwertdaten liefert. Zucker-/kalorienreduzierte Varianten (Red Bull Sugarfree/Zero, Monster Ultra Zero, effect Zero Sugarfree, 28 Black Açaí Zero) haben produktgenaue Ausnahmen (0 g Zucker), damit die Demo-Werte dem Produktnamen nicht offensichtlich widersprechen. In der UI mit Caption „Demo-Werte, keine geprüften Herstellerangaben“ gekennzeichnet — für eine echte Datenquelle genügt es, `NUTRITION_MOCK_BY_PRODUCT` zu befüllen oder `getNutritionPer100ml()` umzustellen.
- Verifiziert über einen temporären lokalen Server (mobile Viewport 375×812): Angebotskarte → Bild-Klick öffnet Produktdetail (Nährwerte + ähnliche Produkte, keine Preisdaten/Chart); „Preisverlauf“-Klick öffnet weiterhin nur den Preisverlauf (Produktname + Händlerzeile + Chart, keine Nährwerte/Vorschläge); Zurück-Pfeil und simulierte Touch-Wischgesten (>110px → schließt, <110px → federt zurück, `scrollTop>0` → kein Drag) per dispatched `TouchEvent`s geprüft; keine Konsolenfehler.

**Wichtige Entscheidungen:**
- Bestehende Inhalte von `#histOverlay` (Badges, weitere Händler, Preisalarm, CTA) bewusst **nicht** entfernt — die Aufgabenstellung schließt explizit nur Nährwerte/Produktvorschläge/„zusätzliche Produktdetails“ aus der Preisverlauf-Ansicht aus; die bestehenden Preisvergleichs-Funktionen sind keine Produktdetails im engeren Sinn und sollten laut Vorgabe „bisher funktionierende Abläufe“ erhalten bleiben.
- `openProductDetail()` ist an `productId` (Katalog) statt `dealId` (Angebot) adressiert, weil die neue Ansicht produktbezogen ist und auch für Produkte ohne aktuelles Angebot funktionieren soll (siehe „ähnliche Produkte“ ggf. ohne Preis).
- Wischgeste per Touch-Events (nicht Pointer-Events) umgesetzt, konsistent mit dem Mobile-First-Charakter der App; Maus-Drag auf Desktop schließt die Sheet daher nicht (dort bleibt der Pfeil/Klick-außerhalb-Weg).

**Offen:**
- Keine offenen Rückfragen.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nicht committet/gepusht (auf ausdrücklichen Wunsch des Nutzers für diese Änderung).

---

## 2026-09-02 — PWA-Icons neu generiert, SEO-Landingpages farblich angeglichen

**Umgesetzt:**
- Auf Rückfrage (siehe „Offen“ im vorherigen Eintrag) hat der Nutzer beide offenen Punkte bestätigt: Icons neu generieren, Landingpages angleichen; Push zum Remote weiterhin **nicht** gewünscht.
- **PWA-Icons neu generiert** (`icons/favicon-16.png`, `favicon-32.png`, `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon-180.png`): Aus dem Logo-SVG die drei eigenständigen `<path>`-Elemente außerhalb der Buchstaben-`<g>` isoliert (Dose + Standort-Pin-Symbol, ohne Schriftzug) und deren exakte Bounding Box per `getBBox()` im Browser ermittelt. Icons per `<canvas>` gerendert: flacher `--brand-primary-dark`-Hintergrund (kein Verlauf mehr, passend zum neuen flachen Design), Symbol in Weiß/`--brand-primary-blue`, dieselben Eckenradien-Proportionen wie zuvor (abgerundet für alle Größen außer `icon-maskable-512`, das als volles Quadrat ohne eigene Rundung für den OS-Maskierungs-Safe-Zone-Bereich exportiert wurde, mit kleinerem Symbol-Anteil). Visuell verglichen mit den alten Icons (gleiches Seitenverhältnis/Padding-Gefühl), Dateigrößen/Dimensionen verifiziert.
- **`service-worker.js`**: `CACHE_NAME` von `canspot-cache-v2` auf `canspot-cache-v3` erhöht (Icons sind Teil des precached App-Shells; laut CLAUDE.md-Regel bei jeder relevanten Änderung an gecachten Dateien nötig, damit wiederkehrende Besucher/PWA-Installationen die neuen Icons statt der alten aus dem Cache bekommen).
- **CLAUDE.md** aktualisiert: Icon-Beschreibung im PWA-Abschnitt (alter Blitz → neues Dose/Pin-Symbol, Generierungsweg dokumentiert) sowie die `CACHE_NAME`-Versionsangabe.
- **7 SEO-Landingpages** (`arnsberg-59823.html`, `carabao.html`, `gonrgy.html`, `location.html`, `monster.html`, `red-bull.html`, `rockstar.html`) — alle teilen ein identisches, minimales inline-`<style>` (eigene alte Farbpalette, unabhängig vom Haupt-App-CSS, noch aus einer Vor-„CanSpot"-Iteration mit Titel „Energy Deals v3.2“). Deren 10 hartcodierte Hex-Werte 1:1 auf die neue Palette gemappt (u. a. Hero-Hintergrund/Text `#111827`→`#1b213f`, „BESTER PREIS“-Badge von Grün/Erfolg auf Amber/Highlight umgestellt, passend zur Hauptapp). `theme-color`-Meta ebenfalls aktualisiert. Reiner Farb-Swap — Markup, Inhalte, Links, Struktur sowie der (veraltete) „⚡ ENERGY DEALS“-Markentext/-Titel blieben unangetastet (war nicht Teil der Frage/Zustimmung, nur „farblich angleichen“).
- Verifiziert über einen temporären lokalen Server: `index.html` lädt weiterhin fehlerfrei (50 Angebote, Dark/Light, Karte), Icon-Dateien haben korrekte Maße (16/32/192/512/512/180 px), `red-bull.html` und `location.html` stichprobenartig visuell geprüft.

**Wichtige Entscheidungen:**
- Icon-Hintergrund bewusst als flache Farbe statt des alten radialen Verlaufs umgesetzt — konsistent mit dem neuen, insgesamt flacheren Designsystem (auch der App-Header nutzt jetzt eine flache statt verlaufende Fläche).
- Auf den Landingpages wurde ausschließlich die Farbpalette übertragen, nicht der Marken-Name/Text oder das Logo-Bild selbst (dort weiterhin „⚡ ENERGY DEALS“ statt des neuen „canspot“-Schriftzugs) — das war explizit nicht Teil der gestellten/beantworteten Frage („farblich angeglichen“). Bei Bedarf separat ansprechen.
- Weiterhin **kein Push** zum GitHub-Remote — alle Änderungen bleiben lokal committet, bis der Nutzer das anfordert.

**Offen:**
- Marken-Name/-Logo auf den 7 Landingpages (aktuell noch „⚡ ENERGY DEALS“) — nur ansprechen, falls gewünscht.
- Push zum Remote steht weiterhin aus.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nächster Schritt: warten auf weiteres Feedback oder die Freigabe zum Pushen.

---

## 2026-09-02 — Neues Farb-/Design-System und Logo aus Referenzdatei übernommen

**Umgesetzt:**
- Nutzer stellte eine Referenzdatei bereit (`.../ENERGY-APP/20260902/Überarbeitung-Farben-Logo/EnergyBoost — Prototyp-2.html`, ein per Browser „Seite speichern“ exportierter Snapshot). Dieselbe Änderung war zunächst versehentlich an einer lokalen Kopie außerhalb dieses Repos vorgenommen worden; auf Hinweis des Nutzers hier im GitHub-Verzeichnis (`index.html`) nachgeholt.
- Vor der Änderung geprüft: Der `<style>`-Block dieses Repos war byte-identisch mit dem Ausgangszustand der lokalen Kopie (bevor dort das neue Design übernommen wurde) — daher **ausschließlich der komplette `<style>`-Block** 1:1 aus der Referenzdatei übernommen (neues Marken-Blau `#3363ac` statt Orange, systematische Neutral-/Status-Farbskala, Radius-/Schatten-/Hover-/Focus-Tokens) sowie `<div class="brand">` durch das neue Vektor-Logo-SVG („canspot“-Wortmarke) ersetzt.
- Sechs im JavaScript **hart codierte** Farbwerte außerhalb des CSS-Blocks ebenfalls umgestellt: Preisverlauf-Chart (`drawChart()`: Linie/Fläche/Punkte), dessen Legenden-Punkte, `FALLBACK_IMG` und `STORE_FALLBACK_IMG` (Bild-Fallbacks). `<meta name="theme-color">` von `#2a1607` auf `#1b213f` aktualisiert.
- `manifest.webmanifest`: `theme_color` und `background_color` ebenfalls auf die neue dunkle Marken-Palette (`#1b213f`/`#0d0f14`) umgestellt, damit PWA-Installationsbildschirm/Browser-Tinting zum neuen Design passen.
- `deals.json`, alle Angebots-/Filial-/Produktdaten, sämtliche JS-Logik (Suche, Filter, Favoriten, Preisalarme, Persistenz, `loadDeals()`/PWA-Mechanik) sowie die Struktur von `index.html` blieben unverändert.
- Verifiziert über einen temporären lokalen Server: Light-/Dark-Theme, Produktdetail-Chart, Kartenansicht, sowie funktionaler Regressionstest (Suche, Favorit setzen/entfernen inkl. Persistenz, Kartenansicht-Pins) — keine Konsolenfehler (bis auf eine Service-Worker-Registrierungsmeldung, die auf die automatisierte Browser-Umgebung zurückgeht, nicht auf die Änderung).

**Wichtige Entscheidungen:**
- **Nicht** angefasst: die PWA-Icon-Dateien (`icons/*.png`, weiterhin der alte orangene Blitz auf dunkelbraun) und die sieben eigenständigen SEO-Landingpages (`arnsberg-59823.html`, `carabao.html`, `gonrgy.html`, `location.html`, `monster.html`, `red-bull.html`, `rockstar.html`), die jeweils ihr eigenes kleines, ebenfalls noch altes Styling mitbringen. Beides ist zusätzlicher, vom Nutzer noch nicht angeforderter Scope — siehe Rückfrage im Chat.
- Committet, aber **nicht gepusht** — dieses Repo ist mit einem GitHub-Remote verbunden und laut vorherigem Log-Eintrag bereits als GitHub-Pages-PWA live deployed; ein Push würde die öffentliche Version ändern und erfordert explizite Bestätigung.

**Offen:**
- Rückfrage an Nutzer: sollen die PWA-Icons (Favicon/App-Icon) neu generiert werden, damit sie zum neuen Logo/Farbschema passen? Aktuell zeigen sie noch den alten Blitz.
- Rückfrage an Nutzer: sollen die sieben SEO-Landingpages ebenfalls farblich/optisch angeglichen werden?
- Push zum GitHub-Remote steht aus, bis der Nutzer das bestätigt.

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler in `index.html`.
- Nächster Schritt: Rückmeldung des Nutzers zu Icons/Landingpages/Push abwarten.

---

## 2026-09-01 — Angebotsdaten in deals.json ausgelagert, App lädt dynamisch

**Umgesetzt:**
- Vorbereitung auf eine spätere echte Angebotsdatenquelle: Produkte, Händler/Filialen und Angebote sind nicht mehr inline in `index.html` codiert, sondern liegen in [deals.json](deals.json), das die App zur Laufzeit per `fetch()` lädt (`loadDeals()`). Kein UI- oder Verhaltensänderung, keine echten Händlerdaten abgerufen — `deals.json` enthält weiterhin ausschließlich die bisherigen Demo-/Testdaten.
- `deals.json` neu strukturiert (ersetzt den alten, ungenutzten Alt-Bestand aus einem früheren Prototyp-Stand) mit klar dokumentiertem Schema (`_meta.description`/`_meta.fields` im JSON selbst): `products[]`, `stores{}` (Logo + Filiale: Adresse/Geo/Öffnungszeiten je Händler) und `offers[]` — das einheitliche Angebotsformat mit Produkt (`productId`), Marke/Menge (über `products[]`), Händler (`store`), Preis (`regularPrice`/`offerPrice`), Pfand (`pfand`), Angebotszeitraum (`validFrom`/`validUntil`), Entfernung (`distanceKm`) sowie optionalem `priceHistory` für den Preisverlauf. Öffnungsstatus wird bewusst NICHT im Datensatz gespeichert, sondern weiterhin clientseitig aus den Öffnungszeiten gegen die aktuelle Uhrzeit berechnet (sonst würde er veralten).
- Die 50 aktuellen Test-Angebote (10 Basis-Angebote + 20 zuvor zur Laufzeit generierte Zusatz-Angebote für p8–p28, je 2 Händler) wurden einmalig exakt mit der bisherigen Generator-Logik berechnet und 1:1 unverändert nach `deals.json` übernommen (gleiche Preise, IDs, Distanzen, Zeiträume) — keine sichtbare Änderung an den angezeigten Angeboten.
- `index.html`: `products`/`storeLogos`/`storeBranches`/`deals` sind jetzt leere `let`-Platzhalter, befüllt durch `loadDeals()`. Neue Funktionen `buildDeals()` (Anreicherung: Produktdaten, Pfand-/Verpackungs-Fallback, Preisverlauf, „zuletzt geprüft") und `loadDeals()`/`finishInit()`/`showLoadError()`/`delay()`. `getPfand()`/`getPackaging()`-Wrapper entfernt, alle Aufrufstellen lesen jetzt direkt `deal.pfand`/`deal.packaging`.
- Ladezustände: Skeleton-Platzhalter (`renderSkeleton(4)`) bis die Daten da sind (künstliche Mindestwartezeit von 500ms wie bisher, damit das Skelett bei schneller lokaler Antwort nicht nur aufblitzt); bei Fehlschlag (Netzwerkfehler, HTTP-Fehler, kaputtes JSON) neuer Fehlerzustand mit „Erneut versuchen"-Button; bei leerer/gefilterter Angebotsliste weiterhin der bestehende „Keine Angebote gefunden“-Leerzustand (keine neue UI nötig).
- Service Worker (`service-worker.js`): `deals.json` zum Precache der App-Shell hinzugefügt, `CACHE_NAME` auf `canspot-cache-v2` erhöht, damit Offline-Nutzung weiterhin die vollen 50 Angebote zeigt statt nur die App-Hülle.
- Lokal getestet (Server auf Port 8123, dabei jeweils Service-Worker-Cache geleert für saubere Tests): normaler Load (50 Angebote, Preise identisch zu vorher), Fehlerzustand durch temporär entfernte `deals.json` ausgelöst und per „Erneut versuchen“ wiederhergestellt, Leerzustand mit leerer `offers[]`-Liste, Preisverlauf-Chart (Fallback-Generierung funktioniert), Filialdetail-Sheet (Adresse/Öffnungszeiten/Live-Status), Verpackungsfilter „Flasche“ weiterhin 0 Treffer, „Ohne Pfand“-Filter weiterhin 0 Treffer (beides exakt wie zuvor dokumentiertes Verhalten), Kartenansicht, Offline-Modus bei gestopptem Server (App inkl. aller 50 Angebote weiterhin nutzbar).

**Wichtige Entscheidungen:**
- Alt-Inhalt von `deals.json` (aus einem früheren, nicht mehr verwendeten Prototyp-Stand mit anderen Produkt-IDs, ohne Bilder/Historie/Filialdaten, referenziert von inzwischen ungenutzten `*.html`-Einzelseiten) wurde vollständig ersetzt statt parallel gepflegt — die neue Datei ist jetzt die einzige, tatsächlich genutzte Datenquelle.
- Filiale wird pro Händlerkette modelliert (ein Satz Öffnungszeiten/Adresse je Kette in Arnsberg), nicht pro einzelnem Angebot — entspricht exakt dem bisherigen Verhalten. Für echte Mehr-Filial-Daten müsste `offers[]` künftig eine eigene `branchId` bekommen; bewusst nicht vorgebaut, um keine ungenutzte Komplexität einzuführen.
- Preisverlauf und „zuletzt geprüft“ bleiben clientseitig synthetisch/deterministisch erzeugt (unverändertes Verhalten), `deals.json` unterstützt aber schon optional ein eigenes `priceHistory`-Array pro Angebot für eine echte Datenquelle.
- Künstliche 500ms-Mindestwartezeit beim Laden bewusst beibehalten (jetzt als Untergrenze neben dem echten `fetch()`, nicht mehr als reiner Fake-Timer), damit sich am wahrgenommenen Ladeverhalten nichts ändert.

**Offen:**
- Wie in CLAUDE.md dokumentiert: `CACHE_NAME` in `service-worker.js` bei künftigen Änderungen an `deals.json` (oder anderen App-Shell-Dateien) manuell weiter hochzählen.
- Für eine echte Datenquelle später: `loadDeals()`'s `fetch("deals.json")`-Ziel austauschen bzw. `deals.json` durch einen echten, gleich geformten Feed ersetzen; optional `lastCheckedAt`-Zeitstempel statt der deterministischen `checkedMinutesAgo`-Simulation ergänzen (bewusst nicht vorgebaut).

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Änderungen sind lokal committet, noch nicht gepusht (auf Freigabe des Nutzers wartend).

---

## 2026-09-01 — PWA-Deployment auf GitHub Pages bestätigt

**Umgesetzt:**
- Nutzer hat Commit `0821710` (PWA: Manifest, Icons, Service Worker) über GitHub Desktop nach `origin/main` gepusht.
- GitHub Pages ist mit dem neuen Stand aktualisiert.
- Auf echtem Smartphone verifiziert: PWA ist installierbar, App-Icon (Blitz-Symbol) erscheint korrekt auf dem Homescreen.

**Offen:**
- Der bisher offene Punkt „Installierbarkeit auf echten Geräten verifizieren" aus dem vorherigen Eintrag ist damit erledigt.
- `CACHE_NAME` in `service-worker.js` weiterhin manuell hochzählen, sobald sich künftig App-Shell-Dateien ändern (siehe CLAUDE.md).

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nächster Schritt: warten auf weitere Änderungswünsche.

---

## 2026-09-01 — CanSpot als installierbare Progressive Web App (PWA)

**Umgesetzt:**
- `manifest.webmanifest` ergänzt: Name „CanSpot", Kurzname, `theme_color`/`background_color` passend zum bestehenden dunklen Farbschema, `start_url`/`scope` relativ (`./`) für GitHub-Pages-Kompatibilität unter einem Unterpfad.
- Icon-Set unter `icons/` erzeugt (Blitz-Symbol, angelehnt an das bestehende `i-bolt`-Icon und die Akzentfarbe `--accent`, auf dunkelbraunem Hintergrund passend zu `theme-color`): `favicon-16.png`, `favicon-32.png`, `icon-192.png`, `icon-512.png` (purpose „any"), `icon-maskable-512.png` (purpose „maskable", mit Sicherheitsabstand für Androids adaptive Icons), `apple-touch-icon-180.png`. Generiert über ein lokales Canvas-Skript (kein externer Dienst, kein SVG-Konverter nötig) und über einen lokalen Upload-Server direkt als PNG auf die Festplatte geschrieben.
- `service-worker.js` ergänzt: Cache-first-Strategie für Same-Origin-Requests, Precaching der App-Shell (`index.html`, Manifest, Icons) beim Install, Offline-Fallback auf das gecachte `index.html` bei Navigation. Hotlinked Produkt-/Händlerbilder (externe Origins) werden bewusst NICHT gecacht — bleiben wie bisher netzwerkabhängig.
- `index.html` `<head>` ergänzt um: `<link rel="manifest">`, Favicon-Links, `apple-touch-icon`, `apple-mobile-web-app-*`/`mobile-web-app-capable`-Meta-Tags, `viewport-fit=cover`. Service-Worker-Registrierung ganz am Ende des bestehenden `<script>`-Blocks (nach dem bestehenden `setTimeout(...,500)`-Init), hinter `"serviceWorker" in navigator` abgesichert.
- `.claude/launch.json` neu angelegt (fehlte bisher trotz Referenz in CLAUDE.md) mit der `canspot-preview`-Konfiguration (`python3 -m http.server 8123`).
- Lokal getestet: Manifest/Icons/Service-Worker werden korrekt ausgeliefert (HTTP 200, richtige Content-Types), Service Worker registriert sich und aktiviert sich, App-Shell wird vollständig in den Cache geschrieben, Seite lädt vollständig offline (Server während Test gestoppt, Seite weiterhin nutzbar). Bestehende UI/Funktionen (Sheets, Theme, Navigation) nach den Änderungen stichprobenartig erneut geprüft — keine Regressionen.

**Wichtige Entscheidungen:**
- Alle PWA-Pfade (Manifest, Icons, Service Worker, Registrierung) sind relativ ohne führenden `/`, damit die App sowohl lokal als auch unter einem GitHub-Pages-Projektunterpfad (`https://wilderusername.github.io/Energy-deals/`, kein eigener Domain-Root) korrekt funktioniert.
- `apple-mobile-web-app-status-bar-style` auf `black` statt `black-translucent` gesetzt, weil die App aktuell nur `env(safe-area-inset-bottom)` behandelt, aber keinen Top-Safe-Area-Abstand — `black-translucent` hätte Inhalte im Standalone-Modus unter die Statusleiste schieben können.
- Keine bestehende UI/Logik verändert — nur neue Dateien plus rein additive `<head>`-Tags und eine Service-Worker-Registrierung am Skriptende.

**Offen:**
- `CACHE_NAME` in `service-worker.js` muss bei künftigen App-Shell-Änderungen manuell hochgezählt werden (siehe CLAUDE.md), sonst bekommen wiederkehrende Nutzer eine veraltete gecachte Version.
- Kein Push-Notification-Support (nur die bestehende, rein lokale `canspot-notif-*`-Simulation) — falls echte Web-Push-Benachrichtigungen gewünscht sind, wäre das ein separates Feature (Server/Push-Service nötig).

**Bekannte Fehler / nächste Schritte:**
- Keine bekannten Fehler.
- Nächster Schritt: nach dem Push auf GitHub Pages die Installierbarkeit (Chrome-Install-Prompt, iOS „Zum Home-Bildschirm") auf echten Geräten verifizieren — lokal wurde nur der Offline-/Registrierungs-Teil getestet.

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
