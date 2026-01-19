# Životni Ciklus Alarma

## Pregled

Alarm prolazi kroz više faza od aktivacije do razrešenja, sa automatskom eskalacijom ako niko ne reaguje.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ALARM LIFECYCLE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  AKTIVACIJA ──► ACTIVE ──► ESCALATED_1 ──► ESCALATED_2                      │
│                    │                            │                           │
│                    └─────────► RESPONDING ◄─────┘                           │
│                                    │                                        │
│                                    ▼                                        │
│                               ON_SCENE                                      │
│                                    │                                        │
│                                    ▼                                        │
│                               RESOLVED                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Status alarma

| Status | Opis |
|--------|------|
| `ACTIVE` | Alarm je aktivan, čeka reakciju respondera na smeni |
| `ESCALATED_1` | Niko nije reagovao za 90s, notifikovani svi responderi |
| `ESCALATED_2` | Niko nije reagovao za 150s, notifikovani svi članovi |
| `RESPONDING` | Neko je preuzeo alarm i ide na lokaciju |
| `ON_SCENE` | Responder je stigao na lokaciju |
| `RESOLVED` | Situacija je rešena |
| `CANCELLED` | Pošiljalac je otkazao alarm |
| `FALSE_ALARM` | Označeno kao lažni alarm |

---

## Faza 0: Aktivacija

### Korisnik drži panic button 3 sekunde

```
┌─────────────────────────────────────┐
│                                     │
│         🔴                          │
│     DRŽI ZA ALARM                   │
│                                     │
│     ████████████░░░░ 2.1s           │
│                                     │
│     (pusti za otkazivanje)          │
│                                     │
└─────────────────────────────────────┘
```

**Zašto 3 sekunde?**
- Sprečava slučajne klikove
- Daje vreme za odustajanje
- Signal da je akcija namerna

### Forma za detalje

```
┌─────────────────────────────────────┐
│                                     │
│   📍 Lokacija: Detektovana          │
│                                     │
│   Šta se dešava? (opciono)          │
│                                     │
│   [Prate me  ] [Tuča     ]          │
│   [Kradu     ] [Preti mi ]          │
│                                     │
│   [ Drugo: __________________ ]     │
│                                     │
│   [🚨 POŠALJI ALARM]                │
│                                     │
└─────────────────────────────────────┘
```

---

## Faza 1: ACTIVE (T=0)

### Backend logika

```typescript
async function createAlarm({ location, message, group_id }) {
  // 1. Kreiraj alarm
  const alarm = await db.insert("alarms", {
    group_id,
    triggered_by: user._id,
    location_lat: location.lat,
    location_lng: location.lng,
    message,
    status: "ACTIVE",
    created_at: Date.now(),
  });

  // 2. Nađi RESPONDERE na smeni
  const activeResponders = await getActiveResponders(group_id);

  // 3. Sortiraj po blizini
  const sorted = sortByDistance(activeResponders, location);

  // 4. Kreiraj alarm_responses za svakog
  for (const responder of sorted) {
    await db.insert("alarm_responses", {
      alarm_id: alarm._id,
      user_id: responder.user_id,
      notified_at: Date.now(),
      distance_meters: responder.distance,
    });
  }

  // 5. Pošalji Telegram
  await sendTelegramAlert(group_id, alarm);

  // 6. Zakaži eskalaciju
  await scheduler.runAfter(90_000, "escalateAlarm", { alarm_id: alarm._id });

  return alarm;
}
```

### Šta vide responderi na smeni

```
┌─────────────────────────────────────────────┐
│  🚨 ALARM - OŠ Kovačić                      │
│                                             │
│  📍 300m od tebe (Kod fontane)              │
│  💬 "Prate me"                              │
│  👤 Od: Dete Markovića                      │
│                                             │
│  Ti si 1 od 2 dostupna respondera           │
│                                             │
│  ⏱️ Eskalacija za: 1:23                     │
│                                             │
│  [🏃 PREUZIMAM]                             │
│                                             │
│  Ne mogu jer: [ Izaberi razlog ▼ ]          │
│                                             │
│  👁️ Videlo: 2    ✅ Preuzelo: 0             │
└─────────────────────────────────────────────┘
```

### Telegram poruka

```
🚨 *ALARM*

📍 Lokacija: https://maps.google.com/?q=44.81,20.46
💬 "Prate me"
👤 Od: Dete Markovića

👆 Otvori app: https://patrola.rs/alarm/xyz
```

---

## Faza 2: Eskalacija 1 (T=90s)

