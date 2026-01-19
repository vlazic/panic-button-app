# Patrola - Panic Button za Bezbednost Dece

## Problem

Deca u okolini škole OŠ Ivan Goran Kovačić (Beograd, kod Vukovog spomenika) su izložena pretnjama, uznemiravanju i pljačkanju od strane lokalne grupe maloletnika.

**Roditelji su se organizovali u patrole, ali nedostaje im alat za brzu koordinaciju.**

---

## Rešenje - Jednostavno

Aplikacija sa jednim crvenim dugmetom.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│                                                                                 │
│                          ┌───────────────────┐                                  │
│                          │                   │                                  │
│                          │      PANIC        │                                  │
│                          │                   │                                  │
│                          │   (drži 3 sek)    │                                  │
│                          │                   │                                  │
│                          └───────────────────┘                                  │
│                                                                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Šta se desi kada dete pritisne dugme:**

1. Aplikacija šalje GPS lokaciju
2. Telegram grupa dobija notifikaciju
3. Najbliži roditelj kaže "Idem!" i kreće

**To je to.** Cilj je da pomoć stigne za par minuta umesto da dete zove, objašnjava gde je, čeka da neko reaguje...

---

## Faza 1: PoC (Proof of Concept)

**Šta je PoC?** Minimalna verzija koja radi osnovnu stvar - šalje alarm i omogućava preuzimanje.

### Šta PoC ima

- ✅ Panic dugme (drži 3 sekunde da se aktivira)
- ✅ Automatsko slanje GPS lokacije
- ✅ Telegram notifikacija celoj grupi
- ✅ "Preuzimam" dugme - svi vide ko ide
- ✅ Lista aktivnih alarma
- ✅ Jedan zajednički PIN za celu školu

### Šta PoC NEMA

- ❌ Registraciju i profile korisnika
- ❌ Smene i rasporede
- ❌ Admin panel
- ❌ Statistike
- ❌ Više škola/grupa

### Kako izgleda korišćenje

```
DETE:                           RODITELJI (Telegram):
─────                           ────────────────────

1. Otvori aplikaciju
   (PIN se unosi samo prvi put,
    posle app pamti)
2. Drži crveno dugme 3 sek      → Notifikacija stiže

                                3. Marko vidi: "ALARM! Kod fontane"
                                4. Marko klikne link → otvori app
                                5. Marko pritisne "Preuzimam"

                                   Svi vide: "Marko ide, ETA 4 min"

6. Dete u app-u vidi: "Marko ide ka tebi"
   (real-time update, automatski)
```

**Zašto 3 sekunde držanja?** Sprečava slučajno aktiviranje - ako slučajno dodirneš dugme, ništa se ne dešava. Moraš namerno držati.

### Za developere: Tehnički detalji (draft)

> ⚠️ **Napomena:** Ovo je predlog arhitekture, nije finalno.

**Stack:**
- **Frontend:** Next.js (React) kao PWA - instalira se na telefon iz browsera
- **Backend:** Convex - serverless baza sa real-time sync
- **Notifikacije:** Telegram Bot API

**Zašto ovaj izbor:**
- PWA ne zahteva objavu u app store (brže do korisnika)
- Convex ima ugrađen real-time (svi vide promene instant)
- Telegram već svi koriste, notifikacije rade pouzdano

**Data model (jedna tabela):**
```
alarms:
  - sender_name (ko je poslao)
  - lat, lng (GPS)
  - status (ACTIVE / TAKEN / RESOLVED)
  - taken_by (ko je preuzeo)
  - created_at, taken_at
```

**Procena troškova:** ~$2/mesec (praktično besplatno na free tier-ovima)

📄 **Detaljna dokumentacija:**
- [poc/OVERVIEW.md](./docs/poc/OVERVIEW.md) - Kompletan pregled PoC-a
- [poc/USER_FLOWS.md](./docs/poc/USER_FLOWS.md) - Detaljni user flow sa wireframe-ovima
- [poc/DATA_MODEL.md](./docs/poc/DATA_MODEL.md) - Schema baze
- [poc/IMPLEMENTATION.md](./docs/poc/IMPLEMENTATION.md) - Struktura koda
- [poc/DEPLOYMENT.md](./docs/poc/DEPLOYMENT.md) - Kako deployovati

---

## Faza 2: Pun Sistem (sledeći korak)

Kada PoC proradi i dobijemo feedback, možemo dodati:

| Funkcionalnost | Opis |
|----------------|------|
| **Registracija** | SMS verifikacija umesto zajedničkog PIN-a |
| **Uloge** | Admin, Roditelj, Responder, Dete - različite mogućnosti |
| **Smene** | Ko je dostupan kada - raspored patrola |
| **Eskalacija** | Ako niko ne reaguje 90 sek → širi se krug obaveštenih |
| **Admin panel** | Upravljanje članovima, pregled alarma, podešavanja |
| **Statistike** | Izveštaji za policiju - gde i kada su incidenti |
| **Više škola** | Svaka škola ima svoju grupu i podešavanja |

