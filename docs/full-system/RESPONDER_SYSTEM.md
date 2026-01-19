# Responder Sistem

## Pregled

Responder sistem obuhvata:
1. **Sistem smena** - ko je dostupan kada
2. **"Preuzimam" mehanika** - kako se preuzima alarm
3. **Bystander effect mitigacija** - sprečavanje "niko ne reaguje"
4. **Statistike** - praćenje angažmana

---

## Sistem smena

### Vremenski slotovi

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      STANDARDNI SLOTOVI                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   JUTRO (07:00 - 09:00)                                                     │
│   └── Dolazak u školu                                                       │
│                                                                             │
│   RUČAK (12:00 - 14:00)                                                     │
│   └── Pauza, odlazak kući za neke                                           │
│                                                                             │
│   POSLE (14:00 - 17:00)                                                     │
│   └── Kraj nastave, odlazak kući                                            │
│                                                                             │
│   Slotovi su konfigurabilan po grupi                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Nedeljni raspored - Admin view

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     NEDELJNI RASPORED                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│            PON    UTO    SRE    ČET    PET    SUB    NED                    │
│  ┌────────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐               │
│  │ 07-09  │ 👤👤 │ 👤   │ 👤👤 │ 👤   │ 👤👤 │  -   │  -   │               │
│  │ JUTRO  │      │ ⚠️   │      │ ⚠️   │      │      │      │               │
│  ├────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤               │
│  │ 12-14  │ 👤👤 │ 👤👤 │ 👤   │ 👤👤 │ 👤👤 │  -   │  -   │               │
│  │ RUČAK  │      │      │ ⚠️   │      │      │      │      │               │
│  ├────────┼──────┼──────┼──────┼──────┼──────┼──────┼──────┤               │
│  │ 14-17  │ 👤👤 │ 👤👤 │ 👤👤 │ 👤👤 │ 👤   │  -   │  -   │               │
│  │ POSLE  │ 👤   │      │      │      │ ⚠️   │      │      │               │
│  └────────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘               │
│                                                                             │
│  👤 = Responder prijavljen                                                  │
│  ⚠️ = Manje od 2 respondera (potrebno popuniti)                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Prijava dostupnosti - Responder view

```
┌──────────────────────────────────────┐
│  Moja dostupnost ove nedelje         │
│                                      │
│  PONEDELJAK                          │
│  ☐ 07-09 (Jutro)                     │
│  ☐ 12-14 (Ručak)                     │
│  ☑️ 14-17 (Posle)                     │
│                                      │
│  UTORAK                              │
│  ☐ 07-09 (Jutro)                     │
│  ☑️ 12-14 (Ručak)                     │
│  ☐ 14-17 (Posle)                     │
│                                      │
│  ...                                 │
│                                      │
│  [SAČUVAJ]                           │
└──────────────────────────────────────┘
```

### Data model

```typescript
// shifts tabela
{
  membership_id: Id<"memberships">,
  group_id: Id<"groups">,

  date: string,           // "2025-01-20"
  time_slot: "MORNING" | "MIDDAY" | "AFTERNOON",

  // Ili custom vreme
  custom_start: string | null,  // "08:30"
  custom_end: string | null,    // "09:30"

  status: "SCHEDULED" | "ACTIVE" | "COMPLETED" | "MISSED",
}
```

---

## "Preuzimam" mehanika

### Tok preuzimanja

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PREUZIMANJE ALARMA                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. Responder vidi alarm i klikne "PREUZIMAM"                              │
│                                                                             │
│   2. Sistem beleži:                                                         │
│      • Vreme reakcije                                                       │
│      • Lokaciju respondera                                                  │
│      • Kalkuliše ETA                                                        │
│                                                                             │
│   3. Status alarma: ACTIVE → RESPONDING                                     │
│                                                                             │
│   4. Svi vide: "Marko P. ide, ETA ~4 min"                                   │
│                                                                             │
│   5. Drugi responderi mogu kliknuti "I ja idem" (backup)                    │
│                                                                             │
│   6. Zakazana eskalacija se OTKAZUJE                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Interface za respondera koji je preuzeo

