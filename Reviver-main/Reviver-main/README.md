# REVIVER — Emergency Response Platform

REVIVER is a front-end web app that gives people one-tap access to emergency help: calling the police, requesting an ambulance, and registering patients with triage-based priority. It is built with plain HTML, CSS and JavaScript, and uses [Supabase](https://supabase.com) as the backend for accounts and patient records.

> **Status:** prototype / demo. See [Known Limitations](#known-limitations) before using it for anything real.

---

## Features

### Home (`index.html`)
- Landing page with cards linking to the three emergency services
- **Login** and **Create Account** modals, backed by Supabase
- Animated grid background, responsive layout and a back-to-top button

### Call Police (`callpolice.html`)
- One-tap `tel:` call button
- Police helpline (100) shown on the page
- Incident report form (name, location, description) with validation
- Browser geolocation to detect the user's coordinates

### Call Ambulance (`ambulance.html`)
- National ambulance helpline (108) and a one-tap request button
- Patient details form (name, age, condition type)
- Pickup location form (address, auto-detected GPS coordinates, nearest hospital, contact number)
- Dispatch confirmation with an ETA display

### Patient Registration (`registration.html`)
- START-style triage priority selector:
  | Level | Category | Meaning |
  |-------|----------|---------|
  | P1 | Immediate | Life-threatening, needs immediate intervention |
  | P2 | Urgent | Serious but stable, can wait up to 1 hour |
  | P3 | Delayed | Minor injury or illness, can be deferred |
  | P4 | Expectant | Very low survival chance, palliative care |
- Patient form: name, age, gender, contact, blood group, allergies/conditions
- Medical form: complaint category, vitals (BP, heart rate, SpO₂, temperature), department/ward, notes
- Result panel showing a token number, priority, guidance message and queue position
- Saves records to the Supabase `patients` table

---

## Project Structure

```
.
├── index.html          # Landing page, login & account creation
├── callpolice.html     # Police call + incident report
├── ambulance.html      # Ambulance request + patient/pickup details
├── registration.html   # Triage patient registration
└── README.md
```

There is no build step and no dependencies to install. Every page is a self-contained HTML file.

---

## Tech Stack

- **HTML5 / CSS3 / vanilla JavaScript** (ES modules)
- **Supabase** (`@supabase/supabase-js`, loaded from `esm.sh`) for the database
- **Google Fonts:** Bebas Neue, DM Sans, Space Mono
- **Browser Geolocation API** for location detection

---

## Getting Started

### Run locally

1. Clone or download the project.
2. Serve the folder with any static server (needed for ES module imports):
   ```bash
   # Python
   python3 -m http.server 8000

   # or Node
   npx serve .
   ```
3. Open `http://localhost:8000` in your browser.

Geolocation requires `localhost` or HTTPS, so opening the files directly with `file://` may not detect your location.

### Set up Supabase

The app expects two tables. Create them in the Supabase SQL editor:

```sql
create table users (
  id          bigint generated always as identity primary key,
  name        text,
  phone       text,
  email       text unique,
  password    text,
  created_at  timestamptz default now()
);

create table patients (
  id           bigint generated always as identity primary key,
  name         text,
  age          int,
  priority     int,
  gender       text,
  phone        text,
  blood_group  text,
  complaint    text,
  department   text,
  notes        text,
  created_at   timestamptz default now()
);
```

Then replace the project URL and anon key near the top of the `<script type="module">` block in **`index.html`** and **`registration.html`** with your own:

```js
const supabaseUrl = 'https://YOUR-PROJECT.supabase.co';
const supabaseKey = 'YOUR-ANON-KEY';
```

### Configure phone numbers

Both call buttons currently use a placeholder number. Update the `tel:` links:

- `callpolice.html` → `<a href="tel:+919999999999" class="call-btn">`
- `ambulance.html` → `<a href="tel:+919999999999" class="call-btn">`

Use your local emergency number (for example `tel:100` for police and `tel:108` for ambulance in India).

---

## Known Limitations

This is a prototype, and several parts are simulated or unsafe for production:

**Security**
- **Passwords are stored and compared in plain text** in the `users` table. Replace this with Supabase Auth (`supabase.auth.signUp` / `signInWithPassword`) so passwords are hashed.
- The Supabase URL and anon key are hard-coded in client-side code. The anon key is designed to be public, but only if **Row Level Security (RLS) is enabled** on every table with strict policies. Without RLS, anyone can read or write your data.
- Patient records contain sensitive health information. Add proper access control and review applicable privacy regulations before storing real data.

**Simulated behaviour**
- The police report form and ambulance dispatch do **not** contact any real service. They only show a success message.
- The ambulance ETA is a random number between 4 and 11 minutes.
- The queue position in patient registration is randomly generated (except P1, which is always #1).
- The "AI-assisted priority scoring" is not implemented. Priority is selected manually by the user, and defaults to P3 if none is chosen.
- The "< 60s average dispatch time" stat on the home page is static marketing text.
- Geolocation on the police page only updates an input placeholder. It is not sent anywhere.

**Other**
- If the database call fails in `registration.html`, the result panel is still shown with a random token, which could mislead users into thinking the record was saved.
- The login modal does not keep a session. Nothing changes after signing in.

---

## Suggested Next Steps

- Move authentication to Supabase Auth and enable RLS
- Connect police and ambulance requests to a real dispatch backend or SMS/call API
- Store submitted coordinates with each report and request
- Implement real triage scoring from the vitals entered
- Add a staff dashboard to view and manage the patient queue
- Show a clear error when a patient record fails to save

---

## Disclaimer

REVIVER is a demonstration project. It is **not** a substitute for official emergency services. In a real emergency, call your local emergency number directly.

---

## License

Add a license of your choice (for example MIT) before distributing.
