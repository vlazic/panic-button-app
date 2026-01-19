# Pun Sistem - Pregled

## Šta je "Pun Sistem"?

Pun sistem je **production-ready verzija** Panic Button aplikacije sa svim funkcionalnostima potrebnim za skaliranje na više škola i dugoročno korišćenje.

---

## Ključne karakteristike

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PUN SISTEM - PREGLED                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  GRUPE I KORISNICI                                                          │
│  ─────────────────                                                          │
│  • Više škola/grupa (neograničeno)                                          │
│  • SMS verifikacija korisnika                                               │
│  • 4 uloge: ADMIN, RODITELJ, RESPONDER, DETE                                │
│  • Approval workflow za nove članove                                        │
│  • Povezivanje roditelj-dete                                                │
│                                                                             │
│  ALARM SISTEM                                                               │
│  ────────────                                                               │
│  • 3 nivoa eskalacije (90s → 60s → broadcast)                               │
│  • Ciljane notifikacije (po smeni, po blizini)                              │
│  • "Preuzimam" sa ETA i praćenjem                                           │
│  • "Stigao sam" potvrda                                                     │
│  • Detaljne kategorije razrešenja                                           │
│                                                                             │
│  RESPONDER SISTEM                                                           │
│  ────────────────                                                           │
│  • Nedeljni raspored smena                                                  │
│  • Bystander effect mitigacija                                              │
│  • Statistike po responderu                                                 │
│  • Automatski podsjetnici                                                   │
│                                                                             │
│  ADMIN PANEL                                                                │
│  ───────────                                                                │
│  • Upravljanje članovima (approve/suspend/ban)                              │
│  • Pregled smena i alarma                                                   │
│  • Podešavanja grupe                                                        │
│  • Telegram integracija UI                                                  │
│                                                                             │
│  STATISTIKE I IZVEŠTAJI                                                     │
│  ──────────────────────                                                     │
│  • Mesečni izveštaji                                                        │
│  • Mapa "vrućih tačaka"                                                     │
│  • Trend analiza                                                            │
│  • Export za policiju (PDF/CSV)                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Arhitektura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PUN SISTEM - ARHITEKTURA                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                           ┌─────────────┐                                   │
│                           │  CLOUDFLARE │                                   │
│                           │    (CDN)    │                                   │
│                           └──────┬──────┘                                   │
│                                  │                                          │
│         ┌────────────────────────┼────────────────────────┐                 │
│         │                        │                        │                 │
│         ▼                        ▼                        ▼                 │
│  ┌─────────────┐          ┌─────────────┐          ┌─────────────┐          │
│  │   VERCEL    │          │   CONVEX    │          │  TELEGRAM   │          │
│  │  (Frontend) │◄────────►│  (Backend)  │◄────────►│    BOT      │          │
│  │   Next.js   │  WS/HTTP │  Database   │   HTTP   │  (Webhook)  │          │
│  │   PWA       │          │  Scheduled  │          │             │          │
│  └─────────────┘          └──────┬──────┘          └─────────────┘          │
│                                  │                                          │
│                           ┌──────┴──────┐                                   │
│                           │   TWILIO    │                                   │
│                           │    (SMS)    │                                   │
│                           └─────────────┘                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Entiteti sistema

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              ENTITETI                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   KORISNIK (users)                                                          │
│   ├── Telefon (verifikovan SMS-om)                                          │
│   ├── Ime                                                                   │
│   └── Push subscription                                                     │
│                                                                             │
│   GRUPA (groups)                                                            │
│   ├── Naziv (npr. "OŠ Ivan Goran Kovačić")                                  │
│   ├── Admin                                                                 │
│   ├── JOIN PIN                                                              │
│   ├── Telegram integracija                                                  │
│   └── Zona (geofence)                                                       │
│                                                                             │
│   ČLANSTVO (memberships)                                                    │
│   ├── Korisnik + Grupa                                                      │
│   ├── Uloga: ADMIN / RODITELJ / RESPONDER / DETE                            │
│   ├── Status: PENDING / ACTIVE / SUSPENDED / BANNED                         │
│   └── Statistike (za RESPONDER)                                             │
│                                                                             │
│   SMENA (shifts)                                                            │
│   ├── Responder                                                             │
│   ├── Dan + vremenski slot                                                  │
│   └── Status: SCHEDULED / ACTIVE / COMPLETED / MISSED                       │
│                                                                             │
│   ALARM (alarms)                                                            │
│   ├── Lokacija                                                              │
│   ├── Opis                                                                  │
│   ├── Status: ACTIVE / ESCALATED_1 / ESCALATED_2 / RESPONDING / RESOLVED    │
│   └── Vremena eskalacije                                                    │
│                                                                             │
│   ODGOVOR NA ALARM (alarm_responses)                                        │
│   ├── Alarm + Korisnik                                                      │
│   ├── Kada notifikovan/video/reagovao                                       │
│   ├── Odgovor: ACCEPTED / DECLINED / BACKUP                                 │
│   └── Razlog odbijanja                                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Uloge korisnika

