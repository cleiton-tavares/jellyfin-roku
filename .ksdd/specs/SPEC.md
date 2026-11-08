# SPEC.md — JEllo

> A calmer way to watch Jellyfin on Roku. Unofficial, and built by AI agents.

**Version:** 1.0
**Last updated:** 08/11/2026
**Status:** Approved
**Target platform (MVP):** Roku devices, installed by sideloading
**Interface language:** English source copy, plus the 14 other locales inherited from the official client. New copy may appear in English until translated. Household TV language `[to confirm]`.

**Markers:** `[to confirm]` needs an answer from the maintainer. `[to validate]` is an external fact not yet checked. `[hypothesis]` is a proposed value to test.

## 1. Product Vision

### 1.1 Problem

The household has three confirmed pain points with the official client:

| Pain point | How the app works today |
|---|---|
| **Home:** too many rows, hard to see what to watch next | Up to eight sections set on the account, one "latest" row per library, and My List and Favorites added at the end. Two rows are visible at a time. |
| **Libraries:** slow to browse, filter or sort | The grid shares the screen with a block of item details, a dropdown row, a chip row and a letter picker. Items load in pages. |
| **Series:** moving between seasons and episodes is clumsy | The series screen shows a grid of seasons. Picking one opens a separate screen of episodes, and changing season means backing out. |

The right-hand column comes from reading the code, not from watching the app in use. Whether library slowness comes from layout, loading or both hasn't been measured on a device.

Details pages, player controls, search, accounts and settings were not flagged.

### 1.2 Solution

JEllo is a soft fork of the official client that redesigns those three areas around a calm, content-first experience, and leaves everything else working as inherited.

It adds one optional AI feature: "What should we watch tonight" suggestions drawn from the user's own library. The feature is off by default and uses the user's own LLM key. JEllo stays fully usable without it.

### 1.3 Target Audience

- **Primary:** the maintainer's household, made up of technical adults, non-technical adults and kids, sharing one Jellyfin account.
- **Secondary:** other Jellyfin self-hosters on Roku who are comfortable sideloading.
- **Not for:** people who expect a store install or a supported replacement for the official client.

### 1.4 Main Reference

**Apple TV app.** Borrow the principles: one focal point per screen, large artwork, quiet navigation, and an "Up Next" row that merges resuming and next episodes. Don't copy its visual assets or layouts.

This reference is described from general product knowledge. Check it against the real app before `/ksdd:design` `[to validate]`.

## 2. Personas

Derived from the household answers. Ages and individual habits haven't been provided `[to confirm]`.

| | **The Maintainer** | **The Non-Technical Adult** | **The Kid** |
|---|---|---|---|
| **Profile** | Technical adult who runs the Jellyfin server and installs JEllo | Adult who only wants to watch | Child with remote access; age range `[to confirm]` |
| **Context** | Evening viewing, plus setup and testing | Evening viewing, often undecided about what to watch | Returns to favorite shows and episodes |
| **Main frustration** | Having to explain the app to everyone else | Rows and menus get in the way of choosing | Getting lost between seasons and screens |
| **Needs** | Reliable playback, control over AI and privacy | An obvious next thing to watch | Big artwork and few steps to play |

## 3. Visual Identity and Design Direction

### 3.1 Brand Personality

Calm, content-first and quietly helpful. It's honest about being unofficial and experimental.

### 3.2 Color Palette (direction)

A dark, neutral base so artwork carries the color. One restrained accent, reserved for focus and primary actions. Avoid colors that make JEllo look like the official Jellyfin app.

### 3.3 Typography (direction)

Large and readable from across a room. Three clear levels: title, supporting line and metadata. Short line lengths, with no dense blocks of text on browsing screens.

### 3.4 Iconography

Simple and consistent. Primary actions always have a text label. Icon-only controls are limited to universally understood symbols such as play and pause. The app icon and name must not resemble the Jellyfin logo.

### 3.5 Overall Tone

Plain, short and friendly. Avoid technical terms on browsing screens. Error messages say what happened and what to do next.

## 4. Data Model

JEllo has no backend. The Jellyfin server stays the source of truth for media and progress.

