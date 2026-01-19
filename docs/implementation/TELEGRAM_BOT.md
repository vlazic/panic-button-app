# Telegram Bot - Setup i Implementacija

## Pregled

Ovaj dokument opisuje kako kreirati i konfigurisati Telegram bota za Panic Button aplikaciju.

---

## Kreiranje Bota

### Korak 1: BotFather

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  KREIRANJE BOTA PREKO @BotFather                                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. Otvori Telegram i pronađi @BotFather                                        │
│                                                                                 │
│  2. Pošalji komandu: /newbot                                                    │
│                                                                                 │
│  3. BotFather pita za ime bota:                                                 │
│     > Patrola Bot                                                               │
│                                                                                 │
│  4. BotFather pita za username (mora završiti sa "bot"):                        │
│     > patrola_skola_bot                                                         │
│                                                                                 │
│  5. BotFather ti daje TOKEN:                                                    │
│     ┌─────────────────────────────────────────────────────────────────────┐     │
│     │  Done! Congratulations on your new bot.                             │     │
│     │                                                                     │     │
│     │  Use this token to access the HTTP API:                             │     │
│     │  123456789:ABCdefGHIjklMNOpqrsTUVwxyz_0123456789                    │     │
│     │                                                                     │     │
│     │  Keep your token secure and store it safely.                        │     │
│     └─────────────────────────────────────────────────────────────────────┘     │
│                                                                                 │
│  ⚠️ SAČUVAJ TOKEN! Nikada ga ne objavljuj javno.                                │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Korak 2: Podešavanje Bota

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  DODATNA PODEŠAVANJA (opciono)                                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  Pošalji @BotFather sledeće komande:                                            │
│                                                                                 │
│  /setdescription                                                                │
│  > Patrola Bot - Notifikacije o alarmima za bezbednost dece                     │
│                                                                                 │
│  /setabouttext                                                                  │
│  > Bot za koordinaciju roditeljskih patrola oko škola                           │
│                                                                                 │
│  /setuserpic                                                                    │
│  > [Upload sliku/logo]                                                          │
│                                                                                 │
│  /setcommands                                                                   │
│  > start - Pokreni bota                                                         │
│  > help - Pomoć                                                                 │
│  > status - Status sistema                                                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Kreiranje Grupe

### Korak 3: Nova Telegram Grupa

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  KREIRANJE TELEGRAM GRUPE                                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. U Telegram-u klikni na ≡ (hamburger menu)                                   │
│                                                                                 │
│  2. "New Group"                                                                 │
│                                                                                 │
│  3. Dodaj bar jednog člana (može biti tvoj drugi nalog ili @patrola_skola_bot)  │
│                                                                                 │
│  4. Unesi ime grupe:                                                            │
│     > OŠ Kovačić - Patrola                                                      │
│                                                                                 │
│  5. Klikni ✓ da kreiraš grupu                                                   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Korak 4: Dodavanje Bota kao Admina

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  POSTAVLJANJE BOTA KAO ADMINA                                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ⚠️ VAŽNO: Bot MORA biti admin da bi mogao slati poruke u grupu!                │
│                                                                                 │
│  1. Otvori grupu                                                                │
│                                                                                 │
│  2. Klikni na ime grupe (header) da otvoriš info                                │
│                                                                                 │
│  3. Klikni na ✏️ (Edit)                                                         │
│                                                                                 │
│  4. "Administrators"                                                            │
│                                                                                 │
│  5. "Add Administrator"                                                         │
│                                                                                 │
│  6. Pronađi i izaberi @patrola_skola_bot                                        │
│                                                                                 │
│  7. Dodeli permisije (minimum):                                                 │
│     ✓ Post messages                                                             │
│     ✓ Edit messages                                                             │
│                                                                                 │
│  8. Klikni "Done"                                                               │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Dobijanje Chat ID

### Korak 5: Dobijanje ID grupe

