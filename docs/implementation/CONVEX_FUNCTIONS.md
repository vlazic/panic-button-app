# Convex Functions - Backend API

## Pregled

Ovaj dokument opisuje sve Convex funkcije koje čine backend API za Panic Button aplikaciju.

---

## Struktura Fajlova

```
convex/
├── schema.ts              # Database schema
├── alarms.ts              # Alarm CRUD operations
├── telegram.ts            # Telegram integration
├── auth.ts                # Authentication (pun sistem)
├── groups.ts              # Group management (pun sistem)
├── memberships.ts         # Membership management (pun sistem)
├── shifts.ts              # Shift scheduling (pun sistem)
├── stats.ts               # Statistics (pun sistem)
├── _generated/            # Auto-generated types
│   ├── api.d.ts
│   ├── dataModel.d.ts
│   └── server.d.ts
└── internal/
    └── escalation.ts      # Internal escalation functions
```

---

## PoC Funkcije

### alarms.ts

```typescript
// convex/alarms.ts

import { v } from "convex/values";
import { mutation, query, action } from "./_generated/server";
import { internal } from "./_generated/api";

// ═══════════════════════════════════════════════════════════════════════════════
// QUERIES (Real-time reads)
// ═══════════════════════════════════════════════════════════════════════════════

/**
 * Dohvati sve aktivne alarme
 * Real-time subscription - automatski se ažurira
 */
export const getActiveAlarms = query({
  args: {},
  handler: async (ctx) => {
    const alarms = await ctx.db
      .query("alarms")
      .withIndex("by_status", (q) => q.eq("status", "ACTIVE"))
      .order("desc")
      .collect();

    return alarms;
  },
});

/**
 * Dohvati sve alarme (za listu)
 */
export const getAllAlarms = query({
  args: {
    limit: v.optional(v.number()),
  },
  handler: async (ctx, { limit = 50 }) => {
    const alarms = await ctx.db
      .query("alarms")
      .withIndex("by_created")
      .order("desc")
      .take(limit);

    return alarms;
  },
});

/**
 * Dohvati jedan alarm po ID-u
 */
export const getAlarm = query({
  args: { id: v.id("alarms") },
  handler: async (ctx, { id }) => {
    return await ctx.db.get(id);
  },
});

// ═══════════════════════════════════════════════════════════════════════════════
// MUTATIONS (Writes)
// ═══════════════════════════════════════════════════════════════════════════════

/**
 * Kreiraj novi alarm (Panic Button pritisnut)
 */
export const createAlarm = mutation({
  args: {
    sender_name: v.string(),
    message: v.optional(v.string()),
    lat: v.number(),
    lng: v.number(),
  },
  handler: async (ctx, args) => {
    // Validacija koordinata
    if (args.lat < -90 || args.lat > 90) {
      throw new Error("Invalid latitude");
    }
    if (args.lng < -180 || args.lng > 180) {
      throw new Error("Invalid longitude");
    }

    // Kreiraj alarm
    const alarmId = await ctx.db.insert("alarms", {
      sender_name: args.sender_name,
      message: args.message,
      lat: args.lat,
      lng: args.lng,
      status: "ACTIVE",
      created_at: Date.now(),
    });

    // Zakaži Telegram notifikaciju (action)
    await ctx.scheduler.runAfter(0, internal.telegram.sendAlarmNotification, {
      alarmId,
    });

    return alarmId;
  },
});

/**
 * Preuzmi alarm ("Preuzimam" dugme)
 */
export const takeAlarm = mutation({
  args: {
    id: v.id("alarms"),
    taken_by: v.string(),
  },
  handler: async (ctx, { id, taken_by }) => {
    const alarm = await ctx.db.get(id);

    if (!alarm) {
      throw new Error("Alarm not found");
    }

    // Proveri da li je već preuzet
    if (alarm.status === "TAKEN") {
      throw new Error("Alarm is already taken");
    }

    // Proveri da li je aktivan
    if (alarm.status !== "ACTIVE") {
      throw new Error("Alarm is not active");
    }

    // Update alarm
    await ctx.db.patch(id, {
      status: "TAKEN",
      taken_by,
      taken_at: Date.now(),
    });

    // Pošalji Telegram update
    await ctx.scheduler.runAfter(0, internal.telegram.sendTakeNotification, {
      alarmId: id,
      taken_by,
    });

    return { success: true };
  },
});

/**
 * Označi alarm kao rešen
 */
export const resolveAlarm = mutation({
  args: {
    id: v.id("alarms"),
    resolution_type: v.optional(
      v.union(
        v.literal("ESCORTED_HOME"),
        v.literal("THREAT_DISPERSED"),
        v.literal("POLICE_CALLED"),
        v.literal("FALSE_ALARM"),
        v.literal("CANCELLED"),
        v.literal("OTHER")
      )
    ),
  },
  handler: async (ctx, { id, resolution_type }) => {
    const alarm = await ctx.db.get(id);

    if (!alarm) {
      throw new Error("Alarm not found");
    }

    const newStatus = resolution_type === "FALSE_ALARM" ? "FALSE_ALARM" : "RESOLVED";

    await ctx.db.patch(id, {
      status: newStatus,
      resolved_at: Date.now(),
    });

    // Pošalji Telegram update
    await ctx.scheduler.runAfter(0, internal.telegram.sendResolveNotification, {
      alarmId: id,
      resolution_type: resolution_type || "RESOLVED",
    });

    return { success: true };
  },
});

/**
 * Otkaži alarm (pošiljalac otkazuje)
 */
export const cancelAlarm = mutation({
  args: {
    id: v.id("alarms"),
  },
  handler: async (ctx, { id }) => {
    const alarm = await ctx.db.get(id);

    if (!alarm) {
      throw new Error("Alarm not found");
    }

    if (alarm.status !== "ACTIVE") {
      throw new Error("Can only cancel active alarms");
    }

    await ctx.db.patch(id, {
      status: "CANCELLED",
      resolved_at: Date.now(),
    });

    return { success: true };
  },
});
```

