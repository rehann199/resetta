# Technical Case Study

A description of how Resetta is built. Identifiers and constants are generalised.

## 1. Scope and size

Resetta is a single Flutter/Dart codebase targeting Android and iOS. A native Kotlin activity and a small Swift app delegate handle platform duties (a splash activity, App Tracking Transparency and push registration). There are no web or desktop targets.

About 240 hand-written Dart files and 65k lines of app code, excluding generated files. The test folder holds 12 Dart files and about 2.4k lines. Counts describe size only.

## 2. Stack at a glance

Flutter and Dart for the UI; Riverpod for state (notifiers and plain providers, no code-generated providers); `go_router` with a redirect guard driven by onboarding flags; immutable generated models; Hive for local JSON-map storage; Firebase Auth, Firestore, Cloud Messaging and Remote Config; local scheduled notifications; the Flutter in-app purchase library over Play Billing and StoreKit; one ad-mediation SDK through a locally forked plugin; Firebase Analytics, Crashlytics and Performance plus a product-analytics SDK; on-device PDF generation. The README has the table form.

## 3. The scoring pipeline

The pipeline is plain Dart with no framework dependencies, so it is unit-testable in isolation. Conceptually:

1. **Normalise.** Each 0 to 6 answer becomes a 0-100 value.
2. **Orient.** Exhaustion is used directly; the engagement and effectiveness items are reverse-scored so that a higher value always means more burnout in that dimension. Disengagement and inefficacy then read in the same direction as exhaustion.
3. **Smooth.** Each dimension is smoothed with an exponential moving average: the new value is a blend of today's reading and the previous smoothed value. The previous value is the last valid smoothed value, falling back to the baseline, and finally to today's raw reading on a first run. The blending strength is chosen from the user's work profile and is damped during the first days, when there is little history to trust.
4. **Combine.** The three smoothed dimensions are combined with fixed weights into the overall score.
5. **Band.** The score maps to one of four bands, and each band has a user-facing state name: Restored, Taxed, Overloaded, Depleted.
6. **Driver.** The dimension with the highest smoothed value is the dominant driver, with a fixed tie-break order.

The score has memory: a perfect first check-in after a poor baseline does not read as the lowest state. Because smoothing starts from the baseline, retaking it is easy and does not erase history.

The weights, smoothing strengths and band edges are product tuning and are not published. The scoring is documented in a QA specification with worked examples and a boundary table, and unit tests exercise baseline scoring, the daily formula, smoothing, the strength resolver and the dominant-dimension rule.

This is software that summarises self-reported answers. This write-up does not describe the score as a standardised or named instrument.

## 4. Deciding what to show for every date

The timeline shows something for every calendar date, even when the user did not check in. A resolver assigns each date a kind: checked in, interpolated between two real readings, held (carried forward), early estimate (a projection that decays toward the baseline while history is short), projection (stored on the last real check-in), baseline estimate, or no data.

Each kind carries a user-visible label so an estimate is never presented as a measurement. Projected bars decrease by a fixed step from their anchor, so a projection can never trend upward on its own. Past projections are frozen once their day has passed, so a retake or a backfill later does not rewrite what the user saw. The largest test file in the project exercises early-estimate and retake behaviour.

## 5. Early signals and projections

For users without much history, a rule-based layer looks at the level, the short-term change and per-dimension flags. Once enough check-ins exist, premium users see projected days from a heuristic: a simple linear trend with bias correction, and a low-confidence message after consecutive skips. The shipped projection is not a trained model.

A calm, supportive card appears after several consecutive days in the heaviest state, with a cooldown after dismissal. It is a UI behaviour, not an assessment.

## 6. Recovery selection

A schedule intensity (recovery, maintenance or protection) is derived from the score level and its recent direction. A library of short actions is then filtered by band, dominant driver and a maximum duration, excluding actions the user dismissed or completed recently. A longer forward schedule, per-dimension advice and the checklist are premium.

## 7. Persistence and sync

**Local first.** Hive is opened before the app starts. Boxes hold the user profile, baseline, daily check-ins, completed actions, weekly insights and settings, all as JSON maps. Check-ins are written locally first and the UI reads only local state. A storage-open failure is reported as a fatal storage error to analytics.

