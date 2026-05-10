# PLAN

## Mission

Build a small, privacy-first messenger for groups that is easier than Matrix, more private than WhatsApp, and simple enough for normal people to use daily.

## Product principles

- No ads.
- No phone-number-first identity
- E2EE messages and attachments
- Minimal metadata retention
- EU-first hosting
- Web app first, mobile later
- Clear export path; no user lock-in
- No "AI training on user content."
- No server- and client-side scanning of private chats

## Non-goals for v1

- No federation.
- No public communities.
- No bots/platform ecosystem.
- No custom crypto.
- No broad enterprise feature set.
- No voice/video in first release unless absolutely necessary.

## v1 audience

- Small teams
- Families/friend groups
- Privacy-conscious communities that currently use WhatsApp/Discord reluctantly

## Company / org setup

- Create GitHub org.
- Reserve main domain and subdomains.
- Set up password manager, hardware keys, and separate admin accounts
- Set up legal/business entity only when needed for billing and terms (later is ok)

## Repositories

- .github
- webapp
- server
- protocol (?)
- website
- legal
- status

## Technical direction

### Frontend

- SvelteKit
- IndexedDB for local encrypted cache
- Service worker for offline shell
- Optional push notifications

### Backend

- Auth service
- Delivery service
- Attachment service
- Admin/support service separated from core systems
- Postgres for account/device metadata
- Object storage for encrypted blobs

### Crypto / protocol

- Evaluate MLS-based group encryption
- Separate protocol spec from implementation
- Device-based keys
- Encrypted attachments with per-file keys
- Server stores ciphertext and minimal routing metadata only

## Metadata policy draft

- No ad identifiers
- No third-party analytics in app
- Short retention for IP and delivery logs
- Clear user-visible retention table in docs

## Environments

- Local
- Staging
- Production

## Websites

- main marketing site
- docs site
- legal site
- status page
- app domain

## Security baseline

- Mandatory 2FA/hardware keys for admins
- Secret management from day one
- Reproducible builds where possible
- Dependency review and SAST in CI
- External security review before wide launch

## Milestones

### M0 - Foundation

- Name, domain, org, repos, base docs
- Architecture decision records
- UI concept and design principles

### M1 - Clickable prototype

- SvelteKit shell
- Auth flow mock
- DM UI
- Group UI
- Local-only encrypted message store

### M2 - Functional alpha

- Real accounts/devices
- Encrypted DMs
- Encrypted small groups
- Attachment upload/download
- Basic invite flow

### M3 - Private beta

- Reliability fixes
- Export/import
- Push notifications
- Admin ops, backups, monitoring
- Privacy/legal pages live

### M4 - Launch

- Billing or donation flow
- Public security docs (github enough?)
- Independent review summary
- Transparency page
