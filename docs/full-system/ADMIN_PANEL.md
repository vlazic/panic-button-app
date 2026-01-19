# Admin Panel

## Pregled

Admin panel omogućava administratorima grupe da upravljaju članovima, smenama, alarmima i podešavanjima.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🏫 OŠ Ivan Goran Kovačić                                    [⚙️ Podešavanja]│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│   │   ALARMI    │  │   ČLANOVI   │  │   SMENE     │  │  STATISTIKA │        │
│   │   Danas: 0  │  │   32 / 3    │  │  ✅ 2 sad   │  │  Ovaj mesec │        │
│   │   ✅ Mirno  │  │  aktivnih   │  │  ⚠️ Sre 2   │  │   [OTVORI]  │        │
│   └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Upravljanje članovima

### Pregled članova

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ČLANOVI GRUPE                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Filter: [Svi ▼] [Aktivni ▼]    Pretraga: [_______________] 🔍              │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  IME              ULOGA       STATUS    ČLAN OD     AKCIJE            │  │
│  │  ─────────────────────────────────────────────────────────────────    │  │
│  │  Marko Petrović   RESPONDER   ✅ Aktivan  Jan 2025   [···]            │  │
│  │  Ana Jovanović    RODITELJ    ✅ Aktivan  Jan 2025   [···]            │  │
│  │  Petar Nikolić    RESPONDER   ⚠️ Suspend  Dec 2024   [···]            │  │
│  │  Mila Marković    DETE        ✅ Aktivan  Jan 2025   [···]            │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  Prikazano 4 od 32 članova                                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Akcije za člana

```
┌─────────────────────────────────────┐
│  Marko Petrović                     │
│  RESPONDER · Aktivan                │
├─────────────────────────────────────┤
│                                     │
│  [Promeni ulogu]                    │
│    → RODITELJ                       │
│    → ADMIN                          │
│                                     │
│  [Pogledaj aktivnost]               │
│                                     │
│  [Suspenduj]                        │
│                                     │
│  [Ukloni iz grupe]                  │
│                                     │
└─────────────────────────────────────┘
```

### Zahtevi za pristup

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ⏳ ZAHTEVI ZA PRISTUP (2)                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  👤 Jelena Nikolić                                                    │  │
│  │  📞 +381 64 555 1234                                                  │  │
│  │  📅 Zahtev: Pre 2 sata                                                │  │
│  │                                                                       │  │
│  │  [✅ ODOBRI]  [❌ ODBIJ]  [❓ PITAJ ZA VIŠE INFO]                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Detalji člana

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  👤 Marko Petrović                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OSNOVNI PODACI                                                             │
│  📞 +381 63 123 4567                                                        │
│  📅 Član od: 15. Januar 2026                                                │
│  🔑 Odobrio: Admin Grupe                                                    │
│  📊 Uloga: RESPONDER                                                        │
│                                                                             │
│  STATISTIKA (kao RESPONDER)                                                 │
│  ├── Smena prijavljenih ovog meseca: 12                                     │
│  ├── Alarma primljeno: 3                                                    │
│  ├── Alarma prihvaćeno: 2 (67%)                                             │
│  ├── Alarma odbijeno: 1                                                     │
│  └── Prosečno vreme reakcije: 34 sek                                        │
│                                                                             │
│  POSLEDNJA AKTIVNOST                                                        │
│  • 18.01. 14:35 - Preuzeo alarm #127                                        │
│  • 15.01. 08:20 - Prijavio smenu (Pon 14-17)                                │
│  • 12.01. 16:50 - Preuzeo alarm #124                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Upravljanje smenama

### Nedeljni pregled

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  RASPORED SMENA - Ova nedelja                     [◀ Prethodna] [Sledeća ▶]│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│            PON 20    UTO 21    SRE 22    ČET 23    PET 24                   │
│  ┌────────┬─────────┬─────────┬─────────┬─────────┬─────────┐               │
│  │ 07-09  │ Marko   │ Ana     │ Marko   │ Petar   │ Ana     │               │
│  │ JUTRO  │ Ana     │ ⚠️ 1    │ Jovan   │ Marko   │ Marko   │               │
│  ├────────┼─────────┼─────────┼─────────┼─────────┼─────────┤               │
│  │ 12-14  │ Jovan   │ Marko   │ Ana     │ Ana     │ Petar   │               │
│  │ RUČAK  │ Petar   │ Petar   │ ⚠️ 1    │ Jovan   │ Jovan   │               │
│  ├────────┼─────────┼─────────┼─────────┼─────────┼─────────┤               │
│  │ 14-17  │ Ana     │ Jovan   │ Petar   │ Marko   │ ⚠️ 1    │               │
│  │ POSLE  │ Marko   │ Ana     │ Marko   │ Petar   │         │               │
│  │        │ Jovan   │         │         │         │         │               │
│  └────────┴─────────┴─────────┴─────────┴─────────┴─────────┘               │
│                                                                             │
│  ⚠️ = Manje od 2 respondera (treba popuniti)                                │
│                                                                             │
│  [POŠALJI PODSETNIK]  [PODESI SLOTOVE]                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Akcije