### Trigger

Scheduled function se pokreće 90 sekundi nakon kreiranja alarma.

```typescript
async function escalateAlarm({ alarm_id }) {
  const alarm = await db.get(alarm_id);

  // Ako je već neko preuzeo, ne radi ništa
  if (alarm.status !== "ACTIVE") return;

  // Update status
  await db.patch(alarm_id, {
    status: "ESCALATED_1",
    escalated_1_at: Date.now(),
  });

  // Notifikuj SVE respondere (ne samo one na smeni)
  const allResponders = await getAllResponders(alarm.group_id);
  await sendUrgentNotification(allResponders, alarm, "ESCALATED");

  // Zakaži sledeću eskalaciju
  await scheduler.runAfter(60_000, "escalateAlarmFinal", { alarm_id });
}
```

### Poruka za sve respondere

```
┌─────────────────────────────────────────────┐
│  ⚠️ ESKALIRAN ALARM - OŠ Kovačić           │
│                                             │
│  Niko od dostupnih respondera nije          │
│  preuzeo alarm već 90 sekundi!              │
│                                             │
│  📍 Kod fontane                             │
│  💬 "Prate me"                              │
│                                             │
│  [🏃 PREUZIMAM]                             │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Faza 3: Eskalacija 2 - Broadcast (T=150s)

### Trigger

Ako niko nije reagovao ni nakon eskalacije 1.

```typescript
async function escalateAlarmFinal({ alarm_id }) {
  const alarm = await db.get(alarm_id);

  if (alarm.status !== "ESCALATED_1") return;

  await db.patch(alarm_id, {
    status: "ESCALATED_2",
    escalated_2_at: Date.now(),
  });

  // Notifikuj SVE članove grupe (uključujući RODITELJE)
  const allMembers = await getAllMembers(alarm.group_id);
  await sendCriticalNotification(allMembers, alarm);

  // Pozovi admina direktno (opciono)
  await callAdmin(alarm.group_id, alarm);
}
```

### Poruka za sve članove

```
┌─────────────────────────────────────────────┐
│  🆘 KRITIČNO - ALARM BEZ ODGOVORA           │
│                                             │
│  Alarm je aktivan već 2.5 minuta            │
│  i NIKO nije preuzeo!                       │
│                                             │
│  Ako ste u blizini škole Kovačić,           │
│  molimo reagujte!                           │
│                                             │
│  [OTVORI DETALJE]                           │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Faza 4: RESPONDING (neko preuzeo)

### Kada responder klikne "Preuzimam"

```typescript
async function takeAlarm({ alarm_id, taken_by }) {
  const alarm = await db.get(alarm_id);

  if (!["ACTIVE", "ESCALATED_1", "ESCALATED_2"].includes(alarm.status)) {
    throw new Error("Alarm nije u stanju za preuzimanje");
  }

  // Update alarm
  await db.patch(alarm_id, {
    status: "RESPONDING",
    responded_at: Date.now(),
  });

  // Update alarm_response
  await db.patch(responseId, {
    response: "ACCEPTED",
    response_at: Date.now(),
    eta_minutes: calculateETA(responderLocation, alarmLocation),
  });

  // OTKAŽI zakazanu eskalaciju
  await scheduler.cancel(escalationJobId);

  // Telegram update
  await sendTelegramUpdate(`✅ ${taken_by} je preuzeo alarm`);
}
```

### Šta vidi responder

```
┌─────────────────────────────────────────────┐
│  ✅ Preuzeo si alarm                         │
│                                             │
│  📍 Navigacija do lokacije:                 │
│  [OTVORI GOOGLE MAPS]                       │
│                                             │
│  📞 Pozovi dete: +381 63 xxx xxxx           │
│  📞 Pozovi roditelja: +381 64 xxx xxxx      │
│                                             │
│  Javi kad stigneš:                          │
│  [✅ STIGAO SAM]                            │
│                                             │
│  Problem?                                   │
│  [❌ MORAM DA ODUSTANEM]                    │
│                                             │
└─────────────────────────────────────────────┘
```

### Šta vide ostali

```
┌─────────────────────────────────────────────┐
│  🚨 ALARM                                   │
│                                             │
│  ✅ Petar je preuzeo                        │
│  📍 Udaljen ~400m                           │
│  ⏱️ ETA: ~4 min                             │
│                                             │
│  [📍 VIDI LOKACIJU]                         │
│                                             │
└─────────────────────────────────────────────┘
```

