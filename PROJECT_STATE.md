# PROJECT_STATE.md

Laufendes Änderungsprotokoll für CanSpot. Neuester Eintrag oben. Für dauerhafte Projektregeln/technische Hinweise siehe [CLAUDE.md](CLAUDE.md).

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