```bash
# Metoda 1: Preko getUpdates API-ja

# 1. Pošalji bilo koju poruku u grupu

# 2. Pozovi API (zameni <TOKEN> sa tvojim tokenom):
curl "https://api.telegram.org/bot<TOKEN>/getUpdates"

# 3. U JSON response-u, nađi:
# {
#   "message": {
#     "chat": {
#       "id": -100123456789,  <-- OVO JE CHAT_ID
#       "title": "OŠ Kovačić - Patrola",
#       "type": "supergroup"
#     }
#   }
# }

# NAPOMENA: ID grupe je NEGATIVAN broj (počinje sa -)
```

```bash
# Metoda 2: Koristi @userinfobot

# 1. Dodaj @userinfobot u grupu
# 2. Bot će automatski poslati info o grupi, uključujući ID
# 3. Zatim možeš ukloniti bota
```

### Primer Response-a

```json
{
  "ok": true,
  "result": [
    {
      "update_id": 123456789,
      "message": {
        "message_id": 1,
        "from": {
          "id": 12345678,
          "first_name": "Marko"
        },
        "chat": {
          "id": -1001234567890,
          "title": "OŠ Kovačić - Patrola",
          "type": "supergroup"
        },
        "date": 1705671600,
        "text": "Test"
      }
    }
  ]
}
```

---

## Testiranje

### Korak 6: Test Slanja Poruke

```bash
# Test slanja poruke u grupu

curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "-1001234567890",
    "text": "🚀 Test poruka - Patrola Bot je uspešno povezan!",
    "parse_mode": "Markdown"
  }'

# Očekivani response:
# {
#   "ok": true,
#   "result": {
#     "message_id": 2,
#     "chat": {"id": -1001234567890, ...},
#     "text": "🚀 Test poruka - Patrola Bot je uspešno povezan!"
#   }
# }
```

### Test Alarm Poruke

```bash
# Simuliraj alarm poruku

curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "-1001234567890",
    "text": "🚨 *ALARM*\n\n📍 Lokacija: [Otvori mapu](https://maps.google.com/?q=44.8125,20.4612)\n💬 \"Prate me\"\n👤 Od: Marko P.\n\n👆 [Otvori u aplikaciji](https://patrola.rs/alarm/test)",
    "parse_mode": "Markdown",
    "disable_web_page_preview": false
  }'
```

---

## Integracija sa Convex

### Environment Variables

```bash
# Postavi u Convex
npx convex env set TELEGRAM_BOT_TOKEN "123456789:ABCdefGHIjklMNOpqrsTUVwxyz"
npx convex env set TELEGRAM_CHAT_ID "-1001234567890"
npx convex env set APP_URL "https://patrola.rs"

# Proveri
npx convex env list
```

### Convex Action

