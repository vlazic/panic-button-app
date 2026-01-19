# Telegram Integracija

## Pregled

Telegram se koristi kao **pouzdan kanal za notifikacije** jer:
- Korisnici već imaju Telegram
- Notifikacije rade izuzetno pouzdano (čak i na battery saver-u)
- Grupni chat je besplatan
- API je jednostavan

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TELEGRAM ARHITEKTURA                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   PWA radi:                          Telegram radi:                         │
│   • Registracija, grupe              • Notifikacija o alarmu                │
│   • PANIC button                     • Link nazad u app                     │
│   • "Preuzimam" reakcija             • Update poruke                        │
│   • Mapa, istorija                                                          │
│   • Admin panel                      Telegram je samo "zvono"               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Opcije arhitekture

### Opcija A: Jedan bot za sve grupe (preporučeno za početak)

```
                      ┌─────────────┐
                      │  PATROLA    │
                      │    BOT      │
                      └─────────────┘
                             │
            ┌────────────────┼────────────────┐
            ▼                ▼                ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │ TG Grupa:   │  │ TG Grupa:   │  │ TG Grupa:   │
   │ OŠ Kovačić  │  │ OŠ Vuk      │  │ OŠ Tesla    │
   └─────────────┘  └─────────────┘  └─────────────┘

✅ Jednostavnije - jedan bot za ceo sistem
✅ Centralizovano upravljanje
⚠️ Ako bot padne, SVE grupe su offline
⚠️ Rate limiting (30 poruka/sek po botu)
```

### Opcija B: Svaka grupa ima svoj bot

```
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │ BOT Kovačić │  │ BOT Vuk     │  │ BOT Tesla   │
   └─────────────┘  └─────────────┘  └─────────────┘
          │                │                │
          ▼                ▼                ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │ TG Grupa    │  │ TG Grupa    │  │ TG Grupa    │
   └─────────────┘  └─────────────┘  └─────────────┘

✅ Izolacija - problem jednog ne utiče na druge
✅ Nema rate limit problema
⚠️ Komplikovanije - admin mora da kreira svog bota
⚠️ Više tokena za čuvati
```

**Preporuka:** Opcija A za PoC/MVP, razmotriti B ako skalira

---

## Setup bota

### Korak 1: Kreiranje bota

1. Otvori Telegram, pronađi **@BotFather**
2. Pošalji `/newbot`
3. Unesi ime: `Patrola Bot`
4. Unesi username: `patrola_bot`
5. Sačuvaj **BOT_TOKEN**

### Korak 2: Kreiranje grupe

1. Kreiraj novu Telegram grupu
2. Dodaj bota u grupu
3. **Postavi bota kao ADMIN** (obavezno za slanje poruka)

### Korak 3: Dobijanje CHAT_ID

```bash
# Pošalji poruku u grupu, pa pozovi:
curl "https://api.telegram.org/bot<TOKEN>/getUpdates"

# U odgovoru, nađi chat.id (negativan broj za grupe)
# Primer: -100123456789
```

---

## Flow povezivanja grupe

### Kod-bazirano povezivanje

```
═══════════════════════════════════════════════════════════════════════════════
                    ADMIN POVEZUJE TELEGRAM GRUPU
═══════════════════════════════════════════════════════════════════════════════

KORAK 1: Admin dodaje bota u Telegram grupu
────────────────────────────────────────────

    [Telegram app]
    1. Admin kreira grupu "OŠ Kovačić - Patrola"
    2. Dodaje @PatrolaBot u grupu
    3. Postavlja bota kao ADMINA


KORAK 2: Bot šalje kod
──────────────────────

    Bot detektuje da je dodat i šalje:

    ┌─────────────────────────────────────────────┐
    │  👋 Zdravo! Ja sam PatrolaBot.              │
    │                                             │
    │  Da povežete ovu grupu sa aplikacijom,      │
    │  admin treba da unese ovaj kod u app:       │
    │                                             │
    │         🔑 KOD: TG-7X9K2M                   │
    │                                             │
    │  Kod važi 15 minuta.                        │
    └─────────────────────────────────────────────┘

    [Backend čuva: kod "TG-7X9K2M" → chat_id "-100123456"]


KORAK 3: Admin unosi kod u PWA
──────────────────────────────

    [PWA - Podešavanja grupe]

    ┌─────────────────────────────────────────────┐
    │  Poveži Telegram grupu                      │
    │                                             │
    │  Unesite kod koji je bot poslao:            │
    │                                             │
    │  [ TG-7X9K2M_____________ ]                 │
    │                                             │
    │  [POVEŽI]                                   │
    └─────────────────────────────────────────────┘


KORAK 4: Verifikacija
─────────────────────

    Backend:
    1. Proverava da li kod postoji i nije istekao
    2. Uzima chat_id vezan za kod
    3. Šalje TEST poruku u Telegram grupu

    ┌─────────────────────────────────────────────┐
    │  ✅ Uspešno povezano!                        │
    │                                             │
    │  Ova Telegram grupa je sada povezana sa:   │
    │  "OŠ Ivan Goran Kovačić - Roditelji"        │
    │                                             │
    │  Alarmi će stizati ovde.                    │
    └─────────────────────────────────────────────┘
```