| Entity | Key attributes | Relationships | Lives on | Sent to the LLM |
|---|---|---|---|---|
| **Server** | Address, name | Has accounts | Server, address saved on device | Never |
| **Account** | Name, preferences | One account shared by the household | Server | Never |
| **Library** | Name, type | Contains media items | Server | Never |
| **Media item** | Title, type, genres, year, rating, overview, runtime, artwork | Series → Season → Episode; movies stand alone | Server | Title and genres only |
| **Playback state** | Position, watched, favorite, last played | One per item, per account | Server | Never |
| **Home row** | Type, order, items | Built from media items and playback state | Device, derived | Never |
| **Automatic picks setting** | Rating limit | Applies to Spotlight, Something New and "With kids" | Device | Never |
| **AI settings** | Enabled, provider, key, disclosure accepted | One per device | Device | Key goes to the chosen provider only, to authenticate |
| **Suggestion session** | Who's watching, mood, sample, results | Results point to media items | Device, not kept | Mood only, along with the sample's titles and genres |

## 5. Data Sources

| Source | Provides | When |
|---|---|---|
| **Jellyfin server** | Media, artwork, playback state, libraries | Always |
| **Device storage** | Server address, app settings, AI settings | Always |
| **External LLM, chosen by the user** | Suggested titles and a one-line reason | Only when AI is enabled and the user asks |

**AI data rule:**

- **Sent:** titles and genres for a bounded sample of the library, plus the selected mood.
- **Never sent:** watched state, history, progress, favorites, ratings, overviews, years, or anything identifying the account or server.
- **The sample ignores watched state,** so its contents don't reveal what the household has seen. Watched titles are removed on the device after the results arrive.
- **The "who's watching" filter is applied on the device,** before the sample is built, and is not sent as a field.
- **Only real items are shown.** A suggestion that doesn't match an item in the library is dropped.

## 6. User Profiles and Permissions

Everyone shares one Jellyfin account, so JEllo can't tell people apart. Anyone holding the remote can do anything below. The "Intended" column is a household convention, and the app doesn't enforce it.

| Action | Kid | Non-technical adult | Maintainer | Enforced? |
|---|---|---|---|---|
| Browse and play | Yes | Yes | Yes | Server account limits only |
| Mark watched, favorite, My List | Yes | Yes | Yes | Shared across the household |
| Use Tonight suggestions | Yes | Yes | Yes | Only once the maintainer sets it up |
| Change app settings | Intended: no | Intended: rarely | Yes | No |
| Enter or remove AI key | Intended: no | Intended: no | Yes | No, but the key is masked after entry |
| Administer the server | No | No | Outside JEllo | Yes, by the server |

**Known limitations of one shared account, which v1 won't fix:**

- Up Next mixes everyone's progress, so kids' shows and adults' shows appear together. This may limit how far the "hard to see what to watch next" complaint can be solved.
- One person watching an episode marks it watched for everyone.
- There are no per-person restrictions. JEllo adds no parental controls. The rating limit in §7.2 only affects what JEllo picks automatically, and anyone can still browse the full library.

Separate server accounts would address most of this, and the inherited user switching already supports them. Profiles ranked last among the critical flows, so richer profile support is a Phase 3 candidate.

## 7. Page and Screen Structure

### 7.1 Global Navigation

**Objective:** reach any part of the app from Home and Libraries without clutter.

- A slim bar at the top of Home and Library screens. It fades out once focus moves into the content, and returns when focus moves back up.
- Order: Search, Home, one entry per library, Settings.
- Up to five library entries, with the rest under More `[hypothesis]`.
- Music, Live TV, photo and other library entries open their inherited screens.

### 7.2 Home

**Objective:** show something new to watch, with the next episode one press away.

| Area | Content | Behavior |
|---|---|---|
| **Spotlight** | Up to five unwatched titles, favoring recently added and refreshed daily `[hypothesis]`. Each shows large artwork, title, one line of metadata and up to two lines of overview. | Has focus on launch. Left and Right move through the picks, with no auto-rotation. OK opens details. |
| **Tonight card** | First Spotlight card, only when AI is set up | OK opens Tonight (§7.6) |
| **Up Next** | Resume and next episodes in one row, most recent first, one entry per series, with a progress bar | Directly below the Spotlight. OK starts playback. Hidden when empty. |
| **Recently Added** | Newest movies and shows across libraries | OK opens details |
| **Something New** | Other unwatched titles picked from across the library, refreshed daily, without repeating the Spotlight's picks | OK opens details |
| **My List** | The account's list | Hidden when empty |

**Decisions:**

