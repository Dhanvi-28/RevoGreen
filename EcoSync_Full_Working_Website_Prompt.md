# EcoSync — Full-Stack Website Implementation Prompt

## Role
Act as a senior full-stack engineer, MongoDB data architect, application security engineer, and product UI/UX designer. Build a complete working EcoSync website integrated with a two-unit smart recycling vending-machine concept. Do not return only a plan, mockup, or static frontend. Inspect the repository, implement the application, run it, test its main flows, and document setup. Preserve useful existing work.

## Product Context
EcoSync consists of two attached vending units:
1. A **waste-deposit unit** with a screen, QR scanner/RFID reader, shaking plate, camera with flash, AI image classifier, pressure/weight sensor, controller, sorting mechanism, and waste compartments.
2. A **product vending unit** containing configured products such as instant noodles, packaged snacks, and other catalogue items.

A citizen identifies themselves at the waste-deposit unit using their permanent QR or associated RFID card. The machine classifies the waste, measures stable weight, and displays the category, weight, and points. The citizen chooses either **Redeem now** (activate the attached product vending unit and use eligible points for a product) or **Save to account** (keep points in the online wallet for later redemption). The website shows account, QR, deposits, wallet, rewards, and redemptions.

The physical machine may not be connected during initial development. Build honest integration boundaries; do not claim live hardware behavior unless actually connected.

## Non-Negotiable Requirements
1. Use MongoDB as persistent storage; database access must be server-side.
2. Do not use dummy business data: no fake users, deposits, balances, redemptions, AI results, machines, inventory, sensor readings, or live statistics. New accounts start with zero points and empty histories. Use honest empty states.
3. Do not fake AI results. If the model/API is not configured, show a clear unavailable/configuration state.
4. Do not claim live machine, sensor, inventory, or digital-twin integration unless it is actually connected. Label optional simulation mode clearly and separate it from real records.
5. Validate balances, product prices, QR identity, deposits, inventory, and redemptions on the server. Never trust values submitted by the browser.
6. Each user gets a unique, permanent QR identity at registration. It remains bound to that account and can never be assigned to another account.
7. **QR purpose:** identify the citizen at the physical waste-deposit machine. It is not a website login credential. Website login uses normal authentication.
8. Prevent duplicate deposits, wallet credits, redemptions, and dispense confirmations using unique event IDs/idempotency and safe atomic operations.
9. Implement authentication, authorization, validation, error handling, loading states, and responsive UI.
10. Every visible control must work or clearly state why it is unavailable.
11. Store secrets in environment variables and provide `.env.example`.

## Recommended Stack
- Frontend: React + Vite + TypeScript.
- Styling: Tailwind CSS or existing consistent design system.
- Routing: React Router.
- Backend: Node.js + TypeScript + Express unless the existing repository already has a coherent backend.
- Database: MongoDB using Mongoose or the official driver.
- Authentication: secure server-side session or carefully implemented access/session tokens.
- Password hashing: Argon2id or bcrypt.
- QR generation: maintained QR-code library.
- Validation: Zod, Joi, or equivalent.
- Optional charts: Recharts.

Use maintained compatible versions and document setup commands.

## Required Website Areas

### 1. Landing Page
Create a polished public page explaining the two attached units, AI waste classification, weight-based rewards, immediate product redemption, and saving points for later. Include Sign Up and Log In. Do not show fabricated statistics or pretend to have live partner locations.

### 2. Registration, Login, Logout
Implement real account creation and authentication. Registration collects name, email, and password (or another sensible minimum), validates fields on client/server, normalizes and uniquely indexes email, hashes passwords, and reliably creates the user, wallet, and permanent QR identity. Login verifies credentials. Protect pages and API endpoints. Users must not access another user's account by changing URL or request parameters.

