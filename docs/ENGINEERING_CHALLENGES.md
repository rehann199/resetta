# Engineering Challenges

Each entry follows the same shape: the problem, the approach taken, the trade-offs, and the outcome. Constants, identifiers and file names are generalised on purpose. Where there are no tests, the entry says so.

## 1. A noisy self-report should not whip the result around

**Problem.** Daily 0 to 6 ratings follow mood. If the score were computed from today's answers alone, one rough day would flip a user to the heaviest state and one good day would erase a pattern. Either would make the state names meaningless.

**Approach.** Answers are normalised to 0-100 and oriented so that higher always means more burnout (the positively worded items are reverse-scored). Each of the three dimensions is smoothed with an exponential moving average seeded from the baseline, then the three smoothed values are combined with weights into the score. The strength of the smoothing depends on the user's work profile and is damped while history is short. The scoring code is plain Dart and has a written QA specification with worked examples and a boundary table.

**Trade-offs.** Smoothing makes the score lag. A user who feels much better today will see the state move gradually, which is the intended behaviour but can feel slow. The weights and strengths are product tuning rather than the output of a published procedure, so this write-up treats the result as a self-tracking summary, not a measurement.

**Outcome.** A deterministic, explainable pipeline with documented edge cases, for example why a first good day after a poor baseline does not read as the lowest state. Unit tests cover baseline scoring, the daily formula, smoothing, the strength rule and the dominant-dimension rule.

## 2. Never present an estimate as a measurement

**Problem.** People skip days, travel, or reinstall. The timeline still needs a marker for every date, and a made-up value that looks real would undermine trust in everything else on the screen.

**Approach.** A single resolver classifies each calendar date as checked in, interpolated, held, early estimate, projection, baseline estimate or no data. Each kind has a visual style (solid versus dashed markers) and a text label that tells the user when a number is an estimate or the data is stale. Projections are monotonic: bars step down from their anchor and never trend upward on their own. Once a day has passed, its projection is frozen so a later retake or backfill cannot rewrite history.

**Trade-offs.** More states mean more UI cases to design and test, and freezing means a past projection is deliberately never revised after the fact.

**Outcome.** A skip-friendly timeline. The largest test file in the project exercises early-estimate and retake behaviour, and backfilling a past day recomputes the smoothing chain forward.

## 3. Help from day one without pretending to predict

**Problem.** Any look-ahead needs history, but a new user needs something useful immediately.

**Approach.** A ladder. For the first stretch of check-ins a rule-based layer looks at the level, the short-term change and per-dimension flags. After enough readings, premium users see projected days from a simple linear trend with bias correction, plus a low-confidence message after consecutive skips. After several consecutive days in the heaviest state, a calm card suggests small steps, with a cooldown after dismissal.

**Trade-offs.** The projection is intentionally modest. The project contains code structured to host a trained model later, but ships none, and this write-up makes no machine-learning claim. The calm card is wording, not an assessment.

**Outcome.** A coherent rules-first, projection-later progression with honest limits.

## 4. Local-first with safe sync

**Problem.** The app must work with no account and no network, then reconcile across devices after sign-in, without a stale phone overwriting a newer edit.

**Approach.** Hive is the only thing the UI reads. For signed-in users Firestore is the cloud source of truth. An initial sync decides whether to push or pull depending on whether cloud data exists. Writes are batched. A push is skipped only when the cloud copy is strictly newer; local wins ties and un-stamped cloud records. A generation counter invalidates in-flight syncs on sign-out, and the initial sync is a shared future so concurrent callers wait on one operation. Pending edits flush when connectivity returns, where connectivity combines network status with a DNS reachability check.

**Trade-offs.** Timestamp-based last-writer-wins is simple but not field-level merging: two devices editing the same day offline resolve to one winner. The sign-in entry point is controlled by a build-time switch, so this portfolio does not claim sync is active in any particular store build.

**Outcome.** Guest use, offline check-ins, restore after sign-in and multi-device continuity are covered by design. Sync is not covered by automated tests.

