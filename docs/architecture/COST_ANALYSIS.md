# Analiza Troškova

## Pregled

Ovaj dokument analizira troškove pokretanja Panic Button aplikacije na različitim nivoima skaliranja.

---

## Scenariji

### Scenario 1: PoC (1 škola, ~50 korisnika)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  PoC - JEDNA ŠKOLA                                                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PRETPOSTAVKE:                                                                  │
│  • 50 korisnika (roditelji + responderi)                                        │
│  • ~5 alarma mesečno                                                            │
│  • Nema SMS verifikacije                                                        │
│  • Jedna Telegram grupa                                                         │
│                                                                                 │
│  TROŠKOVI:                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  Servis              │  Plan        │  Cena/mesec                       │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  Vercel              │  Hobby       │  $0 (free)                        │    │
│  │  Convex              │  Free        │  $0 (free)                        │    │
│  │  Cloudflare DNS      │  Free        │  $0 (free)                        │    │
│  │  Telegram API        │  -           │  $0 (free)                        │    │
│  │  Domena (.rs)        │  -           │  ~$15/god = ~$1.25/mes            │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  UKUPNO                             │  ~$1-2/mesec                      │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  ✅ Potpuno održivo na free tier-ima                                            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Scenario 2: MVP (3-5 škola, ~200 korisnika)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  MVP - NEKOLIKO ŠKOLA                                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PRETPOSTAVKE:                                                                  │
│  • 5 škola                                                                      │
│  • ~200 korisnika ukupno                                                        │
│  • ~20 alarma mesečno                                                           │
│  • SMS verifikacija (100 SMS/mes)                                               │
│  • 5 Telegram grupa                                                             │
│                                                                                 │
│  TROŠKOVI:                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  Servis              │  Plan        │  Cena/mesec                       │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  Vercel              │  Hobby       │  $0 (free)                        │    │
│  │  Convex              │  Free        │  $0 (free)*                       │    │
│  │  Cloudflare DNS      │  Free        │  $0 (free)                        │    │
│  │  Telegram API        │  -           │  $0 (free)                        │    │
│  │  Twilio SMS          │  Pay-as-go   │  ~$5 (100 SMS × $0.05)            │    │
│  │  Domena              │  -           │  ~$1.25                           │    │
│  │  Sentry (errors)     │  Free        │  $0 (free)                        │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  UKUPNO                             │  ~$5-10/mesec                     │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  * Možda blizu granice free tier-a                                              │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Scenario 3: Srednji (10 škola, ~500 korisnika)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  SREDNJI - VIŠE ŠKOLA                                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PRETPOSTAVKE:                                                                  │
│  • 10 škola                                                                     │
│  • ~500 korisnika                                                               │
│  • ~50 alarma mesečno                                                           │
│  • SMS verifikacija (300 SMS/mes)                                               │
│  • 10 Telegram grupa                                                            │
│                                                                                 │
│  TROŠKOVI:                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  Servis              │  Plan        │  Cena/mesec                       │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  Vercel              │  Pro         │  $20                              │    │
│  │  Convex              │  Pro         │  $25                              │    │
│  │  Cloudflare          │  Free        │  $0                               │    │
│  │  Telegram API        │  -           │  $0                               │    │
│  │  Twilio SMS          │  Pay-as-go   │  ~$15 (300 × $0.05)               │    │
│  │  Domena              │  -           │  ~$1.25                           │    │
│  │  Sentry              │  Team        │  $26                              │    │
│  │  UptimeRobot         │  Free        │  $0                               │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  UKUPNO                             │  ~$80-100/mesec                   │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Scenario 4: Veliki (50+ škola, ~2000 korisnika)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  VELIKI - REGIONALNA IMPLEMENTACIJA                                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PRETPOSTAVKE:                                                                  │
│  • 50 škola                                                                     │
│  • ~2000 korisnika                                                              │
│  • ~200 alarma mesečno                                                          │
│  • SMS verifikacija (1000 SMS/mes)                                              │
│  • 50 Telegram grupa                                                            │
│                                                                                 │
│  TROŠKOVI:                                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  Servis              │  Plan        │  Cena/mesec                       │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  Vercel              │  Pro         │  $20                              │    │
│  │  Convex              │  Pro/Custom  │  ~$100                            │    │
│  │  Cloudflare          │  Pro         │  $20                              │    │
│  │  Telegram API        │  -           │  $0                               │    │
│  │  Twilio SMS          │  Pay-as-go   │  ~$50 (1000 × $0.05)              │    │
│  │  Domena              │  -           │  ~$1.25                           │    │
│  │  Sentry              │  Business    │  $80                              │    │
│  │  Monitoring          │  Various     │  ~$30                             │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  UKUPNO                             │  ~$300-350/mesec                  │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Detaljna Analiza po Servisu