### Šta vidi pošiljalac (dete/roditelj)

```
┌─────────────────────────────────────────────┐
│  ✅ Pomoć je na putu!                        │
│                                             │
│  👤 Petar P. dolazi                         │
│  📍 Udaljen ~400m                           │
│  ⏱️ Očekivano vreme: ~4 min                 │
│                                             │
│  Ako je lažna uzbuna:                       │
│  [OTKAŽI ALARM]                             │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Faza 5: ON_SCENE (stigao)

### Kada responder klikne "Stigao sam"

```typescript
async function arriveAtScene({ alarm_id }) {
  await db.patch(alarm_id, {
    status: "ON_SCENE",
  });

  await db.patch(responseId, {
    arrived_at: Date.now(),
  });

  await sendTelegramUpdate("📍 Responder je stigao na lokaciju");
}
```

### Interface za razrešenje

```
┌─────────────────────────────────────────────┐
│  📍 Na licu mesta                           │
│                                             │
│  Kad se situacija reši, označi:             │
│                                             │
│  [✅ REŠENO - Sve OK]                       │
│                                             │
│  [⚠️ REŠENO - Potrebna intervencija]        │
│     (treba pozvati policiju, roditelje...)  │
│                                             │
│  [❌ LAŽNI ALARM]                           │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Faza 6: RESOLVED (razrešeno)

### Forma za razrešenje

```
┌─────────────────────────────────────────────┐
│  Kako je rešeno?                            │
│                                             │
│  [○] Situacija procenjena kao neaktuelna    │
│  [○] Dete sprovedeno do bezbedne lokacije   │
│  [○] Pozvana policija                       │
│  [○] Pozvani roditelji                       │
│  [○] Drugo                                  │
│                                             │
│  Dodatne napomene:                          │
│  [ ___________________________________ ]    │
│                                             │
│  [ZAVRŠI]                                   │
│                                             │
└─────────────────────────────────────────────┘
```

### Backend

```typescript
async function resolveAlarm({ alarm_id, resolution, notes }) {
  await db.patch(alarm_id, {
    status: "RESOLVED",
    resolved_at: Date.now(),
    resolution_notes: notes,
    resolved_by: user._id,
  });

  // Update responder statistike
  await updateResponderStats(responderId, {
    alarmsResolved: increment(1),
    // ...
  });

  // Telegram
  await sendTelegramUpdate("✅ Alarm razrešen");

  // Audit log
  await db.insert("audit_log", {
    action: "ALARM_RESOLVED",
    alarm_id,
    user_id: user._id,
    details: JSON.stringify({ resolution, notes }),
  });
}
```

---

## Dijagram state machine

```
                              CANCELLED
                                 ▲
                                 │
        ┌────────────────────────┤
        │                        │
        │                        │
        ▼                        │
     ACTIVE ──────90s────► ESCALATED_1 ──────60s────► ESCALATED_2
        │                        │                         │
        │                        │                         │
        │    ┌───────────────────┴─────────────────────────┘
        │    │
        │    │ neko preuzme
        ▼    ▼
     RESPONDING
        │
        │ stigao
        ▼
     ON_SCENE
        │
        │ reši
        ▼
     RESOLVED ◄──────────────────────────────────────── FALSE_ALARM
```

---

## Vremenska linija tipičnog alarma

```
T=0      Alarm kreiran, status: ACTIVE
         → Notifikovani responderi na smeni (2)

T=15s    Responder 1 video alarm
T=32s    Responder 1 kliknuo "Ne mogu" (razlog: na poslu)

T=45s    Responder 2 video alarm
T=52s    Responder 2 kliknuo "PREUZIMAM"
         → Status: RESPONDING
         → Eskalacija otkazana

T=4min   Responder 2 kliknuo "STIGAO SAM"
         → Status: ON_SCENE

T=9min   Responder 2 označio kao RESOLVED
         → Status: RESOLVED
         → Ukupno vreme: 9 minuta
```

---

## Edge case-ovi

| Situacija | Ponašanje |
|-----------|-----------|
| Responder preuzme pa odustane | Alarm se vraća u ACTIVE, eskalacija kreće ispočetka |
| Dva respondera preuzmu istovremeno | Oba se beleže kao ACCEPTED, koordinacija u app-u |
| Pošiljalac otkaže dok neko ide | Responder dobija notifikaciju "Alarm povučen" |
| Niko ne reaguje ni na broadcast | Ostaje u ESCALATED_2, admin notifikovan |

---

*Dokument kreiran: Januar 2025*
