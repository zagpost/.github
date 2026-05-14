# Zag Post - Project Context Document

## What is Zag Post?

Zag Post is a **web-first, privacy-first group messaging app** targeting families and friend groups. It is being built as a direct alternative to WhatsApp, addressing WhatsApp's core privacy weaknesses (metadata collection, Meta linkage, GDPR exposure) while being significantly easier to use than existing privacy-focused alternatives like Signal or Matrix/Element.

The project is currently in **early development**, targeting a private beta by end of 2026.

---

## The Problem Being Solved

Major messaging apps fail users in predictable ways:

- **WhatsApp** encrypts message content but still collects and shares extensive metadata with Meta (who talks to whom, when, from where). It is widely considered legally risky for EU business use under GDPR.
- **Signal** has strong encryption but requires a phone number for registration, has poor multi-device and web support, and is architecturally centralised with no self-hosting.
- **Matrix/Element** is decentralised and technically capable, but is widely criticised as too complex for normal users - confusing device verification, slow UX, and a high setup barrier.
- **Discord** has become a general community platform but has serious scraping problems, mandatory biometric age verification (as of 2026), and closed infrastructure.

No existing product in the mainstream or privacy-focused space simultaneously offers:

- WhatsApp-level ease of use
- Strong, verifiable E2EE with minimal metadata
- No phone number required
- EU-hosted, GDPR-clean infrastructure
- A web-first PWA that works without a native app install

---

## Target Users (v1)

**Primary:** Families and friend groups (3–100 people) who currently use WhatsApp, Telegram or Signal reluctantly, primarily because they have no better option.

**Secondary:** Privacy-conscious small teams and clubs in the EU who want group messaging without Meta or US-cloud exposure.

**Not the target for v1:** Large communities, public channels, bots/integrations, enterprise features, voice/video calls.

---

## Core Product Principles

These are non-negotiable constraints, not aspirational goals:

1. **No ads, ever.** Revenue comes from donations and paid mobile apps in the future.
2. **No phone number required.** Users sign up with a username and password. A phone number can optionally be added later, with a toggle controlling whether it is discoverable by others.
3. **End-to-end encrypted messages and attachments.** The server stores only ciphertext. Zag Post staff cannot read messages.
4. **Minimal metadata retention.** Message routing metadata is short-lived. Retained data is documented publicly with explicit retention periods.
5. **No server- and client-side content scanning.** Private chats are not scanned, moderated, or used for AI training.
6. **EU hosting by default.** Data does not leave the EU without explicit user consent.
7. **No lock-in.** Users can export their data and leave at any time.
8. **Honest about trade-offs.** Where a convenience–privacy compromise exists, it is documented clearly in public architecture docs.

---

## What Zag Post Is NOT

- Not a Matrix client or Matrix-compatible service
- Not a Signal fork or Signal-protocol product
- Not a public community platform (no public servers/channels in v1)
- Not a voice/video calling app (v1)
- Not a self-hostable product (v1 - planned for later)
- Not open-source yet (client code planned as AGPLv3 in a future release; server initially closed)
- Not a business/enterprise product (v1 - planned for later)

---

## Technical Stack

### Frontend (Web)

- **Framework:** SvelteKit (SPA/PWA mode)
- **Language:** TypeScript
- **Styling:** TailwindCSS
- **Local storage:** Dexie.js (IndexedDB wrapper) - encrypted message cache stored locally on the user's device
- **Crypto:** WebCrypto API (browser-native, no custom algorithms)

### Mobile (Later)

Native apps for iOS and Android. Not a Capacitor/WebView wrapper - see decision rationale below.

- **iOS:** Swift + SwiftUI; CryptoKit + Secure Enclave for key storage; SwiftData for local message cache
- **Android:** Kotlin + Jetpack Compose; Android Keystore + Tink for key storage; Room for local message cache
- **Real-time:** Native WebSocket clients (`URLSessionWebSocketTask` on iOS, Ktor/OkHttp on Android)

### Backend

- **Language:** Rust (for performance and safety in crypto handling)
- **Object storage:** S3-compatible (TDB: Cloudflare R2 preferred - zero egress fees - however, EU-hosted options are disputed as of mid-2026)
- **Push notifications:** Web Push API (web); APNs / FCM (native apps, later)

