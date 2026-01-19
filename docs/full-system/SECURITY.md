# Bezbednosna Razmatranja

## Pregled

Ovaj dokument opisuje bezbednosne mere implementirane u sistemu Panic Button, potencijalne rizike i njihove mitigacije.

---

## Rate Limiting

### SMS Verifikacija

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  SMS RATE LIMITING                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OGRANIČENJA:                                                               │
│  • Max 3 SMS-a po broju telefona na sat                                     │
│  • Max 10 SMS-a po broju telefona na dan                                    │
│  • Max 5 pokušaja unosa koda (zatim novi kod)                               │
│  • Kod ističe za 10 minuta                                                  │
│                                                                             │
│  IMPLEMENTACIJA:                                                            │
│  sms_codes tabela prati:                                                    │
│  • attempts: broj pokušaja                                                  │
│  • created_at: vreme kreiranja                                              │
│  • expires_at: vreme isteka                                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Panic Button

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PANIC BUTTON RATE LIMITING                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OGRANIČENJA:                                                               │
│  • Max 1 aktivan alarm po korisniku                                         │
│  • Cooldown: 5 minuta između alarma                                         │
│  • Drži 3 sekunde za aktivaciju (sprečava slučajne klikove)                 │
│                                                                             │
│  RAZLOG:                                                                    │
│  Sprečava spam alarma i potencijalnu zloupotrebu                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### PIN Brute-force Zaštita

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  PIN ZAŠTITA                                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  OGRANIČENJA:                                                               │
│  • Max 5 pokušaja za unos PIN-a                                             │
│  • Nakon 5 pokušaja: lockout 1 sat                                          │
│  • Praćenje po IP adresi + korisničkom agentu                               │
│                                                                             │
│  IMPLEMENTACIJA:                                                            │
│  • Redis/Convex čuva broj pokušaja                                          │
│  • TTL 1 sat za lockout                                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Autentifikacija

### JWT Tokeni

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  JWT KONFIGURACIJA                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ACCESS TOKEN:                                                              │
│  • Expiry: 24 sata                                                          │
│  • Sadrži: user_id, role, group_ids                                         │
│  • Potpisan sa JWT_SECRET                                                   │
│                                                                             │
│  REFRESH TOKEN (pun sistem):                                                │
│  • Expiry: 30 dana                                                          │
│  • Čuva se u HTTP-only cookie                                               │
│  • Rotira se pri svakom korišćenju                                          │
│                                                                             │
│  VALIDACIJA:                                                                │
│  • Svaki API poziv proverava token                                          │
│  • Expired token vraća 401                                                  │
│  • Invalid token vraća 403                                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Session Management

```typescript
// Primer validacije
async function validateRequest(ctx) {
  const token = ctx.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    throw new Error('Unauthorized');
  }

  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    return decoded;
  } catch (e) {
    throw new Error('Invalid token');
  }
}
```

---

## Autorizacija

### Role-based Access Control (RBAC)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  RBAC IMPLEMENTACIJA                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PROVERA ULOGE:                                                             │
│  • Svaka mutacija proverava ulogu korisnika                                 │
│  • membership tabela čuva user_id + group_id + role                         │
│  • Korisnik može imati različite uloge u različitim grupama                 │
│                                                                             │
│  PRIMER:                                                                    │
│  async function adminAction(ctx, { group_id }) {                            │
│    const membership = await getMembership(ctx, user._id, group_id);         │
│    if (membership.role !== 'ADMIN') {                                       │
│      throw new Error('Forbidden - Admin required');                         │
│    }                                                                        │
│    // ... nastavi                                                           │
│  }                                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Privatnost podataka

### GDPR Compliance

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  GDPR ZAHTEVI                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PRAVO NA PRISTUP:                                                          │
│  • Korisnik može da vidi sve svoje podatke                                  │
│  • Export u JSON formatu                                                    │
│                                                                             │
│  PRAVO NA BRISANJE:                                                         │
│  • Korisnik može da obriše nalog                                            │
│  • Briše: profil, članstva, alarm_responses                                 │
│  • NE briše: alarme (anonymizuje sender)                                    │
│                                                                             │
│  MINIMIZACIJA PODATAKA:                                                     │
│  • Čuvamo samo neophodne podatke                                            │
│  • Lokacije se čuvaju samo za aktivne alarme                                │
│  • Stari alarmi se automatski anonimizuju nakon 6 meseci                    │
│                                                                             │
│  OBAVEŠTENJE:                                                               │
│  • Jasna politika privatnosti                                               │
│  • Saglasnost pri registraciji                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Vidljivost podataka

