# PoC - Deployment

## Pregled

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POC DEPLOYMENT STACK                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│   │   VERCEL    │     │   CONVEX    │     │  TELEGRAM   │                   │
│   │  (Frontend) │     │  (Backend)  │     │    BOT      │                   │
│   │   FREE      │     │   FREE      │     │   FREE      │                   │
│   └─────────────┘     └─────────────┘     └─────────────┘                   │
│                                                                             │
│   Ukupni mesečni trošak: $0                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Korak 1: Telegram Bot Setup (10 min)

### 1.1 Kreiraj bota

1. Otvori Telegram i pronađi **@BotFather**
2. Pošalji `/newbot`
3. Unesi ime bota: `Patrola Kovacic Bot`
4. Unesi username: `patrola_kovacic_bot`
5. **Sačuvaj BOT TOKEN** koji dobiješ (npr. `123456789:ABC-DEF_xyz`)

### 1.2 Kreiraj Telegram grupu

1. Kreiraj novu Telegram grupu: `OŠ Kovačić - Patrola`
2. Dodaj bota u grupu
3. Postavi bota kao **ADMINA** (potrebno za slanje poruka)

### 1.3 Dobij CHAT_ID

```bash
# Zameni TOKEN sa svojim
curl "https://api.telegram.org/bot<TOKEN>/getUpdates"
```

Ili:
1. Pošalji bilo koju poruku u grupu
2. Pozovi URL iznad
3. U JSON odgovoru nađi `chat.id` (npr. `-100123456789`)
4. **Sačuvaj CHAT_ID**

---

## Korak 2: Convex Setup (15 min)

### 2.1 Kreiraj Convex projekat

```bash
# Kreiraj novi Next.js projekat sa Convex-om
npx create-next-app@latest panic-button-poc --typescript --tailwind
cd panic-button-poc

# Dodaj Convex
npm install convex
npx convex init
```

### 2.2 Dodaj schema i funkcije

Kreiraj fajlove kao što je opisano u [IMPLEMENTATION.md](./IMPLEMENTATION.md):
- `convex/schema.ts`
- `convex/alarms.ts`
- `convex/telegram.ts`

### 2.3 Deploy Convex funkcije

```bash
# Pokreni Convex dev (lokalno testiranje)
npx convex dev

# Ili deploy u produkciju
npx convex deploy
```

### 2.4 Dodaj environment varijable u Convex

