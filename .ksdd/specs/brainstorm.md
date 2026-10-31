# Brainstorm — The Un-official Jellyfin Roku Client by IA

> The Jellyfin Roku client with a better TV experience, built by AI agents.

**Date:** 31/10/2026
**Status:** Approved

## 1. Concept in one sentence

An open-source soft fork of the official Jellyfin Roku client that modernizes its TV interface, is built by AI agents, and adds optional AI features, starting with one household.

## 2. Problem

The maintainer's household finds the official client's UI/UX dated or awkward on the TV. This is the problem to solve first. Playback, performance and feature gaps were not selected as priorities.

The specific pain points haven't been listed yet and there is no usage data. This is a first-hand observation, not a validated market need.

## 3. Proposed solution

Start from the official client (v3.2.5, BrightScript/SceneGraph, 15 locales) and redesign the screens people use most to find and watch something. What the client already does, including trickplay, lyrics and media segments, is inherited rather than rebuilt.

AI agents do the development work. A human reviews the changes and checks them on a real TV, because UX quality can't be judged from a build passing.

AI features are a secondary layer: optional, off by default, and never required to browse or play media. The working candidates are natural-language library search and "what should we watch tonight" suggestions drawn from the user's own library. Both are hypotheses until the SPEC picks one.

Releases are published in a public repo as a zip with sideload instructions. The store version of the official client stays installed alongside it (`docs/DEVGUIDE.md` confirms sideloading doesn't affect it), so the household always has a fallback.

## 4. Differentiator

Better UX on the TV is the differentiator, and the only one selected. AI-built development is how the product gets made and AI features are extras, so neither is the reason to choose it.

- **Official Jellyfin Roku client:** same server and media capabilities, backed by the Jellyfin project, with a one-click store install. This fork gives up the store install and official backing in exchange for a redesigned interface.
- **Other unofficial Roku clients:** not surveyed `[to validate]`.

## 5. Target audience

- **Primary:** the maintainer and their household.
- **Secondary:** other Jellyfin self-hosters on Roku who are comfortable enabling developer mode and sideloading.
- **Not for:** people who expect a store install or a supported replacement for the official client, and anyone unwilling to tolerate occasional breakage from an experimental, AI-built project.

## 6. References

- **Official `jellyfin-roku` client:** the base. Keep its features and server compatibility; replace the UX this project exists to improve.
- **Other products:** none provided. One or two UX references should be picked before the SPEC (see §10).

## 7. MVP scope

**In v1:**

- A redesigned watch path for movies and TV: home, library grid, item details and player controls. This scope is a proposal, to be confirmed against the household's pain-point list.
- Its own name and icon, so it can't be mistaken for the official app.
- A public repo with zip releases and sideload instructions.
- A repeatable check for agent-made changes: build, lint and a real-device checklist.
- One optional AI feature, only if the backend question in §10 is settled in time. Otherwise it moves to v1.1 and doesn't hold the release.

**Later:**

- Redesign of music, Live TV, photos and settings. These keep working as inherited.
- Additional AI features.
- A Roku Channel Store listing.
- Changes to playback behavior, other than bug fixes.

## 8. Business model (initial hypothesis)

Free and open source, under the GPL-2.0 license inherited from the official client. No revenue is planned. The one likely running cost is LLM usage if an AI feature calls an external API; the working hypothesis is that users bring their own key, so the project itself carries no ongoing cost.

## 9. Known constraints

- **Project limits:** deadline, budget, team size and available Roku test devices were not provided. They are unknown, not assumed to be unconstrained.
- **No on-device AI:** Roku apps run BrightScript and can't run models locally, so any AI feature depends on something off the device.
- **Weak safety net for AI-built code:** the repo has no automated tests and CI only builds. `docs/DEVGUIDE.md` asks contributors to run `npm run lint`, but `package.json` only defines `build`. Agents have little to verify their work against today.
- **Sideloading:** each device needs developer mode enabled. A device is believed to hold only one sideloaded app at a time `[to validate]`.
- **Branding:** `manifest` still has `title=jellyfin` and `package.json` describes the project as the "Official Roku app". Both must change before a public release. Whether the Jellyfin name may be used in the product's name needs checking against the project's branding rules `[to validate]`.
- **Upstream policy:** the Jellyfin project may restrict LLM-generated contributions `[to validate]`.

External policy items are unverified because web lookups weren't available when this was written.

## 10. Open questions

1. **Can AI-built fixes go upstream?** This comes first because it decides whether "send fixes back" is realistic. If Jellyfin restricts LLM-generated contributions, the fork needs another path, such as a human rewriting fixes or only reporting bugs.
2. **How does a soft fork survive a UX overhaul?** A redesign will diverge from upstream and create merge conflicts, and upstream is likely to accept fixes but not the redesign. Suggested principle for the SPEC: isolate UX changes from core and API code, and only send back fixes that apply regardless of the redesign.
3. **Where does the AI run?** The options are an external LLM API called from the app, or a server-side component such as a Jellyfin plugin or sidecar. The choice has to weigh API key handling on a TV, cost under a free model, and the privacy expectations of self-hosters, since library and viewing data may leave the home.
4. **Which AI feature comes first?** The candidates in §3 are hypotheses. Pick one once question 3 is answered.
5. **What exactly is wrong with the UX today?** List the household's concrete pain points by screen and choose UX references. This list confirms or changes the v1 scope in §7.
6. **What is the product called?** It needs a name and icon, subject to the branding check in §9.
7. **How are changes verified?** Decide which Roku devices are available for testing and how much of the real-device check can be automated.
8. **What are the real limits?** Confirm any deadline, budget and team size.

---

**Next step:** After this brainstorm is approved, run `/ksdd:spec` to generate the detailed specification.
