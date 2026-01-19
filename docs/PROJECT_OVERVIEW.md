# Pregled Projekta - Panic Button App

## Pozadina

### Problem

U okolini OŠ Ivan Goran Kovačić, blizu Vukovog spomenika u Beogradu, u poslednja dva meseca učestale su:
- Pretnje učenicima osnovne škole
- Slučajevi uznemiravanja
- Pljačkanja osnovaca

Počinioci su uglavnom uigrana ekipa lokalnih mangupa starosti **14-18 godina**.

### Trenutna situacija

- Roditelji i osoblje škole su se intenzivno obraćali **policiji**
- Uz sva deklarativna obećanja, policija nije uspela (a ni pokušala) da promeni situaciju na terenu
- Na pomolu je spontana organizacija **roditeljske patrole**

### Inspiracija

Ideja je inspirisana mnogim volonterskim poduhvatima viđenim na društvenim mrežama:
- Razne mape
- Liste
- Koordinacione platforme

---

## Rešenje

### Vizija

Besplatna neprofitna aplikacija koja bi služila:
- Primarno: Roditeljima OŠ Ivan Goran Kovačić
- Dugoročno: Drugim školama i roditeljskim grupama širom zemlje

### Ključne funkcionalnosti

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PANIC BUTTON APP                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ZA DECU/UČENIKE:                                                           │
│  ────────────────                                                           │
│  • Panik dugme - jedan pritisak šalje lokaciju i alarm                      │
│  • "Stigao sam bezbedno" - check-in funkcija                                │
│  • GPS praćenje (opciono)                                                   │
│                                                                             │
│  ZA RODITELJE/PATROLE:                                                      │
│  ─────────────────────                                                      │
│  • Raspored patrola - ko, kada i gde patrolira                              │
│  • Mapa zone - prikaz "vrućih tačaka" gde su incidenti česti                │
│  • Grupni chat - brza koordinacija                                          │
│  • Push notifikacije - hitna obaveštenja                                    │
│                                                                             │
│  PRIJAVLJIVANJE INCIDENATA:                                                 │
│  ──────────────────────────                                                 │
│  • Obrazac za prijavu - šta, kada, gde, opis                                │
│  • Anonimna prijava - za one koji se plaše odmazde                          │
│  • Arhiva incidenata - statistika za pritisak na policiju                   │
│                                                                             │
│  DODATNO:                                                                   │
│  ────────                                                                   │
│  • "Bezbedne kuće" - označeni lokali gde dete može potražiti pomoć          │
│  • Kontakti - policija, škola, roditelji                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Suština

Aplikacija spaja:
1. **Brzu komunikaciju** - alarm stiže u sekundi
2. **Koordinaciju volontera** - ko ide, ko je blizu
3. **Dokumentovanje problema** - statistika za institucije

---

## Pristup implementaciji

### Minimalni PoC (Proof of Concept)

Za brzi početak, fokus na apsolutni minimum:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  MINIMALNI POC                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Panic dugme + opcioni tekst                                                │
│         │                                                                   │
│         ▼                                                                   │
│  Šalje lokaciju i poruku na API endpoint                                    │
│         │                                                                   │
│         ▼                                                                   │
│  API prosleđuje u Telegram/Viber/WhatsApp grupu                             │
│         │                                                                   │
│         ▼                                                                   │
│  Web front za pregled aktivnih alarma                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Zašto Telegram?

- Većina korisnika već ima Telegram
- Notifikacije rade **izuzetno pouzdano** (čak i na battery saver-u)
- Grupni chat je besplatan
- API je jednostavan za integraciju

### PWA umesto native app

- Nema App Store review procesa
- Instalira se direktno iz browser-a
- Radi offline (sa ograničenjima)
- Brže do korisnika

---

## Arhitektura - High Level

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              KORISNICI                                      │
│                                                                             │
│          📱 PWA (mobilni browser)        📱 Telegram (notifikacije)         │
│                      │                              ▲                       │
│                      │                              │                       │
│                      ▼                              │                       │
│              ┌─────────────┐                        │                       │
│              │   VERCEL    │                        │                       │
│              │  (Frontend) │                        │                       │
│              └──────┬──────┘                        │                       │
│                     │                               │                       │
│                     ▼                               │                       │
│              ┌─────────────┐              ┌─────────────┐                   │
│              │   CONVEX    │─────────────►│  TELEGRAM   │                   │
│              │  (Backend)  │              │    BOT      │                   │
│              │  (Database) │              └─────────────┘                   │
│              └─────────────┘                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Tech Stack

| Komponenta | Tehnologija | Obrazloženje |
|------------|-------------|--------------|
| Frontend | Next.js + React | PWA podrška, brz development |
| Backend | Convex | Real-time, serverless, besplatan tier |
| Database | Convex (ugrađen) | Transakcije, subscriptions |
| Notifikacije | Telegram Bot API | Pouzdanost, besplatno |
| SMS (pun sistem) | Twilio | Verifikacija telefona |
| Hosting | Vercel | Automatic deploys, CDN |
| CDN | Cloudflare | Besplatno, globalna distribucija |

---

## Faze razvoja

### Faza 1: PoC (1-2 sata)
- Panic button sa lokacijom
- Telegram notifikacija
- "Preuzimam" dugme
- Jedna škola, zajednički PIN

### Faza 2: MVP (1-2 nedelje)
- SMS verifikacija
- RESPONDER uloga
- Osnovne smene
- 1 nivo eskalacije
- Osnovne statistike

### Faza 3: Pun sistem (1-2 meseca)
- Više škola/grupa
- Sve uloge (ADMIN, RODITELJ, RESPONDER, DETE)
- 3 nivoa eskalacije
- Admin panel
- Kompletan reporting
- Geofencing

### Faza 4: Skaliranje
- Native aplikacije (iOS/Android)
- API za integraciju
- White-label za druge gradove

---

## Ciljevi projekta

### Kratkoročni (PoC)
1. Funkcionalan panic button za jednu školu
2. Validacija koncepta sa stvarnim korisnicima
3. Prikupljanje feedback-a

### Srednjoročni (6 meseci)
1. Stabilna platforma za 5-10 škola u Beogradu
2. Dokumentovana statistika incidenata
3. Izveštaji za policiju i institucije

### Dugoročni (1+ godina)
1. Platforma dostupna svim školama u Srbiji
2. Partnerstva sa školama i lokalnim samoupravama
3. Merljivo smanjenje incidenata

---

## Neprofitni karakter

Ovo je **volonterski projekat** bez komercijalnih ciljeva:
- Kod je open-source
- Nema pretplata ni naplaćivanja
- Troškovi infrastrukture su minimalni (~$5-15/mesec za jednu školu)
- Moguće tražiti sponzorstva od tech kompanija (Twilio.org, Vercel OSS, itd.)

---

## Kontakt i doprinosi

Projekat je otvoren za:
- Programere koji žele da doprinesu kodom
- Dizajnere za UI/UX poboljšanja
- Roditelje koji žele da testiraju
- Škole koje žele da se priključe

---

*Dokument kreiran: Januar 2026*
*Poslednje ažuriranje: Januar 2026*