- **The layout is fixed and curated on the device.** It doesn't follow the server's home section settings. The account is shared, so changing those to fix the TV would also change the web client for the whole household.
- **At most four rows sit below the Spotlight.**
- **Removed from Home:** per-library Latest rows, On Now and Continue Listening. Their content stays reachable through its libraries. Favorites moves to the existing library Favorites filter.
- **Settings for Home rows that no longer apply are hidden,** so no setting appears to do nothing.
- **Automatic picks respect a rating limit.** Spotlight and Something New choose titles on their own, and kids use this TV, so they only pick items at or below a limit set on the device. Unrated items are excluded. The default limit is `[to confirm]`. Recently Added and browsing aren't filtered.

**States:** placeholders while loading; a message and a path to Settings if no libraries are available; inherited error handling if the server is unreachable.

### 7.3 Library

**Objective:** get to a title quickly and make sort and filter feel immediate.

- **Header:** library name and one row with Sort, Filter and Genre, plus the current choices as removable chips and a "Clear all" action.
- **Main area:** a large grid of consistent posters. Item details shrink to one compact line for the focused title.
- **Letter jump:** kept from the inherited app, with a simpler presentation.
- **Favorites:** kept as a filter.

**Behavior:**

- Placeholders appear immediately, and more items load before the user reaches the end of the grid.
- Sort and filter choices are remembered per library.
- Returning from details restores the same position and focus.
- Changing a sort or filter never blocks the screen.

**States:** an empty result explains which filters are active and offers "Clear all".

### 7.4 Series

**Objective:** see where you are and move between seasons and episodes without changing screen.

- **Top area:** artwork, title, one line of metadata and up to three lines of overview.
- **Primary action:** names the episode, such as "Resume S2 E5" or "Play S2 E6".
- **Secondary actions:** Shuffle and Extras, with Watched and Favorite under More.
- **Season tabs:** a horizontal row. Moving between tabs updates the episodes on the same screen. Hidden for single-season series.
- **Episode list:** thumbnail, number, title, runtime, and watched or progress state.

**Behavior:**

- The screen opens on the next-up episode's season, with its episode in view.
- OK on an episode plays or resumes it. Options opens episode details, including audio and subtitle choices.
- Returning from playback restores focus and updates watched state.
- The separate season screen goes away. Anything that used to open it, such as a search result or "Go To Season", now opens the Series screen with that season selected.

**States:** a series with no episodes shows a short message. Specials appear as the last tab.

### 7.5 Movie and Episode Details

**Objective:** stay familiar while matching the new look.

Phase 1 is a **visual consistency pass only**: artwork treatment, type, focus style, buttons and navigation behavior. The action list and technical information stay as they are. Structural redesign moves to Phase 2, because these screens weren't flagged as pain points.

### 7.6 Tonight (optional AI)

**Objective:** help an undecided household choose something from their own library.

1. **Choose:**
   - Who's watching: "With kids" (the default each time) or "Just adults".
   - Mood: Light, Exciting, Thoughtful or Surprise me `[hypothesis]`.
   - A Suggest button.
2. **Waiting:** skeleton tiles in the three result slots. After about 10 seconds it stops waiting and uses the fallback `[hypothesis]`.
3. **Results:** three titles with artwork, a one-line reason and an "AI suggestion" label, plus Try Again. OK opens details.

"With kids" applies the rating limit and excludes unrated items. "Just adults" removes the limit.

**Fallback:** if the provider fails, or too few valid suggestions remain after filtering, JEllo fills the three slots with picks from the library using the same filters. These are labeled "From your library" and show a short note.

### 7.7 AI Setup (inside Settings)

**Objective:** one adult turns the feature on knowingly, once.

- **Off by default.** The Tonight card doesn't exist until setup is complete.
- **Disclosure before enabling.** Draft: "To suggest something, JEllo sends titles and genres from your library, plus the mood you pick, to your chosen provider using your own key. Watch history isn't sent. Your provider's terms and charges apply."
- **Key entry.** Typing a long key with a remote is a real usability problem, so `/ksdd:tech` must choose the entry method. The key is masked after entry and never shown in full again.
- **Test on save.** JEllo checks the key and reports the result.
- **Removal.** "Remove key" deletes it from the device and turns the feature off.

| State | What the user sees |
|---|---|
| No key | Feature off, with setup available in Settings. No Tonight card. |
| Invalid key | Settings shows the key was rejected. Tonight shows the fallback and a note. |
| Offline or timed out | Tonight shows the fallback and a note. |
| Quota exhausted | Tonight shows the fallback and a note to check the provider account. |

