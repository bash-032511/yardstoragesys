# ECG Container Yard: Yard Storage System

> **"Track. Manage. Control."**

A web-based Container Yard Management System built for **ECG Brokerage Inc.** It tracks containers coming into and going out of the yard, shows where each one is stored, monitors yard capacity in TEUs, and alerts staff when containers stay too long.

The whole app is a **single `index.html` file** with a **Firebase Realtime Database** backend, so changes sync live across all devices.

**Version:** v8.0
**System Developer:** Jayson Alejo

---

## Table of Contents

- [Features](#features)
- [Modules](#modules)
- [User Roles and Permissions](#user-roles-and-permissions)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Firebase Setup](#firebase-setup)
- [AI Yard Assistant (Gemini)](#ai-yard-assistant-gemini)
- [Data Structure](#data-structure)
- [Security Notes](#security-notes)
- [Project Structure](#project-structure)
- [Credits](#credits)

---

## Features

- **Real-time sync:** Data updates instantly on every connected device, with a live connection badge ("Connected") in the header.
- **Gate In / Gate Out:** Record container arrival and release with driver, plate number, yard slot, and remarks.
- **Yard Locator:** Find the exact slot of any container, and update a container's slot when it moves.
- **Public Yard Locator:** Anyone can search for a container from the login page without signing in (read-only, via anonymous Firebase session).
- **TEU Capacity Tracking:** Live utilization against a **1,000 TEU** capacity (1 x 20ft = 1 TEU, 1 x 40ft = 2 TEUs).
- **Aging / Dwell Time Alerts:** Containers stored longer than the configured threshold (default **14 days**) are highlighted in red and trigger notifications.
- **Notifications:** Bell icon shows gate-in and gate-out activity from the last 24 hours plus aging alerts.
- **Reports and Export:** Filter by status and date range, then export to **PDF** (jsPDF + AutoTable) or **CSV** (PapaParse).
- **Duplicate Container Detection:** Warns when a container number already exists.
- **Auto Uppercase Input:** Text fields on the Gate In form are forced to ALL CAPS for consistent data.
- **User Management:** Admin creates users, approves or rejects self-registrations, and sets per-user tab permissions.
- **Self-Registration:** New users can request an account, which stays pending until an admin approves it.
- **Account Settings:** Users can update their profile photo and password.
- **AI Yard Assistant:** Chatbot (Google Gemini) that answers questions about live inventory and TEU utilization, with Tagalog support.
- **Auto Logout:** Session expires after **50 minutes** of inactivity.
- **Session Restore and Hash Routing:** Deep links such as `#/inventory` and `#/register`, with browser back/forward support.
- **Dark Mode** toggle.
- **Responsive Design:** Sidebar on desktop, drawer and bottom navigation on mobile.
- **Company Branding:** Editable company name, address, and logo.
- **Mini Games:** Built-in break-time games (stacker, brick breaker, and a block puzzle).

---

## Modules

| Tab | Description |
| --- | --- |
| **Dashboard** | Yard operational overview: stats, TEU utilization bar, and summaries. |
| **Yard Locator** | Search containers and view or update their yard slot. |
| **Gate In** | New gate-in entry form and the list of gated-in containers (edit or delete). |
| **Gate Out** | Release stored containers, with the option to revert a release back to stored. |
| **Container Inventory** | Full inventory with age tracker, sorting, search, and pagination. |
| **Reports** | Detailed movement and aging reports with filters and PDF/CSV export. |
| **Settings** | Master lists (Shipping Lines, ISO Size/Type, Yard Slots), aging threshold, and company profile. *(Admin only)* |
| **Admin Panel** | User management, pending approvals, and AI key configuration. *(Admin only)* |

### Gate In fields

| Field | Required |
| --- | --- |
| Container Number (e.g. `MSCU1234567`) | Yes |
| ISO Size / Type | Yes |
| Shipping Line | Yes |
| Consignee | No |
| Date and Time | Yes |
| Gate In Driver Name | Yes |
| Plate Number | Yes |
| Yard Slot | Yes |
| Remarks | No |

Container status is either **STORED** or **RELEASED**.

---

## User Roles and Permissions

| Role | Access |
| --- | --- |
| **Yard Admin** | Full control of every tab, including Settings and Admin Panel. |
| **Yard User** | Custom permissions per tab (Dashboard, Yard Locator, Gate In, Gate Out, Inventory, Reports). Cannot open Settings or Admin Panel. |
| **Guest (Public)** | Read-only Yard Locator search from the login page. |

Usernames are mapped to an email behind the scenes for Firebase Authentication.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| UI | HTML, Tailwind CSS (CDN), Font Awesome 6.4 |
| Logic | Vanilla JavaScript (no build step) |
| Database | Firebase Realtime Database |
| Authentication | Firebase Authentication (email/password + anonymous for guests) |
| PDF Export | jsPDF 2.5.1 + jsPDF-AutoTable 3.5.31 |
| CSV Export | PapaParse 5.4.1 |
| AI Assistant | Google Gemini API |

---

## Getting Started

No installation or build is needed.

1. Clone or download this repository.
2. Open `index.html` in a modern browser, **or** host it on any static host (Firebase Hosting, GitHub Pages, Netlify, etc.).
3. Make sure you have an internet connection, since libraries and Firebase load from CDNs.
4. Sign in with your credentials, or use **Locate Container** on the login page for public search.

```bash
# Optional: run locally with a simple static server
npx serve .
# or
python3 -m http.server 8000
```

---

## Firebase Setup

To use your own Firebase project, edit the `firebaseConfig` object near the top of `index.html`:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "your-project.firebaseapp.com",
  databaseURL: "https://your-project-default-rtdb.<region>.firebasedatabase.app",
  projectId: "your-project",
  storageBucket: "your-project.firebasestorage.app",
  messagingSenderId: "...",
  appId: "..."
};
```

Then in the Firebase Console:

1. Create a **Realtime Database**.
2. Enable **Authentication**, then turn on **Email/Password** and **Anonymous** sign-in.
3. Set your **Realtime Database Security Rules** (see [Security Notes](#security-notes)).

---

## AI Yard Assistant (Gemini)

The chat assistant reads the live inventory and TEU utilization and answers questions about it.

1. Get a Gemini API key from [Google AI Studio](https://aistudio.google.com/).
2. Log in as **Yard Admin**, go to **Admin Panel**, and save the key under the AI settings section.
3. The key is stored in Firebase at `aiConfig/geminiApiKey` and shared with users.

You can also set `DEFAULT_GEMINI_API_KEY` in the script, but avoid committing real keys to a public repository.

Example prompt: *"Bigyan mo ako ng summary ng kasalukuyang inventory at TEU utilization."*

---

## Data Structure

Realtime Database paths used by the app:

| Path | Purpose |
| --- | --- |
| `containers/` | All container records (gate in, gate out, slot, status) |
| `settings/` | Master lists and aging threshold (`agingAlertDays`) |
| `companyProfile/` | Company name, address, and logo |
| `users/<uid>` | User profile, role, and permissions |
| `pendingUsers/` | Registration requests awaiting admin approval |
| `aiConfig/geminiApiKey` | Gemini API key for the AI assistant |

If Firebase is unavailable, the app falls back to `localStorage` for settings.

---

## Security Notes

- Firebase web config values are not secret, but **your database rules are what protect your data**. Restrict reads and writes by authenticated user, and limit `users`, `pendingUsers`, and `aiConfig` to admins.
- Guests use anonymous sign-in, so rules should allow **read-only** access to only the data needed for the public locator.
- The Gemini key is readable by signed-in users. Use a key with restricted quota and referrer limits, and never commit it to a public repo.
- Client-side permission checks only hide tabs. Enforce permissions in your Firebase rules too.

---

## Project Structure

```
.
├── index.html      # Entire app: UI, styles, and JavaScript
└── README.md
```

---

## Credits

- **Company:** ECG Brokerage Inc.
- **System Developer:** Jayson Alejo
- **Tagline:** "Track. Manage. Control."