### 3. Permanent QR Identity — Critical
On registration:
- Generate a cryptographically secure, unpredictable unique QR identifier/token on the server.
- Enforce uniqueness with a MongoDB unique index and retry on rare collisions.
- Store a token hash where practical; do not expose internal secrets.
- Encode only an opaque identifier/secure token—not name, email, password, raw database ID, or personal data.
- Bind the QR permanently to exactly one user.
- Show the same QR after refresh, logout/login, and normal profile edits; do not regenerate it during normal usage.
- Never let users edit, claim, transfer, or assign a QR to another account.
- Never use sequential or guessable IDs as QR secrets.

The QR identifies a user at the **waste-deposit unit**, not for website login. The website must still use normal login authentication. A QR can be copied, so the machine/backend must validate the QR and deposit session, authenticate machine-controller requests, and add safeguards where practical to prevent copied QR codes being used for unauthorized reward claims. Do not silently change a user's permanent QR during normal use.

An RFID card can be associated with the same user account through a secure, unique card identifier. Never store or expose card secrets unnecessarily.

### 4. Citizen Dashboard
Show only the authenticated user's real data:
- Available points.
- Lifetime points earned.
- Verified deposit count.
- Total accepted weight when actual readings exist.
- Recent deposit activity.
- Recent redemption history.
- Quick links to QR identity, scanner, wallet, and rewards store.

New accounts must show zeros and empty states. Never fabricate impact numbers or charts.

### 5. AI Waste Scanner — Scan & Reimagine
Allow image upload and browser-camera capture where supported. Validate type/size, preview the image, call a real configured classification model/service, and display category/confidence only if returned by that model. Display reuse/recycling guidance only from a configured ruleset or real configured AI service. Include loading, error, unsupported-file, unavailable-model, and retry states. Website photo scans never award points. If no model is configured, show “AI classification is not configured.”

### 6. Wallet and Transaction History
Show current balance, lifetime points earned, points spent, deposits, and redemption transactions. Use immutable wallet ledger entries as the source of truth; do not rely only on a client-side balance. Keep wallet and ledger consistent with MongoDB transactions where supported or another safe atomic approach.

### 7. Rewards Store
Create a shopping-style catalogue backed by actual MongoDB reward product records. Each product can have name, description, point price, image, active/availability status, category, and optional stock. Do not invent products or redemptions. If no products exist, show an empty state and document how an authorized admin can configure the catalogue.

For redemption:
- Authenticate the user.
- Fetch current price and stock from MongoDB.
- Validate server-side balance and product availability.
- Deduct points and record redemption/ledger transaction atomically.
- Prevent duplicate requests and negative balances.
- Show confirmation only after the server confirms success.
- Do not claim product delivery unless fulfillment is integrated.

### 8. Deposit History
Show only actual backend-recorded deposits: timestamp, machine, accepted category, measured weight and validation status where available, points awarded, and transaction status. A frontend request containing category and weight is not proof of a valid deposit. Reward-bearing deposits must originate from an authenticated and authorized machine-controller integration or a clearly isolated test path. Never expose a public endpoint that lets a user award themselves points.

### 9. Two-Unit Redemption Workflow
The website must represent the real intended flow:
1. Citizen scans their permanent QR or uses their associated RFID card at the waste-deposit unit.
2. Machine classifies eligible waste and obtains a stable weight reading.
3. Backend validates the deposit and calculates points.
4. The waste-unit screen displays category, weight, and points and offers **Redeem now** or **Save to account**.
5. If **Save to account** is selected, points stay in the user's wallet for later redemption through the website or another authorized visit.
6. If **Redeem now** is selected, the backend checks available points and product availability, creates an authorized redemption session, and activates the product vending unit if connected.
7. User selects a configured product within the available points balance.
8. The backend records the points deduction and redemption. If the physical machine supports dispense confirmation, record that confirmation; otherwise clearly show that hardware dispensing is not connected.

Do not claim that a product was physically dispensed merely because a button was clicked. Keep deposit credit and product redemption as separate linked transactions. Define a safe policy for failed dispensing (for example, a pending/reversal process) so points are not lost if a machine reports failure.