### telegram.ts

```typescript
// convex/telegram.ts

import { v } from "convex/values";
import { action, internalAction } from "./_generated/server";
import { internal } from "./_generated/api";

// ═══════════════════════════════════════════════════════════════════════════════
// INTERNAL ACTIONS (Called by scheduler)
// ═══════════════════════════════════════════════════════════════════════════════

/**
 * Pošalji notifikaciju o novom alarmu
 */
export const sendAlarmNotification = internalAction({
  args: {
    alarmId: v.id("alarms"),
  },
  handler: async (ctx, { alarmId }) => {
    // Dohvati alarm iz baze
    const alarm = await ctx.runQuery(internal.alarms.getAlarmInternal, {
      id: alarmId,
    });

    if (!alarm) {
      console.error("Alarm not found:", alarmId);
      return;
    }

    const botToken = process.env.TELEGRAM_BOT_TOKEN;
    const chatId = process.env.TELEGRAM_CHAT_ID;
    const appUrl = process.env.APP_URL || "https://patrola.rs";

    if (!botToken || !chatId) {
      console.error("Telegram credentials not configured");
      return;
    }

    // Formatiraj poruku
    const mapsLink = `https://maps.google.com/?q=${alarm.lat},${alarm.lng}`;
    const appLink = `${appUrl}/alarm/${alarmId}`;

    const message = `🚨 *ALARM*

📍 Lokacija: [Otvori mapu](${mapsLink})
${alarm.message ? `💬 "${alarm.message}"` : ""}
👤 Od: ${alarm.sender_name}

👆 [Otvori u aplikaciji](${appLink})`;

    // Pošalji poruku
    try {
      const response = await fetch(
        `https://api.telegram.org/bot${botToken}/sendMessage`,
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            chat_id: chatId,
            text: message,
            parse_mode: "Markdown",
            disable_web_page_preview: false,
          }),
        }
      );

      const result = await response.json();

      if (!result.ok) {
        console.error("Telegram API error:", result);
      }
    } catch (error) {
      console.error("Failed to send Telegram notification:", error);
    }
  },
});