**Cloud sync for signed-in users.** The design rule is that the cloud is the source of truth and Hive is a cache the UI reads. On sign-in or start-up an initial sync checks whether cloud data exists: if it does, local edits are pushed and then cloud data is fetched into Hive; if not, local data is pushed up. Pushes are batched. Local wins ties and un-stamped cloud records, and a cloud document blocks a push only if its timestamp is strictly newer, so a stale device cannot overwrite a newer edit from another device. Real-time listeners merge incoming changes into Hive.

**Safety around lifecycle.** A generation counter ensures an in-flight sync cannot outlive a sign-out. The in-flight initialisation is shared so concurrent callers wait for the same sync. When a failure looks like "this user can no longer read their data", the app re-verifies the account by forcing a token refresh, signs out locally if the account is gone and explains why in a dialog.

**Connectivity.** Platform network status is combined with a DNS reachability check, because being on Wi-Fi is not the same as being online. An offline banner shows globally and pending changes are pushed when connectivity returns.

**Accounts.** Email, Google and Apple sign-in, password reset, "reset app" and account deletion (cloud data first, then the auth user) exist in the codebase. The sign-in entry point is controlled by a build-time switch, so this write-up does not claim it is active in a particular store build.

The project's Firestore rules scope per-user data to its owner.

## 8. Notifications

The daily reminder is a scheduled local notification at the user's chosen time, repeating by time of day, with exact alarms and a fallback to inexact scheduling. On Android a boot receiver reschedules after a restart. Beyond the reminder, three groups of messages are chosen on the device from check-in state: engagement (a missed day, an improvement, a pattern shift), lifecycle (an unfinished assessment, a first check-in) and conversion (premium nudges), with no cloud functions involved.

The reminder title is split-tested with a sticky 50/50 assignment per install, with an exposure event. Push messages arrive through Cloud Messaging; in the foreground they are re-shown locally so taps keep their deep links. Payloads carry an internal deep-link format used for notification taps, not for external links. An in-app pre-prompt asks for notification permission and applies a retry cooldown.

## 9. Monetisation

**Subscriptions.** Premium is an auto-renewing subscription with monthly and annual options. The app maps its own package concepts to store products and base plans, which differ by platform. On Android a purchase reports the same product for every base plan, so the selected package is persisted at purchase time to know which entitlement period applies, and intro prices are read from the first paid pricing phase.

Entitlement is determined on the device from store state at launch. Server-side verification would be the next step.

**Paywall.** Several layouts exist and can be selected through remote configuration, with placements after onboarding steps and on return to the home screen. A local trial tracker shows dialogs at trial milestones.

**Gating.** Premium unlocks recovery advice, the checklist, journal notes, weekly insights, longer history, the multi-day schedule, personalised actions and the monthly PDF export, and it removes ads.

**PDF export.** A monthly report is built on the device with a PDF library and handed to the share sheet.

## 10. Ads

One mediation SDK sits behind a thin wrapper with several adapter networks. Free-tier formats are an app-open ad at start-up, a dashboard banner, native cards and interstitials at onboarding milestones. Each placement can be switched off remotely, and premium suppresses ads at initialisation and on each refresh. A locally forked Flutter plugin adds banner and native preloading.

A start-up gate waits for consent (App Tracking Transparency on iOS, a consent flow after SDK start-up), then tries an app-open ad under timeouts, retries and a hard ceiling, so launch cannot hang. A per-zone throttle fronts every ad request because bursts are treated as no-fill, and a preload coordinator keeps one ad warm per format.

## 11. Analytics

Alongside Firebase Analytics, the app integrates a product-analytics SDK. Events come from a typed catalogue with controlled value sets, defined from a written specification. A transport-level guard drops parameters on a prohibited list (free text, exact answers). Events are buffered until analytics is enabled after the first frame, and on iOS collection stays off until the tracking prompt has been shown. A catalogue test enforces naming and structure rules.

## 12. Tests and tooling

Unit tests cover scoring, smoothing, day-state resolution and retakes, check-in streaks, notification copy and the analytics catalogue. There are no automated tests for sync, auth, billing, ads or notification scheduling.

Release builds on Android enable code shrinking and resource shrinking. The release artefact is an Android App Bundle. There is no automated release pipeline.

## 13. Trade-offs and next steps

Server-side receipt verification is the natural next step for entitlement. Optional at-rest encryption would harden journal text. Sync, billing and ad coordination would benefit from automated tests. The projection logic is simple by design. Copy is English-only today.
