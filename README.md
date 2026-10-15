# EcoSync

EcoSync is a MongoDB-backed recycling account and reward website. It includes account registration/login, a permanent encrypted QR identity, user-scoped dashboard/wallet/deposit views, an actual configured reward catalogue, idempotent redemption requests, authenticated machine event ingestion, and an honest unavailable state for hardware and AI integrations.

## Requirements

- Node.js 20 or newer
- MongoDB 6+ (local or hosted). A replica set is required for the multi-record transaction flows used for deposits and redemptions.

## Setup

1. Copy `.env.example` to `.env` and set `MONGODB_URI` to your MongoDB deployment.
2. Generate independent random secrets for `SESSION_SECRET`, `QR_ENCRYPTION_KEY`, and (when using controller requests) `MACHINE_API_KEY`. Keep the QR encryption key backed up: changing it prevents decoding existing permanent QR identities.
3. Install and start:

   ```sh
   npm install
   npm start
   ```

4. Open `http://localhost:3000` and create an account. New accounts start with zero balance and no activity.
5. To create the first operator, run `npm run admin:create` and enter an email and a password of at least 14 characters. There are no default credentials.

The application refuses to start without `MONGODB_URI`. Sessions are stored in MongoDB when configured. In production, set `NODE_ENV=production`, serve behind HTTPS, and configure a trusted reverse proxy and network firewall.

## Configure real catalogue records

There is no seed data. An operator may add actual rewards through a trusted administrative script or MongoDB tooling. A reward document has fields such as:

```json
{"name":"Configured product","description":"Real catalogue description","pointsPrice":100,"category":"Accessories","active":true,"stock":4}
```

Do not use unverified products or inventory. The current website only displays active database records. A future admin catalogue UI should be protected with the existing server role check.

## Machine controller integration

`POST /api/machine/events` accepts an event from the waste controller. Configure `MACHINE_API_KEY` and a comma-separated `AUTHORIZED_MACHINE_IDS` allowlist first. Send `X-Machine-Key: <MACHINE_API_KEY>` and JSON `{ "eventId":"controller-generated-unique-id", "qrToken":"opaque citizen QR token", "machineId":"configured-machine-id", "category":"plastic", "weightGrams":120 }`. The server verifies the QR hash, validates the configured machine ID, category, and weight, calculates points (one point for each complete 50 grams), records the unique event, and creates its wallet ledger entry in a MongoDB transaction. Duplicate event IDs are idempotent. This endpoint requires a replica-set transaction-capable MongoDB deployment. Restrict it to the machine network and rotate controller credentials if exposed.

The machine must scan `ecosync://citizen/<opaque-token>` from the permanent QR and securely relay the token. QR is for machine identity only and is never a website login credential. `POST /api/machine/dispense` accepts authenticated `{ "eventId":"controller-generated-unique-id", "redemptionId":"...", "machineId":"configured-product-unit-id", "outcome":"dispensed" }` or `outcome:"failed"`. A confirmed failure atomically restores points and finite stock and records a compensating ledger transaction; an actual dispense is reported only after the controller confirms it. Both unit IDs must be in the `AUTHORIZED_MACHINE_IDS` allowlist. The physical product-unit connection remains unavailable until configured and tested.

## AI classifier integration

Website photo scans validate and preview JPEG, PNG, and WebP files up to 5 MB. They never award points. The classifier adapter is currently unavailable and produces no category, confidence, or recycling advice. `AI_CLASSIFIER_URL` and `AI_CLASSIFIER_API_KEY` are placeholders for a future authenticated adapter; configuring a URL alone does not enable or fake a model result.

## API overview

- `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/me`
- `GET /api/identity`, `GET /api/dashboard`, `GET /api/wallet`, `GET /api/deposits`
- `GET /api/rewards`, `POST /api/redeem` (authenticated; requires `Idempotency-Key`)
- `POST /api/scan` (authenticated; unavailable until a real adapter is implemented)
- `POST /api/machine/events`, `POST /api/machine/dispense` (controller key authenticated, machine allowlisted, and idempotent)
- `GET /api/admin/overview` (admin role only)

Citizen data is always scoped to the active session. Password hashes and QR token hashes/ciphertext are never returned to the browser. Registration/login are rate limited. The frontend is plain HTML/CSS/JavaScript served by Express; no separate frontend server is required.

## Implemented and integration-dependent

Implemented: responsive EcoSync-inspired public page and citizen dashboard, signup/login/logout, persistent permanent QR, MongoDB-backed wallet ledger and history, real catalogue query, server-validated/idempotent reward redemption, machine-event API, role-protected operations overview, image selection/camera capture with unavailable-model messaging, empty states, and setup documentation.

Needs external setup or further integration: MongoDB service and production secrets, real machine registrations/controller credentials and physical actuator verification, real AI classifier adapter, admin management UI, RFID association, map/location data, and a persisted digital twin. A secured dispense confirmation endpoint implements the pending/failed reversal policy; it has not been exercised against hardware. The UI reports hardware and AI as unavailable until connected. There are no demo records, hard-coded statistics, fake users, or fake AI outputs.
