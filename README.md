# WavuMall – Customer App

A Flutter customer application for the **WavuMall** marketplace. Built **offline‑first** so customers can browse vendors, manage a cart, and place orders without a network connection, then sync with the backend on demand (or automatically).

---

## Features

- **Offline‑first** – vendors, products, cart, and orders are all cached in a local SQLite database.
- **Local password unlock** – the app unlocks with a locally stored password hash (bcrypt). No network call is required to open the app.
- **Recovery PIN** – a 4‑digit PIN is available **only for local password recovery**. It is never sent to the server.
- **Configurable base URL** – set the backend endpoint from the Settings screen.
- **Manual + automatic sync**
  - Manual sync button in the app bar.
  - Optional auto‑sync with a configurable interval (2–60 minutes).
- **RTC conflict resolution** – 4‑timestamp relative‑time comparison (`T1`, `L_sync`, `L_edit`, `S_updated`) applied consistently across profile, orders, and any future synced entities.
- **Light / dark mode** – theme toggle in the drawer, persisted across launches.
- **Vendor browsing** – search, filter, and view vendor details including logo, description, location, and featured products.
- **Cart with per‑vendor grouping** – one order per vendor; the cart keeps items grouped so “Place Order” is scoped correctly.
- **Order history** – shows payment status, delivery state, and sync state (Synced / Pending Sync).

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| UI | Flutter (Material 3) |
| State | Riverpod |
| Local DB | `sqflite` |
| HTTP | `dio` |
| Key/Value | `shared_preferences` |
| IDs | `uuid` |
| Hashing | `bcrypt` |

---

## Project Structure

```
lib/
├── main.dart
├── models/
│   ├── customer_profile.dart
│   ├── orders.dart
│   ├── products.dart
│   └── vendor.dart
├── database/
│   ├── database_helper.dart
│   ├── order_repository.dart
│   ├── product_repository.dart
│   ├── vendor_repository.dart
│   ├── customer_profile_repository.dart
│   └── sync_metadata_repository.dart
├── services/
│   ├── api_client.dart
│   ├── sync_service.dart
│   └── auto_sync_service.dart
├── providers/
│   └── providers.dart
└── screens/
    ├── login_screen.dart
    ├── home_screen.dart
    ├── vendor_list_screen.dart
    ├── vendor_detail_screen.dart
    ├── cart_screen.dart
    ├── order_history_screen.dart
    └── settings_screen.dart
```

---

## Getting Started

### Prerequisites

- Flutter **3.16+**
- Dart **3.2+**
- A running WavuMall backend (see the backend repo) reachable from the device/emulator

### Install

```bash
flutter pub get
```

### Run

```bash
flutter run
```

### Configure the base URL

1. Open the app.
2. Drawer → **Settings**.
3. Enter your server URL (e.g. `http://192.168.1.100:5001` – no trailing slash).
4. Tap **Test** to verify connectivity, then **Save**.

> Android emulators should use `http://10.0.2.2:<port>` to reach a server on the host machine.

---

## Sync Model

### The four timestamps (RTC)

Conflict resolution uses relative‑time comparison rather than absolute wall clock, so device/server clock drift doesn’t cause silent data loss:

| Symbol | Meaning | Stored where |
|--------|---------|--------------|
| `T1` | Server time at last successful sync | `sync_metadata.lastSyncServerTime` |
| `L_sync` | Device time at last successful sync | `sync_metadata.lastSyncLocalTime` |
| `L_edit` | Device time when the record was last edited | `*.local_updated_at` |
| `S_updated` | Server‑stamped time of the record | `*.server_updated_at` |

The winner is decided by comparing `L_edit − L_sync` against `S_updated − T1`, with a **5‑second clock‑drift tolerance** before falling back to a tie.

### Manual vs Auto sync

- **Manual** – press the sync icon in the app bar. Always available.
- **Auto** – enable “Auto‑Sync (Online Mode)” in Settings. Auto‑sync runs in the background while the app is open, at the interval you choose. Switching to offline disables the timer; the manual button still works.

---

## Offline‑First Flow

1. **First launch**
   - No profile exists → the app opens into **Settings** so you can create a profile.
2. **Returning user**
   - Login screen asks for the local password.
   - Password is verified against `customer_profile.password_hash` (bcrypt).
   - No server call on unlock.
3. **Forgot password**
   - Use the recovery flow with the 4‑digit PIN (local only).
4. **Sync**
   - Profile is synced on **Save Profile** (Settings).
   - Orders are uploaded/downloaded via the sync button or auto‑sync.

---

## Backend Endpoints Used

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/customer/login` | Optional web login (returns JWT) |
| `POST` | `/customer/register` | Optional registration |
| `GET`  | `/api/vendors` | Vendor list |
| `GET`  | `/api/vendors/:uuid` | Single vendor |
| `GET`  | `/api/vendors/:uuid/featured` | Featured products for a vendor |
| `GET`  | `/api/images/product/:uuid` | Product image |
| `POST` | `/customer/orders/upload` | Upload pending orders |
| `GET`  | `/customer/orders/download` | Download new orders |
| `POST` | `/customer/profile/sync` | Sync customer profile (RTC) |
| `GET`  | `/customer/profile` | Fetch customer profile |

---

## Permissions

The Android manifest enables cleartext traffic so the app can talk to a local development server over plain HTTP. For production, use HTTPS and remove `android:usesCleartextTraffic="true"`.

---

## Known Limitations / Roadmap

- Product image loading relies on the backend image endpoint; offline image caching is not yet implemented.
- Customer profile sync relies on server‑side RTC support (mirrors the vendor profile controller).
- Multi‑vendor orders are intentionally split into one order per vendor.
- No push notifications yet.

---

## Contributing

1. Fork the repo.
2. Create a feature branch (`git checkout -b feature/your-feature`).
3. Commit changes (`git commit -m 'Add …'`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Open a Pull Request.

---

## License

Proprietary — © WavuMall. All rights reserved.

---

## Related Repos

- **WavuMall Backend** – Node.js / TypeScript / PostgreSQL API
- **WavuMall Vendor App** – Flutter app for vendors and employees

---

If you'd like, I can also add:

- A **screenshots section** (once you have captures).
- A **CHANGELOG.md** with the current version and feature list.
- A **CONTRIBUTING.md** with coding standards and PR checklist.
- A short **architecture.md** explaining the sync engine and RTC in more depth.

Just say the word.