```typescript
// convex/telegram.ts

import { internalAction } from "./_generated/server";
import { v } from "convex/values";

export const sendMessage = internalAction({
  args: {
    text: v.string(),
    parseMode: v.optional(v.string()),
  },
  handler: async (ctx, { text, parseMode = "Markdown" }) => {
    const botToken = process.env.TELEGRAM_BOT_TOKEN;
    const chatId = process.env.TELEGRAM_CHAT_ID;

    if (!botToken || !chatId) {
      throw new Error("Telegram not configured");
    }

    const response = await fetch(
      `https://api.telegram.org/bot${botToken}/sendMessage`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          chat_id: chatId,
          text,
          parse_mode: parseMode,
          disable_web_page_preview: false,
        }),
      }
    );

    const result = await response.json();

    if (!result.ok) {
      console.error("Telegram API error:", result);
      throw new Error(result.description || "Failed to send message");
    }

    return result;
  },
});
```

---

## API Reference

### Korišćeni Endpoints

| Endpoint | Metoda | Opis |
|----------|--------|------|
| `/getMe` | GET | Info o botu |
| `/getUpdates` | GET | Dobij nove poruke |
| `/sendMessage` | POST | Pošalji poruku |
| `/editMessageText` | POST | Izmeni poruku |
| `/deleteMessage` | POST | Obriši poruku |

### sendMessage Parametri

```typescript
interface SendMessageParams {
  chat_id: string | number;      // ID grupe
  text: string;                  // Tekst poruke
  parse_mode?: "Markdown" | "HTML" | "MarkdownV2";
  disable_web_page_preview?: boolean;
  disable_notification?: boolean;
  reply_to_message_id?: number;
  reply_markup?: InlineKeyboardMarkup;
}
```

### Markdown Formatiranje

```markdown
*bold*
_italic_
`code`
```code block```
[link](http://example.com)
```

---

## Napredne Funkcije (Pun Sistem)

### Inline Keyboard za Akcije

```typescript
// Poruka sa dugmićima

const message = {
  chat_id: chatId,
  text: "🚨 *ALARM*\n\nNeko treba pomoć!",
  parse_mode: "Markdown",
  reply_markup: {
    inline_keyboard: [
      [
        {
          text: "🏃 Preuzimam",
          url: "https://patrola.rs/alarm/abc123?action=take"
        }
      ],
      [
        {
          text: "📍 Lokacija",
          url: "https://maps.google.com/?q=44.81,20.46"
        }
      ]
    ]
  }
};
```

### Webhook (umesto Polling)

```typescript
// Setup webhook za primanje poruka od bota

// 1. Registruj webhook
// POST https://api.telegram.org/bot<TOKEN>/setWebhook
// body: { "url": "https://xxx.convex.site/telegram-webhook" }

// 2. Convex HTTP endpoint
import { httpAction } from "./_generated/server";

export const telegramWebhook = httpAction(async (ctx, request) => {
  const body = await request.json();

  // Handle bot commands
  if (body.message?.text?.startsWith("/")) {
    const command = body.message.text.split(" ")[0];
    const chatId = body.message.chat.id;

    switch (command) {
      case "/start":
        await sendWelcomeMessage(chatId);
        break;
      case "/status":
        await sendStatusMessage(chatId);
        break;
    }
  }

  return new Response("OK");
});
```

### Kod-bazirano Povezivanje Grupe

```typescript
// Kada bot detektuje da je dodat u grupu, šalje kod

export const handleBotAdded = internalAction({
  args: {
    chatId: v.string(),
    chatTitle: v.string(),
  },
  handler: async (ctx, { chatId, chatTitle }) => {
    // Generiši kod
    const code = `TG-${Math.random().toString(36).substring(2, 8).toUpperCase()}`;

    // Sačuvaj u bazu
    await ctx.runMutation(internal.telegram.saveLinkCode, {
      code,
      chatId,
      chatTitle,
      expiresAt: Date.now() + 15 * 60 * 1000, // 15 min
    });

    // Pošalji poruku sa kodom
    await ctx.runAction(internal.telegram.sendMessage, {
      chatId,
      text: `👋 Zdravo! Ja sam PatrolaBot.

Da povežete ovu grupu sa aplikacijom, admin treba da unese ovaj kod u app:

🔑 *KOD: ${code}*

Kod važi 15 minuta.`,
    });
  },
});
```

---

## Troubleshooting

### Česti Problemi

| Problem | Rešenje |
|---------|---------|
| Bot ne šalje poruke | Proveri da li je bot ADMIN u grupi |
| "Chat not found" | Proveri CHAT_ID (mora biti negativan za grupe) |
| "Bot was blocked" | Korisnik je blokirao bota |
| Rate limit exceeded | Implementiraj queue za poruke |
| "Forbidden" | Bot nema permisije u grupi |

### Debug Koraci

```bash
# 1. Proveri da li bot radi
curl "https://api.telegram.org/bot<TOKEN>/getMe"

# 2. Proveri recent updates
curl "https://api.telegram.org/bot<TOKEN>/getUpdates"

# 3. Proveri webhook status (ako koristiš)
curl "https://api.telegram.org/bot<TOKEN>/getWebhookInfo"

# 4. Test poruka
curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -d "chat_id=<CHAT_ID>&text=test"
```

---

## Bezbednost

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  BEZBEDNOSNE PREPORUKE                                                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ✓ Nikada ne objavljuj BOT_TOKEN javno                                          │
│  ✓ Koristi environment variables                                                │
│  ✓ Rotiraj token ako je kompromitovan (/revoke @BotFather)                      │
│  ✓ Minimalna poruka u TG (detalji u app-u)                                      │
│  ✓ Verifikuj webhook requests (ako koristiš)                                    │
│  ✓ Rate limit za slanje poruka                                                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

*Dokument kreiran: Januar 2025*
