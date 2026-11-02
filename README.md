# Forms App policy site

This folder is a standalone static site. It needs no npm packages, build command, external fonts, or JavaScript. Pages use relative links so they work on GitHub Pages project sites and custom domains.

## Publish using a separate GitHub repository

1. Create a public repository for the policy site.
2. Copy **all contents of this folder**, including `.nojekyll`, into that repository's root and commit them. Do not place them inside another `github-pages` directory.
3. In the repository's **Settings → Pages**, choose **Deploy from a branch**, select your branch (usually `main`), and choose **/(root)**. Save.
4. Wait for deployment and open the URL shown by GitHub. No Flutter source or app configuration needs to be uploaded.

GitHub's branch publishing selector supports the repository root or `/docs`; it cannot select an arbitrary `/github-pages` folder in the Flutter repository. Copying this folder's contents to a dedicated repository root is the simplest deployment. If publishing from this repository instead, use a Pages Actions workflow that uploads this folder, or put the site's files under `/docs`.

The published site uses these URLs:

- [Home](https://firebirdappsdev.github.io/formsApp/)
- [Privacy policy](https://firebirdappsdev.github.io/formsApp/privacy-policy.html)
- [Terms of use](https://firebirdappsdev.github.io/formsApp/terms-of-use.html)

## Configure the app after publishing

`PRIVACY_POLICY_URL` and `TERMS_URL` are configured with these published URLs in the app's Dart defaults, sample configuration, and local `config/app_config.json`. `SUPPORT_EMAIL` is `firebirdapps.dev@gmail.com`. If moving the site, update the URLs in the app configuration and rebuild. Run these commands from the Flutter app repository:

```powershell
flutter build apk --debug --dart-define-from-file=config/app_config.json
# For a signed Play release, after configuring the upload key:
flutter build appbundle --release --dart-define-from-file=config/app_config.json
```

These are build-time values; changing the JSON does not update an already-installed app. Also configure the privacy URL and site links in Play Console and your Google OAuth consent configuration as appropriate. For a custom domain, follow GitHub's domain setup and use the resulting HTTPS URLs.

## Contents and maintenance

- [Home and support](index.html)
- [Privacy policy](privacy-policy.html)
- [Terms of use](terms-of-use.html)
- [Shared responsive styles](styles.css)
- [App icon assets](assets/): the header uses the same icon as the Android app, with 32px favicon and 180px Apple touch icon variants. Include this directory when copying or deploying the site. If the app icon changes, update the source copy and resized variants here.

The documents describe the current Android implementation: Google Forms/Drive access, response reading, device-based notifications, local preferences, exports, Firebase Analytics without advertising ID collection, and voluntarily submitted anonymous Firebase feedback. They do not claim a crash-reporting service, app-level analytics opt-out, or automatic feedback deletion.

Before publishing, confirm the effective date, Firebirdapps contact identity, and that your Firebase Analytics retention settings and actual support/feedback retention practices match the policy. Update the documents when features, providers, data use, or retention practices change. All three public pages contain the configured support email. There are no owner-value placeholders in the public pages or the app's policy URL configuration. Android release-signing values remain separate; see [remaining configuration](../docs/PLACEHOLDERS.md).

## Implementation review: 2 November 2026

The public policy and terms were checked against the current Android code. The review covered:

| Behavior | Implementation checked |
| --- | --- |
| Google access and disconnect | [Account controller](../lib/core/app_controller.dart) and [Google repository](../lib/data/forms_repository.dart): `forms.body`, `forms.responses.readonly`, and full `drive`; Google account sign-in with no separate Forms App/Firebase account. |
| Analytics | [Startup](../lib/main.dart) and [Android manifest](../android/app/src/main/AndroidManifest.xml): Firebase Analytics starts before sign-in; no dedicated crash-reporting SDK, app-level analytics opt-out, advertising ID collection, or account/form data attached to custom analytics events. |
| Feedback | [Feedback payload](../lib/core/app_feedback.dart), [rating dialog](../lib/ui/rating_dialog.dart), and [database rules](../database.rules.json): explicit unauthenticated submission with message/rating and app metadata; star-only selections do not create feedback records. Unauthenticated reads of the live feedback collection and a sample record path returned HTTP 401 during this review. No feedback was submitted or downloaded. |
| Local data and reminders | [Preferences](../lib/core/app_controller.dart), [demo storage](../lib/data/demo_repository.dart), and [rating reminders](../lib/core/rating_prompts.dart): local language/appearance/favorites, demo data, task counts and recent form IDs; unsaved live edits and preview answers remain in memory. |
| Notifications | [Native monitor](../android/app/src/main/kotlin/com/firebirdapps/forms_app/ResponseNotifications.kt), [worker](../android/app/src/main/kotlin/com/firebirdapps/forms_app/ResponseNotificationWorker.kt), [checkpoints](../android/app/src/main/kotlin/com/firebirdapps/forms_app/ResponseNotificationStore.kt), and [backup exclusions](../android/app/src/main/res/xml/data_extraction_rules.xml): periodic local polling of response IDs/timestamps, permission-dependent default monitoring, form-title alerts, and clearing the active account/checkpoints on sign-out. Per-account choices remain local. |
| Preview, media, and exports | [Editor](../lib/ui/editor_screen.dart), [preview](../lib/ui/preview_tab.dart), [response exports](../lib/ui/responses_tab.dart), and [sharing](../lib/ui/share_sheet.dart): finish/open-live saves edits without submitting preview answers; preview images contact their host; YouTube opens externally; clipboard links and print/share caches can remain after sign-out. |

Retention settings in the Firebase/Google Analytics consoles and the developer's actual handling of support messages cannot be determined from app code. The documents deliberately do not promise a specific retention period, automatic feedback expiry, deletion of anonymous analytics by email address, or deletion of exported/shared files on sign-out. Confirm those operational practices before publishing; no cloud settings or rules were changed during this review.
