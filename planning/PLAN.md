# PLAN

## Mission

Build a small, privacy-first messenger for families and friend groups that is easier than Matrix, more private than WhatsApp, and simple enough for normal people to use daily.

## Product principles

- No ads, ever
- No phone-number-first identity
- E2EE messages and attachments — server stores ciphertext only
- Minimal metadata retention, documented publicly
- EU-first hosting
- Web app first, native mobile later
- Clear export path; no user lock-in
- No AI training on user content
- No server- or client-side scanning of private chats

## Non-goals for v1

- No federation
- No public communities or channels
- No bots or platform ecosystem
- No custom cryptography
- No enterprise features
- No voice/video
- No self-hosting
- No native mobile apps (post-beta)

## v1 audience

- Families and friend groups (3–100 people) currently using WhatsApp reluctantly
- Privacy-conscious small teams and clubs in the EU

---

## Org setup

- GitHub org: github.com/Zag-Post
- Reserve main domain and subdomains
- Set up password manager, hardware keys, and separate admin accounts
- Legal/business entity only when needed for billing (later is fine)

## Repositories

| Repo      | Visibility        | Purpose                                                |
| --------- | ----------------- | ------------------------------------------------------ |
| `.github` | Public            | Org profile README, SECURITY.md, contribution guidelines |
| `app`     | Public            | SvelteKit PWA (plain project, no monorepo)             |
| `website` | Public            | Marketing site and landing pages                       |
| `legal`   | Public            | Privacy policy, terms of service, transparency reports |
| `server`  | Private (for now) | Backend API, auth, delivery, attachment services       |
| `infra`   | Private           | Terraform/Docker/deployment config                     |

---

## Technical direction

### Frontend

- SvelteKit — plain project, no monorepo tooling
- `src/lib/crypto/` — WebCrypto wrappers; clear internal boundary, future audit target. Not a separate package (only consumed by the web app)
- Dexie.js + IndexedDB for local encrypted message cache
- Service worker for offline shell
- Web Push for notifications

### Mobile (post-beta)

- Native iOS (Swift + SwiftUI) and Android (Kotlin + Jetpack Compose)
- Key storage: iOS Secure Enclave via CryptoKit; Android Keystore via Tink
- Local cache: SwiftData (iOS), Room (Android)
- Not Capacitor — WebView crypto has a weaker security posture and iOS kills WebView background processes aggressively

### Backend

- Rust (Actix-web or similar)
- TimescaleDB for account/device/delivery metadata
- S3-compatible object storage for encrypted attachment blobs (Cloudflare R2 preferred; EU hosting TBD)
- WebSockets for real-time delivery (gateway service TBD)
- Auth, delivery, and attachment services separated from each other
- APNs/FCM for native push (post-beta)

### Crypto / protocol

- Hybrid E2EE: message encrypted with symmetric key, symmetric key encrypted with recipient public key(s)
- Evaluating MLS (RFC 9420) for group key agreement — forward secrecy and post-compromise security
- Device-based key material, generated client-side, never transmitted in cleartext
- Encrypted attachments with per-file keys; decryption key travels inside the E2EE envelope
- TOFU for v1; opt-in key verification UI
- Phone numbers stored as hashes only (SHA-256 + global salt)

### Metadata policy

- No ad identifiers
- No third-party analytics in the app
- Short retention for IP and delivery logs
- User-visible retention table published in public docs

---

## Environments

- Local
- Staging
- Production

## Websites / domains

- Marketing site
- App domain
- Docs site
- Status page
- Legal pages

---

## Security baseline

- Mandatory 2FA / hardware keys for all admins
- Secret management from day one
- Dependency review and SAST in CI
- External security review before wide launch
- Reproducible builds where feasible
- 90-day coordinated disclosure policy; no legal action against good-faith researchers

---

## Milestones

Target: **private beta by December 2026**

### M0 - Foundation (May–Jun 2026)

- Name, domain, org, repos, SECURITY.md
- PLAN.md and ARCHITECTURE.md (ADRs)
- UI concept, design principles, and non-functional prototype

### M1 - Auth & Plumbing (Jul 2026)

- Real accounts and device registration
- Login flow
- WebSocket connection
- Plaintext messaging (no E2EE yet — functional scaffolding only)

### M2 - E2EE (Aug–Sep 2026)

- End-to-end encrypted DMs
- End-to-end encrypted group messages
- Encrypted attachment upload and download

### M3 - Media & Polish (Oct 2026)

- Photo and file sharing
- Voice notes
- UX polish and reliability fixes

### M4 - Push & Beta Prep (Nov 2026)

- Web push notifications
- Data export/import
- Admin ops, backups, monitoring
- Privacy policy, terms of service, and transparency page live

### M5 - Private Beta (Dec 2026)

- 50–100 invited users
- Donation flow live (Stripe/Ko-fi)
- Public security documentation
- Independent review summary (or pre-review engagement)