### Server Framework

No drop-in Rust messaging server exists (unlike WhatsApp's Ejabberd). Build on:

- **Axum** — HTTP + WebSocket handling (Tokio-native)
- **Tokio** — async runtime
- **NATS** — internal pub/sub bus for cross-node message routing (replaces Erlang/OTP message passing)
- **tokio-tungstenite** — WebSocket protocol (used under Axum)
  NATS routes messages between Rust nodes so horizontal scaling works without sticky sessions:
  `Client A → Rust Node 1 → NATS → Rust Node 2 → Client B`

**Conduwuit** (Rust Matrix homeserver) is the only "existing server" option, but only relevant if adopting the Matrix protocol and federation — not suitable for a closed WhatsApp-like app.

### Database Layer

#### Valkey (Redis-compatible) — Ephemeral Messages

- Stores encrypted messages per recipient as a Redis List: `msg:{user_id}`
- TTL of 7 days; deleted on delivery
- Rust crate: `redis` with `deadpool-redis` for connection pooling
- Valkey chosen over Redis for cleaner open-source licensing (BSD, Linux Foundation)

#### PostgreSQL — Pre-keys

- Stores user identity keys and one-time pre-keys
- Pre-keys are atomically claimed and deleted on use (`FOR UPDATE SKIP LOCKED`)
- Rust crate: `sqlx` with two pools (write → primary, read → replica)
- Migrations via `sqlx-cli`

### Scaling

#### Vertical

- Upgrade DB server hardware; cheapest first step
- Always use **PgBouncer** in front of Postgres (`transaction` pool mode, ~20 real connections behind 1000 client connections)

#### Horizontal

- **Rust app servers**: stateless, scale freely behind NGINX (`least_conn` for WebSocket fairness)
- **Postgres**: primary for writes, read replica for pre-key lookups
- **Valkey**: single node + replica for most workloads; Valkey Cluster for high throughput (auto-shards keys, supported natively by `redis` crate)

#### Order of operations

PgBouncer → more app servers → read replica → Valkey Cluster → vertical scale DBs → Postgres sharding (last resort)

### Infrastructure & Deployment

- **Hosting:** EU region, SvelteKit (SSG mode) deployed on Cloudflare Workers; App is TDB (maybe CF Workers or Pages); Hetzner Dedicated Server or similar for backend (TDB)
- **CI/CD:** GitHub Actions - type check + lint on every PR, preview deployments for PRs
- Dockerized via `docker-compose` (app, Postgres, Valkey)
- Multi-stage Dockerfile for lean Rust binary
- NGINX as load balancer with WebSocket upgrade headers
- PgBouncer as connection pooler between app nodes and Postgres

#### ADR: Marketing Site Deployment

**Decision:** Deploy the marketing site using SvelteKit (SSG mode) on Cloudflare Workers.
**Rationale:** SSG perfectly fits a static marketing site. Choosing Workers over Pages keeps the deployment toolchain consistent across our services.

### Tech Stack Summary

| Layer                   | Technology |
| ----------------------- | ---------- |
| Server language         | Rust       |
| HTTP + WebSockets       | Axum       |
| Async runtime           | Tokio      |
| Cross-node messaging    | NATS       |
| Ephemeral message store | Valkey     |
| Pre-key store           | PostgreSQL |
| Connection pooler       | PgBouncer  |
| Load balancer           | NGINX      |
| Migrations              | sqlx-cli   |

### Crypto / Protocol

- **Hybrid E2EE:** Messages are encrypted client-side using a symmetric key, which is then encrypted with the recipient's public key(s) and sent to the server. The server stores only the encrypted blobs.
- **Approach:** Use established standards, no custom cryptography
- **Group encryption:** Evaluating MLS (RFC 9420) for group key agreement with forward secrecy and post-compromise security
- **Attachments:** Client-side encrypted before upload; per-file decryption key sent inside the E2EE message envelope
- **Phone number discovery:** Phone numbers are hashed (SHA-256 + global salt) before storage. Contact discovery sends hashes only, never raw numbers.
- **Identity:** Each device gets its own key material. TOFU (Trust on First Use) for v1 - key verification UI is opt-in, not mandatory.

---

## Repository Structure

All repos live under the **`Zag Post`** GitHub organisation.

| Repo      | Visibility        | Purpose                                                |
| --------- | ----------------- | ------------------------------------------------------ |
| `.github` | Public            | Org profile README, SECURITY.md, guidelines, etc       |
| `app`     | Public            | SvelteKit PWA                                          |
| `website` | Public            | Marketing site and landing pages                       |
| `legal`   | Public            | Privacy policy, terms of service, transparency reports |
| `server`  | Private (for now) | Backend API, auth, delivery, attachment services       |
| `infra`   | Private           | Terraform/Docker/deployment config                     |

### Web app layout (`app` repo)

A plain SvelteKit project - no monorepo tooling, no workspace packages.

```
app/
├── src/
│   ├── lib/
│   │   └── crypto/ - WebCrypto wrappers (future audit target)
│   │       ├── keys.ts
│   │       ├── messages.ts
│   │       ├── attachments.ts
│   │       ├── groups.ts
│   │       ├── identity.ts
│   │       └── recovery.ts
│   ├── routes/
│   └── app.html
├── PLAN.md
├── ARCHITECTURE.md - Architecture Decision Records (ADRs)
├── package.json
└── svelte, vite, ... config stuff
```

#### `src/lib/crypto/` module

- `keys.ts` - generateKeyPair(), exportKey(), importKey(), storeKeyInIndexedDB()
- `messages.ts` - encryptMessage(plaintext, recipientPublicKey) → ciphertext --- decryptMessage(ciphertext, myPrivateKey) → plaintext
- `attachments.ts` - encryptFile(file) → { encryptedBlob, fileKey } --- decryptFile(encryptedBlob, fileKey) → File
- `groups.ts` - deriveGroupKey(), rotateGroupKey(), addMemberToGroup()
- `identity.ts` - fingerprint(publicKey) → human-readable safety number
- `recovery.ts` - encryptKeyBackup(keyMaterial, password) → encryptedBackup --- decryptKeyBackup(encryptedBackup, password) → keyMaterial

---

## Development Roadmap

Target: **Private beta by December 2026** (7 months from project start in May 2026).

| Milestone             | Months       | Deliverable                                                              |
| --------------------- | ------------ | ------------------------------------------------------------------------ |
| M0 - Foundation       | May-Jun 2026 | Name, org, repos, PLAN.md, ARCHITECTURE.md, UI prototype (no backend)    |
| M1 - Auth & Plumbing  | Jul 2026     | Real accounts, login, WebSocket connection, plaintext messaging          |
| M2 - E2EE             | Aug–Sep 2026 | End-to-end encrypted DMs and group messages; encrypted attachment upload |
| M3 - Media & Polish   | Oct 2026     | Photo/file sharing, voice notes, UX polish                               |
| M4 - Push & Beta Prep | Nov 2026     | Web push notifications, export/import, admin ops, monitoring             |
| M5 - Private Beta     | Dec 2026     | 50–100 invited users, donation flow live, public security docs           |

Native mobile apps are post-beta, not on the v1 roadmap.

---

## Monetization

- **Web app:** Free. Sustained by voluntary donations (Stripe/Ko-fi).
- **Mobile apps (iOS/Android, later):** One-time purchase, €0.50–€1.00 on App Store / Play Store.
- **No freemium tiers, no paid feature walls, no ads.**

The model is: free on the web for everyone, a nominal charge for native app convenience. This mirrors the GrapheneOS philosophy - the core product is free and open; sustainability comes from people who want to support the mission.

---

## Competitive Positioning

|                                   | WhatsApp | Signal | Matrix/Element         | **Zag Post** |
| --------------------------------- | -------- | ------ | ---------------------- | ------------ |
| E2EE content                      | ✅       | ✅     | ✅                     | ✅           |
| Minimal metadata                  | ❌       | ✅     | ⚠️ (depends on server) | ✅           |
| No phone number required          | ❌       | ❌     | ✅                     | ✅           |
| Easy for non-technical users      | ✅       | ⚠️     | ❌                     | ✅ (goal)    |
| Web-first / no app install needed | ⚠️       | ❌     | ⚠️                     | ✅           |
| EU hosting / GDPR clean           | ❌       | ❌     | ⚠️                     | ✅           |
| No Meta/Google dependency         | ❌       | ✅     | ✅                     | ✅           |
| Open source                       | ❌       | ✅     | ✅                     | Planned      |
| Self-hostable                     | ❌       | ❌     | ✅                     | Planned      |

---

## Key Decisions and Rationale

### Why SvelteKit?

Small compiled bundles (important for privacy-hardened browsers), client-heavy architecture suits doing crypto in the browser, good PWA support, ergonomic stores for local-first state.

### Why a plain SvelteKit project and not a monorepo?

There is only one web app and no packages shared between multiple consumers. A monorepo with Turborepo and pnpm workspaces would add build tooling overhead (separate `package.json`, tsconfig, build steps per package, workspace linking) for no real benefit. If a second consumer ever appears, extracting a package is a straightforward refactor. Starting simple is the right default.

### Why is the crypto module in `src/lib/crypto/` and not a separate package?

The crypto module is only consumed by the web app - there is no second consumer that would justify the overhead of a standalone package. It lives in `src/lib/crypto/` to enforce a clear internal boundary: crypto logic cannot import anything from routes or UI components, which is verified by code review. An auditor can be pointed at this directory just as cleanly as a separate package. If a native mobile client eventually needs shared crypto logic, that decision can be made at the time with full context.

### Why native mobile apps instead of Capacitor?

Zag Post's core value proposition is E2EE and verifiable privacy. Capacitor runs crypto inside a WKWebView (iOS) or WebView (Android), which means private keys end up in IndexedDB within a WebView sandbox - a meaningfully weaker security posture than platform-native key storage. Native apps can use the iOS Secure Enclave (via CryptoKit) and Android Keystore, where private keys can be hardware-bound and non-exportable. Additionally, iOS aggressively kills WebView background processes, which creates reliability problems for a messaging app's WebSocket connection. The team has the skills to build native (Swift/SwiftUI, Kotlin/Jetpack Compose), so Capacitor's main advantage - avoiding native development - does not apply here.

### Why not build on Matrix?

Matrix is a federated protocol, not a product. Building on it means inheriting its complexity, UX problems, and homeserver model. Zag Post's goal is to be simple enough for Grandma - that is incompatible with Matrix's current UX ceiling.

### Why not use the Signal Protocol?

Signal's server infrastructure is closed and does not allow third-party clients or self-hosting. The Signal Protocol library itself is open source and could be used, but MLS (RFC 9420) is newer, standardised by the IETF, and better suited to multi-device group messaging scenarios.

### Why closed server code initially?

Moving fast to beta without the overhead of community contributions to the server. The client code (SvelteKit, crypto module) will be open-sourced first because that is what users can verify. Server open-sourcing and self-hosting support come in a later phase.

### Why no federation in v1?

Federation adds massive complexity to identity, key distribution, message routing, and abuse handling. A centralised v1 lets the team focus on UX quality and crypto correctness. Federation or interoperability is a v3+ consideration.

---

## Security & Privacy Architecture (Summary)

- **Threat model:** Protect against a compromised or subpoenaed Zag Post server. The server should be able to deliver messages without being able to read them.
- **Keys:** Generated client-side on each device. Never transmitted to the server in cleartext.
- **Message storage:** Server stores encrypted envelopes only. No plaintext ever touches the server.
- **Metadata minimisation:** IPs are not logged long-term. Delivery metadata (timestamps, recipient device IDs) is retained only as long as needed for delivery, then deleted.
- **Phone numbers:** Stored as hashes only. The raw number is never persisted server-side.
- **Attachments:** Encrypted client-side before upload. The decryption key travels inside the E2EE message, never separately.
- **Admin access:** No Zag Post employee can read a user's messages. This is a technical guarantee, not a policy-only claim.

---

## Contact & Security

- **Security reports:** security@zagpost.com or GitHub Private Vulnerability Reporting
- **Disclosure policy:** 90-day coordinated disclosure; no legal action against good-faith researchers
- **Organisation:** github.com/zagpost

---

_Last updated: May 2026 - update this document whenever a significant decision is made._