### Za developere: Arhitektura punog sistema (draft)

> ⚠️ **Napomena:** Ovo je samo nacrt za dalji razvoj.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│    KORISNICI          FRONTEND           BACKEND            SERVISI             │
│                                                                                 │
│    ┌───────┐         ┌───────┐          ┌───────┐         ┌───────┐            │
│    │ Dete  │────────►│       │          │       │────────►│Telegram│            │
│    └───────┘         │       │          │       │         └───────┘            │
│                      │  PWA  │◄────────►│Convex │                               │
│    ┌───────┐         │(React)│ WebSocket│  DB   │         ┌───────┐            │
│    │Roditelj────────►│       │          │       │────────►│Twilio │            │
│    └───────┘         └───────┘          └───────┘         │ (SMS) │            │
│                                                           └───────┘            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Dodatne tabele:** users, groups, memberships, shifts, alarm_responses, audit_log

**Procena troškova:** $50-100/mesec za 10 škola

📄 **Detaljna dokumentacija:**
- [full-system/OVERVIEW.md](./docs/full-system/OVERVIEW.md) - Pregled punog sistema
- [full-system/ROLES_AND_PERMISSIONS.md](./docs/full-system/ROLES_AND_PERMISSIONS.md) - Sistem uloga
- [full-system/ALARM_LIFECYCLE.md](./docs/full-system/ALARM_LIFECYCLE.md) - Životni ciklus alarma
- [full-system/RESPONDER_SYSTEM.md](./docs/full-system/RESPONDER_SYSTEM.md) - Smene i preuzimanje
- [full-system/TELEGRAM_INTEGRATION.md](./docs/full-system/TELEGRAM_INTEGRATION.md) - Telegram integracija
- [full-system/ADMIN_PANEL.md](./docs/full-system/ADMIN_PANEL.md) - Admin funkcije
- [full-system/STATISTICS.md](./docs/full-system/STATISTICS.md) - Statistike za policiju
- [full-system/SECURITY.md](./docs/full-system/SECURITY.md) - Bezbednost

---

## Dodatna Dokumentacija

### Specifikacije
- [specs/MVP_DEFINITION.md](./docs/specs/MVP_DEFINITION.md) - Šta je PoC vs MVP vs Pun sistem
- [specs/REQUIREMENTS.md](./docs/specs/REQUIREMENTS.md) - Svi zahtevi detaljno
- [specs/FEATURE_COMPARISON.md](./docs/specs/FEATURE_COMPARISON.md) - Uporedni pregled funkcija

### Arhitektura (draft)
- [architecture/OVERVIEW.md](./docs/architecture/OVERVIEW.md) - Dijagrami sistema
- [architecture/DATA_MODEL.md](./docs/architecture/DATA_MODEL.md) - Kompletna schema baze
- [architecture/TECH_STACK.md](./docs/architecture/TECH_STACK.md) - Zašto ove tehnologije
- [architecture/DEPLOYMENT.md](./docs/architecture/DEPLOYMENT.md) - CI/CD i hosting
- [architecture/COST_ANALYSIS.md](./docs/architecture/COST_ANALYSIS.md) - Detaljni troškovi

### Implementacija (za developere)
- [implementation/SETUP_GUIDE.md](./docs/implementation/SETUP_GUIDE.md) - Korak po korak setup
- [implementation/CONVEX_FUNCTIONS.md](./docs/implementation/CONVEX_FUNCTIONS.md) - Backend API
- [implementation/FRONTEND_COMPONENTS.md](./docs/implementation/FRONTEND_COMPONENTS.md) - React komponente
- [implementation/TELEGRAM_BOT.md](./docs/implementation/TELEGRAM_BOT.md) - Telegram bot setup

### Pozadina projekta
- [PROJECT_OVERVIEW.md](./docs/PROJECT_OVERVIEW.md) - Motivacija, ciljevi, kontekst

---

## Otvorena Pitanja

Neke stvari još nisu definisane:

1. **Pravni disclaimer** - Tekst da aplikacija ne garantuje bezbednost
2. **Povezivanje roditelj-dete** - Kako se nalozi povezuju
3. **Postupak za lažne alarme** - Šta sa ponavljačima
4. **Vikendi i praznici** - Da li sistem radi van školskih dana

---

## Sledeći Koraci

1. **Sada:** Napraviti PoC za jednu školu (OŠ Kovačić)
2. **Zatim:** Testirati sa malim brojem roditelja
3. **Feedback:** Sakupiti utiske, šta fali, šta smeta
4. **Iteracija:** Poboljšati na osnovu feedbacka
5. **Skaliranje:** Ako radi - proširiti na pun sistem

---

## Disclaimer

```
Ovaj sistem je volonterski projekat i NE GARANTUJE bezbednost.
Roditelji su i dalje primarno odgovorni za bezbednost svoje dece.
Sistem služi kao dodatna mera koordinacije, ne kao zamena za nadležne organe.
U slučaju ozbiljne opasnosti - UVEK zvati 192 (policija).
```

---

*Poslednje ažuriranje: Januar 2026*
