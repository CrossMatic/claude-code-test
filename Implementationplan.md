# Implementationsplan: CrossMatic Client Dashboard

## Übersicht

Ein separates, eigenständiges Client-Dashboard auf `dashboard.getcrossmatic.com`, das optisch exakt mit der bestehenden CrossMatic Website harmoniert. Kunden loggen sich ein und sehen ihre wöchentlich gelieferten Leads.

---

## Tech Stack

| Bereich | Technologie | Begründung |
|---|---|---|
| Framework | **Next.js 15** (App Router) | Server-side Auth, API Routes, Sicherheit |
| Styling | **Tailwind CSS + shadcn/ui** | Identisch zur bestehenden Website |
| Auth + DB | **Supabase** | Bereits vorhanden, RLS aktiv |
| Deployment | **Vercel** | Einfach, schnell, passt zu Next.js |

---

## Design-System (exakt von nebula-flow übernommen)

```css
--background:        hsl(222 50% 7%)   /* Fast-schwarz Navy */
--foreground:        hsl(210 40% 98%)  /* Fast-weiss */
--card:              hsl(220 30% 14%)  /* Dunkle Card-Fläche */
--border:            hsl(220 30% 20%)  /* Subtile Border */
--primary:           hsl(210 100% 65%) /* Bright Blue Akzent */
--muted-foreground:  hsl(215 20% 65%)  /* Gedämpfter Text */
--radius:            0.5rem
```

- **Font:** Inter (Google Fonts)
- **Icons:** lucide-react
- **Komponenten:** shadcn/ui (style: default, baseColor: slate)
- **Animationen:** tailwindcss-animate + framer-motion (optional)

---

## Projektstruktur

```
crossmatic-dashboard/
├── src/
│   ├── app/
│   │   ├── globals.css          # CSS-Variablen (identisch zu nebula-flow)
│   │   ├── layout.tsx           # Root Layout mit Font + Dark Mode
│   │   ├── page.tsx             # Redirect: / → /login
│   │   ├── login/
│   │   │   └── page.tsx         # Login-Seite
│   │   └── dashboard/
│   │       ├── layout.tsx       # Dashboard Layout (Auth Guard)
│   │       └── page.tsx         # Lead-Übersicht
│   ├── components/
│   │   ├── ui/                  # shadcn/ui Komponenten
│   │   ├── LeadCard.tsx         # Einzelne Lead-Karte
│   │   ├── LeadGrid.tsx         # Grid aller Leads
│   │   └── Navbar.tsx           # Dashboard-Navigation
│   └── lib/
│       ├── supabase/
│       │   ├── client.ts        # Browser Supabase Client
│       │   └── server.ts        # Server Supabase Client (SSR)
│       └── utils.ts             # cn() Helper
├── middleware.ts                 # Auth-Schutz aller /dashboard Routen
├── .env.local                   # NEXT_PUBLIC_SUPABASE_URL + ANON_KEY
└── package.json
```

---

## Seiten & Komponenten

### 1. Login-Seite (`/login`)

- Zentriertes Card auf dunklem Hintergrund
- CrossMatic Logo oben
- Email + Passwort Felder (shadcn/ui `Input`)
- "Einloggen →" Button (primary blue)
- Fehlermeldung bei falschen Credentials
- Nach Login → Redirect zu `/dashboard`

### 2. Dashboard (`/dashboard`)

**Navbar:**
- CrossMatic Logo links
- Begrüssung: "Willkommen, [Kundenname]"
- Logout-Button rechts

**Lead-Übersicht:**
- Überschrift: "Ihre Leads dieser Woche"
- Datum der letzten Lieferung
- Grid aus Lead-Cards (2 Spalten Desktop, 1 Spalte Mobile)

**Lead-Card:**
```
┌─────────────────────────────────────┐
│  [Firmenname]                       │
│  [Branche/Kategorie]                │
│─────────────────────────────────────│
│  👤 Kontaktperson                   │
│  ✉️  email@beispiel.com             │
│  📞 +41 79 000 00 00                │
│  🔗 LinkedIn / Social Media         │
│─────────────────────────────────────│
│  Warum ein guter Fit:               │
│  [fit_description Text]             │
└─────────────────────────────────────┘
```

- Dunkler Card-Hintergrund (`--card`)
- Subtile Border (`--border`)
- Hover-Effekt: leichter Border-Glow in Primary Blue
- Social Media Link öffnet in neuem Tab

---

## Supabase Integration

### Bestehende Tabellen (bereits erstellt)

**`profiles`**
- `id` → verknüpft mit `auth.users`
- `agency_name` → Kundenname für Begrüssung
- `contact_email`

**`leads`**
- `id`, `client_id` (FK → profiles)
- `company_name`, `contact_name`, `contact_email`
- `phone`, `social_media_url`, `fit_description`
- `website`, `status`, `week_added`, `created_at`

### RLS (bereits aktiv)
- Kunden sehen nur ihre eigenen Leads via `client_id = auth.uid()`

### Auth Flow
```
1. Kunde gibt Email + Passwort ein
2. Supabase Auth validiert → gibt Session zurück
3. Next.js Middleware prüft Session bei jedem Request
4. Kein Session → Redirect zu /login
5. Mit Session → Dashboard mit gefilterten Leads
```

---

## Middleware (Auth-Schutz)

`middleware.ts` schützt alle Routen unter `/dashboard/*`:
- Liest Supabase Session aus Cookie
- Kein Login → automatischer Redirect zu `/login`
- Nach Login → Redirect zurück zur ursprünglichen Seite

---

## Implementationsschritte (Reihenfolge)

1. **Projekt aufsetzen** — `create-next-app`, Tailwind, shadcn/ui installieren
2. **Design-System** — `globals.css` mit exakten CSS-Variablen aus nebula-flow
3. **Supabase Client** — Browser + Server Client einrichten, `.env.local`
4. **Middleware** — Auth-Schutz für `/dashboard`
5. **Login-Seite** — UI + Supabase Auth Logic
6. **Dashboard Layout** — Navbar + Auth Guard
7. **Lead-Karten** — `LeadCard.tsx` + `LeadGrid.tsx` Komponenten
8. **Dashboard-Seite** — Leads von Supabase laden + anzeigen
9. **Testdaten** — Einen Test-Kunden + Leads in Supabase einfügen
10. **Deployment** — Vercel, Subdomain `dashboard.getcrossmatic.com`

---

## Entscheidungen (geklärt)

- [x] **Lead-Status ändern:** Nicht nötig fürs erste — nur Lesezugriff
- [x] **Mobile:** Ja, vollständig mobile-optimiert (responsive Grid, Touch-freundliche Cards)
- [x] **Logo:** Kein SVG vorhanden — `crossmatic-logo.png` aus nebula-flow Repo verwenden
- [x] **Deployment:** Kein Vercel — nur lokale Entwicklung vorerst