/**
 * Pošalji notifikaciju da je neko preuzeo alarm
 */
export const sendTakeNotification = internalAction({
  args: {
    alarmId: v.id("alarms"),
    taken_by: v.string(),
  },
  handler: async (ctx, { alarmId, taken_by }) => {
    const botToken = process.env.TELEGRAM_BOT_TOKEN;
    const chatId = process.env.TELEGRAM_CHAT_ID;

    if (!botToken || !chatId) {
      return;
    }

    const message = `✅ *${taken_by}* je preuzeo alarm`;

    try {
      await fetch(`https://api.telegram.org/bot${botToken}/sendMessage`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          chat_id: chatId,
          text: message,
          parse_mode: "Markdown",
        }),
      });
    } catch (error) {
      console.error("Failed to send take notification:", error);
    }
  },
});

/**
 * Pošalji notifikaciju da je alarm rešen
 */
export const sendResolveNotification = internalAction({
  args: {
    alarmId: v.id("alarms"),
    resolution_type: v.string(),
  },
  handler: async (ctx, { alarmId, resolution_type }) => {
    const botToken = process.env.TELEGRAM_BOT_TOKEN;
    const chatId = process.env.TELEGRAM_CHAT_ID;

    if (!botToken || !chatId) {
      return;
    }

    const resolutionText: Record<string, string> = {
      ESCORTED_HOME: "Sprovedeno do kuće",
      THREAT_DISPERSED: "Pretnja se razišla",
      POLICE_CALLED: "Pozvana policija",
      FALSE_ALARM: "Lažni alarm",
      CANCELLED: "Otkazano",
      OTHER: "Rešeno",
      RESOLVED: "Rešeno",
    };

    const message = `✅ Alarm razrešen: ${resolutionText[resolution_type] || "Rešeno"}`;

    try {
      await fetch(`https://api.telegram.org/bot${botToken}/sendMessage`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          chat_id: chatId,
          text: message,
          parse_mode: "Markdown",
        }),
      });
    } catch (error) {
      console.error("Failed to send resolve notification:", error);
    }
  },
});

// ═══════════════════════════════════════════════════════════════════════════════
// PUBLIC ACTION (Test connection)
// ═══════════════════════════════════════════════════════════════════════════════

/**
 * Test Telegram konekcije
 */
export const testConnection = action({
  args: {},
  handler: async () => {
    const botToken = process.env.TELEGRAM_BOT_TOKEN;
    const chatId = process.env.TELEGRAM_CHAT_ID;

    if (!botToken || !chatId) {
      return { success: false, error: "Credentials not configured" };
    }

    try {
      const response = await fetch(
        `https://api.telegram.org/bot${botToken}/sendMessage`,
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            chat_id: chatId,
            text: "🔔 Test konekcije uspešan!",
          }),
        }
      );

      const result = await response.json();

      return { success: result.ok, result };
    } catch (error) {
      return { success: false, error: String(error) };
    }
  },
});
```

---

## Pun Sistem - Dodatne Funkcije

### auth.ts

```typescript
// convex/auth.ts

import { v } from "convex/values";
import { mutation, query, action } from "./_generated/server";

/**
 * Pošalji SMS verifikacioni kod
 */