---

## Format poruka

### Alarm notifikacija

```
🚨 *ALARM*

📍 Lokacija: https://maps.google.com/?q=44.81,20.46
💬 "Prate me"
👤 Od: Dete Markovića

👆 Otvori: https://patrola.rs/alarm/xyz
```

**Zašto minimalna poruka:**
- Privatnost - detalji ostaju u app-u
- Link vodi na app gde su puni podaci
- Članovi grupe ne vide sve lokacije

### Update poruke

```
✅ Marko P. je preuzeo alarm

📍 Responder je stigao na lokaciju

✅ Alarm razrešen
```

---

## Problemi i rešenja

### Problem 1: Neko poveže tuđu grupu

**Scenario:** Marko ima kod, Jovan ga nekako sazna i poveže sa svojom app grupom.

**Rešenje:** Dvostepena potvrda

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  VERIFIKACIJA                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Opcija 1: Kod se šalje kao REPLY samo adminu TG grupe                      │
│                                                                             │
│  Opcija 2: Bot traži potvrdu pre povezivanja:                               │
│                                                                             │
│     ┌─────────────────────────────────────────────┐                         │
│     │  ⚠️ Neko pokušava da poveže ovu grupu       │                         │
│     │  sa app grupom "OŠ Kovačić"                │                         │
│     │                                             │                         │
│     │  Admin TG grupe, potvrdite:                 │                         │
│     │  [✅ DOZVOLI]  [❌ ODBIJ]                   │                         │
│     └─────────────────────────────────────────────┘                         │
│                                                                             │
│  Samo ako TG admin klikne DOZVOLI → povezivanje uspeva                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Problem 2: Jedna TG grupa, više app grupa

**Rešenje:** Striktno 1:1 mapiranje

```typescript
if (existing_group_with_this_chat_id) {
  return error("Ova Telegram grupa je već povezana sa: {group_name}");
}
```

### Problem 3: Bot uklonjen iz grupe

**Rešenje:** Detekcija i obaveštenje

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  DETEKCIJA                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. Bot prima "removed from group" event → markira u bazi                   │
│                                                                             │
│  2. App prikazuje UPOZORENJE adminu:                                        │
│                                                                             │
│     ┌─────────────────────────────────────────────┐                         │
│     │  ⚠️ PROBLEM SA TELEGRAM INTEGRACIJOM        │                         │
│     │                                             │                         │
│     │  Bot je uklonjen iz Telegram grupe.        │                         │
│     │  Alarmi se NE DOSTAVLJAJU!                 │                         │
│     │                                             │
│     │  [POVEŽI PONOVO]                           │                         │
│     └─────────────────────────────────────────────┘                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Problem 4: Rate limiting

**Telegram limiti:**
- 30 poruka po sekundi po botu
- 20 poruka po minuti u istu grupu

**Rešenje:** Queue sistem

```typescript
const sendQueue = [];

async function sendTelegramAlert(chat_id, message) {
  sendQueue.push({ chat_id, message, attempts: 0 });
}

// Worker procesira queue sa pauzama
setInterval(async () => {
  const item = sendQueue.shift();
  if (item) {
    try {
      await telegram.sendMessage(item.chat_id, item.message);
    } catch (e) {
      if (e.code === 429) { // Rate limited
        item.attempts++;
        if (item.attempts < 3) {
          sendQueue.push(item); // Vrati u queue
        }
      }
    }
  }
}, 100); // Max 10 poruka/sek
```

---

## Data model

```typescript
// Telegram link codes
telegram_link_codes: defineTable({
  code: v.string(),              // "TG-7X9K2M"
  chat_id: v.string(),           // "-100123456789"
  chat_title: v.string(),        // "OŠ Kovačić Patrola"
  created_at: v.number(),
  expires_at: v.number(),        // created_at + 15min
  used: v.boolean(),
  used_by_group: v.optional(v.id("groups")),
})

// U groups tabeli
groups: defineTable({
  // ...
  telegram_chat_id: v.optional(v.string()),
  telegram_connected_at: v.optional(v.number()),
  telegram_connected_by: v.optional(v.id("users")),
})
```

---

## Admin UI za Telegram

```
┌─────────────────────────────────────────────┐
│  Telegram integracija                       │
│                                             │
│  Status: ✅ Povezano                        │
│  Grupa: "OŠ Kovačić Patrola"               │
│  Povezano: 15. jan 2025.                   │
│                                             │
│  [TEST KONEKCIJE]                          │
│                                             │
│  [PREKINI VEZU]  [PROMENI GRUPU]           │
└─────────────────────────────────────────────┘
```

---

## Checklist za implementaciju

### MVP
- [ ] Jedan bot za ceo sistem
- [ ] Kod-bazirano povezivanje (15min expiry)
- [ ] 1:1 mapiranje
- [ ] Provera da li je bot u grupi pre slanja
- [ ] Minimalna poruka + link
- [ ] Admin UI za connect/disconnect

### Pun sistem
- [ ] Potvrda od TG admina pre linkovanja
- [ ] Queue sistem za rate limiting
- [ ] Opcija: svaka škola svoj bot
- [ ] Webhook umesto polling

---

*Dokument kreiran: Januar 2025*
