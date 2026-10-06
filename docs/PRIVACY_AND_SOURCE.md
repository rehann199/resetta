# Privacy and Source

## What this repository is

A portfolio showcase for a production application. It describes the engineering of Resetta, a burnout and daily stress self-tracking app for Android and iOS. It is not a source distribution.

## What is published

- Written descriptions of architecture, design decisions and trade-offs
- Original diagrams created for this showcase. Diagram labels are conceptual and do not correspond to production identifiers.
- Real screenshots from the Android build, cropped to the app content (system status bar and ad-placeholder strips removed), captured with sample/placeholder data
- The app icon, re-encoded

## What is intentionally not published

- Production source code, in whole or in part, and directory layouts of the production project
- Scoring weights, smoothing constants, band cut-offs and other tuning values
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration, API keys and tokens
- Ad network identifiers, placement identifiers and account details
- Billing product identifiers, prices, trial lengths and store configuration
- Internal endpoints, collection names and event names
- Version control history of the production repository
- User data of any kind, and any store statistics

## Images

- **Screenshots** are real captures from the Android build, taken with sample/placeholder data. The score result is labelled as a sample session. No notifications, account details, debug overlays or ad creatives appear. The home screen avatar is a generic illustration.
- **The settings screenshot** is cropped to the top of the settings list, above the About section.
- **Diagrams** are original vector illustrations.
- Illustrations and branding shown in screenshots belong to the application and are covered by [RIGHTS.md](../RIGHTS.md).

## Wellness positioning

Resetta is a self-reflection and tracking tool. It does not diagnose, treat or replace professional advice. The app states on its baseline questionnaire and its "How it works" screen that it provides wellness insights, is not medical advice, and that users should consult a healthcare professional for medical concerns. This repository describes software engineering only and makes no claims about health outcomes, how well scores reflect a person's condition, or the effectiveness of recovery suggestions. The scoring is the app's own summary of what a user entered. This repository does not describe the scoring as a standardised or named instrument.

## Data handling: what can be said from the code

This is a summary, not a privacy policy. The in-app privacy policy and the store listings are the authoritative description for users.

- The app works without an account. Check-ins are written to local storage first.
- Where a user signs in, data is synced to a cloud database. The project defines rules that limit per-user data to its owner.
- Local storage relies on the platform's app sandbox. This showcase does not claim additional at-rest encryption.
- The free tier shows ads through a mediation SDK, and ad networks can use advertising identifiers. The app presents consent prompts, including the App Tracking Transparency prompt on iOS.
- The app uses product analytics (Firebase Analytics and a third-party product-analytics SDK) and crash reporting. A typed event catalogue and a parameter guard limit what the app sends from its own code. This repository makes no claim about whether analytics data can or cannot identify an individual.
- Subscription status is checked on the device against the store on every launch. This showcase does not claim server-side entitlement verification.
- The app requests no location, contacts, camera, microphone or health-platform access. Android permissions are limited to networking, notifications, exact alarms for reminders, boot-completed rescheduling, vibration and billing.
- A signed-in user can delete their account in the app; cloud data is removed before the sign-in account. A "reset app" action clears local data.

## Access

Source access for hiring or due-diligence purposes can be discussed at the owner's discretion.