export const sendVerificationCode = action({
  args: {
    phone: v.string(),
  },
  handler: async (ctx, { phone }) => {
    // Normalize phone number
    const normalizedPhone = phone.replace(/\s/g, "");

    // Check rate limiting
    const recentCodes = await ctx.runQuery(internal.auth.getRecentCodes, {
      phone: normalizedPhone,
    });

    if (recentCodes.length >= 3) {
      throw new Error("Too many attempts. Try again in 1 hour.");
    }

    // Generate 6-digit code
    const code = Math.floor(100000 + Math.random() * 900000).toString();

    // Save code to database
    await ctx.runMutation(internal.auth.saveVerificationCode, {
      phone: normalizedPhone,
      code,
      expires_at: Date.now() + 10 * 60 * 1000, // 10 minutes
    });

    // Send SMS via Twilio
    const accountSid = process.env.TWILIO_ACCOUNT_SID;
    const authToken = process.env.TWILIO_AUTH_TOKEN;
    const fromNumber = process.env.TWILIO_PHONE_NUMBER;

    if (!accountSid || !authToken || !fromNumber) {
      throw new Error("SMS service not configured");
    }

    try {
      const response = await fetch(
        `https://api.twilio.com/2010-04-01/Accounts/${accountSid}/Messages.json`,
        {
          method: "POST",
          headers: {
            Authorization: `Basic ${Buffer.from(`${accountSid}:${authToken}`).toString("base64")}`,
            "Content-Type": "application/x-www-form-urlencoded",
          },
          body: new URLSearchParams({
            To: normalizedPhone,
            From: fromNumber,
            Body: `Vaš Patrola kod je: ${code}`,
          }),
        }
      );

      const result = await response.json();

      if (result.error_code) {
        throw new Error(result.error_message);
      }

      return { success: true };
    } catch (error) {
      console.error("SMS error:", error);
      throw new Error("Failed to send SMS");
    }
  },
});

/**
 * Verifikuj SMS kod
 */
export const verifyCode = mutation({
  args: {
    phone: v.string(),
    code: v.string(),
  },
  handler: async (ctx, { phone, code }) => {
    const normalizedPhone = phone.replace(/\s/g, "");

    const storedCode = await ctx.db
      .query("sms_codes")
      .withIndex("by_phone_code", (q) =>
        q.eq("phone", normalizedPhone).eq("code", code)
      )
      .first();

    if (!storedCode) {
      // Increment attempts
      const latestCode = await ctx.db
        .query("sms_codes")
        .withIndex("by_phone", (q) => q.eq("phone", normalizedPhone))
        .order("desc")
        .first();

      if (latestCode) {
        await ctx.db.patch(latestCode._id, {
          attempts: latestCode.attempts + 1,
        });
      }

      throw new Error("Invalid code");
    }

    if (storedCode.expires_at < Date.now()) {
      throw new Error("Code expired");
    }

    if (storedCode.attempts >= 5) {
      throw new Error("Too many attempts");
    }

    // Mark as verified
    await ctx.db.patch(storedCode._id, {
      verified_at: Date.now(),
    });

    // Create or get user
    let user = await ctx.db
      .query("users")
      .withIndex("by_phone", (q) => q.eq("phone", normalizedPhone))
      .first();

    if (!user) {
      const userId = await ctx.db.insert("users", {
        phone: normalizedPhone,
        name: "",
        phone_verified: true,
        verified_at: Date.now(),
        status: "ACTIVE",
        created_at: Date.now(),
      });
      user = await ctx.db.get(userId);
    }

    // Return user info (in real app, would return JWT)
    return {
      success: true,
      userId: user!._id,
      isNewUser: !user!.name,
    };
  },
});
```

### groups.ts

```typescript
// convex/groups.ts

import { v } from "convex/values";
import { mutation, query } from "./_generated/server";

/**
 * Kreiraj novu grupu
 */
