# Dramora — Blogger Micro-Drama Platform

Dramora is a free micro-drama streaming platform built on a Blogger XML theme.

## Project decisions
- Frontend: Blogger XML theme (not React).
- Visual direction: premium cinematic dark UI with purple/neon accents.
- Authentication and user data: Firebase Authentication + Cloud Firestore.
- Media storage: Blogger is primary for supported uploads; Backblaze B2 is the secondary/fallback provider for larger files or Blogger upload failures.
- Firestore stores media URLs and metadata, not video binaries.
- Community: authenticated-only public chat and private one-to-one messaging; text and URLs only.
- Monetization: completely free; no VIP, premium membership, coins, payment gateways, or paid-content locks.

## Final package contents (V8)
The release package is expected to contain:
- Blogger XML theme with Discover, series pages, media manager and smart player.
- Firestore security rules and composite indexes.
- Cloudflare Worker source for the B2 upload gateway.
- Admin utility for setting Firebase admin custom claims.
- Worker tests and deployment checklist.

## Important deployment note
This repository entry records the project overview. The V8 ZIP archive and full source files still need to be committed as binary/source assets; they have not been uploaded by this README-only commit.

## Deployment checklist
1. Configure the Firebase Web App config in the Blogger XML.
2. Deploy Firestore rules and indexes.
3. Enable Email/Password and Google Authentication providers.
4. Deploy the Cloudflare Worker and configure B2 secrets as Worker secrets.
5. Install the Blogger XML theme and create the required Blogger pages.
6. Register video assets and link each episode to its canonical Blogger post.
7. Test authentication, rules, upload fallback, playback seeking, watch resume, series navigation and community moderation on the live domain.

Never place B2 application keys or Firebase service-account credentials in Blogger/browser code.
