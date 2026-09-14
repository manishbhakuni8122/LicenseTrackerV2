# 🗂️ License Tracker — Zippee BizOps

A single-file, no-build web app for tracking statutory licenses (Trade License, FSSAI, Shop & Establishment, CLRA, Fire NOC, PTEC, and any custom license types you add) across all dark stores — expiries, renewal history, compliance-team billing, and step-by-step task tracking, all in one place.

No backend framework, no build step, no `npm install` — it's one `.html` file that runs entirely in the browser and syncs to a shared Supabase table.

---

## ✨ Features

- **📊 Dashboard** — per-store or all-stores compliance % completion, drill into any license's checklist
- **📅 Expiry Tracker** — auto-calculated expiry (Issue Date + Validity), color-coded urgency, one-click renewal (old data archived automatically)
- **📜 History** — two tabs:
  - **Renewals** — full renewal cycle history per store/license
  - **Payments** — every logged bill payment across every store, with running totals
- **💰 Bills & Payments (Ledger)** — one place to manage compliance-team billing:
  - KPI cards (Total Bills / Paid / Pending / Invoice Value)
  - Filters by store, team, payment status, search
  - Per-bill popup: vendor, applied date, amount, invoice number, **Google Drive invoice link**, and a running **payment history log** (not just a single overwritable number — every payment is logged with date + note)
  - Editable rate cards per compliance team per license type
  - "+ New Bill" flow that can attach any license (existing or brand-new) to any store on the spot
  - Styled Excel export (bordered, colored headers, auto-sized columns)
- **✅ Task Trackers** — multiple parallel tracker sheets (e.g. per brand/batch), each store row has Assigned On / Assigned By / All Docs Received fields, and any license can have a **step-by-step checklist** (built-in templates for CLRA, FSSAI, Fire NOC, Trade License, or fully custom/editable steps) so the whole team can see exactly which stage a license is at
- **📖 Fee Reference** — rough statutory government fee ranges by state (for reference, not billing)
- **🏪 Store Master** — add/edit stores and toggle which licenses apply to each
- **📁 Docs** — password-gated link to the shared Google Drive folder for all license documents
- **📄 AI-assisted date extraction** — upload a scanned license/certificate and Claude reads the issue date, expiry, application number, etc. automatically

All document/bill storage uses **Google Drive links** — there is no file upload anywhere in the app; you upload PDFs to Drive yourself and paste the link.

---

## 🧱 Tech stack

| Layer | What's used |
|---|---|
| UI | React 18 + ReactDOM (via CDN, no bundler) |
| JSX compilation | Babel Standalone — compiles JSX **in the browser** on page load |
| Excel export | [xlsx-js-style](https://www.npmjs.com/package/xlsx-js-style) (styled: borders, colored headers, merged cells) |
| Data storage | **Supabase** (Postgres) — one shared table, `app_storage` |
| Hosting | Any static host (Cloudflare Pages, Netlify, GitHub Pages, S3...) — no build step required |

There is no server-side code. The browser talks directly to Supabase over HTTPS using the public `anon` key.

---

## 🚀 Setup

### 1. Create a Supabase project
Go to [supabase.com](https://supabase.com) → New Project (free tier is plenty).

### 2. Run the setup SQL
In your project's **SQL Editor**, run the script in [`supabase_setup.sql`](./supabase_setup.sql):

```sql
create table app_storage (
  key text primary key,
  value text,
  updated_at timestamptz default now()
);

alter table app_storage enable row level security;

create policy "anon read"   on app_storage for select using (true);
create policy "anon insert" on app_storage for insert with check (true);
create policy "anon update" on app_storage for update using (true);
```

This creates **one table** that stores the entire app's data as a single JSON blob under the key `"zippee-lt-data-v1"`.

### 3. Add your credentials
Open `License_Tracker_v2.html`, find these two lines near the top of the `<script type="text/babel">` block, and fill in your project's values (**Project Settings → API** in Supabase):

```js
const SUPABASE_URL="https://YOUR-PROJECT-REF.supabase.co";
const SUPABASE_ANON_KEY="YOUR-ANON-PUBLIC-KEY";
```

If these are left as placeholders, the app still runs (and shows a warning banner on the login screen) but nothing will be saved.

> The `anon` key is **meant** to be public — it ships inside the HTML that reaches every browser regardless, so there's nothing to hide. **Never** put a Supabase `service_role` key here.

### 4. Rename & deploy
Rename the file to `index.html` and deploy it to any static host:

**Cloudflare Pages (recommended, free):**
1. [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages** → **Create** → **Pages** → **Upload assets**
2. Upload `index.html`
3. Deploy — you'll get a `*.pages.dev` URL

**Or GitHub Pages:** push this repo, enable Pages on the `main` branch, done.

No build command, no install step — it's static.

---

## 🔑 In-app login

The app has its own username/password screen (`Bizops@2026` / `Zippee@1234` by default, plus an `admin`/`admin123` account — see the `US` constant near the top of the script to change these). This is a **UI gate only**, not real authentication — anyone with direct access to the Supabase anon key and URL could bypass it. Fine for an internal team tool behind an unlisted URL; if you need real access control, Supabase Auth would need to be layered in.

---

## 📁 Repo contents

| File | Purpose |
|---|---|
| `index.html` (or `License_Tracker_v2.html`) | The entire app — open it in a browser, or deploy it as-is |
| `supabase_setup.sql` | One-time SQL to run in your Supabase project |
| `README.md` | This file |

---

## 🛠️ Customizing

Everything lives in one script tag, edited directly (no rebuild needed — just save and refresh):

- **Default license types** — `DEF_L` constant
- **Store list / seed data** — `ST` constant
- **Statutory fee reference** — `FI` constant
- **Login credentials** — `US` constant
- **Compliance teams / rate cards** — `DEF_VENDORS`, `DEF_RATES`
- **Step-checklist templates** (CLRA, FSSAI, Fire NOC, Trade License) — `STEP_TEMPLATES` constant
- **Shared Google Drive folder link** — `GD` constant

---

## ⚠️ Known limitations

- No real authentication — see above
- No offline support — requires a live connection to Supabase
- Single shared dataset — there's no per-team or per-brand data isolation; everyone who has the URL sees the same data
- Excel exports run entirely client-side; very large exports may be slow on low-end devices
