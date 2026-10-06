# Product Overview

## What Resetta is

Resetta is a mobile self-tracking app for work-related burnout, aimed at people with demanding jobs: knowledge workers, managers, founders, freelancers and students who also work. It is published for Android and iOS (see the store links in the [README](../README.md)).

The product scope is deliberately small. A user answers a one-off baseline of twelve questions, then three short questions each day. The app turns those answers into a single 0-100 score, a named state, and a view of which dimension is contributing most. From there it offers small recovery actions matched to that dimension.

It is a self-reflection and tracking tool. It does not diagnose, treat or replace professional advice, and the app carries its own not-medical-advice disclaimer on the baseline questionnaire and the "How it works" screen. This repository describes the software; it makes no claim about health outcomes.

## The three dimensions

The questions are grouped into three dimensions:

| Dimension | What the daily question asks about |
|---|---|
| Exhaustion | How physically and emotionally worn out the user feels from work |
| Disengagement | How detached or uncaring the user felt about work |
| Inefficacy | How capable and effective the user felt (scored in reverse, so a lower sense of capability raises the dimension) |

Each dimension has its own character illustration in the check-in. The exhaustion question, shown below, uses a tired figure whose body is a battery with one segment left.

<p align="center">
  <img src="../assets/screenshots/resetta-daily-check-in-question.webp" width="260" alt="Daily check-in question about exhaustion with a battery character and a 0 to 6 scale" />
</p>

The daily wording for each dimension is drawn from a small pool of variants, and the app avoids repeating the previous wording when the screen is reopened. That keeps the daily ritual from feeling like the same form every time.

## The user journey

1. **Splash and introduction.** A short intro, then a profile step that asks for a few work-profile details and for the time the user wants the daily reminder.
2. **Baseline.** Twelve statements across the three dimensions, answered on a 0 to 6 scale. The baseline can be retaken from settings; retaking does not delete history.
3. **First result.** Completing the baseline produces the first score and state, and seeds today's reading so the user is not asked to check in again immediately.
4. **Home.** A dashboard with a first-run coach-mark tour.
5. **Daily loop.** A reminder at the chosen time opens the three-question check-in. The result screen shows the state name, a short message and the three dimension values.
6. **Recovery.** From the result or from the home screen's mode switch, the user moves into Recovery Mode.

<table>
  <tr>
    <td align="center"><img src="../assets/screenshots/resetta-baseline-intro.webp" width="230" alt="Baseline introduction screen"/><br/><sub>Baseline intro</sub></td>
    <td align="center"><img src="../assets/screenshots/resetta-checkin-time-picker.webp" width="230" alt="Check-in time picker"/><br/><sub>Check-in time picker</sub></td>
    <td align="center"><img src="../assets/screenshots/resetta-score-result.webp" width="230" alt="Score result screen from a sample session"/><br/><sub>Score result screen (sample session)</sub></td>
  </tr>
</table>

## One score, four named states

The score is a weighted combination of three smoothed dimension values. It maps to one of four states, ordered from lightest to heaviest load:

**Restored, Taxed, Overloaded, Depleted.**

The state names are the app's labels for positions on its own scale; they are not descriptions of a condition or a clinical category. Names matter here. A number such as 63 means little at a glance, while "Overloaded" is something a user can say out loud. Each state also drives the visual identity of the home screen: the app ships a separate illustrated backdrop per state, and the Depleted backdrop in the screenshots below uses rain clouds and lightning.

The smoothing is the quiet part of the product. The score is not today's raw answers; it is today's answers blended with recent history. That design choice has visible consequences: a single exhausting day nudges the state rather than slamming it, and a single good day after a rough stretch does not reset the picture. The exact weights, smoothing strengths and band cut-offs are proprietary and are not published here.

The dominant driver is the dimension with the highest smoothed value. It determines which advice leads.

## Home: two modes

The home screen has a persistent two-segment control at the bottom that switches between **Burnout Mode** and **Recovery Mode**.