```
┌─────────────────────────────────────────────┐
│  🏃 Na putu si                               │
│                                             │
│  📍 Navigacija: [OTVORI MAPU]               │
│  📞 Pozovi dete: 063/xxx-xxx                │
│  📞 Pozovi roditelja: 064/xxx-xxx           │
│                                             │
│  [✅ STIGAO SAM]  [❌ ODUSTAJEM]            │
└─────────────────────────────────────────────┘
```

### Ako responder odustane

```
Tok:
1. Klik na "ODUSTAJEM"
2. Alarm se vraća u prethodni status (ACTIVE/ESCALATED)
3. Eskalacija kreće ispočetka
4. Beleži se kao "DECLINED" u statistikama
```

---

## Bystander Effect Mitigacija

### Problem

**Bystander effect**: Kada više ljudi vidi incident, svako pretpostavlja da će neko drugi reagovati, pa na kraju niko ne reaguje.

### Rešenje: Psihološke tehnike

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  BYSTANDER EFFECT MITIGACIJA                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. DIREKTNA ODGOVORNOST                                                    │
│     "Ti si 1 od 3 dostupna respondera"                                      │
│     Ne "poslato svima" nego personalizovano                                 │
│                                                                             │
│  2. VIDLJIV NEDOSTATAK AKCIJE                                               │
│     "Videlo: 3   Preuzelo: 0"                                               │
│     Prikazuje da NIKO nije reagovao                                         │
│                                                                             │
│  3. IMENOVANJE                                                              │
│     "Marko, Jovan i Petra su na smeni"                                      │
│     Imena, ne anonimna masa                                                 │
│                                                                             │
│  4. BLIZINA KAO PRIORITET                                                   │
│     "Ti si NAJBLIŽI (300m)"                                                 │
│     Jasna implikacija da si ti odgovoran                                    │
│                                                                             │
│  5. VREMENSKA URGENCIJA                                                     │
│     Countdown: "Eskalacija za 47 sekundi"                                   │
│     Stvara pritisak za brzu akciju                                          │
│                                                                             │
│  6. NEGATIVNA POTVRDA                                                       │
│     Mora kliknuti "NE MOGU" sa razlogom                                     │
│     Ne može samo ignorisati                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Implementacija u UI

```
┌─────────────────────────────────────────┐
│  🚨 ALARM                               │
│                                         │
│  Ti si NAJBLIŽI od 2 dostupna          │
│  respondera.                            │
│                                         │
│  👁️ Videlo: 2    ✅ Preuzelo: 0         │
│  ⏱️ Eskalacija za: 0:47                 │
│                                         │
│  [🏃 PREUZIMAM]                         │
│                                         │
│  [Ne mogu jer: ▼___________________]    │
│    • Na poslu sam                       │
│    • Nisam u blizini                    │
│    • Zdravstveni razlog                 │
│    • Drugo                              │
└─────────────────────────────────────────┘
```

### Razlozi za "Ne mogu"

| Razlog | Validnost |
|--------|-----------|
| Na poslu sam | Validno |
| Nisam u blizini | Validno |
| Zdravstveni razlog | Validno |
| Već sam na drugom alarmu | Validno |
| Drugo (custom tekst) | Validno |

Sistem beleži razloge za kasniju analizu (npr. ako neko stalno ima iste izgovore).

---

## Statistike respondera

