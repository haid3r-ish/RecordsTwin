# RecordTwin — Notification System Documentation

## Overview
This document outlines the notification architecture for the RecordTwin app, based on the Master Architecture (F1-F16). It defines the specific types of notifications triggered by the system, whether they are dismissable, and how they are categorized in the UI to prevent user cognitive overload.

## Notification Categories (UI Tabs)

We use a simple, 3-tab segmented control for the notification center to keep the mental model clear for the target demographic:

| Tab | Purpose |
|---|---|
| **All** | The default combined view showing everything chronologically. |
| **Alerts 🔴** | Non-dismissable, action-required critical warnings (e.g., storage limits, sync conflicts). |
| **Activity** | Dismissable, informational background logs (e.g., AI text-washing complete, drafts saved). |

**UI Behavior Rule:** If the user is viewing the "Activity" tab and there are unread items in the "Alerts" category, a persistent red dot must be shown on the "Alerts" tab. Users should never be able to accidentally ignore a critical alert.

---

## Complete Notification Mapping

The following 11 notification types are extracted from the core features (F1-F16) and mapped to their appropriate category:

| # | Trigger / Event | Source Feature | Notification Modality | Category Tab | Dismissable? | Action Required |
|---|---|---|---|---|---|---|
| 1 | AI text-washing complete | F2/F15 | Push (FCM) | Activity | ✅ Yes | None |
| 2 | Background text sync complete | F8/F15 | Push (FCM) | Activity | ✅ Yes | None |
| 3 | Image sync complete (manual) | F8/F15 | Local | Activity | ✅ Yes | None |
| 4 | Storage pre-purge warning ("X docs will lose images") | F9/F15 | Local | **Alerts** | ❌ No | User must tap Confirm or Cancel |
| 5 | Pinned storage overflow ("Pinned images taking too much space") | F9 | Local | **Alerts** | ❌ No | User must unpin or increase limit |
| 6 | "X images not backed up" badge | F9/F13 | Local | **Alerts** | ❌ No | Tapping redirects to Sync Manager |
| 7 | 7-day un-synced image warning | F9/F15 | Local | **Alerts** | ❌ No | Forces user to sync or acknowledge risk |
| 8 | Draft saved (incremental auto-save) | F1 | Local | Activity | ✅ Yes | None |
| 9 | Urdu STT model not installed | F1/F16 | In-App (First launch) | **Alerts** | ✅ Yes | Disappears after setup completes |
| 10 | Sync conflict (409 Merge/Override required) | F8 | Push/Local | **Alerts** | ❌ No | User must resolve conflict via dialog |
| 11 | OCR completion (Phase 2 future feature) | F15 | Push (FCM) | Activity | ✅ Yes | None |

---

## Notification Delivery Modalities

- **Push Notifications (FCM):** Used for events that happen remotely on the server (e.g., AI Text-Washing, Background Sync).
- **Local Notifications:** Triggered by the device itself without server intervention (e.g., Cache limits, un-synced image warnings).
- **The "First 3 Days" Rule:** To avoid notification permission fatigue during onboarding, critical notifications appear purely as in-app UI badges (not push/system notifications) for the first 3 days after install.
