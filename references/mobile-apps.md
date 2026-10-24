# Mobile app rewrites

Read this when the system being replaced (or the replacement) is an iOS or Android app: native to a cross-platform framework, cross-platform to native, one framework to another, or a new app replacing an old one. On mobile you can't roll back what users have installed, and many identities are bound to the app's identifiers and signing keys. The rules below are about the platforms and stores, not any particular framework; where a framework matters, the text says what to look for in yours.

## Contents
1. Ship as an update, not a new app
2. On-device data and secure storage
3. Things bound to identifiers and keys
4. Old versions in the wild
5. Rollout and rollback on the stores
6. Over-the-air (OTA) updates
7. Checklist

## 1. Ship as an update, not a new app

Default: the new app is published as an **update to the existing store listing**, with the same iOS bundle identifier and Android application ID. That keeps installs, ratings, reviews, search ranking, and on-device data.

- **Android:** the update must be signed with the same app signing key (if Play App Signing holds it, you need the same upload key or a registered key reset). A different key means the update can't install over the old app.
- **iOS:** same bundle ID, same Apple Developer team. Moving teams is an app transfer, which has its own constraints (for example Sign in with Apple identifiers).
- **Version numbers** must go up: the new build's version code / build number must be above the old app's highest submitted build, including test tracks.
- In a cross-platform framework, set the bundle identifier, application ID and version fields explicitly in the app configuration, and point the build service at the existing signing credentials instead of letting it generate new ones.

Shipping as a separate new listing is sometimes right (different product, different legal entity). If so, plan the migration of users across: an in-app prompt in the old app, account-based restore in the new one, and the old listing kept up (or delisted) on purpose.

## 2. On-device data and secure storage

A new framework usually reads storage from different places than the old one. Users lose sessions, cached content, settings and drafts unless the first launch of the new app migrates them.

- **Inventory what the old app stores locally:** auth tokens, refresh tokens, user preferences, offline data and downloaded content, onboarding flags, feature-flag caches, analytics IDs. Note the exact keys and storage mechanism.
- **iOS Keychain:** items are readable by the new app if it has the same bundle, team and access group, **and** you query with the same service and account names the old app used. Secure-storage libraries use their own default service names, so set them explicitly or read the old items through a small native module.
- **iOS UserDefaults, files, Core Data, SQLite** and **Android SharedPreferences, files, Room, SQLite, Keystore-wrapped keys:** the new app's storage library won't see these automatically. Write a one-time first-launch migration (read old → write new → mark done; never delete old until the new copy is verified).
- **Android Keystore keys** created by the old app are only usable if the new app uses the same alias. Data encrypted with a key the new app can't access is gone. Find out which data that is before cutover.
- Test the upgrade path for real: install the **old store build**, sign in, generate state, then install the new build **over it** through a test track. A fresh install proves nothing about migration.

## 3. Things bound to identifiers and keys

Inventory each and confirm it survives:

- **Push:** APNs tokens are per bundle ID and environment; FCM tokens change if the Firebase project or sender changes. Plan re-registration on first launch and keep the backend able to send to both old and new tokens during coexistence.
- **Universal links and App Links:** the `apple-app-site-association` file (team ID + bundle ID) and `assetlinks.json` (package + SHA-256 signing certificate fingerprint). Custom URL schemes the old app registered (other apps, emails, QR codes and SMS may use them).
- **Sign in with Apple and Google Sign-In:** user identifiers are scoped to the team or OAuth client. Same team and client IDs = same users.
- **Deep links in the wild:** links in already-sent emails, SMS, printed material, partner sites. Every old path must still open something sensible.
- **Analytics and attribution:** event names, user ID mapping, install-attribution SDK IDs. Dashboards break silently if names change.
- **In-app purchases and subscriptions:** product IDs and receipt validation must carry over.
- **App Groups, widgets, extensions, watch apps:** group identifiers and shared containers.

## 4. Old versions in the wild

Some users never update. The backend will serve old app versions for a long time.

- Before cutover, make sure the **old** app has a **minimum-version gate** (remote config or an API response that tells it to show "please update"). If it doesn't, ship one in the old app now. It takes store review time plus weeks for adoption, so it goes on the critical path.
- Decide the support policy: which old versions keep working against which APIs, and for how long.
- Log the client version on every request so you can see who is still on old builds before turning anything off.

## 5. Rollout and rollback on the stores

- **No real rollback on the stores.** Once a build is installed you can only ship a newer build. "Rollback" means a newer build that restores old behaviour (or flags in the new app that switch features off), and that needs another review.
- Keep a **remote kill switch and feature flags** in the new app for risky features, so you can switch behaviour off without a store release.
- Use **phased release** on iOS and **staged rollout** on Android. Both can be paused; a paused rollout stops new users getting the build but doesn't uninstall it.
- Budget for **review time**, including the chance of rejection, especially for a big rewrite (new permissions, payments, login flows). Submit early and use expedited review only for emergencies.
- Watch crash-free sessions, ANRs, sign-in success and payment success per build during rollout. Set thresholds for pausing.

## 6. Over-the-air (OTA) updates

Some frameworks can push JavaScript or asset updates without a store release (for example Expo EAS Update, CodePush, or a framework's own OTA channel).

- OTA updates only change the interpreted layer. Native changes (new modules, permissions, SDK upgrades, build configuration) need a store build. Use a runtime-version mechanism so an update never reaches a binary it doesn't fit.
- OTA is the fastest way to fix a bad change after cutover. Make sure the channel setup and a tested "republish previous update" procedure exist **before** launch.
- Moving from a native app to an OTA-capable framework: the first build must come through the store. OTA only helps after that.

## 7. Checklist

- [ ] Same bundle ID / application ID / team / signing key confirmed, or a separate listing chosen on purpose
- [ ] Build number above every previously submitted build
- [ ] Local storage inventory with keys, plus a first-launch migration written and tested old-build → new-build
- [ ] Keychain service and access group, and Keystore aliases, match
- [ ] Push re-registration and dual-token sending during coexistence
- [ ] Universal links, App Links, URL schemes and old deep links verified
- [ ] Social sign-in identifiers unchanged
- [ ] Analytics event names mapped
- [ ] Minimum-version gate live in the old app
- [ ] Kill switches and feature flags in the new app
- [ ] Phased or staged rollout plan with pause thresholds
- [ ] OTA channel and rollback procedure tested (if the framework supports OTA)
