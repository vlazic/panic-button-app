# PoC (Proof of Concept) - Pregled

## Šta je PoC?

PoC (Proof of Concept) je **minimalna funkcionalna verzija** aplikacije koja služi za:
1. Testiranje osnovnog koncepta
2. Validaciju sa stvarnim korisnicima
3. Prikupljanje ranog feedback-a

**Cilj:** Odgovoriti na pitanje "Da li ovo uopšte radi?" sa minimalnim ulaganjem vremena.

---

## Opseg PoC-a

### Šta JE uključeno

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  POC FUNKCIONALNOSTI                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ✅ Panic button (drži 3 sekunde za aktivaciju)                             │
│  ✅ Automatsko preuzimanje GPS lokacije                                     │
│  ✅ Opcioni tekstualni opis incidenta                                       │
│  ✅ Opcioni unos imena                                                      │
│  ✅ Slanje alarma u Telegram grupu                                          │
│  ✅ "Preuzimam" dugme (sa unosom imena)                                     │
│  ✅ Real-time prikaz ko je preuzeo                                          │
│  ✅ Lista aktivnih alarma                                                   │
│  ✅ Razrešenje alarma (REŠENO / LAŽNI / OTKAZANO)                           │
│  ✅ PIN-bazirana autentifikacija                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Šta NIJE uključeno

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  IZOSTAVLJENO IZ POC-a                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ❌ Više škola/grupa (samo jedna, hardcoded)                                │
│  ❌ SMS verifikacija (samo PIN)                                             │
│  ❌ Uloge korisnika (svi jednaki)                                           │
│  ❌ Sistem smena (svi dobijaju sve)                                         │
│  ❌ Eskalacija (nema timeout-a)                                             │
│  ❌ Admin panel (ručna administracija)                                      │
│  ❌ Statistike i izveštaji                                                  │
│  ❌ PWA offline podrška                                                     │
│  ❌ Korisnički profili                                                      │
│  ❌ Approval workflow                                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Arhitektura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POC ARHITEKTURA                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│     KORISNIK                                                                │
│         │                                                                   │
│         ▼                                                                   │
│  ┌─────────────┐      POST /alarm       ┌─────────────┐                     │
│  │  WEB APP    │ ──────────────────────►│   CONVEX    │                     │
│  │  (React)    │◄─────────────────────► │  (Backend)  │                     │
│  │             │      Real-time         └──────┬──────┘                     │
│  │  • Panic    │      subscription             │                            │
│  │  • Status   │                               │ HTTP                       │
│  │  • Preuzmi  │                               ▼                            │
│  └─────────────┘                        ┌─────────────┐                     │
│                                         │  TELEGRAM   │                     │
│                                         │    GRUPA    │                     │
│                                         └─────────────┘                     │
│                                                                             │
│  AUTH: Jedan PIN za celu školu (hardcode ili env var)                       │
│  HOSTING: Vercel (free) + Convex (free)                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Autentifikacija

### Pristup sistemu

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  POC AUTENTIFIKACIJA                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  • Jedan zajednički PIN za celu školu                                       │
│    Primer: "kovacic2025"                                                    │
│                                                                             │
│  • PIN se deli:                                                             │
│    - Na roditeljskom sastanku                                               │
│    - Lično od admina grupe                                                  │
│    - Preko privatne poruke                                                  │
│                                                                             │
│  • Čuvanje:                                                                 │
│    - PIN se čuva u localStorage                                             │
│    - Korisnik ne mora ponovo da ga unosi                                    │
│                                                                             │
│  • Korisnik dodatno unosi:                                                  │
│    - Ime (opciono, pamti se za sledeći put)                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Ograničenja i rizici

| Rizik | Mitigacija |
|-------|------------|
| Bilo ko sa PIN-om može sve | PIN se deli samo pouzdanim osobama |
| Nema verifikacije identiteta | Za PoC je to prihvatljivo |
| PIN može da "procuri" | PIN se može promeniti |

---

## Ograničenja PoC-a

### Funkcionalna ograničenja

1. **Jedna škola** - Sistem je hardcoded za jednu školu
2. **Nema uloga** - Svi korisnici imaju iste mogućnosti
3. **Nema smena** - Svi dobijaju sve notifikacije
4. **Nema eskalacije** - Ako niko ne reaguje, ništa se ne dešava
5. **Nema statistika** - Podaci se ne analiziraju

### Tehnička ograničenja

1. **Hardcoded Telegram CHAT_ID** - Jedna grupa
2. **Nema offline podrške** - Zahteva internet
3. **Osnovni error handling** - Minimalan
4. **Nema audit log-a** - Akcije se ne loguju

### Bezbednosna ograničenja

1. **Zajednički PIN** - Svi koriste isti
2. **Nema rate limiting-a** - Moguća zloupotreba
3. **Nema bruteforce zaštite** - PIN se može pogađati

---

## Šta PoC NIJE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  POC NIJE:                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ✗ Produkciono rešenje                                                      │
│  ✗ Skalabilna platforma                                                     │
│  ✗ Zamena za policiju                                                       │
│  ✗ Garancija bezbednosti                                                    │
│  ✗ Kompletna aplikacija                                                     │
│                                                                             │
│  PoC je DOKAZ KONCEPTA - testira da li osnovna ideja funkcioniše            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Ciljna grupa za PoC

- **5-10 pouzdanih roditelja** koji su voljni da testiraju
- Roditelji koji razumeju da je ovo rana verzija
- Tehnički pismene osobe koje mogu dati konstruktivan feedback

---

## Kriterijumi uspeha

| Kriterijum | Mera |
|------------|------|
| Alarm se uspešno šalje | ✓/✗ |
| Telegram prima notifikaciju | ✓/✗ |
| Real-time update radi | ✓/✗ |
| "Preuzimam" funkcioniše | ✓/✗ |
| Korisnici razumeju interfejs | Feedback |
| Nema kritičnih bug-ova | 0 blocker-a |

---

## Sledeći koraci nakon PoC-a

```
POC USPEŠAN?
    │
    ├── DA ──► Nastavak na MVP
    │          • SMS verifikacija
    │          • RESPONDER uloga
    │          • Smene
    │          • Eskalacija
    │
    └── NE ──► Analiza i pivotiranje
               • Šta nije radilo?
               • Da li koncept ima smisla?
               • Potrebne izmene
```

---

## Vremenska procena

| Aktivnost | Vreme |
|-----------|-------|
| Telegram bot setup | 10 min |
| Convex backend | 30 min |
| React frontend | 45 min |
| Deploy | 15 min |
| Testiranje | 20 min |
| **UKUPNO** | **~2 sata** |

---

*Dokument kreiran: Januar 2026*