export const createGroup = mutation({
  args: {
    name: v.string(),
    description: v.optional(v.string()),
    creatorId: v.id("users"),
  },
  handler: async (ctx, { name, description, creatorId }) => {
    // Generate random PIN
    const pin = Math.floor(100000 + Math.random() * 900000).toString();

    // Create group
    const groupId = await ctx.db.insert("groups", {
      name,
      description,
      access_pin: pin,
      escalation_1_seconds: 90,
      escalation_2_seconds: 60,
      min_responders_per_shift: 2,
      active_from: "07:00",
      active_to: "18:00",
      weekend_active: false,
      status: "ACTIVE",
      created_at: Date.now(),
      created_by: creatorId,
    });

    // Add creator as ADMIN
    await ctx.db.insert("memberships", {
      user_id: creatorId,
      group_id: groupId,
      role: "ADMIN",
      status: "ACTIVE",
      joined_at: Date.now(),
    });

    return { groupId, pin };
  },
});

/**
 * Pridruži se grupi sa PIN-om
 */
export const joinGroup = mutation({
  args: {
    pin: v.string(),
    userId: v.id("users"),
  },
  handler: async (ctx, { pin, userId }) => {
    // Find group by PIN
    const group = await ctx.db
      .query("groups")
      .withIndex("by_pin", (q) => q.eq("access_pin", pin))
      .first();

    if (!group) {
      throw new Error("Invalid PIN");
    }

    if (group.status !== "ACTIVE") {
      throw new Error("Group is not active");
    }

    // Check if already member
    const existing = await ctx.db
      .query("memberships")
      .withIndex("by_user_group", (q) =>
        q.eq("user_id", userId).eq("group_id", group._id)
      )
      .first();

    if (existing) {
      throw new Error("Already a member");
    }

    // Create membership (PENDING for approval)
    await ctx.db.insert("memberships", {
      user_id: userId,
      group_id: group._id,
      role: "RODITELJ",
      status: "PENDING",
      joined_at: Date.now(),
    });

    return { groupId: group._id, status: "PENDING" };
  },
});

/**
 * Dohvati grupe korisnika
 */
export const getUserGroups = query({
  args: {
    userId: v.id("users"),
  },
  handler: async (ctx, { userId }) => {
    const memberships = await ctx.db
      .query("memberships")
      .withIndex("by_user", (q) => q.eq("user_id", userId))
      .filter((q) => q.eq(q.field("status"), "ACTIVE"))
      .collect();

    const groups = await Promise.all(
      memberships.map(async (m) => {
        const group = await ctx.db.get(m.group_id);
        return {
          ...group,
          role: m.role,
          membership_id: m._id,
        };
      })
    );

    return groups.filter(Boolean);
  },
});
```

### stats.ts

```typescript
// convex/stats.ts

import { v } from "convex/values";
import { query } from "./_generated/server";

/**
 * Dohvati mesečne statistike za grupu
 */
export const getMonthlyStats = query({
  args: {
    groupId: v.id("groups"),
    year: v.number(),
    month: v.number(),
  },
  handler: async (ctx, { groupId, year, month }) => {
    const startOfMonth = new Date(year, month - 1, 1).getTime();
    const endOfMonth = new Date(year, month, 0, 23, 59, 59).getTime();

    // Get all alarms in month
    const alarms = await ctx.db
      .query("alarms")
      .withIndex("by_group", (q) => q.eq("group_id", groupId))
      .filter((q) =>
        q.and(
          q.gte(q.field("created_at"), startOfMonth),
          q.lte(q.field("created_at"), endOfMonth)
        )
      )
      .collect();

    // Aggregate
    const total = alarms.length;
    const resolved = alarms.filter((a) => a.status === "RESOLVED").length;
    const falseAlarms = alarms.filter((a) => a.status === "FALSE_ALARM").length;
    const escalated = alarms.filter((a) => a.escalated_2_at !== undefined).length;

    // Average response time
    const responseTimes = alarms
      .filter((a) => a.taken_at && a.created_at)
      .map((a) => a.taken_at! - a.created_at);

    const avgResponseTime =
      responseTimes.length > 0
        ? responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length
        : 0;

    // Distribution by hour
    const byHour: Record<number, number> = {};
    alarms.forEach((a) => {
      const hour = new Date(a.created_at).getHours();
      byHour[hour] = (byHour[hour] || 0) + 1;
    });

    return {
      summary: {
        total,
        resolved,
        falseAlarms,
        escalated,
        avgResponseTime: Math.round(avgResponseTime / 1000), // seconds
      },
      byHour,
    };
  },
});
```

---

## Internal Functions

### internal/escalation.ts

```typescript
// convex/internal/escalation.ts

