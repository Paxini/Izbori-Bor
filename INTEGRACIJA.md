# Integracija eksternog softvera za praćenje izlaznosti

## Pregled

Dashboard čita podatke o izlaznosti iz **Firebase Realtime Database**. Svaki eksterni softver koji može da šalje HTTP zahteve može da upisuje podatke direktno u bazu, a dashboard će ih automatski prikazati u realnom vremenu.

## Endpoint

```
https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app
```

## Autentifikacija

Baza zahteva autentifikaciju (`auth != null`). Pre slanja podataka, potrebno je dobiti **ID token** putem Firebase Auth REST API-ja.

### Korak 1 — Dobijanje tokena

```
POST https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=AIzaSyBB2yq4b2lLGhI8PMD7NuXkUa6qRHO1FoM
Content-Type: application/json

{
  "email": "korisnik@example.com",
  "password": "lozinka",
  "returnSecureToken": true
}
```

Odgovor sadrži polje `idToken` koje se koristi u svim narednim zahtevima. Token važi **1 sat**. Kada istekne, ponoviti ovaj poziv ili koristiti `refreshToken` iz odgovora za obnovu.

### Korak 2 — Slanje podataka sa tokenom

Svaki zahtev ka bazi mora da sadrži `auth` parametar:

```
?auth=<idToken>
```

## Struktura podataka

Putanja u bazi:

```
/turnout/{bmId}/{slotIndex}
```

| Polje       | Tip    | Opis                                                        |
|-------------|--------|-------------------------------------------------------------|
| `bmId`      | string | Broj biračkog mesta (1–47)                                  |
| `slotIndex` | string | Indeks vremenskog preseka: `"0"` do `"5"`                   |
| vrednost    | number | Ukupan broj birača koji su glasali do tog preseka           |

### Mapiranje slotIndex → vreme preseka

| slotIndex | Vreme |
|-----------|-------|
| 0         | 09:00 |
| 1         | 11:00 |
| 2         | 13:00 |
| 3         | 15:00 |
| 4         | 17:00 |
| 5         | 19:00 |

## Primeri

### Prijava jednog preseka za jedno BM

Upisati da je na BM 12 u preseku od 13h glasalo 345 birača:

```
PUT https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app/turnout/12/2.json?auth=<idToken>
Content-Type: application/json

345
```

### Prijava celog preseka za jedno BM

Upisati sve preseke odjednom za BM 5:

```
PUT https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app/turnout/5.json?auth=<idToken>
Content-Type: application/json

{
  "0": 89,
  "1": 156,
  "2": 241
}
```

> Ovo će postaviti preseke 09h, 11h i 13h. Preseci koji nisu navedeni ostaju nepromenjeni.
> Ako želite da ažurirate bez brisanja postojećih preseka, koristite `PATCH` umesto `PUT`.

### Masovna prijava — sva BM odjednom

Upisati presek od 11h za svih 47 biračkih mesta u jednom zahtevu:

```
PATCH https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app/turnout.json?auth=<idToken>
Content-Type: application/json

{
  "1": { "1": 120 },
  "2": { "1": 198 },
  "3": { "1": 167 },
  ...
  "47": { "1": 203 }
}
```

> `PATCH` spaja podatke sa postojećim — ne briše ranije preseke ni ostala BM.

### Brisanje prijave

Obrisati presek od 15h za BM 8:

```
DELETE https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app/turnout/8/3.json?auth=<idToken>
```

## Kompletni primer (curl)

```bash
# 1. Dobijanje tokena
TOKEN=$(curl -s -X POST \
  'https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=AIzaSyBB2yq4b2lLGhI8PMD7NuXkUa6qRHO1FoM' \
  -H 'Content-Type: application/json' \
  -d '{"email":"korisnik@example.com","password":"lozinka","returnSecureToken":true}' \
  | python -c "import sys,json; print(json.load(sys.stdin)['idToken'])")

# 2. Prijava: BM 12, presek 13h, 345 glasova
curl -X PUT \
  "https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app/turnout/12/2.json?auth=$TOKEN" \
  -d '345'
```

## Kompletni primer (Python)

```python
import requests

API_KEY = "AIzaSyBB2yq4b2lLGhI8PMD7NuXkUa6qRHO1FoM"
DB_URL = "https://izbori-bor-2026-default-rtdb.europe-west1.firebasedatabase.app"
EMAIL = "korisnik@example.com"
PASSWORD = "lozinka"

# Autentifikacija
auth_resp = requests.post(
    f"https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key={API_KEY}",
    json={"email": EMAIL, "password": PASSWORD, "returnSecureToken": True}
)
token = auth_resp.json()["idToken"]

# Prijava jednog preseka
requests.put(
    f"{DB_URL}/turnout/12/2.json?auth={token}",
    json=345
)

# Masovna prijava celog preseka (npr. slot 1 = 11h)
presek_11h = {
    "1":  {"1": 120},
    "2":  {"1": 198},
    "3":  {"1": 167},
    # ... ostala BM
}
requests.patch(
    f"{DB_URL}/turnout.json?auth={token}",
    json=presek_11h
)
```

## Napomene

- Vrednosti moraju biti **celobrojne** (broj glasalih, ne procenat).
- Svaka vrednost predstavlja **kumulativni** broj birača koji su glasali do tog preseka (ne razliku od prethodnog).
- Dashboard automatski detektuje promene i ažurira prikaz u realnom vremenu — nema potrebe za dodatnim signaliziranjem.
- Biračka mesta su numerisana **1–47** za opštinu Bor.
- Token ističe nakon 1 sata. Za dugotrajne procese, implementirati obnovu tokena pre isteka.