### 10. Admin / Operations
Use server-enforced role-based access control. Authorized operators can view actual users, both machine units and their integration status, verified deposits, category/weight records, product catalogue/inventory, redemptions, dispensing outcomes, and operational metrics. Do not fabricate connectivity, readings, inventory, locations, or collection events. If hardware is not connected, say so. Provide a secure documented way to create the first admin; do not ship public default credentials.

### 11. Digital Twin / Machine Status
If implementing a digital-twin page, connect it to persisted machine state and actual incoming events. Show the two attached units as a connected system. If no machine or location is configured, show an informative unavailable state. Any simulation mode must be explicitly labelled and separated from real production records.

## MongoDB Data Model
Design suitable collections and indexes for:
- `users`: name, normalized email, password hash, role, timestamps.
- `qrIdentities`: user reference, permanent identifier, token hash/secure representation, created timestamp, status. Unique indexes prevent QR reuse and ensure one identity per user.
- `rfidCards`: user reference, unique card identifier/token representation, status, linked timestamp.
- `wallets` or a cached wallet summary.
- `walletTransactions`: user, type, points delta, related deposit/redemption, idempotency key, timestamp.
- `deposits`: unique event ID, user, waste-unit machine ID, category, weight, AI result reference, validation state, points, timestamp, idempotency key.
- `rewards`: name, description, points price, image, active state, optional stock.
- `redemptions`: user, reward, points spent, status, product-unit reference, idempotency key, timestamps, dispense status.
- `machines`: unique identifier, type (`waste_deposit` or `product_vending`), supported categories/products, integration status, last-seen time, location only when configured.
- `compartments` or machine state: category and capacity/fill only when actual data or explicitly labelled simulation provides it.
- Optional `aiScans`: user, image metadata, model version, category, confidence, status; avoid retaining uploaded images unnecessarily.
- Optional `machineEvents`: authenticated source, unique event ID, event type, timestamp, processing state.

Add indexes for unique normalized email, unique QR identifier, one QR per user, unique RFID card identifier, unique deposit event IDs, and idempotency keys. Never return password hashes or QR token secrets to the browser.

## API Requirements
Document APIs for:
- Registration, login, logout/session, current user.
- Current user's permanent QR identity.
- RFID association/validation where supported.
- Dashboard summary.
- AI classification.
- Wallet and wallet transaction history.
- Rewards catalogue/details and redemption.
- Current user's deposit history.
- Admin machine, deposit, user, reward, inventory, and operations endpoints.
- Authenticated machine/controller event ingestion.
- Product vending unit activation/redemption session and dispense confirmation, if hardware integration is available.

Authorize every protected endpoint. Validate request bodies, images, IDs, categories, and weights. Return useful errors without leaking secrets.

Machine event ingestion must authenticate the machine/controller, validate the unique event ID, prevent replay/duplicates, validate category and weight ranges, apply acceptance rules, calculate points on the server, and record the deposit plus wallet ledger consistently. A retry of an already processed event must not credit points again.

## Design Direction
Use the EcoSync reference as guidance: deep forest-green navigation, white/off-white surfaces, emerald accents, clean typography, consistent spacing, subtle borders, restrained shadows, and responsive layouts. The site should feel like polished sustainability technology.

Citizen navigation:
- Home
- Scan Waste
- My Eco ID
- My Wallet
- Rewards Store
- Recycling History
- Profile

Admin navigation:
- Overview
- Machines (show both attached units)
- Deposits
- Users
- Rewards and Inventory
- Redemptions/Dispensing
- Digital Twin / Machine Status
- Analytics
- Settings

Build real pages with loading, success, empty, validation, unavailable, and error states. Keep admin pages separate and protected.

## Strict No-Dummy-Data Policy
- Do not seed fake business records into development or production.
- Do not hard-code total waste, active users, machines, CO₂ saved, deposits, balances, AI outputs, product inventory, or redemption statistics.
- Calculate dashboard metrics from persisted records.
- Show empty states when collections are empty.
- Explain missing configuration when a feature is unavailable.
- Automated tests may use isolated fixtures, but never mix them into the application database or present them as live data.
- If a demo/simulation mode is provided, require explicit activation and label it clearly.