## 5. Detecting a deleted account without a backend

**Problem.** If an account is removed elsewhere, the device may keep a cached session while every read is rejected by the rules.

**Approach.** Force a token refresh on start-up (with a timeout), on app resume and when a sync fails with a permission-style error. If the account is gone, sign out locally and show a one-time "session expired" dialog.

**Trade-offs.** It adds a network round trip at predictable moments; the timeout keeps start-up from waiting on it.

**Outcome.** Stale sessions are cleaned up without a server component.

## 6. A notification system that runs entirely on the device

**Problem.** Reminders must fire with the app closed, survive a reboot, respect time zones, and also serve activation and conversion goals, with no server to schedule them.

**Approach.** OS-scheduled daily reminders with exact alarms and an inexact fallback, rescheduled after reboot on Android. A campaign selector reads local check-in state and picks engagement, lifecycle or conversion messages. The reminder title is split-tested with a sticky assignment per install. Push messages are re-shown locally in the foreground so taps keep their deep link. An in-app pre-prompt asks for permission before the system dialog and applies a retry cooldown.

**Trade-offs.** On-device selection cannot use server-side knowledge, and exact alarms depend on a permission that some users decline.

**Outcome.** A campaign engine with no cloud functions. Scheduling is not covered by automated tests; notification copy and lifecycle triggers are.

## 7. An ads-first start-up that cannot hang the app

**Problem.** Ad SDK initialisation, consent prompts and ad loads vary widely in latency, and the ad network rate-limits repeated requests per zone, so several independent callers asking at once cause no-fill that then persists.

**Approach.** A start-up gate with per-platform timeouts, retry caps and a hard ceiling, which logs its outcome. Consent is requested natively on activation and de-duplicated. A per-zone throttle sits in front of every request. A preload coordinator waits for the first visible ads to settle and then keeps one ad warm per format. A locally forked plugin adds preloading for banner and native views. Remote switches control the network, each placement and the preload mode. Premium users skip all of it.

**Trade-offs.** A fork has a maintenance cost when the upstream plugin changes. Remote switches mean ad behaviour depends on a configuration document being readable before sign-in.

**Outcome.** Bounded launch time and coordinated requests. No revenue or fill-rate claims are made.

## 8. Subscriptions across two billing models

**Problem.** The two stores model subscriptions differently: flat products on one side, one product with base plans and offers on the other. On Android a restore reports the same product for every base plan, and entitlement checks are client-side.

**Approach.** Map the app's own package concepts to store products and base plans. Persist the selected package at purchase time. Re-check through restore or query on every launch, and read intro prices from the first paid pricing phase.

**Trade-offs.** Entitlement checks are client-side; server-side receipt verification is a natural next step.

**Outcome.** Working premium gating with a graceful fall-back to free.

## 9. A typed, guarded analytics catalogue

**Problem.** Ad hoc event names drift over time, and a wellness app should be careful about what it sends.

**Approach.** A written specification becomes typed trackers and controlled value sets. A transport-level guard drops parameters on a prohibited list (free text, exact answers). Events are buffered until analytics is enabled after the first frame, and on iOS collection starts only after the tracking prompt. A separate product-analytics SDK is also integrated. A catalogue test enforces naming and structure.

**Trade-offs.** The guard is name-based, so it protects only against parameters it knows about. A catalogue needs discipline to stay in step with the product.

**Outcome.** A consistent, testable event surface in an analytics layer of roughly 5k lines (a size figure only).

## 10. Remote control without a release

**Problem.** Monetisation experiments and incident response should not need an app update.

**Approach.** Configuration documents for ads that the client reads (before sign-in) but does not write, plus Remote Config for paywall layouts and limited-time offers.

**Trade-offs.** Anything remotely switchable has to fail safe if the configuration cannot be read.

**Outcome.** Server-side toggles for ads and paywall behaviour.

## What would come next

Server-side entitlement verification, automated tests around sync and billing, optional at-rest encryption for journal text, and, only if a model were genuinely added, a clear statement of what it does and does not do.