### Profil respondera (vidljiv adminu)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    RESPONDER PROFIL                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   👤 Marko Petrović                                                         │
│   📱 +381 63 xxx xxxx                                                       │
│   📅 Član od: Januar 2026                                                   │
│                                                                             │
│   OVOG MESECA:                                                              │
│   ├── Smena prijavljenih:     12                                            │
│   ├── Alarma primljeno:       3                                             │
│   ├── Preuzeto:               2  ✅                                         │
│   ├── Opravdano odbijeno:     1                                             │
│   └── Ignorisano:             0  ✅                                         │
│                                                                             │
│   Prosečno vreme reakcije:    34 sekunde                                    │
│                                                                             │
│   ────────────────────────────────────────────────────────────────────      │
│                                                                             │
│   ISTORIJA (poslednja 3 meseca):                                            │
│   • Jan: 12 smena, 2/3 preuzeto                                             │
│   • Dec: 10 smena, 3/3 preuzeto                                             │
│   • Nov: 8 smena, 1/2 preuzeto                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Šta se prati

| Metrika | Opis |
|---------|------|
| Smene prijavljene | Koliko smena je responder prijavio |
| Smene odradjene | Koliko smena je zaista bio aktivan |
| Alarmi primljeni | Koliko alarma je dobio |
| Alarmi prihvaćeni | Koliko je preuzeo |
| Alarmi odbijeni | Koliko je kliknuo "Ne mogu" |
| Alarmi ignorisani | Koliko nije reagovao uopšte |
| Prosečno vreme reakcije | Prosek vremena do klika "Preuzimam" |

### Svrha statistika

1. **Accountability** - Admin vidi ko je aktivan, ko nije
2. **Motivacija** - Responder vidi svoj doprinos
3. **Identifikacija problema** - Npr. neko "skuplja smene" a ne reaguje
4. **Nije za kažnjavanje** - Volonterski sistem, bez pritiska

---

## Podsjetnici

### Automatski podsjetnici

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PODSJETNICI                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  NEDELJNA PRIJAVA                                                           │
│  Svake nedelje (npr. nedelja uveče):                                        │
│  "Popunite dostupnost za sledeću nedelju!"                                  │
│                                                                             │
│  NEPOPUNJENE SMENE                                                          │
│  Ako slot ima <2 respondera:                                                │
│  "Sreda 14-17h nema dovoljno respondera. Možete li pomoći?"                 │
│                                                                             │
│  DAN UNAPRED                                                                │
│  "Sutra ste na smeni: 14:00-17:00"                                          │
│                                                                             │
│  POČETAK SMENE                                                              │
│  "Vaša smena počinje za 15 minuta"                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Implementacija (Convex scheduled functions)

```typescript
// Nedeljni podsjetnik
export const weeklyReminder = internalMutation({
  handler: async (ctx) => {
    // Svake nedelje u 18h
    const groups = await ctx.db.query("groups").collect();

    for (const group of groups) {
      const responders = await getResponders(group._id);

      for (const responder of responders) {
        await sendNotification(responder.user_id, {
          title: "Popunite dostupnost",
          body: "Prijavite smene za sledeću nedelju",
        });
      }
    }
  },
});
```

---

## Backup mehanizam

### "I ja idem" funkcionalnost

Kada neko već preuzme alarm, drugi mogu kliknuti "I ja idem":

```
┌─────────────────────────────────────────────┐
│  🚨 ALARM                                   │
│                                             │
│  ✅ Marko je preuzeo (ETA 4 min)            │
│                                             │
│  [🏃 I JA IDEM]                             │
│  (backup, ako je situacija ozbiljna)        │
│                                             │
└─────────────────────────────────────────────┘
```

### Kada koristiti backup

- Situacija zvuči ozbiljna (fizički napad)
- Primarni responder je daleko
- Alarm se ponavlja na istoj lokaciji

---

## Edge case-ovi

| Situacija | Ponašanje |
|-----------|-----------|
| Alarm noću (van svih smena) | Odmah broadcast svim responderima |
| Responder je sam u problemu | Može i RESPONDER da pošalje PANIC |
| Svi odbiju | Eskalacija na sve članove |
| Niko nije prijavio smene | Notifikacija adminu |

---

*Dokument kreiran: Januar 2026*
