# Uloge i Dozvole

## Pregled uloga

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              ULOGE                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ADMIN                                                                     │
│   ─────                                                                     │
│   • Puna kontrola nad grupom                                                │
│   • Odobrava/odbija nove članove                                            │
│   • Može da menja uloge drugih                                              │
│   • Pristup statistikama                                                    │
│   • Upravlja podešavanjima                                                  │
│                                                                             │
│   RODITELJ (default)                                                        │
│   ────────────────                                                          │
│   • Može da šalje alarme za svoje dete                                      │
│   • Prima notifikacije o alarmima                                           │
│   • NE očekuje se fizička reakcija                                          │
│   • Vidi status alarma                                                      │
│                                                                             │
│   RESPONDER (opt-in)                                                        │
│   ─────────────────                                                         │
│   • Sve što i RODITELJ                                                      │
│   • Prijavljuje smene                                                       │
│   • Očekuje se da reaguje na alarme                                         │
│   • Ima "Preuzimam" dugme                                                   │
│   • Vidi statistiku svog angažmana                                          │
│                                                                             │
│   DETE (ograničen)                                                          │
│   ────────────────                                                          │
│   • SAMO panic button                                                       │
│   • Ne vidi tuđe alarme                                                     │
│   • Ne vidi detalje članova                                                 │
│   • Povezan sa roditeljem                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Matrica dozvola

| Akcija | ADMIN | RODITELJ | RESPONDER | DETE |
|--------|-------|----------|-----------|------|
| Slanje alarma | ✅ | ✅ | ✅ | ✅ |
| Pregled alarma | ✅ | ✅ | ✅ | ❌ |
| Preuzimanje alarma | ✅ | ❌ | ✅ | ❌ |
| Razrešenje alarma | ✅ | ❌ | ✅ | ❌ |
| Prijava smena | ✅ | ❌ | ✅ | ❌ |
| Pregled članova | ✅ | ⚡ | ⚡ | ❌ |
| Approve članova | ✅ | ❌ | ❌ | ❌ |
| Promena uloga | ✅ | ❌ | ❌ | ❌ |
| Suspend/Ban | ✅ | ❌ | ❌ | ❌ |
| Podešavanja grupe | ✅ | ❌ | ❌ | ❌ |
| Telegram setup | ✅ | ❌ | ❌ | ❌ |
| Statistike | ✅ | ❌ | ⚡ | ❌ |
| Export izveštaja | ✅ | ❌ | ❌ | ❌ |

**Legenda:** ✅ Da | ❌ Ne | ⚡ Ograničeno

---

## ADMIN

### Opis

Administrator grupe ima **punu kontrolu** nad grupom i njenim članovima.

### Kako se postaje ADMIN

1. **Kreiranje nove grupe** → automatski postaje ADMIN
2. **Promocija od strane postojećeg ADMIN-a**

### Mogućnosti

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ADMIN MOGUĆNOSTI                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  UPRAVLJANJE ČLANOVIMA                                                      │
│  ├── Pregled svih članova sa statusima                                      │
│  ├── Approve / Reject zahteva za pristup                                    │
│  ├── Promena uloge: RODITELJ ↔ RESPONDER                                    │
│  ├── Suspend člana (privremeno)                                             │
│  ├── Ban člana (trajno)                                                     │
│  └── Promocija u ADMIN (max 3 admina)                                       │
│                                                                             │
│  PODEŠAVANJA GRUPE                                                          │
│  ├── Ime i opis                                                             │
│  ├── PIN (pregled i regeneracija)                                           │
│  ├── Timeout-i eskalacije                                                   │
│  ├── Radno vreme (aktivne sate)                                             │
│  └── Zona škole (geofence)                                                  │
│                                                                             │
│  TELEGRAM                                                                   │
│  ├── Povezivanje grupe                                                      │
│  ├── Test konekcije                                                         │
│  └── Prekid veze                                                            │
│                                                                             │
│  STATISTIKE                                                                 │
│  ├── Mesečni izveštaji                                                      │
│  ├── Istorija alarma                                                        │
│  └── Export PDF/CSV                                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Ograničenja

- Max 3 ADMIN-a po grupi
- Ne može sam sebe da degradira ako je jedini ADMIN
- Potrebna potvrda za kritične akcije (ban, brisanje grupe)

---

## RODITELJ

### Opis

Standardna uloga za roditelje učenika. Može da šalje alarme, ali se ne očekuje da fizički reaguje.

### Kako se postaje RODITELJ

1. SMS verifikacija
2. Unos PIN-a grupe
3. **Odobrenje od ADMIN-a**

### Mogućnosti

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  RODITELJ MOGUĆNOSTI                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ALARMI                                                                     │
│  ├── Slanje panic alarma                                                    │
│  ├── Otkazivanje sopstvenog alarma                                          │
│  └── Pregled aktivnih alarma                                                │
│                                                                             │
│  PROFIL                                                                     │
│  ├── Promena imena                                                          │
│  └── Notifikacije on/off                                                    │
│                                                                             │
│  DETE                                                                       │
│  ├── Kreiranje naloga za dete                                               │
│  └── Praćenje alarma od deteta                                              │
│                                                                             │
│  PREGLED                                                                    │
│  ├── Lista članova (samo imena)                                             │
│  └── Ko je na smeni (bez detalja)                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Ograničenja

