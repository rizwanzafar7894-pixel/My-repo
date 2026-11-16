# Dramora V11 — Firebase, Firestore and Community Audit

Date: 2026-11-16
Branch: `dramora-v9`
Audited source: `Dramora/dramora-blogger-starter.xml` (V10 baseline)

## Scope and limits

This is a static source audit of the Blogger XML currently committed to the work branch. It is not a Firebase project inspection, Firestore Rules test, live Blogger test, or production security certification. The repository search did not locate a `firestore.rules`, `firebase.json`, `firestore.indexes.json`, or Cloudflare Worker source file in the repository's default-branch search results. Do not infer the contents of the deployed Firebase project from the front-end code.

## Findings

### 1. Firebase setup is not production-configured

- The theme still contains placeholder Firebase Web App values: `REPLACE_WITH_FIREBASE_WEB_API_KEY`, `REPLACE_WITH_PROJECT_ID`, and `REPLACE_WITH_FIREBASE_APP_ID`.
- `DRAMORA_MEDIA_API_URL` is also a placeholder Cloudflare Worker URL.
- Firebase initialization is guarded and will not start until the required values are replaced.
- These Web App configuration values are client-side identifiers, not service-account credentials. Never put Firebase Admin/service-account credentials or Backblaze application secrets in the XML.

**Status:** Setup incomplete; a real Firebase Web App and deployed Worker endpoint are required.

### 2. Authentication flows present in the client

- Email/password signup and login.
- Google sign-in with popup and redirect fallback.
- Password reset email.
- Email verification send/resend.
- Local Firebase Auth persistence.
- Auth-state listener, logout, and account profile synchronization.

**Gap:** The UI displays an email-verification panel, but the inspected auth flow does not block unverified users from entering the authenticated community/account flows. The product owner must decide whether verification is mandatory for all signed-in access or only for posting/messaging. Enforce the chosen policy in Firestore Rules/backend as well as the UI; a client-only check is not security.

### 3. Personal Firestore data model present

- `users/{uid}` profile document.
- `users/{uid}/favorites/{contentId}`.
- `users/{uid}/watchHistory/{contentId}`.
- `users/{uid}/watchProgress/{contentId}`.
- Public member discovery via `publicProfiles/{uid}`.

The personal data API scopes reads and writes under the current user's UID. This is good client-side structure, but only Firestore Rules can prevent a modified client from accessing another user's documents. Profile synchronization writes the email address to the private `users/{uid}` document; public profiles intentionally contain display name, avatar URL and search name rather than email.

### 4. Community and private messaging code present

- Public room messages at `communityRooms/public/messages`.
- Private conversations at `conversations/{conversationId}/messages`, with deterministic sorted member IDs.
- Authenticated client checks, real-time listeners, 30-message initial windows and load-older pagination.
- Message text is rendered as text nodes; detected links are restricted to HTTP/HTTPS and opened with `noopener noreferrer`.
- Blocking, unblocking, message/conversation reports and emoji reactions are implemented in the client.
- Private conversation membership is checked in the UI before displaying the message stream.

**Security dependency:** UI membership checks, hidden controls and client-side block checks can be bypassed. Rules must enforce authentication, exact two-member conversation membership, sender identity, participant-only message reads/writes, and block relationships. Public room reads/writes must follow the authenticated-only product requirement.

### 5. Rate limiting is not enforced by the inspected client

The send function writes `communityRateLimits/{uid}.lastSentAt` in the same batch as a message, but it does not read/check the previous timestamp or calculate a cooldown. A client can bypass this logic entirely. Treat this as a data marker only, not an active rate limiter.

**Required:** Enforce rate limits in trusted server-side logic or a carefully designed Firestore Rules strategy, with bounded message size and abuse controls. Do not rely on a disabled button or client timestamp.

### 6. Reports and moderation

- Client creates reports with a bounded reason and excerpt, initially marked `open`.
- Admin UI checks the Firebase custom claim `admin === true`.
- Admin actions write moderation status and user restrictions.

**Required Rules:** Only the authenticated reporter may create a report with their own reporter UID and initial status; only trusted moderators/admins may read reports, change review fields, or write user restrictions. Admin custom claims must be assigned only by a trusted Admin SDK environment, never from browser code.

### 7. Missing deployment artifacts / tests

Before changing any security/data contract, obtain the exact active production/test project artifacts:
- Firestore Security Rules currently deployed.
- Firestore composite indexes (if any).
- Cloudflare Worker source and its environment-variable names (never secret values).
- Confirmation of the Firebase project ID and authorized Blogger domains.
- Confirmation of the required email-verification policy.

Then test with at least two ordinary accounts and one admin test account. Verify cross-user denial, private conversation isolation, blocked-user behavior, report privacy, message rate limits, and admin-only moderation using the Rules Simulator/emulator and real test accounts.

## Priority remediation sequence

1. Import the exact deployed Firestore Rules and index definitions into the repository; do not overwrite them with guessed rules.
2. Replace placeholders only after the user supplies/sets their own Firebase Web App config and Worker URL in the intended Blogger theme.
3. Decide and implement the email-verification access policy.
4. Implement enforceable server-side/rules-based rate limits and block-aware conversation authorization.
5. Validate all collection paths/fields against Rules and indexes.
6. Run two-user and admin authorization tests; record results before marking security complete.

## Honest status

- Client-side feature code exists for authentication, profile sync, favorites, watch history/progress, public chat, private messaging, blocking, reports and custom-claim admin UI.
- Firebase configuration is still placeholder-based in the committed theme.
- Firestore Rules, index deployment, Worker deployment, email-verification enforcement, and production authorization tests are not verified by this audit.
- No production-ready or secure-live claim is made.
