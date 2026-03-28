# Izborni Dan Dashboard — Sevojno 2026: Zbirni prompt svih izmena

## 2. Integracija odstupanja u tabelu
Obojena odstupanja iz sekcije "NAJVEĆA ODSTUPANJA" su integrisana direktno u tabelu izlaznosti kao nova kolona "Odst." sa klasama `dev-cell positive` (zeleno) i `dev-cell negative` (crveno). Sekcija "NAJVEĆA ODSTUPANJA" je potpuno uklonjena zajedno sa `renderAlerts()` i `alertHTML()` funkcijama.

## 5. Pin markeri biračkih mesta
Vizuelni markeri za biračka mesta na mapi bazirani na koordinatama iz `locations.json`. Prvobitno implementirani kao SVG pin ikone, zatim zamenjeni kružnim indikatorima (`bm-circle`) prečnika 26px sa brojem BM-a u centru, obojeni prema trenutnom filteru (odstupanje ili prioritet). Imaju beli border i senku za vidljivost.

## 6. Firebase Realtime Database — live sync
Implementirana integracija sa Firebase Realtime Database:
- Dodat `firebase-database-compat.js` SDK
- Dodat `databaseURL` u `firebase-config.js`
- `initTurnoutData()` inicijalizuje strukturu sa `null` slotovima
- `setupFirebaseSync()` sluša promene sa `on('value')` i poziva `refreshAll()`
- `submitReport()` koristi optimistic update (lokalno ažurira odmah, piše u Firebase u pozadini)
- `deleteReport()` briše podatak lokalno i iz Firebase
- Kreiran `database.rules.json` sa pravilima za autentifikovane korisnike
- Dodat `"database"` odeljak u `firebase.json`

## 7. Toast notifikacije i Firebase status
- Toast sistem (`showToast()`) za vizuelne povratne informacije: zeleni za uspeh, crveni za grešku, narandžasti za brisanje
- Firebase konekcioni indikator u donjem desnom uglu: "Povezano" (zeleno) / "Offline" (crveno) / "Greška sinhronizacije"
- Koristi `.info/connected` listener za praćenje konekcije

## 8. Deselektovanje poligona klikom van mape
Klik na prazan deo mape (van poligona/markera) automatski poništava selekciju. Implementirano preko `clickedOnFeature` flag-a koji se setuje u polygon/marker click handler-ima, a proverava u `electionMap.on('click')`.

## 9. Teren mode
Dugme "Teren" na mapi prebacuje između tamne (CartoDB Dark) i topografske (Esri World Topo) mape. U teren modu: poligoni imaju samo tanke ljubičaste obrise bez fill-a, kružni indikatori su jednobojni ljubičasti. Tastaturna prečica: **M**.

## 10. Email/password autentifikacija
- Dodat `firebase-auth-compat.js` SDK
- Login ekran sa email/password formom prikazuje se pre dashboarda
- `onAuthStateChanged` observer kontroliše prikaz: login → dashboard transition sa fade animacijom
- Poruke grešaka na srpskom (pogrešna lozinka, korisnik nije pronađen, previše pokušaja)
- Email korisnika prikazan u stats bar-u sa "Odjava" dugmetom
- Database pravila ažurirana na `"auth != null"`

## 11. Promena domena
Kreiran novi Firebase hosting sajt sa nasumičnim imenom (`panel-fe63129c`) umesto podrazumevanog `izbori-sevojno-2026.web.app`. Koristi `firebase hosting:sites:create` i `firebase target:apply` sa `"target": "main"` u `firebase.json`.

## 12. Filter dropdown
Dropdown menu u header-u tabele (pored "+ Prijavi") sa opcijama:
- **Live** — poslednji prijavljeni podaci za svako BM
- **Presek u 09h–20h** — podaci za specifičan vremenski presek
- **Prioritetnost** — bojenje prema prioritetu iz kolone M Excel fajla

Filter utiče na: boje poligona, boje kružnih indikatora, odstupanje u tabeli, tooltip, statistike u stats bar-u, legendu mape. Dropdown koristi `position: fixed` sa `z-index: 9000` da bude ispred svih panela.

## 13. Prioritetnost
Učitava se iz polja `prioritet_lokalni` u `sevojno.json` (kolona M iz `BirackaMesta.xlsx`). Svaka vrednost 1–8 ima svoju boju:
- 1: `#16a34a` (tamno zelena) → 8: `#dc2626` (tamno crvena)

Kada je filter "Prioritetnost" aktivan:
- Mapa i krugovi obojeni po prioritetu
- Legenda prikazuje skalu 1–8
- Kolona u tabeli prikazuje "Prior." sa numeričkom vrednošću
- Tooltip prikazuje prioritet umesto izlaznosti

## 14. Tastaturne prečice
- **0-9**: Brzi skok na BM po broju
- **Enter**: Potvrdi unos broja BM / potvrdi prijavu u modalu
- **P**: Pinuj/otpinuj BM
- **R**: Reset zoom mape
- **C**: Zoom na centar grada
- **T**: Uvećaj trend grafikon (fullscreen overlay)
- **M**: Teren / standardna mapa
- **F**: Fullscreen browser
- **H** (ili / ili ?): Prikaži tastaturnu pomoć
- **Esc**: Zatvori modal / overlay / poništi selekciju

## Fajlovi
- `index.html` — jedini HTML fajl, sadrži sav CSS i JS
- `js/firebase-config.js` — Firebase konfiguracija sa `databaseURL`
- `database.rules.json` — pravila za Realtime Database (`auth != null`)
- `firebase.json` — hosting + database deployment konfiguracija
- `data/sevojno.json` — statički podaci o opštini i biračkim mestima (uključuje `prioritet_lokalni`)
- `data/locations.json` — koordinate biračkih mesta
- `data/uzice-sevojno.geojson` — poligoni biračkih mesta za mapu
