# Aura & Co. — Curated Lifestyle Boutique

**Live:** https://aura-co-e-commerce.vercel.app

A production e-commerce storefront with an AI concierge, payment integration, and a Firebase
backend. Deployed on Vercel.

---

## What it does

- Product catalogue and browsing, built with React 19 and Tailwind CSS 4
- **AI concierge** — a Gemini-backed assistant (`api/gemini-chat.ts`) that answers product
  questions inside the storefront
- **Payment flow** — order creation, verification, and config, isolated into serverless functions
  under `api/` rather than bundled into the client
- **Passcode verification** — `api/verify-passcode.ts` gates access paths
- **Firebase backend** — Firestore with rules committed as `firestore.rules` and an
  `firebase-blueprint.json` for reproducible provisioning
- **TypeScript throughout**, with a shared `src/types.ts` and typed API boundaries

## Stack

| Layer | Choice |
|---|---|
| Frontend | React 19 · TypeScript 5.8 · Vite 6 · Tailwind CSS 4 · Lucide · Motion |
| Backend | Express (`server.ts`) + Vercel serverless functions under `api/` |
| Data | Firebase (Firestore) |
| AI | Google Gemini via `@google/genai` |
| Payments | Serverless order create/verify functions |
| Testing | Vitest + Testing Library + jsdom |
| Hosting | Vercel |

## Architecture notes

**The payment and AI calls live in `api/`, not in the client.** This is the decision that matters
most in the codebase: secrets stay server-side, and the browser never handles a payment
verification directly. `api/payment-create-order.ts` and `api/payment-verify.ts` are separate
functions so the verification path can be reasoned about independently.

**Firestore rules are committed.** Access control is a reviewable artifact in the repo rather
than console configuration that drifts invisibly.

**Tests cover the two areas where bugs are expensive** — `tests/currency.test.ts` (money
arithmetic) and `tests/security.test.ts`. Those are deliberate choices: currency formatting and
access control are where a quiet mistake costs real money or leaks data.

## Running it

```bash
cp .env.example .env.local     # add your Firebase and Gemini keys
npm install
npm run dev                    # vite + express via tsx
npm run test                   # vitest
npm run lint                   # tsc --noEmit
```

## Documentation

`docs/` holds the specifications this was built against:

1. [Product Requirements](./docs/1_PRD.md)
2. [Technical Architecture](./docs/2_TECHNICAL_ARCHITECTURE.md)
3. [Security & Access](./docs/3_SECURITY_AND_ACCESS.md)
4. [Frontend Specification](./docs/4_FRONTEND_SPECIFICATION.md)
5. [Feature Ticket List](./docs/5_FEATURE_TICKET_LIST.md)
6. [Firebase & Cloud Run Setup](./docs/6_FIREBASE_AND_CLOUD_RUN_SETUP.md)

---

## A note on how this was built

**To be precise: I wrote the specification and the architecture. AI wrote nearly all of the code.**

My contribution is the six documents in `docs/` — product requirements, technical architecture,
the security and access model, the frontend spec, the feature ticket list, and the Firebase setup
— plus reviewing and correcting what came back. I did not hand-write the application code.

The documents in `docs/` are the specification I worked from, and they are where the thinking
actually lives. They are worth reading before the source, because they explain why the code is
shaped the way it is.

What I'm happy to discuss in detail:

- **Why payment handling is isolated into serverless functions** under `api/` rather than living
  in the client. Secrets stay server-side, and splitting order creation from verification means
  the verification path can be reasoned about — and tested — independently.
- **Why Firestore rules are committed to the repo** as `firestore.rules` alongside
  `firebase-blueprint.json`, rather than configured in a console where they drift invisibly.
  Access control becomes a reviewable artifact and a diff.
- **Why the tests target currency arithmetic and access control specifically.** Those are the two
  places where a quiet mistake costs real money or leaks data. Everything else failing loudly in
  development is acceptable; these failing silently is not.
