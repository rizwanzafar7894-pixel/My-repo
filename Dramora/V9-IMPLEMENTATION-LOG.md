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

## V9.3 — Dynamic Trending and Latest feeds (implemented)

File: `Dramora/dramora-blogger-starter.xml`

- Added a Trending feed section backed by Blogger's `/feeds/posts/default/-/Trending` JSON feed.
- Trending is explicitly editorial curation using the `Trending` label. It is not presented as a real-time popularity ranking.
- Added a Latest Episodes section backed by Blogger's newest-post JSON feed.
- Feed cards use safe URL parsing, DOM creation, textContent for post text, lazy image loading, and empty/error states.
- Added mobile grid refinements and reduced-motion CSS handling.

## V9.4 — Drama sharing and release checklist (implemented)

- Added a Share Drama action to Blogger post pages using the Web Share API, with clipboard fallback where supported.
- Added `Dramora/V9-REMAINING-WORK-AND-RELEASE-CHECKLIST.md` documenting publishing conventions, test matrix, security checks, and release blockers.
- No new Firebase collections, Firestore rules, or media storage contracts were introduced.

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


## V9.5 — Blogger-native Blog1 rendering and installation audit (implemented)

- Replaced the empty `Blog1` widget body with explicit Blogger v2 `main` and `dramoraPost` includables.
- The main includable iterates `data:posts` and delegates each post to the local post includable.
- Single-item views render the full Blogger post body; multi-item views render the title, snippet, and a direct Watch episode link.
- Post labels are rendered as Blogger label links, and index pages receive older/newer navigation when Blogger supplies those URLs.
- Kept the existing widget ID, type, version, settings, containing section, Firebase code, media code, community code, and V9 homepage feeds.
- Used a small purpose-built Blogger rendering layer rather than copying SoraFlix's 73-includable widget wholesale. The reference contains legacy markup/dependencies that are not required by Dramora.
- The theme still requires Blogger Pages to be created in the dashboard for the client-rendered account/community routes. Required slugs are documented in the release checklist.

## V9.5 verification limits

- GitHub write completed on `dramora-v9`; `main` remains unchanged.
- Static preflight confirmed one `Blog1` widget, explicit `main` and `dramoraPost` includables, a matching main-to-post include, and balanced CDATA markers.
- This is not a live Blogger parser/install test. Blogger may still reject an unsupported data expression or widget construct; the test blog remains the final authority.
- Do not claim the generic Blogger “Update failed” message is resolved until the updated XML is accepted by Blogger.


## V9.6 — Blogger-native uploaded episode video support (implemented)

- Detects Blogger's native uploaded-video markup: `object.BLOG_video_class` and IDs beginning with `BLOG_video-`.
- Preserves Blogger's native player instead of replacing it with a guessed MP4 URL or injecting a duplicate Firestore-registered player.
- Adds responsive sizing for the native object embed and marks the post's separator image (or first post image) as the episode poster.
- Uses the same post image as the poster source for Dramora's existing favorites/watch-history metadata, so library cards can display it automatically.
- This implementation does not extract undocumented Blogger video tokens or bypass Blogger's player.

### V9.6 verification limits

- GitHub commit completed on `dramora-v9`; no live Blogger playback test was performed in this change.
- Blogger's native video is an `OBJECT` embed, not a standard HTML `VIDEO` element. Browser-side watch-progress/resume tracking currently attaches only to standard `video` elements; it cannot reliably read playback time from Blogger's cross-origin/native player. Progress tracking for these native embeds remains an explicit limitation until a supported player API is available.
- Verify native playback on Android and desktop, portrait/landscape layout, and the poster image selection on the live test blog.


## V9.7 — Single-post hybrid multi-episode series experience (implemented)

- Added a Blogger-post parser for `.dramora-episodes` containing `.dramora-episode[ep-id]` entries.
- Reads the outer `description.dramora-series-description` as the series synopsis and each episode's `description.dramora-episode-description` as its episode description.
- Reads optional `ep-title` / `data-episode-title`; otherwise labels entries as Episode N.
- Sorts episode IDs numerically, rejects duplicate/invalid IDs, and ignores entries without a supported media element.
- Builds a hybrid series page inside the Blogger post: poster + title + synopsis, selected episode player, episode description, and responsive episode selector.
- Supports direct episode selection with the `?ep=N` query parameter and updates the URL when users select another episode.
- Moves the original Blogger-native `object.BLOG_video_class` element into the selected player slot; does not invent a media URL or convert Blogger content IDs into MP4 links.
- Series UI initialization is independent of Firebase being configured. Existing Firebase favorites and other account features remain separate.
- The feature uses DOM-created nodes and `textContent` for descriptions/titles rather than injecting post text as HTML.