### Convex

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CONVEX PRICING                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  FREE TIER (dovoljno za PoC/MVP):                                               │
│  • 1M function calls/mesec                                                      │
│  • 1GB database storage                                                         │
│  • 1GB bandwidth                                                                │
│  • Unlimited projects                                                           │
│                                                                                 │
│  PRO ($25/mesec):                                                               │
│  • 10M function calls                                                           │
│  • 10GB storage                                                                 │
│  • 10GB bandwidth                                                               │
│  • Priority support                                                             │
│  • Point-in-time recovery                                                       │
│                                                                                 │
│  PROCENA KORIŠĆENJA (10 škola):                                                 │
│  ├── Page loads: 500 users × 10 loads/dan × 30 = 150K calls                     │
│  ├── Real-time subscriptions: ~500K calls                                       │
│  ├── Mutations: ~50K calls                                                      │
│  └── TOTAL: ~700K calls/mesec (OK za free)                                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Vercel

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  VERCEL PRICING                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  HOBBY (Free):                                                                  │
│  • 100GB bandwidth                                                              │
│  • Unlimited deployments                                                        │
│  • SSL                                                                          │
│  • 1 custom domain                                                              │
│  • Community support                                                            │
│                                                                                 │
│  PRO ($20/mesec):                                                               │
│  • 1TB bandwidth                                                                │
│  • Team features                                                                │
│  • Password protection                                                          │
│  • Preview deployments                                                          │
│  • Analytics                                                                    │
│                                                                                 │
│  KADA UPGRADE:                                                                  │
│  • Više od 100GB bandwidth                                                      │
│  • Tim od 2+ developera                                                         │
│  • Potrebna analytics                                                           │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Twilio SMS

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TWILIO SMS PRICING                                                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CENA ZA SRBIJU:                                                                │
│  • Outbound SMS: ~$0.05/poruka                                                  │
│  • Phone number: $1/mesec                                                       │
│                                                                                 │
│  PROCENA (po školi):                                                            │
│  ├── Registracija: ~10 novih korisnika/mesec × 2 SMS = 20 SMS                   │
│  ├── Password reset: ~5 SMS/mesec                                               │
│  └── TOTAL: ~25 SMS × $0.05 = $1.25/škola/mesec                                 │
│                                                                                 │
│  OPTIMIZACIJE:                                                                  │
│  • Koristiti Telegram umesto SMS za verifikaciju (ako moguće)                   │
│  • Rate limiting da se spreči zloupotreba                                       │
│  • Samo SMS za kritične operacije                                               │
│                                                                                 │
│  ALTERNATIVA:                                                                   │
│  • Infobip: Lokalniji, slična cena                                              │
│  • Bez SMS: Samo PIN (kao u PoC-u)                                              │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Monitoring (Sentry)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  SENTRY PRICING                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  DEVELOPER (Free):                                                              │
│  • 5K errors/mesec                                                              │
│  • 1 user                                                                       │
│  • 30-day retention                                                             │
│                                                                                 │
│  TEAM ($26/mesec):                                                              │
│  • 50K errors                                                                   │
│  • Unlimited users                                                              │
│  • 90-day retention                                                             │
│  • Performance monitoring                                                       │
│                                                                                 │
│  BUSINESS ($80/mesec):                                                          │
│  • 100K errors                                                                  │
│  • Advanced features                                                            │
│  • 1-year retention                                                             │
│                                                                                 │
│  PREPORUKA:                                                                     │
│  • Free tier dovoljan za PoC/MVP                                                │
│  • Team za production sa više škola                                             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Grafikon Troškova

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  TROŠKOVI PO BROJU ŠKOLA                                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  $400│                                                          ○              │
│      │                                                         ╱               │
│  $300│                                                       ○╱                │
│      │                                                      ╱                  │
│  $200│                                                    ╱                    │
│      │                                                  ╱                      │
│  $100│                              ○─────────────────○╱                       │
│      │                            ╱                                            │
│   $50│            ○─────────────○╱                                             │
│      │          ╱                                                              │
│    $0│○────────○                                                               │
│      └─────────────────────────────────────────────────────────────────────    │
│        1      3      5     10     20     30     40     50    Škola             │
│                                                                                 │
│  LEGENDA:                                                                       │
│  ○ Prelomne tačke (upgrade tier-a)                                              │
│                                                                                 │
│  NAPOMENA:                                                                      │
│  • Do 5 škola: Većinom free tier                                                │
│  • 5-10 škola: $80-100/mes (Pro tiers)                                          │
│  • 10-50 škola: Linearan rast, ~$5-7/škola/mes                                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Finansiranje