### 7.8 Inherited Screens

These keep working as they do today, and must stay reachable.

| Screen | Phase 1 treatment | Reached from |
|---|---|---|
| Search | Unchanged | Navigation bar |
| Player controls, subtitles, audio, next-episode prompt | Unchanged | Playback |
| Settings | Rebranded, with obsolete Home options hidden and AI setup added | Navigation bar |
| Change user, change server, sign out | Unchanged | Settings |
| Login and server setup | Rebranded | First launch |
| Music, Live TV, photos, other libraries | Unchanged | Navigation bar |

## 8. Global Reusable Components

| Component | Used in | Notes |
|---|---|---|
| **Content tile** | Rows, grid | Poster and landscape variants; default, focused, watched and in-progress states |
| **Spotlight** | Home | Artwork, title and metadata for the current pick |
| **Content row** | Home | Labeled and focus-preserving |
| **Navigation bar** | Home, Library | Fades when focus moves into content |
| **Labeled button** | All redesigned screens | Primary and secondary |
| **Season tabs** | Series | Scroll for long series |
| **Episode row** | Series | Watched and progress states |
| **Sort and filter panel** | Library | Choices appear as chips |
| **Chip** | Library, Tonight | Selectable or removable |
| **Loading placeholder** | All redesigned screens | Used instead of blocking spinners |
| **Empty and error message** | All redesigned screens | Says what happened and what to do next |
| **Source label** | Tonight | "AI suggestion" or "From your library" |

## 9. Critical Touchpoints

| Touchpoint | Where | Rule |
|---|---|---|
| **Unofficial notice** | First launch and an About item in Settings | States that JEllo is unofficial, AI-built and not affiliated with the Jellyfin project |
| **Where to report problems** | About and the repo README | Points to the JEllo repo, so upstream doesn't get JEllo's bugs |
| **Source code link** | About | Links to the public GPL-2.0 source |
| **Tonight entry** | First Spotlight card | Appears only after AI setup |
| **AI disclosure** | AI setup | Must be accepted before the key is used |
| **Rating limit** | Settings, and prompted during AI setup | Shared by automatic picks and "With kids" |
| **Client identification** | Requests to the server | JEllo reports itself to the server as JEllo, so the server's activity view can tell it apart from the official app |
| **Fallback to the official app** | Sideload instructions | Explains that the store version can stay installed alongside JEllo |

## 10. Responsiveness

This is a TV app, so there are no breakpoints.

- **One Full HD canvas.** Roku's scaling on HD TVs needs checking on a real device `[to validate]`.
- **Title-safe margins** on every screen, since TVs may crop the edges.
- **Readable at about 10 feet.** All required information must be legible from a couch.
- **Test devices `[to confirm]`.** Include the slowest Roku the household owns, because it sets the limit for artwork and motion.

## 11. Interactions and Behaviors

| Remote key | Behavior |
|---|---|
| **D-pad** | Moves focus. There's always exactly one clearly visible focused element. |
| **OK** | Plays in Up Next and in the episode list. Opens details everywhere else. |
| **Back** | Returns to the previous screen and restores focus. On Home, it moves focus to the Spotlight first, then follows the inherited exit behavior. |
| **Options (\*)** | Opens context actions, including details for items where OK plays. |
| **Play/Pause** | Plays the focused item directly from any browsing screen. |

- **Focus replaces hover.** Focus changes are the only way the interface reacts to attention.
- **Artwork updates wait briefly** when focus moves quickly, so fast scrolling doesn't flicker or lag.
- **Motion is short and subtle.** No auto-playing previews and no auto-rotating content.
- **Lower-end devices drop effects first.** Navigation speed takes priority over visual effects.
- **Accessibility doesn't regress.** Existing audio-guide labels are kept, and new controls get them too.

## 12. Business Model (Impact on Interface)

JEllo is free and open source under GPL-2.0, with no accounts, ads, plans or paywalls. The interface impact is limited to the bring-your-own-key setup in §7.7 and the AI-off state, which must feel complete and never look like a locked feature.

## 13. Critical Flows (User Journeys)

Listed in the household's priority order.

**1. Browse for something new**