### V9.7 authoring format

Use one Blogger post per series. Put the series poster image and series description outside the episode wrapper. Each episode must have a unique numeric `ep-id` and contain its Blogger-uploaded native video object. Optional `ep-title` provides a custom episode title.

```html
<div class="separator">
  <!-- Add the series poster image using Blogger's image uploader -->
</div>
<description class="dramora-series-description">
  The series synopsis goes here.
</description>
<div class="dramora-episodes">
  <div class="dramora-episode" ep-id="1" ep-title="The First Meeting">
    <!-- Insert Blogger's native uploaded-video object here -->
    <description class="dramora-episode-description">Episode 1 summary.</description>
  </div>
  <div class="dramora-episode" ep-id="2" ep-title="A New Beginning">
    <!-- Insert Blogger's native uploaded-video object here -->
    <description class="dramora-episode-description">Episode 2 summary.</description>
  </div>
</div>
```

### V9.7 verification limits

- GitHub branch code updated; live Blogger rendering and native playback still require a test post.
- Blogger may normalize or sanitize unknown `description` tags/attributes in the post editor. Confirm the saved post HTML retains the `class`, `ep-id`, and `ep-title` attributes. If Blogger strips them, use equivalent `data-*` attributes or a Blogger-safe wrapper structure.
- Native Blogger OBJECT playback remains opaque to page JavaScript; per-episode playback position/resume cannot be reliably read from that player. Standard HTML `video` tracking remains separate.


### V9.8 dedicated hidden thumbnail image

- Added a dedicated Blogger post-body thumbnail wrapper: `<div class="dramora-thumbnail"><img ... /></div>`.
- The wrapper is hidden on the single-post page with `display:none !important`, so the thumbnail does not appear in the visible series/player layout or leave a blank gap.
- The post image resolver prioritizes `.dramora-thumbnail img`, allowing the dedicated image to be used as the content thumbnail for cards and saved content metadata.
- The hybrid series poster selector and native-video poster selector explicitly skip the hidden thumbnail, so it is not moved into the visible series overview or displayed as an episode poster.
- Existing post images remain supported as fallback thumbnails when no dedicated thumbnail is provided.
- Live Blogger feed thumbnail extraction and editor preservation still require verification on a test blog.

Recommended authoring format:

```html
<div class="dramora-thumbnail">
  <!-- Upload the thumbnail image with Blogger's image uploader -->
  <img src="YOUR_BLOGGER_IMAGE_URL" alt="Drama thumbnail" />
</div>

<div class="dramora-series-description">
  Series description goes here.
</div>

<div class="dramora-episodes">
  <div class="dramora-episode" ep-id="1" ep-title="Episode 1">
    <!-- Insert Blogger's native uploaded-video object here -->
    <div class="dramora-episode-description">Episode 1 description.</div>
  </div>
</div>
```

Use the Blogger editor's actual uploaded image URL; do not leave the placeholder URL in a published post.


### V9.9 responsive mobile bottom navigation

- Added a fixed five-item mobile bottom bar: Home, Discover, My List, Community, and Profile.
- The bar is hidden on desktop/tablet widths above 760px; the existing desktop top navigation remains unchanged.
- Active destination is detected from the current URL path and receives an active purple treatment plus `aria-current="page"`.
- Added translucent dark/glass styling, compact icon-and-label layout, keyboard focus indication, and bottom safe-area padding for phones with gesture navigation.
- Added page bottom spacing on mobile so fixed navigation does not cover the final content.
- Admin remains accessible through the existing desktop/account navigation and is intentionally not a public bottom-bar destination.
- GitHub source update is complete; test the Blogger XML installation and tap targets on real Android/iOS viewport sizes before treating the feature as live-verified.

## V9.10 — Mobile carousel gestures and thumbnail-only series poster (implemented)

- Added horizontal swipe navigation to the homepage featured carousel on touch devices. Vertical page scrolling remains available; links, buttons, and form controls are excluded from swipe capture.
- A touch gesture pauses autoplay while active and restarts it after release/cancel, while reduced-motion and hidden-tab behavior remain respected.
- Updated hybrid series poster handling: a dedicated `.dramora-thumbnail img` is preferred, used as the post/card metadata image source, then removed from the visible single-post DOM. It is not reused as the visible series cover or episode poster.
- The series overview no longer reserves a poster column, preventing an empty image gap on the detail page.
- Existing fallback poster discovery remains available when no dedicated thumbnail wrapper exists.

### V9.10 verification limits