## Security and Reliability
- Hash passwords securely.
- Use secure HTTP-only same-site cookies where appropriate and CSRF protection if needed.
- Enforce HTTPS in production and narrowly configured CORS.
- Rate-limit login, registration, QR validation, image upload, and redemption.
- Validate image type/size and never execute uploads.
- Use least-privilege MongoDB credentials.
- Never expose MongoDB to the browser.
- Prevent IDOR by deriving citizen identity from authenticated sessions, not arbitrary client-supplied IDs.
- Make deposits, point credits, redemptions, and dispense events idempotent.
- Keep an auditable wallet ledger.
- Never expose passwords, QR token secrets, machine credentials, or internal stack traces.
- Handle failed dispense attempts with a transparent pending/reversal policy.

## Documentation and Configuration
Create:
- `.env.example` with placeholders for `MONGODB_URI`, authentication/session secrets, frontend URL, AI model endpoint/key, and machine integration secrets.
- `README.md` with prerequisites, installation, environment setup, database setup, run/build/test commands, first-admin setup, and troubleshooting.
- Instructions to connect the real classifier, waste-unit controller, and product-unit controller.
- An explicit list of implemented features versus features requiring external hardware/services.

Never put real credentials in `.env.example`.

## Build Order
1. Inspect the repository and state a concise plan.
2. Set up app structure and MongoDB.
3. Implement registration/login and authorization.
4. Generate the unique permanent QR during registration and test persistence.
5. Implement wallet and ledger.
6. Implement rewards catalogue and safe redemption.
7. Implement authenticated machine-event ingestion and duplicate protection.
8. Build dashboard, QR, history, wallet, and rewards store.
9. Integrate the real classifier or show an honest unavailable state.
10. Implement the two-unit redemption flow and dispense status boundaries.
11. Build admin pages from actual database records.
12. Connect machine/digital-twin state only where integration exists.
13. Run tests, fix errors, and verify the end-to-end journey.

Do not stop after writing a plan. Continue implementing unless the environment prevents it; document any blocker precisely.

## Acceptance Criteria
- [ ] App runs using documented commands.
- [ ] MongoDB persistence works.
- [ ] Registration/login/logout work.
- [ ] Registration creates one unique permanent QR per user.
- [ ] QR persists across refresh and future sessions.
- [ ] No two accounts can share a QR.
- [ ] QR identifies users at the waste-deposit machine and is not website login.
- [ ] Users cannot access another user's QR, wallet, deposits, or redemptions.
- [ ] New users have zero points and empty history.
- [ ] Scanner never fabricates AI results and photo scans do not award points.
- [ ] Only validated, authorized, unique deposit events award points.
- [ ] Duplicate events do not double-credit.
- [ ] Wallet and ledger remain consistent.
- [ ] Actual configured rewards can be redeemed securely.
- [ ] Two-unit workflow distinguishes saving points from immediate product redemption.
- [ ] No product is reported dispensed without valid dispense confirmation.
- [ ] Failed dispensing has a documented safe points policy.
- [ ] Admin endpoints enforce roles.
- [ ] Metrics come from actual database records.
- [ ] Hardware/model unavailability is clearly disclosed.
- [ ] No fake business data is seeded.
- [ ] Tests cover QR persistence/uniqueness, authorization, deposit idempotency, points, redemption, and failed dispense behavior.
- [ ] README setup instructions work.

## Final Instruction
Build EcoSync as a complete, secure, MongoDB-backed website. Prioritize permanent QR identity for machine identification, secure website authentication, reliable deposit/points records, the choice between immediate product redemption and saving points, and clear integration boundaries for the AI model and both physical units. Do not substitute fake data or simulated success messages for working functionality. If hardware or AI is unavailable, provide a truthful unavailable state and a documented integration path.