- Ne može da preuzima alarme
- Ne može da vidi statistike
- Ne može da menja podešavanja grupe

---

## RESPONDER

### Opis

Volonter koji je spreman da fizički reaguje na alarme. Ima sve mogućnosti RODITELJA plus sistem smena i preuzimanje alarma.

### Kako se postaje RESPONDER

1. Prvo biti RODITELJ (verified član)
2. Opt-in: izabere "Želim da budem Responder"
3. Admin odobrava (opciono)

### Mogućnosti

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  RESPONDER MOGUĆNOSTI                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SVE ŠTO I RODITELJ, PLUS:                                                  │
│                                                                             │
│  SMENE                                                                      │
│  ├── Prijava dostupnosti po danima/slotovima                                │
│  ├── Pregled rasporeda                                                      │
│  └── Otkazivanje prijavljene smene                                          │
│                                                                             │
│  ALARMI                                                                     │
│  ├── "Preuzimam" dugme                                                      │
│  ├── "Stigao sam" potvrda                                                   │
│  ├── Razrešenje alarma                                                      │
│  └── Navigacija do lokacije                                                 │
│                                                                             │
│  STATISTIKE (sopstvene)                                                     │
│  ├── Broj preuzetih alarma                                                  │
│  ├── Prosečno vreme reakcije                                                │
│  └── Broj odradjenih smena                                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Odgovornosti

- Prijavljuje dostupnost unapred
- Reaguje na alarme kada je na smeni
- Mora dati razlog ako ne može da reaguje

---

## DETE

### Opis

Ograničen nalog za učenike. Ima pristup samo panic button-u i ništa više.

### Kako se kreira DETE nalog

1. RODITELJ kreira nalog za dete
2. Dete dobija svoj PIN za pristup
3. Nalog je automatski povezan sa roditeljem

### Mogućnosti

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  DETE MOGUĆNOSTI                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ✅ Panic button                                                            │
│  ✅ Pregled statusa sopstvenog alarma                                       │
│  ✅ Otkazivanje sopstvenog alarma                                           │
│                                                                             │
│  ❌ Pregled tuđih alarma                                                    │
│  ❌ Pregled članova grupe                                                   │
│  ❌ Pregled ko je na smeni                                                  │
│  ❌ Bilo kakva podešavanja                                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Privatnost

- Dete NE vidi lokacije drugih alarma
- Dete NE vidi ko je preuzeo alarm (osim svog)
- Roditelj dobija notifikaciju ako dete pošalje alarm

---

## Promena uloga

### RODITELJ → RESPONDER

```
Tok:
1. RODITELJ klikne "Želim da budem Responder"
2. (Opciono) Admin odobrava
3. Status se menja na RESPONDER
4. Dobija pristup smenama i preuzimanju
```

### RESPONDER → RODITELJ

```
Tok:
1. RESPONDER klikne "Ne želim više da budem Responder"
2. Status se menja na RODITELJ
3. Gubi pristup smenama i preuzimanju
4. Postojeće prijavljene smene se brišu
```

### Promocija u ADMIN

```
Tok:
1. Postojeći ADMIN bira člana
2. Klikne "Promoviši u Admin"
3. Potvrda (2FA ili PIN)
4. Član dobija ADMIN pristup
```

---

## Status članstva

| Status | Opis | Mogućnosti |
|--------|------|------------|
| `PENDING` | Čeka odobrenje | Nema pristup |
| `ACTIVE` | Aktivan član | Sve po ulozi |
| `SUSPENDED` | Privremeno suspendovan | Nema pristup |
| `BANNED` | Trajno uklonjen | Nema pristup |

### Flow statusa

```
        ┌───────────┐
        │  PENDING  │
        └─────┬─────┘
              │ Admin odobri
              ▼
        ┌───────────┐
        │  ACTIVE   │
        └─────┬─────┘
              │
    ┌─────────┴─────────┐
    │                   │
    ▼                   ▼
┌───────────┐     ┌───────────┐
│ SUSPENDED │────►│  BANNED   │
└───────────┘     └───────────┘
    │
    │ Admin vrati
    ▼
┌───────────┐
│  ACTIVE   │
└───────────┘
```

---

## Data model

```typescript
// Membership entity
{
  user_id: Id<"users">,
  group_id: Id<"groups">,

  role: "ADMIN" | "RODITELJ" | "RESPONDER" | "DETE",

  status: "PENDING" | "ACTIVE" | "SUSPENDED" | "BANNED",

  joined_at: number,
  approved_by: Id<"users"> | null,

  // Za DETE - povezan roditelj
  parent_user_id: Id<"users"> | null,

  // Statistike (za RESPONDER)
  stats: {
    alarms_received: number,
    alarms_accepted: number,
    alarms_declined: number,
    alarms_ignored: number,
    avg_response_time_sec: number,
  } | null,
}
```

---

*Dokument kreiran: Januar 2026*