- GitHub branch updates completed. Touch swipe behavior and thumbnail extraction still need testing on an installed Blogger test blog and real mobile browsers.
- Blogger's feed `media$thumbnail` behavior can vary with the editor's image markup; verify that the dedicated thumbnail is returned in feed cards and saved-content metadata.
- This change does not alter Firebase data contracts, Firestore rules, media storage, or the Blogger-first / Backblaze B2 fallback architecture.


## V9.11 — Blogger Layout editor sections
- Added five editable Blogger Layout gadget areas: header/announcement, homepage top, homepage after hero, homepage bottom, and footer.
- Header brand text now uses the Blogger blog title (`data:blog.title`) so the visible name follows Blogger blog settings.
- Added responsive styling for inserted Layout gadgets; existing navigation, featured carousel, Blogger post feed, Firebase logic, and episode player code are otherwise unchanged.
- Gadget areas are empty by default and can be populated through Blogger Dashboard → Layout → Add a Gadget.
- Blogger install/save and live Layout editor behavior still require testing on the user's test blog.


## V9.12 — Layout-managed navigation and categories
- Added a dedicated Blogger Layout section inside the desktop navigation for a native LinkList gadget (`dramoraNavWidgets`).
- Added a Blogger Layout section beside the homepage mood chips for a native Label gadget (`dramoraCategoryWidgets`).
- The original built-in navigation and category chips remain as safe fallbacks. Once the corresponding native Blogger gadget is actually added, front-end initialization hides only that fallback to avoid duplicate menus/chips.
- Styled Blogger LinkList items as the existing horizontal navigation on desktop and a vertical list in the mobile expanded menu; Label gadget links use Dramora pill styling.
- Existing five V9.11 gadget areas, fixed mobile bottom navigation, Blogger post feeds, Firebase features, and episode player code remain in place.
- Setup: add a LinkList gadget to the Dramora navigation menu area and configure its URLs; add a Label gadget to the Dramora categories area and select the labels to show. Keep category labels on posts consistent with those selected in the Label gadget.
- Static Blogger Pages still must be created separately in Dashboard → Pages; adding a menu link does not create its destination page.
- GitHub source changes are not a Blogger installation test. Confirm the Layout editor accepts both sections and that LinkList/Label gadget output matches the intended CSS on a private test blog.


## V9.13 — Blog widget Layout registration and single-post recovery (code updated; live test pending)

- Moved the existing locked `Blog1` widget section out of the homepage-versus-inner-page conditional so Blogger can register the Blog section consistently in the Layout structure.
- Kept the existing `Blog1` ID, Blog widget type, settings, and `dramoraPost` renderer.
- The Blog widget now renders its post loop only on single-item views; homepage content remains handled by the existing Dramora homepage layout.
- The full post body remains sourced from Blogger's `data:post.body` field.
- This is a targeted structural change; it does not alter Firebase, Firestore, player, community, or media-storage contracts.
- GitHub update completed on `dramora-v9`. The live Blogger Layout editor and single-post output have not yet been re-tested; do not mark this as fixed until the user installs the updated XML and confirms the Blog Posts gadget and post content appear.


## V10 — Reference-led Blogger layout and archive rendering (code updated; live test pending)

- Re-inspected the supplied SoraFlix Free Version 1.0 XML as the primary architecture/layout reference and Smart Tube Free as a secondary thumbnail/recent-content reference. Dramora uses the references for layout patterns only; no vendor branding, obfuscated code, redirects, or old third-party dependencies were copied.
- Reworked the homepage into a responsive main-content + sidebar composition while retaining the Dramora featured carousel, continue-watching area, mood/category links, recent/trending/latest feeds, and home Layout gadget zones.
- Added configurable Blogger sidebar gadget zones for homepage and post/archive views, with Dramora genre and account quick links.
- Moved the inner-page title/back navigation into the Blog1 content column so the post renderer and sidebar share one coherent layout.
- Extended the existing Blog1 includable to render archive/search/label cards with Blogger thumbnails, snippets, Watch links, and older/newer pagination. The archive loop is suppressed on the homepage to avoid duplicating the custom Dramora feed. Single-item pages continue to render the full Blogger post body.
- Added responsive archive-card and sidebar styling in Dramora's existing dark cinematic purple visual system.
- Existing Firebase Auth/Firestore, favorites, watch history/progress, episode navigation, Blogger-first media storage and Backblaze B2 fallback decisions were left unchanged. No premium, paid-membership, or payment flow was added.
- XML tag nesting was checked after the edits, and static feature-presence checks passed. This is not a Blogger server/parser validation: installation, Layout gadget registration, archive pagination, and live post rendering still require a test-blog confirmation.