| Uloga | Opis | Mogućnosti |
|-------|------|------------|
| **ADMIN** | Administrator grupe | Sve + upravljanje članovima |
| **RODITELJ** | Roditelj učenika | Slanje alarma, pregled statusa |
| **RESPONDER** | Volonter patrole | Sve što RODITELJ + preuzimanje, smene |
| **DETE** | Učenik | Samo panic button, ograničen prikaz |

---

## Flow alarma

```
┌────────────┐
│   PANIC    │
│  BUTTON    │
└─────┬──────┘
      │
      ▼ T=0
┌────────────┐     Niko ne reaguje      ┌────────────┐
│   ACTIVE   │─────────90 sek──────────►│ ESCALATED_1│
└─────┬──────┘                          └─────┬──────┘
      │ Neko preuzme                          │
      ▼                                       │ Niko ne reaguje
┌────────────┐                                │ 60 sek
│ RESPONDING │◄───────────────────────────────┘
└─────┬──────┘                                │
      │ Stigao                                ▼
      ▼                               ┌────────────┐
┌────────────┐                        │ ESCALATED_2│
│  ON_SCENE  │                        │ (broadcast)│
└─────┬──────┘                        └────────────┘
      │ Rešeno
      ▼
┌────────────┐
│  RESOLVED  │
└────────────┘
```

---

## Eskalacija

| Nivo | Vreme | Ko dobija notifikaciju |
|------|-------|------------------------|
| 1 | T=0 | Samo responderi na smeni |
| 2 | T+90s | Svi responderi (i van smene) |
| 3 | T+150s | Svi članovi grupe + admin direktno |

---

## Bystander Effect Mitigacija

Sistem koristi psihološke tehnike za sprečavanje "niko ne reaguje":

| Tehnika | Implementacija |
|---------|----------------|
| Direktna odgovornost | "Ti si 1 od 3 dostupna respondera" |
| Vidljiv nedostatak akcije | "0 preuzelo, 3 videlo" |
| Imenovanje | "Marko, Jovan i Petra su na smeni" |
| Blizina kao prioritet | "Ti si NAJBLIŽI (300m)" |
| Vremenska urgencija | Countdown do eskalacije |
| Negativna potvrda | Mora da klikne "NE MOGU" sa razlogom |

---

## Admin Panel - Funkcije

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ADMIN PANEL                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ČLANOVI                                                                    │
│  ├── Pregled svih članova                                                   │
│  ├── Approve / Reject zahteva                                               │
│  ├── Promena uloge (RODITELJ ↔ RESPONDER)                                   │
│  ├── Suspend / Ban                                                          │
│  └── Pregled aktivnosti                                                     │
│                                                                             │
│  SMENE                                                                      │
│  ├── Nedeljni pregled                                                       │
│  ├── Upozorenja za nepopunjene                                              │
│  └── Podsjetnici                                                            │
│                                                                             │
│  ALARMI                                                                     │
│  ├── Istorija                                                               │
│  ├── Detalji po alarmu                                                      │
│  ├── Timeline događaja                                                      │
│  └── Export                                                                 │
│                                                                             │
│  PODEŠAVANJA                                                                │
│  ├── Ime i opis grupe                                                       │
│  ├── PIN (regeneracija)                                                     │
│  ├── Eskalacija (timeout-i)                                                 │
│  ├── Radno vreme                                                            │
│  └── Telegram konekcija                                                     │
│                                                                             │
│  STATISTIKE                                                                 │
│  ├── Mesečni pregled                                                        │
│  ├── Mapa vrućih tačaka                                                     │
│  └── Export PDF / CSV                                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Procena troškova

| Komponenta | 1 škola | 10 škola |
|------------|---------|----------|
| Vercel | $0 | $20 |
| Convex | $0 | $25 |
| Twilio SMS | ~$10 | ~$50 |
| Cloudflare | $0 | $0 |
| Railway | $0-5 | $5 |
| **UKUPNO** | **~$10-15** | **~$100** |

---

## Vremenska procena

| Faza | Vreme |
|------|-------|
| Core sistem (auth, grupe, alarmi) | 1-2 nedelje |
| Responder sistem (smene, eskalacija) | 1 nedelja |
| Admin panel | 1 nedelja |
| Statistike | 3-5 dana |
| Polish i testiranje | 1 nedelja |
| **UKUPNO** | **1-2 meseca** |

---

## Dokumentacija punog sistema

- [ROLES_AND_PERMISSIONS.md](./ROLES_AND_PERMISSIONS.md) - Detalji o ulogama
- [ALARM_LIFECYCLE.md](./ALARM_LIFECYCLE.md) - Kompletan flow alarma
- [RESPONDER_SYSTEM.md](./RESPONDER_SYSTEM.md) - Smene i bystander mitigacija
- [TELEGRAM_INTEGRATION.md](./TELEGRAM_INTEGRATION.md) - Telegram setup
- [ADMIN_PANEL.md](./ADMIN_PANEL.md) - Admin funkcionalnosti
- [STATISTICS.md](./STATISTICS.md) - Izveštaji i statistike
- [SECURITY.md](./SECURITY.md) - Bezbednost

---

*Dokument kreiran: Januar 2026*