1. Open JEllo. Focus starts on the Spotlight.
2. Left and Right move through picks. Down moves to Recently Added and Something New.
3. OK opens details, then Play starts the title.
4. *With AI set up:* OK on the Tonight card, choose who's watching and mood, Suggest, OK on a result, Play.
5. *If AI fails:* "From your library" picks appear, with a short note.

**2. Play the next episode**

1. Open JEllo, then press Down to Up Next.
2. OK starts the first entry, for 2 presses in total.
3. *From a series:* the primary action already names the next episode.
4. *During playback:* the inherited next-episode prompt applies.

**3. Find a specific title**

1. From Home, press Up to the navigation bar, Left to Search, then OK.
2. Use the inherited search.
3. *Alternative:* open a library and use the letter jump.

**4. Resume what I was watching**

Same path as flow 2: Down, then OK on the Up Next entry. Playback continues from the saved position.

**5. Change subtitles or audio during playback**

Uses the inherited player controls, unchanged. This is a regression check rather than a redesign.

**6. Switch to another household member's profile**

Settings, then Change user. It's unchanged and rarely needed with one shared account.

**Supporting flow: set up AI**

Settings, AI setup, read and accept the disclosure, set the rating limit, enter the key, test it. The Tonight card then appears on Home.

## 14. Delivery Phases

**Scope change from the brainstorm:** details and player controls were proposed for v1, but the household didn't flag them. Series navigation was flagged, so it joins v1. Details get a consistency pass only, and the player moves to Phase 2.

### Phase 1 — MVP

- JEllo name, icon and client identification; removal of "official" wording.
- Global navigation.
- Redesigned Home, Library and Series.
- Rating limit for automatic picks.
- Consistency pass on movie and episode details.
- Unofficial notice, About, and problem-reporting links.
- Public repo with zip releases and sideload instructions.
- Release checklist: build, lint and real-device checks of the six critical flows, including playback regressions.
- **Last and allowed to slip to v1.1:** Tonight and AI setup. They don't hold the release.

### Phase 2 — Finish the watch path

- Structural redesign of movie and episode details.
- Redesigned player controls.
- Search redesign.
- Settings cleanup.

### Phase 3 — Beyond video

- Redesign of music, Live TV and photos.
- Better support for separate household accounts, and parental controls.
- More AI features, such as natural-language search.
- A Roku Channel Store listing, subject to Roku's and Jellyfin's rules `[to validate]`.

## 15. Success Metrics

JEllo has no telemetry, to respect self-hosters' privacy. All targets are `[hypothesis]` and measured by hand. Baselines come from the official client on the same device and library.

| Metric | Target | Deadline |
|---|---|---|
| **Household default:** JEllo is the app used, with no return to the official client over UX | 4 consecutive weeks | Within 3 months of v1 |
| **Next episode:** presses from Home to playing | 2 or fewer | v1 |
| **Series:** presses from the primary action to an episode in the adjacent season, without changing screen | 4 or fewer | v1 |
| **Search:** presses from Home to open Search | 3 or fewer | v1 |
| **Library speed:** time to open a library and to apply a sort or filter | 30% faster than baseline on the slowest test device | v1 |
| **Unaided use:** a non-technical adult and a kid complete flows 1 and 2 without help | First attempt | v1 |
| **Playback regressions** compared with the official client | 0 on the release checklist | Every release |
| **AI accuracy:** suggestions outside the library, or above the limit under "With kids" | 0 on the release checklist | When Tonight ships |
| **AI usefulness:** Tonight sessions ending in playback, from a simple household log | At least 1 in 3 | 4 weeks after enabling |

## 16. Open Items

**For the maintainer:**

- The name spelling "JEllo" was confirmed on 08/11/2026. It's close to the JELL-O trademark `[to validate]`. This doesn't block a sideloaded household app, but check it before any store listing.
- Test devices, household TV language and kids' ages.
- Default rating limit for automatic picks.
- Any deadline or budget.

**For `/ksdd:tech`:**

- LLM provider options, sample size, and AI key entry and storage.
- Whether fixes can go upstream under Jellyfin's rules on LLM-generated contributions `[to validate]`, and how to isolate UX changes so tracking upstream stays practical.
- Automating as much of the release checklist as possible, given that the repo has no automated tests today.
- Sideload limits per device `[to validate]`.

---

**Next steps:**

- `/ksdd:tech` for technical architecture
- `/ksdd:design` for the complete design system (DESIGN.md in Google Stitch format)