import { v } from "convex/values";
import { internalMutation, internalAction } from "../_generated/server";
import { internal } from "../_generated/api";

/**
 * Eskalacija Level 1 - nakon 90 sekundi
 * Obavesti sve respondere
 */
export const level1 = internalMutation({
  args: {
    alarmId: v.id("alarms"),
  },
  handler: async (ctx, { alarmId }) => {
    const alarm = await ctx.db.get(alarmId);

    // Proveri da li je alarm još aktivan
    if (!alarm || alarm.status !== "ACTIVE") {
      return; // Već preuzet ili rešen
    }

    // Update status
    await ctx.db.patch(alarmId, {
      status: "ESCALATED_1",
      escalated_1_at: Date.now(),
    });

    // Zakaži Level 2 eskalaciju
    await ctx.scheduler.runAfter(
      60000, // 60 sekundi
      internal.escalation.level2,
      { alarmId }
    );

    // Pošalji notifikaciju svim responderima
    await ctx.scheduler.runAfter(0, internal.telegram.sendEscalation1, {
      alarmId,
    });
  },
});

/**
 * Eskalacija Level 2 - nakon dodatnih 60 sekundi
 * Obavesti sve članove grupe
 */
export const level2 = internalMutation({
  args: {
    alarmId: v.id("alarms"),
  },
  handler: async (ctx, { alarmId }) => {
    const alarm = await ctx.db.get(alarmId);

    // Proveri da li je alarm još eskaliran (nije preuzet)
    if (!alarm || !["ACTIVE", "ESCALATED_1"].includes(alarm.status)) {
      return;
    }

    // Update status
    await ctx.db.patch(alarmId, {
      status: "ESCALATED_2",
      escalated_2_at: Date.now(),
    });

    // Pošalji notifikaciju svim članovima
    await ctx.scheduler.runAfter(0, internal.telegram.sendEscalation2, {
      alarmId,
    });
  },
});
```

---

## API Reference

### Queries

| Function | Arguments | Returns | Description |
|----------|-----------|---------|-------------|
| `alarms.getActiveAlarms` | - | `Alarm[]` | Svi aktivni alarmi |
| `alarms.getAllAlarms` | `limit?` | `Alarm[]` | Svi alarmi (paginated) |
| `alarms.getAlarm` | `id` | `Alarm \| null` | Jedan alarm |
| `groups.getUserGroups` | `userId` | `Group[]` | Grupe korisnika |
| `stats.getMonthlyStats` | `groupId, year, month` | `Stats` | Mesečne statistike |

### Mutations

| Function | Arguments | Returns | Description |
|----------|-----------|---------|-------------|
| `alarms.createAlarm` | `sender_name, lat, lng, message?` | `alarmId` | Kreiraj alarm |
| `alarms.takeAlarm` | `id, taken_by` | `{success}` | Preuzmi alarm |
| `alarms.resolveAlarm` | `id, resolution_type?` | `{success}` | Reši alarm |
| `alarms.cancelAlarm` | `id` | `{success}` | Otkaži alarm |
| `groups.createGroup` | `name, creatorId` | `{groupId, pin}` | Kreiraj grupu |
| `groups.joinGroup` | `pin, userId` | `{groupId, status}` | Pridruži se grupi |

### Actions

| Function | Arguments | Returns | Description |
|----------|-----------|---------|-------------|
| `telegram.testConnection` | - | `{success, error?}` | Test TG konekcije |
| `auth.sendVerificationCode` | `phone` | `{success}` | Pošalji SMS kod |

---

*Dokument kreiran: Januar 2025*