- **Pošalji podsetnik** - Notifikacija svim responderima da popune smene
- **Podesi slotove** - Custom vremena za školu

---

## 3. Istorija alarma

### Lista alarma

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ISTORIJA ALARMA                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Filter: [Svi ▼]  Period: [Ovaj mesec ▼]    [IZVEZI CSV]  [IZVEZI PDF]     │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  DATUM/VREME      LOKACIJA      STATUS       REAKCIJA    RESPONDER   │  │
│  │  ─────────────────────────────────────────────────────────────────    │  │
│  │  18.01. 14:32     Kod fontane   ✅ Rešeno    47 sek     M. Petrović  │  │
│  │  15.01. 08:15     Iza škole     ✅ Rešeno    1:23       A. Jovanović │  │
│  │  12.01. 16:45     Park          ⚠️ Lažni     32 sek     P. Nikolić   │  │
│  │  08.01. 13:20     Prodavnica    ✅ Rešeno    2:05       M. Petrović  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Detalji alarma

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ALARM #127 - 18. Januar 2026, 14:32                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OSNOVNI PODACI                                                             │
│  📍 Lokacija: 44.8125, 20.4612 (Kod fontane)                                │
│  👤 Prijavio: Dete M. (DETE)                                                │
│  💬 Opis: "Grupa od 4-5 starijih, prate me"                                 │
│                                                                             │
│  TIMELINE                                                                   │
│  14:32:00  Alarm kreiran                                                    │
│  14:32:02  Notifikovani: Marko P., Ana J. (na smeni)                        │
│  14:32:15  Marko P. video alarm                                             │
│  14:32:21  Ana J. videla alarm                                              │
│  14:32:47  Marko P. preuzeo (udaljenost: 350m)                              │
│  14:36:12  Marko P. stigao na lokaciju                                      │
│  14:41:33  Alarm označen kao REŠEN                                          │
│                                                                             │
│  REZOLUCIJA                                                                 │
│  Tip: Sprovedeno do kuće                                                    │
│  Napomena: "Dete sprovedeno do kuće, grupa se razišla pri našem dolasku"   │
│                                                                             │
│  [📋 KOPIRAJ ZA IZVEŠTAJ]  [🗺️ VIDI NA MAPI]                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Podešavanja grupe

### Osnovna podešavanja

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PODEŠAVANJA GRUPE                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OSNOVNO                                                                    │
│  Ime grupe:    [ OŠ Ivan Goran Kovačić_____________ ]                       │
│  Opis:         [ Roditelji učenika 1-8 razred_______ ]                      │
│                                                                             │
│  PRISTUP                                                                    │
│  Trenutni PIN: 482916  [🔄 GENERIŠI NOVI]                                   │
│  Automatska rotacija: [ Svake nedelje ▼ ]                                   │
│                                                                             │
│  ESKALACIJA                                                                 │
│  Prva eskalacija:  [ 90  ] sekundi                                          │
│  Druga eskalacija: [ 60  ] sekundi (nakon prve)                             │
│  Min respondera:   [ 2   ] po smeni                                         │
│                                                                             │
│  RADNO VREME                                                                │
│  Aktivno od: [ 07:00 ] do: [ 18:00 ]                                        │
│  Vikend:     [ ☐ Uključen ]                                                 │
│                                                                             │
│  [SAČUVAJ PROMENE]                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Telegram podešavanja

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  TELEGRAM INTEGRACIJA                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Status: ✅ Povezano                                                        │
│  Grupa:  "OŠ Kovačić Patrola"                                              │
│  Od:     15. Januar 2026                                                    │
│                                                                             │
│  [TEST KONEKCIJE]                                                          │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────     │
│                                                                             │
│  [PREKINI VEZU]  [PROMENI GRUPU]                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Zona škole (Geofence)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ZONA ŠKOLE                                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                                                                       │  │
│  │                    [MAPA SA KRUGOM]                                   │  │
│  │                                                                       │  │
│  │                         🏫                                            │  │
│  │                        ╱   ╲                                          │  │
│  │                      ╱       ╲   ← radius                             │  │
│  │                    ╱    ○      ╲                                      │  │
│  │                      ╲       ╱                                        │  │
│  │                        ╲   ╱                                          │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  Centar: 44.8125, 20.4612                                                   │
│  Radius: [ 500 ] metara                                                     │
│                                                                             │
│  [POSTAVI NA MAPI]                                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Opasne operacije

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ⚠️ OPASNA ZONA                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  [OBRIŠI GRUPU]                                                            │
│                                                                             │
│  Brisanje grupe je NEPOVRATNO i obuhvata:                                   │
│  • Sve članove                                                              │
│  • Sve alarme                                                               │
│  • Sve statistike                                                           │
│  • Telegram konekciju                                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Responsivnost

Admin panel je optimizovan za:
- Desktop (puna funkcionalnost)
- Tablet (prilagođeni layout)
- Mobile (osnovne funkcije)

---

*Dokument kreiran: Januar 2026*