1. Otvori [Convex Dashboard](https://dashboard.convex.dev)
2. Selektuj projekat
3. Idi na **Settings → Environment Variables**
4. Dodaj:
   - `TELEGRAM_BOT_TOKEN` = tvoj bot token
   - `TELEGRAM_CHAT_ID` = tvoj chat ID
   - `APP_URL` = `https://tvoj-projekat.vercel.app`

---

## Korak 3: Vercel Deploy (15 min)

### 3.1 Push na GitHub

```bash
# Inicijalizuj git (ako već nije)
git init
git add .
git commit -m "Initial commit - PoC"

# Kreiraj repo na GitHub-u i push-uj
git remote add origin https://github.com/username/panic-button-poc
git push -u origin main
```

### 3.2 Connect sa Vercel-om

1. Otvori [Vercel](https://vercel.com)
2. Klikni **Add New → Project**
3. Importuj GitHub repo
4. Vercel će automatski detektovati Next.js

### 3.3 Dodaj environment varijable

U Vercel dashboard-u, dodaj:

| Variable | Value |
|----------|-------|
| `NEXT_PUBLIC_SCHOOL_PIN` | `kovacic2025` |
| `NEXT_PUBLIC_SCHOOL_NAME` | `OŠ Ivan Goran Kovačić` |
| `CONVEX_DEPLOYMENT` | (automatski se dodaje) |

### 3.4 Deploy

Klikni **Deploy** i sačekaj ~1 minut.

Dobijaš URL: `https://panic-button-poc.vercel.app`

---

## Korak 4: Testiranje (20 min)

### 4.1 End-to-end test

1. Otvori deploy-ovanu aplikaciju
2. Unesi PIN (`kovacic2025`)
3. Drži panic dugme 3 sekunde
4. Unesi opis i ime
5. Klikni POŠALJI
6. **Proveri Telegram grupu** - treba da stigne notifikacija
7. U aplikaciji, klikni "PREUZIMAM"
8. Unesi ime i potvrdi
9. **Proveri Telegram** - treba da stigne update
10. Klikni "REŠENO"
11. Alarm treba da nestane

### 4.2 Checklist

- [ ] PIN login radi
- [ ] Panic button aktivira alarm
- [ ] Lokacija se šalje
- [ ] Telegram prima notifikaciju
- [ ] "Preuzimam" radi
- [ ] Telegram prima update
- [ ] Razrešenje radi
- [ ] Real-time update radi (otvori u dva browser-a)

---

## Struktura URL-ova

```
Production:
https://panic-button-poc.vercel.app          # Frontend
https://xxx.convex.cloud                      # Convex API

Telegram:
https://t.me/patrola_kovacic_bot             # Bot
```

---

## Monitoring

### Vercel Dashboard

- Pregled deploy-ova
- Error logs
- Analytics (visits)

### Convex Dashboard

- Function calls
- Database pregled
- Logs

### Telegram

- Provera da li bot prima/šalje poruke

---

## Troubleshooting

### Problem: Telegram ne prima poruke

**Provere:**
1. Da li je `TELEGRAM_BOT_TOKEN` ispravan?
2. Da li je `TELEGRAM_CHAT_ID` ispravan (negativan broj)?
3. Da li je bot admin u grupi?
4. Proveri Convex logs za error-e

**Debug:**
```bash
# Test direktno sa Telegram API
curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{"chat_id": "<CHAT_ID>", "text": "Test poruka"}'
```

### Problem: Lokacija nije dostupna

**Uzroci:**
1. Korisnik nije dozvolio pristup lokaciji
2. HTTPS nije uključen (lokacija zahteva HTTPS)
3. Browser ne podržava Geolocation API

**Rešenje:**
- Prikaži fallback da korisnik ručno unese opis lokacije

### Problem: PIN ne radi

**Provere:**
1. Da li je `NEXT_PUBLIC_SCHOOL_PIN` postavljen u Vercel-u?
2. Da li je PIN case-sensitive provera ispravna?

---

## Održavanje

### Promena PIN-a

1. Idi na Vercel Dashboard → Settings → Environment Variables
2. Promeni `NEXT_PUBLIC_SCHOOL_PIN`
3. Redeploy (ili sačekaj automatski)

### Dodavanje korisnika u Telegram grupu

1. Admin grupe dodaje nove članove
2. Članovi dobijaju PIN od admina
3. Nema potrebe za promenama u aplikaciji

### Backup podataka

Convex automatski čuva podatke. Za export:
1. Convex Dashboard → Data
2. Export kao JSON

---

## Skaliranje (ako PoC uspe)

```
PoC (sada):                    MVP (sledeće):
─────────────                  ──────────────
1 škola                   →    1 škola
Hardcoded config          →    Dynamic config
$0/mesec                  →    ~$10-15/mesec (SMS)
Zajednički PIN            →    SMS verifikacija
Nema uloga                →    RESPONDER uloga
```

---

## Checklist za produkciju

Pre nego što se aplikacija podeli sa pravim korisnicima:

- [ ] Telegram bot radi
- [ ] Deploy na Vercel završen
- [ ] PIN postavljen
- [ ] End-to-end test prošao
- [ ] Telegram grupa kreirana
- [ ] Minimum 5 članova u grupi
- [ ] Svi razumeju kako se koristi

---

## Vremenska procena

| Aktivnost | Vreme |
|-----------|-------|
| Telegram bot setup | 10 min |
| Convex setup | 15 min |
| Vercel deploy | 15 min |
| Testiranje | 20 min |
| **UKUPNO** | **~1 sat** |

---

*Dokument kreiran: Januar 2026*
