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

### Frontend

- **Framework:** SvelteKit (SPA/PWA mode)
- **Language:** TypeScript
- **Styling:** TailwindCSS
- **Local storage:** Dexie.js (IndexedDB wrapper) - encrypted message cache stored locally on the user's device
- **Crypto:** WebCrypto API (browser-native, no custom algorithms)
- **Mobile (later):** Capacitor wrapping the same SvelteKit codebase for iOS/Android OR native apps (TBD based on feasibility and resource constraints)

### Backend

- **Language:** Rust (for performance and safety in crypto handling)
- **Web framework:** Actix-web or similar lightweight Rust web framework
- **Database:** TimescaleDB (PostgreSQL-based, optimized for time-series data)
- **Object storage:** S3-compatible (TDB: Cloudflare R2 preferred - zero egress fees - however, EU-hosted options are disputed as of mid-2026)
- **Real-time:** WebSockets (TDB: Maybe a Gateway Websocket service)
- **Push notifications:** Web Push API

### Infrastructure

- **Hosting:** EU region, Cloudflare Pages for the marketing site; App is TDB; Hetzner Dedicated Server or similar for backend (TDB)
- **CI/CD:** GitHub Actions - type check + lint on every PR, preview deployments for PRs
- **Monorepo tooling:** Turborepo (pnpm workspaces)

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

| Repo      | Visibility        | Purpose                                                               |
| --------- | ----------------- | --------------------------------------------------------------------- |
| `.github` | Public            | Org profile README, SECURITY.md, guidelines, etc                      |
| `app`     | Public            | SvelteKit monorepo: `apps/web/` + `packages/ui/` + `packages/crypto/` |
| `website` | Public            | Marketing site and landing pages                                      |
| `legal`   | Public            | Privacy policy, terms of service, transparency reports                |
| `server`  | Private (for now) | Backend API, auth, delivery, attachment services                      |
| `infra`   | Private           | Terraform/Docker/deployment config                                    |

### Monorepo layout (`app` repo)

```
app/
├── apps/
│   └── web/             ← SvelteKit PWA
├── packages/
│   ├── ui/              ← Shared component library
│   └── crypto/          ← WebCrypto wrappers - clean boundary, future audit target
├── PLAN.md
├── ARCHITECTURE.md      ← Architecture Decision Records (ADRs)
└── turbo.json
```

#### `crypto` package

- `keys.ts` - generateKeyPair(), exportKey(), importKey(), storeKeyInIndexedDB()
- `messages.ts` - encryptMessage(plaintext, recipientPublicKey) → ciphertext --- decryptMessage(ciphertext, myPrivateKey) → plaintext
- `attachments.ts` - encryptFile(file) → { encryptedBlob, fileKey } --- decryptFile(encryptedBlob, fileKey) → File
- `groups.ts` - deriveGroupKey(), rotateGroupKey(), addMemberToGroup()
- `identity.ts` - fingerprint(publicKey) → human-readable safety number
- `recovery.ts` - encryptKeyBackup(keyMaterial, password) → encryptedBackup --- decryptKeyBackup(encryptedBackup, password) → keyMaterial

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

Small compiled bundles (important for privacy-hardened browsers), client-heavy architecture suits doing crypto in the browser, good PWA support, ergonomic stores for local-first state. The same codebase can be wrapped with Capacitor for native mobile apps later.

### Why not build on Matrix?

Matrix is a federated protocol, not a product. Building on it means inheriting its complexity, UX problems, and homeserver model. Zag Post's goal is to be simple enough for Grandma - that is incompatible with Matrix's current UX ceiling.

### Why not use the Signal Protocol?

Signal's server infrastructure is closed and does not allow third-party clients or self-hosting. The Signal Protocol library itself is open source and could be used, but MLS (RFC 9420) is newer, standardised by the IETF, and better suited to multi-device group messaging scenarios.

### Why closed server code initially?

Moving fast to beta without the overhead of community contributions to the server. The client code (SvelteKit, crypto package) will be open-sourced first because that is what users can verify. Server open-sourcing and self-hosting support come in a later phase.

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
- **Organisation:** github.com/Zag-Post

---

_Last updated: May 2026 - update this document whenever a significant decision is made._
