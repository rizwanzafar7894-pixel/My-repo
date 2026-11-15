# Dramora V9 implementation log

## Baseline and working method

- Frontend remains the Blogger XML theme.
- V9 work is isolated on branch `dramora-v9`; the existing `main` branch is not changed.
- V8 is the baseline. Preserve Firebase Authentication, Firestore user data, favorites, watch history/progress, episode navigation, community/moderation, and the established media-storage design.
- The platform remains entirely free: no premium memberships, paid episodes, coins, or payment upsells.
- Blogger remains primary media hosting (user-stated 100 MB/video limit); Backblaze B2 remains fallback for larger videos or Blogger upload failures. Firestore stores metadata and URLs, not media binaries.
- SoraFlix Free and Smart Tube Free are design references only. Do not copy their obfuscated scripts, `eval()`, redirect behavior, attribution injection, or unverified dependencies.

## V9.1 — Homepage featured carousel (implemented in this branch)

File: `Dramora/dramora-blogger-starter.xml`

- Replaced the static homepage hero with a carousel sourced from the existing Blogger `data:posts` collection.
- Each slide links directly to its Blogger post and displays its title, snippet, and thumbnail when available.
- Added previous/next buttons and generated slide indicators.
- Autoplay pauses on hover, keyboard focus, and hidden browser tabs; it respects `prefers-reduced-motion`.
- A fallback hero remains when the homepage has no posts.
- Added lazy loading and asynchronous image decoding to homepage post thumbnails.

## Static checks performed for this change

- The update was applied to the existing V8 XML file on a separate Git branch.
- The carousel markup uses Blogger post data and existing Blogger page URLs; it does not add a new backend or dependency.
- Existing Firebase/authentication/player functions were not intentionally edited by this change.
- No reference-template scripts or external packages were added.

## V9.2 — Personalized Continue Watching (implemented in this branch)

File: `Dramora/dramora-blogger-starter.xml`

- Added a homepage Continue Watching section that is visible only to an authenticated user.
- Reads the signed-in user's existing Firestore `watchHistory` through `DramoraData.listWatchHistory(8)`; it does not introduce a new collection or alter the V8 data contract.
- Cards use the saved canonical content URL, title, poster URL, position, and duration. URLs are passed through the existing `safeUrl` validation before use.
- Displays a progress indicator when a duration is available and a friendly empty state when no history exists.
- Handles loading and query errors without inserting Firestore text as HTML.
- Hides the section on sign-out and guards asynchronous responses against rendering data after the active account changes.
- Added a Trending Blogger label shortcut. It is a label navigation link, not a fabricated ranking or analytics feed.

## V9.2 static verification notes

- Confirmed the homepage host, authenticated-history query, URL validation, progress rendering, sign-out hiding, account-switch guard, and auth-render hook are present in the updated XML.
- The feature reuses the existing V8 Firestore watch-history contract; no rules or indexes were changed.
- Not yet verified on a live Blogger blog or against a live Firebase project. Actual rendered Blogger data, security rules, and mobile layout still require runtime testing.

## Live verification still required

1. Install the branch XML on a Blogger test blog and confirm Blogger accepts the template.
2. Confirm `data:post.snippet` renders as expected on the active Blogger theme/runtime.
3. Test zero, one, and multiple published posts.
4. Test previous/next buttons, slide indicators, hover/focus pause, tab visibility pause, and reduced-motion behavior.
5. Test mobile swipe/viewport behavior (the first increment currently uses buttons/dots; swipe gestures are not implemented).
6. Run the V8 regression suite against real Firebase configuration: auth flows, favorites, watch progress, episode navigation, community, Firestore rules/indexes, and media playback.
7. Confirm no visual regressions on desktop, tablet, and small Android screens.

## Next implementation sequence

1. Complete a V8 regression and Firebase/rules audit before changing data contracts.
2. Refine homepage sections and card presentation.
3. Upgrade drama detail/episode browsing while retaining current media registration and navigation.
4. Improve player states and progress reliability.
5. Finish responsive, performance, SEO, and security passes.
