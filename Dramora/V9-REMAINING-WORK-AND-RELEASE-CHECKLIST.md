# Dramora V9 — Release Readiness and Remaining Work

## What is implemented on `dramora-v9`

- Blogger XML remains the frontend; no React migration.
- Homepage featured carousel uses the existing Blogger post loop, previous/next controls, indicators, reduced-motion handling, and pause behavior.
- Homepage Continue Watching reads the signed-in user's existing Firestore `watchHistory` collection. It does not create a new collection or change the V8 data contract.
- Homepage Trending section loads posts from the Blogger `Trending` label feed. This is editorial curation: publishers must apply the `Trending` label; it is not a view-count ranking.
- Homepage Latest Episodes section loads the newest Blogger feed entries.
- Feed cards validate post/image URLs, build DOM nodes with `textContent`, lazy-load images, and show empty/error states.
- Drama post pages have a native share action with Web Share API and clipboard fallback.
- Responsive feed/Continue Watching grids and reduced-motion CSS refinements are included.
- Existing V8 Firebase, favorites, watch-progress, community, media registration and Blogger-first/B2-fallback code were retained rather than replaced.

## Blogger publishing contract

1. Publish each drama or episode as a normal Blogger post.
2. Add a clear title and a high-quality first image (used as the poster where Blogger exposes it).
3. Add genre labels such as `Romance`, `Thriller`, `Comedy`, or `Action`.
4. Add the `Trending` label only to editorially selected posts. Remove it when the post should no longer be featured.
5. Add the `Latest` label if you want it available in the existing Latest label archive. The homepage Latest feed itself reads the newest posts, regardless of label.
6. For multi-episode dramas, keep the existing V8 media registration metadata and episode numbering consistent. Do not publish a fake video URL or expose storage credentials in post HTML.

## Required live acceptance tests (cannot be completed from a GitHub-only edit)

### Blogger template installation
- Back up the current live Blogger theme.
- Install the XML from this branch on a private/test Blogger blog first.
- Confirm Blogger accepts the XML without template parser errors.
- Test homepage with zero posts, one post, and multiple posts.
- Confirm Blogger JSON feeds are enabled and same-origin feed requests return JSON.
- Test a post with an image, a post without an image, and a post with unusual characters in its title.
- Test Trending feed with zero, one, and multiple posts carrying the exact `Trending` label.
- Test Latest feed ordering and links.
- Test desktop, tablet, and small Android viewport widths.

### Firebase Authentication and Firestore
- Use the real Firebase Web App configuration belonging to the site; never commit service-account JSON or private API secrets into the public theme.
- Confirm authorized domains include the exact Blogger custom domain and the `blogspot.com` test domain as applicable.
- Enable only the sign-in providers intended by the product (email/password and Google).
- Test signup, email verification, login, logout, Google sign-in, password reset, persistence, and account switching.
- Verify Firestore rules in the Firebase Rules Simulator and with two separate test accounts.
- Confirm a user cannot read/write another user's favorites, profile, watch history, or watch progress.
- Confirm community reads/writes require authentication and test block/report/rate-limit behavior.
- Confirm public media reads are limited to records explicitly marked public and ready.
- Deploy the required Firestore indexes only after checking the actual queries and index build status.

### Media and player
- Confirm Blogger-hosted videos up to the user's stated 100 MB limit play on supported devices.
- Confirm larger files or Blogger upload failures use the secured Backblaze B2 fallback flow.
- Verify no B2 application key, Firebase service-account credential, or privileged worker secret appears in page source, network responses, or browser storage.
- Test playback on Android Chrome and desktop Chrome/Firefox/Safari where available.
- Test seeking, pause/resume, reload/resume, ended/completed state, next/previous episode, autoplay preference, and failed/unsupported media.
- Confirm the browser's native player controls remain usable and the player does not force autoplay without user preference.

### Security, privacy, and abuse
- Review Firestore rules and Worker authorization against the actual deployed Firebase project and Worker environment.
- Test malicious HTML, javascript: URLs, external links, oversized messages, repeated messages, blocked users, and report submission.
- Confirm private messages are readable only by their two participants and authorized moderators where explicitly designed.
- Confirm no user-uploaded files or Firebase Storage upload surface is enabled in community.
- Confirm user data is not exposed through unauthenticated collection queries.

### SEO and performance
- Check canonical URLs, page titles, descriptions, Open Graph metadata, Blogger sitemap, and robots settings on the live domain.
- Run Lighthouse on the actual public domain; record mobile performance, accessibility, best practices, and SEO.
- Check image dimensions, lazy loading, layout shifts, keyboard navigation, focus visibility, and color contrast.
- Test with reduced-motion enabled and with slow network/device throttling.

## Release blockers / honest status

- A GitHub-only code edit cannot install a theme into the user's Blogger account or access their Firebase/Backblaze production consoles.
- No claim is made that Blogger accepted the XML, Firebase rules passed live authorization tests, media streams work on real devices, or production secrets are configured.
- Do not merge or deploy until the acceptance tests above pass.
- The Trending feed is editorially curated through a Blogger label. A true popularity-ranked Trending feed requires reliable view/watch analytics and abuse-resistant counting; it is deliberately not simulated.
- If the current production V8 rules/Worker source are not present in the repository, bring those exact files into the repository before attempting a rules/Worker rewrite. Do not infer or replace production security policy from the front-end theme.

## Branch / review

- Work branch: `dramora-v9`
- Base branch: `main`
- Existing draft PR: #1
- Keep the PR in draft until live acceptance tests are complete.