<table>
  <tr>
    <td align="center"><img src="../assets/screenshots/resetta-home-dashboard.webp" width="260" alt="Home dashboard in Burnout Mode"/><br/><sub><b>Burnout Mode</b><br/>State pill, daily insights, breakdown</sub></td>
    <td align="center"><img src="../assets/screenshots/resetta-burnout-breakdown.webp" width="260" alt="Burnout breakdown with status rings"/><br/><sub><b>Breakdown</b><br/>One status ring and advice line per dimension</sub></td>
    <td align="center"><img src="../assets/screenshots/resetta-recovery-insights.webp" width="260" alt="Recovery Mode: locked advice and checklist (premium)"/><br/><sub><b>Recovery Mode: locked advice and checklist (premium)</b><br/>Advice and checklist (premium)</sub></td>
  </tr>
</table>

**Burnout Mode** answers "where am I today?" It shows the score ring and state pill, a Daily Insights card with a timeline, and the burnout breakdown with one ring per dimension. Days without a check-in appear as dashed placeholders instead of solid markers, and the app labels estimated values as estimates rather than as measurements.

**Recovery Mode** answers "what do I do about it?" It leads with an insight card that names the dimensions that are elevated and offers a one-line suggestion for each. Below it sit dimension-specific recovery advice and a short checklist. A schedule matched to the dominant driver is also part of the recovery design: the codebase chooses between recovery, maintenance and protection schedule intensities from the score level and its recent direction, and filters a library of short actions by state, driver and duration. Actions the user dislikes or completed recently are excluded.

After several consecutive days in the heaviest state, the app can show a calm card that suggests small steps, such as a lighter day or talking to someone they trust. The card has a cooldown after dismissal. It is wording and software behaviour; it is not an escalation procedure.

## Skip-friendly by design

Life interrupts habits. The app treats missing days as normal:

- The question screen has a visible Skip control.
- Missing days are held or interpolated for display and never penalised.
- A past day can be backfilled; the smoothing chain is recomputed forward.
- The score label says when a value is an estimate or when the data is stale, and it can recommend retaking the baseline.

## Reminders and notifications

The user chooses a daily check-in time during setup and can change it later. Reminders are scheduled on the device and survive restarts. On top of the daily reminder, the app selects engagement messages on the device based on check-in state, for example a nudge after a missed day or a note when a pattern shifts. The wording of the daily reminder is split-tested per install.

## Free and premium

| Capability | Free | Premium |
|---|---|---|
| Baseline, daily check-in, score, state, dominant driver | Yes | Yes |
| Burnout breakdown per dimension | Yes | Yes |
| Recovery insight summary | Yes | Yes |
| Recovery advice by dimension | Locked preview | Yes |
| Recovery checklist | Locked preview | Yes |
| Journal notes | Teaser | Yes |
| Weekly insights | Locked | Yes |
| Longer history | Short window | Yes |
| Multi-day recovery schedule | Locked | Yes |
| Monthly PDF export | Locked | Yes |
| Ads | Shown | Suppressed |

<p align="center">
  <img src="../assets/screenshots/resetta-recovery-advice-locked.webp" width="260" alt="Recovery advice by dimension shown as locked premium cards" />
</p>

The locked cards in Recovery Mode preview the layout with blurred text and a lock badge so the value is legible before purchase. Pricing is shown by the stores and is not quoted here.

## Settings

Settings groups preferences, journal, a "how it works" explainer, subscriptions, privacy policy and terms, then a Data section: monthly PDF export, retake baseline and reset app. A reset clears local data and restarts onboarding. A signed-in user can also delete their account, which removes their cloud data before the sign-in account.

<p align="center">
  <img src="../assets/screenshots/resetta-settings.webp" width="260" alt="Settings list with data section" />
</p>

## Accounts

Resetta works without an account, and check-ins are stored on the device first. The codebase contains email, Google and Apple sign-in plus cloud sync for people who want continuity across devices. This portfolio does not state which store build exposes the sign-in entry point.

## Visual identity

The palette is warm paper beige with terracotta, orange, teal and amber accents. Headings use a serif face, body copy a light sans. Illustration carries a lot of the product: the smiling rings mark, character art per dimension, a state-specific weather backdrop on the home screen and per-dimension colour on the status rings. The goal is a calm tone for an app people open when they are already tired.

## What Resetta is not

- This repository describes software; it is not a diagnostic or medical tool description, and the app does not replace professional advice.
- Projected days use simple trend logic, not a trained model.
- This repository makes no claim that following the suggestions reduces burnout.
- The interface is in English only today.