| Podatak | Ko vidi |
|---------|---------|
| Lokacija alarma | Samo članovi grupe |
| Telefon korisnika | Samo ADMIN + sami sebe |
| Ime korisnika | Članovi grupe |
| Statistike | ADMIN + RESPONDER (svoje) |
| Istorija alarma | ADMIN |

---

## Bezbednost komunikacije

### HTTPS

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HTTPS KONFIGURACIJA                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  • Sva komunikacija preko HTTPS (TLS 1.3)                                   │
│  • Vercel automatski obezbeđuje SSL certifikat                              │
│  • HSTS header uključen                                                     │
│  • HTTP automatski redirect na HTTPS                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Telegram komunikacija

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  TELEGRAM BEZBEDNOST                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  • Bot token čuvan kao environment variable                                 │
│  • CHAT_ID verifikovan pre slanja                                           │
│  • Minimalna poruka u TG (detalji u app-u)                                  │
│  • Webhook koristi HTTPS                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Sprečavanje zloupotrebe

### Lažni alarmi

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  MITIGACIJA LAŽNIH ALARMA                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PREVENTIVNO:                                                               │
│  • Hold 3 sekunde za aktivaciju                                             │
│  • Cooldown 5 minuta                                                        │
│  • Verified korisnici (SMS)                                                 │
│  • Approval workflow za nove članove                                        │
│                                                                             │
│  DETEKTIVNO:                                                                │
│  • FALSE_ALARM se beleži                                                    │
│  • Admin vidi istoriju po korisniku                                         │
│  • Pattern detekcija (više lažnih → warning)                                │
│                                                                             │
│  KOREKTIVNO:                                                                │
│  • Suspend za ponavljače                                                    │
│  • Ban za ozbiljne slučajeve                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Neovlašćen pristup

| Vektor napada | Mitigacija |
|---------------|------------|
| PIN brute-force | Rate limit, lockout |
| Token krađa | Short expiry, HTTPS only |
| Neovlašćen član | Approval workflow |
| Insider threat | Audit log, RBAC |

---

## Audit Log

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  AUDIT LOG                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ŠTA SE LOGUJE:                                                             │
│  • Kreiranje/brisanje grupe                                                 │
│  • Approve/Reject člana                                                     │
│  • Promena uloge                                                            │
│  • Suspend/Ban                                                              │
│  • Promena podešavanja grupe                                                │
│  • Telegram connect/disconnect                                              │
│                                                                             │
│  FORMAT:                                                                    │
│  {                                                                          │
│    user_id: "abc123",                                                       │
│    group_id: "xyz789",                                                      │
│    action: "MEMBER_BANNED",                                                 │
│    details: { target_user: "def456", reason: "Spam alarmi" },               │
│    timestamp: 1705671600000,                                                │
│    ip_address: "192.168.1.1"                                                │
│  }                                                                          │
│                                                                             │
│  RETENCIJA: 1 godina                                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Checklist pre produkcije

### Kritično

- [ ] HTTPS enforced
- [ ] JWT_SECRET je jak i čuvan bezbedno
- [ ] Rate limiting implementiran
- [ ] Input validacija na svim endpoint-ima
- [ ] SQL injection zaštita (Convex ga nema, ali validacija ostaje)
- [ ] XSS zaštita (React automatski escape-uje)

### Važno

- [ ] PIN bruteforce zaštita
- [ ] SMS rate limiting
- [ ] Audit log aktivan
- [ ] GDPR disclaimer na registraciji
- [ ] Error poruke ne otkrivaju interne detalje

### Preporučeno

- [ ] Sentry za error tracking
- [ ] Health check endpoint
- [ ] Backup strategija
- [ ] Incident response plan

---

## Poznata ograničenja

1. **PoC nema SMS verifikaciju** - Prihvatljivo za test
2. **Zajednički PIN u PoC-u** - Svako sa PIN-om može sve
3. **Nema 2FA** - Može se dodati kasnije
4. **Lokacija nije enkriptovana** - Vidljiva članovima grupe

---

## Disclaimer

```
NAPOMENA: Ovaj sistem je volonterski projekat i NE GARANTUJE bezbednost.
Roditelji su i dalje primarno odgovorni za bezbednost svoje dece.
Sistem služi kao dodatna mera koordinacije, ne kao zamena za nadležne organe.
```

---

*Dokument kreiran: Januar 2026*