### Opcije za Pokrivanje Troškova

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  OPCIJE FINANSIRANJA                                                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. VOLONTERSKI MODEL (PoC)                                                     │
│     • Developeri rade besplatno                                                 │
│     • Troškovi ~$5/mes pokriveni iz džepa                                       │
│     • Održivo za jednu školu                                                    │
│                                                                                 │
│  2. DONACIJE RODITELJA                                                          │
│     • ~$2/roditelj/godina = ~$100/škola/godina                                  │
│     • Pokriva troškove za jednu školu                                           │
│     • Jednostavan model                                                         │
│                                                                                 │
│  3. SPONZORSTVO KOMPANIJE                                                       │
│     • Lokalne kompanije kao sponzori                                            │
│     • Logo u app-u (nije intruzivan)                                            │
│     • $50-100/mesec po sponzoru                                                 │
│                                                                                 │
│  4. GRANT OD OPŠTINE/GRADA                                                      │
│     • Projekat od javnog interesa                                               │
│     • Jednokratna donacija za godinu dana                                       │
│     • ~$1000-2000 za pokrivanje troškova                                        │
│                                                                                 │
│  5. OPEN SOURCE CREDITS                                                         │
│     • Vercel for Open Source: Free Pro tier                                     │
│     • Sentry for Open Source: Free                                              │
│     • Zahteva: Public repo, non-profit                                          │
│                                                                                 │
│  6. HYBRID MODEL                                                                │
│     • Škola plaća simbolično (~$10/mes)                                         │
│     • Ostatak iz donacija/sponzorstava                                          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Open Source Credits

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  OPEN SOURCE / NONPROFIT PROGRAMI                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  VERCEL FOR OPEN SOURCE:                                                        │
│  • Pro tier besplatno                                                           │
│  • Zahtevi: Public repo, OSS license                                            │
│  • Apply: vercel.com/oss                                                        │
│                                                                                 │
│  SENTRY FOR GOOD:                                                               │
│  • Free za nonprofit                                                            │
│  • Apply: sentry.io/for/good                                                    │
│                                                                                 │
│  CLOUDFLARE PROJECT GALILEO:                                                    │
│  • Za organizacije koje štite javni interes                                     │
│  • Apply: cloudflare.com/galileo                                                │
│                                                                                 │
│  TWILIO.ORG:                                                                    │
│  • Credits za nonprofit                                                         │
│  • Impact Access Program                                                        │
│  • Apply: twilio.org                                                            │
│                                                                                 │
│  GITHUB SPONSORS:                                                               │
│  • Ako je projekat open source                                                  │
│  • Korisnici mogu sponzorisati                                                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## ROI Analiza

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  RETURN ON INVESTMENT                                                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  KVANTITATIVNI BENEFITI:                                                        │
│  • Brža reakcija na incidente (minuti → sekunde)                                │
│  • Dokumentovani podaci za policiju                                             │
│  • Koordinacija volontera                                                       │
│                                                                                 │
│  KVALITATIVNI BENEFITI:                                                         │
│  • Osećaj sigurnosti kod roditelja i dece                                       │
│  • Prevencija (vidljivo prisustvo patrola)                                      │
│  • Zajednica koja se brine                                                      │
│                                                                                 │
│  COST PER SCHOOL:                                                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  Scenario        │  Trošak/mes  │  Trošak/učenik/mes                    │    │
│  │  ────────────────────────────────────────────────────────────────────   │    │
│  │  PoC (1 škola)   │  ~$2         │  ~$0.004 (500 učenika)                │    │
│  │  MVP (5 škola)   │  ~$10        │  ~$0.004 (2500 učenika)               │    │
│  │  Srednji (10)    │  ~$100       │  ~$0.02 (5000 učenika)                │    │
│  │  Veliki (50)     │  ~$350       │  ~$0.014 (25000 učenika)              │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
│  ZAKLJUČAK:                                                                     │
│  Troškovi su MINIMALNI u odnosu na potencijalnu vrednost                        │
│  ~$0.01-0.02/učenik/mesec za bezbednost                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Preporuke

### Za PoC

- Koristi sve free tier-ove
- Bez SMS-a (samo PIN)
- Jedna Telegram grupa
- Procenjeni trošak: **~$2/mesec**

### Za MVP (3-5 škola)

- Free tier Vercel + Convex
- Minimalan SMS (samo registracija)
- Apply za open source credits
- Procenjeni trošak: **~$10/mesec**

### Za Scale (10+ škola)

- Pro tier-ovi
- Sponzorstva ili grant
- Simbolična naknada po školi
- Procenjeni trošak: **$5-10/škola/mesec**

---

*Dokument kreiran: Januar 2026*
