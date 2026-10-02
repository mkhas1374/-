# بستهٔ کامل گزارش ممیزی GameNexa

## مشخصات یکسان ممیزی

| مورد | مقدار |
|---|---|
| Repository | `mkhas1374/GameNexa-release-2026` |
| Branch | `main` |
| Commit SHA | `05348f2df8b1f6c30ee233071be13a899d969e25` |
| وضعیت کلی | **NOT RELEASE-READY** |
| نوع بررسی | Read-only؛ بدون تغییر، commit، push، merge یا deploy |

> این فایل شامل هر پنج خروجی تولیدشده است. ترتیب بخش‌ها مطابق خروجی‌های ممیزی است. برای جلوگیری از حذف evidence، متن هر خروجی به‌صورت کامل حفظ شده است؛ خروجی نهایی خود شامل appendices تخصصی نیز هست، بنابراین برخی مطالب تخصصی عمداً بیش از یک‌بار ظاهر می‌شوند.

## فهرست مطالب

1. [خروجی ۱ — گزارش نهایی و ارزیابی Release Readiness](#خروجی-۱--گزارش-نهایی-و-ارزیابی-release-readiness)
2. [خروجی ۲ — ممیزی Android، UI، State و Offline](#خروجی-۲--ممیزی-android-ui-state-و-offline)
3. [خروجی ۳ — ممیزی Backend، API، Database و Transactions](#خروجی-۳--ممیزی-backend-api-database-و-transactions)
4. [خروجی ۴ — ممیزی Security، Authentication و Authorization](#خروجی-۴--ممیزی-security-authentication-و-authorization)
5. [خروجی ۵ — ممیزی CI/CD، Runtime، Docker و Test Quality](#خروجی-۵--ممیزی-cicd-runtime-docker-و-test-quality)

---

## خروجی ۱ — گزارش نهایی و ارزیابی Release Readiness

> فایل منبع این بخش: `GameNexa-Audit-Report.md`

# GameNexa Release-Grade Audit Report

## 1. Executive Summary

**Result: NOT RELEASE-READY.** This audit covers the exact GitHub `main` revision identified below. Four independent evidence-based passes completed: Android, backend/API/database, security, and CI/CD/runtime. A fifth end-to-end business-flow pass and the reducer workflow were not completed because the session agent-credit budget was exhausted; those areas are explicitly marked **UNVERIFIED** below. No source files were modified, committed, pushed, merged, or deployed.

Confirmed evidence from completed passes includes:

- **1 release blocker**: release-validation failed before producing a signed APK/AAB.
- **Multiple High/P1 defects** affecting financial correctness, licensing, customer data handling, authentication/session security, API/runtime readiness, and release integrity.
- **Cross-layer contract defects**: Android customer VIP rules authentication mismatch; Android/customer passwords are persisted and exposed locally; Android GN purchase is interpreted as wallet top-up by the backend.
- **Financial/data-integrity defects**: guest shares can disappear from invoices; station-start prepayments are stored but not reconciled; VIP supersession can strand paid reservation funds; manual wallet approvals lack amount/ledger controls.
- **Security/control defects**: caller-controlled trial identity, non-revocable 24-hour JWTs, weak password policy, and customer account-state enumeration.
- **Operational gaps**: CI does not run backend/PostgreSQL/Docker/integration/deployment tests; fresh Compose startup does not initialize schema; health checks do not prove DB readiness; VPS/source/container identity is unverified.

The completed findings are not a claim that every runtime behavior was exercised. The local environment lacked Android SDK, Docker, and a listening PostgreSQL/API test stack; the exact GitHub debug run and release-validation run were inspected where available.

## 2. Exact GitHub Commit Tested

| Field | Value |
|---|---|
| Repository | `mkhas1374/GameNexa-release-2026` |
| Branch | `main` |
| Commit SHA | `05348f2df8b1f6c30ee233071be13a899d969e25` |
| Commit message | `Fix station conflicts, customer deletion, diagnostics, and notch layout` |
| Local checkout | `/home/ubuntu/GameNexa-release-2026` |
| Source modifications | None; clean working tree verified |
| Audit boundary | Repository state at this commit only; no assumptions about older/newer revisions |

## 3. Project Architecture and Inventory

- Android: one included Gradle module `:app`; 58 Kotlin files under the reviewed Android tree, Compose UI, Room database/DAO/entities, repository, Retrofit/OkHttp and direct self-hosted network client, ViewModels, alarms, localization/Jalali helpers, tests and resources.
- Backend: Node/Express service centered on `backend/server.js` and `backend/canonical_routes.js`, with `financialService.js`, `reservationService.js`, `bootstrap.js`, PostgreSQL `schema.sql`, Docker Compose, Dockerfile and nginx.
- API surface: backend specialist extracted **108 Express routes**; Android specialist extracted Retrofit declarations and direct/raw paths. The full route inventory is retained in the backend appendix below.
- Persistence: Android Room database `gamenet_manager_db`; backend PostgreSQL schema with manager/customer/session/reservation/payment/invoice/wallet/trial/subscription tables and constraints.
- CI/CD: three GitHub Actions workflows. Android debug and release-validation workflows primarily run Android JVM/static contract checks; no backend integration, Docker, migration or deployment pipeline is defined.
- Configuration/secrets: root/backend `.env.example`, Gradle properties, release signing secrets, JWT/database/CORS/bootstrap settings, Docker and nginx configuration.

## 4. Audit Method, Evidence Standard, and Limitations

A finding is included only when current source, configuration, test output, or recorded GitHub run evidence establishes the behavior. Every detailed specialist section preserves file paths, current line numbers, symbols/routes, snippets, impact, reproduction, fix and verification guidance. Static/mock tests are labelled as such and are not treated as runtime proof.

Not performed or not possible in this sandbox:

- No production mutation or destructive external testing.
- No running PostgreSQL-backed integration suite: local test run stopped at `ECONNREFUSED` on port 5432.
- No Docker execution: Docker CLI unavailable.
- No local Android Gradle execution: Android SDK location unavailable; the exact GitHub debug run did complete its configured JVM/build steps.
- No proof that the running VPS/container corresponds to this SHA; a public `/api/v1/time` HTTP 200 proves reachability only.
- The dedicated end-to-end business-flow reviewer did not complete after the workflow credit limit was reached. Therefore unexecuted scenarios are **UNVERIFIED**, not PASS or FAIL.

## 5. Confirmed Findings Index (deduplicated)

Severity uses the requested scale; priority is assigned from release/security/financial impact. IDs from specialist reports are retained for traceability. `AND-003` and `F-08` are one duplicated release-lint finding; `GN-BE-07` and `SEC-01` are one duplicated trial-identity finding.

| ID | Severity | Priority | Component | Confirmed issue |
|---|---|---|---|---|
| F-01 | CRITICAL | P0 | CI/CD / Release | Release validation failed at stable signing credentials; no signed APK/AAB was produced. |
| AND-001 | HIGH | P1 | Android / Security / Data Integrity | Customer passwords are stored in ordinary Room, displayed in plaintext and copied to clipboard. |
| AND-002 | HIGH | P1 | Android / API / Authentication | Customer VIP reservation-rules request uses manager/base headers while backend requires customer auth. |
| GN-BE-01 | HIGH | P1 | Backend / Authorization / Subscription | Post-login manager device binding bypasses the entitlement device cap. |
| GN-BE-02 | HIGH | P1 | Backend / Financial / Session | Guest payer shares are included in allocation but skipped when invoices are created. |
| GN-BE-03 | HIGH | P1 | Backend / Financial / Session | Start-time prepayments are accepted/stored but never applied to invoices or financial ledgers. |
| GN-BE-04 | HIGH | P1 | Backend / Financial / Reservation | VIP supersession can strand paid normal-reservation funds in a non-cancellable state. |
| GN-BE-05 | HIGH | P1 | Backend / Financial / Authorization | Generic manual wallet approval accepts arbitrary/negative amounts without ledger/audit control. |
| GN-BE-06 | HIGH | P1 | Android / Backend / GN | Android “Buy GN” requests are silently handled as wallet top-ups, not GN purchases. |
| F-02 | HIGH | P1 | CI/CD / Integration | CI omits backend tests, dependency install, Docker build, migrations, deployment and post-deploy checks. |
| F-03 | HIGH | P1 | CI/CD / Test Safety | Backend suite is integration/destructive, coupled to external DB/API and unsafe as written. |
| F-04 | HIGH | P1 | VPS/Runtime / Database | Compose cannot initialize a fresh database; schema and external volume are assumptions. |
| F-05 | HIGH | P1 | VPS/Runtime / Configuration | Environment templates omit required runtime variables. |
| F-06 | HIGH | P1 | VPS/Runtime / Database | Time endpoint can make API/nginx health green while DB/schema is unusable. |
| F-07 | HIGH | P1 | CI/CD / Android Release | Version metadata conflicts: artifacts use `2`/`1.0.1` while properties state `4`/`1.0.2`. |
| SEC-01 | MEDIUM | P2 | Security / Trial | Trial eligibility trusts caller-chosen identities and only blocks managers when a bearer is voluntarily supplied. |
| SEC-02 | MEDIUM | P2 | Authentication / Session | Logout and password changes do not revoke issued 24-hour JWTs. |
| SEC-03 | MEDIUM | P2 | Authentication | Customer passwords may be 4 characters; manager update accepts any non-empty password. |
| SEC-04 | LOW | P3 | Authentication / Privacy | Customer login returns distinguishable account-state errors enabling enumeration. |
| F-08 | MEDIUM | P2 | Android / CI | Release lint, dependency lint and failure gates are disabled; no lint task runs in CI. **Duplicate of AND-003.** |
| F-09 | MEDIUM | P2 | Android / CI | Instrumented Android testing is absent and the only instrumented assertion conflicts with application ID. |
| F-10 | MEDIUM | P2 | Android / Test Quality | JVM tests provide little behavioral assurance and include false-positive patterns. |
| F-11 | MEDIUM | P2 | Test Quality | Several Python suites are placeholders; a large legacy suite targets another architecture. |
| F-12 | MEDIUM | P2 | CI/CD / Integration | Contract/audit scripts are static-only and cannot verify VPS/runtime behavior. |
| F-13 | MEDIUM | P2 | Docker / Supply Chain | Docker build can copy host `node_modules` and is not reproducible. |
| F-14 | MEDIUM | P2 | CI/CD / Supply Chain | Dependency/reproducibility controls are incomplete. |
| F-15 | MEDIUM | P2 | CI/CD / Release | No release publication, deployment, artifact provenance or operational retention workflow exists. |
| F-16 | MEDIUM | P2 | VPS/Runtime | VPS/running API consistency with GitHub SHA remains unverified. |
| GN-BE-08 | MEDIUM | P2 | Backend / Reservation | Direct reservation bypasses disabled/non-reservable station policy. |

## 6. Release Blockers

### F-01 — Release validation cannot produce a signed artifact

- **Reason:** exact GitHub release-validation run `36964812496` failed at `Require stable release signing credentials`; signed build/upload steps were skipped.
- **Evidence:** `.github/workflows/android-release-validation.yml:66-92`; exact run metadata in `GameNexa-Audit-ci.md`, lines 19-24 and 50-75.
- **Impact:** no verified signed APK/AAB exists for this SHA; debug success is not release validation.
- **Required fix:** provision the four protected signing secrets, rerun the release workflow, verify certificate fingerprint and decoded artifact metadata.
- **Verification:** successful signed `assembleRelease`/`bundleRelease`, artifact upload and independent signature/version verification.

The High/P1 findings in the index are also release-blocking from a product-risk standpoint until fixed and regression-tested, especially financial underbilling/loss, credential exposure, authorization/entitlement bypasses, and missing database/runtime validation.

## 7. Complete Test Matrix

Only `PASS`, `FAIL`, `BLOCKED`, and `UNVERIFIED` are used. A PASS means the stated static/build/test action actually executed; it does not imply end-to-end correctness.

| ID | Feature | Scenario | Result | Severity | Priority | Evidence |
|---|---|---|---|---|---|---|
| T-01 | GitHub | Debug workflow for exact SHA | PASS | INFO | P3 | CI report §1, run `36964812531` |
| T-02 | GitHub | Release validation and signed artifact | FAIL | CRITICAL | P0 | CI report F-01, run `36964812496` |
| T-03 | Backend | `npm run check` syntax | PASS | INFO | P3 | CI/backend reports |
| T-04 | Backend | Static isolation/contract/deep audit scripts | PASS | INFO | P3 | `test_isolation_static.js`, `android_backend_contract_check.py`, `deep_release_audit.py` |
| T-05 | Backend | PostgreSQL integration suite | BLOCKED | HIGH | P1 | `npm test` stopped at DB `ECONNREFUSED` |
| T-06 | Android | Local Gradle compile/unit execution | BLOCKED | HIGH | P1 | Android SDK location unavailable locally; GitHub debug run executed configured task |
| T-07 | Docker/DB | Compose config/startup/schema migration | BLOCKED | HIGH | P1 | Docker CLI unavailable; source evidence F-04/F-06 |
| T-08 | Authentication | White-box JWT, bcrypt, role and middleware review | PASS | INFO | P3 | Security report §§ authentication and controls |
| T-09 | Manager isolation | Static/source review of tenant predicates | PASS | INFO | P3 | Security/backend reports; no confirmed Manager A/B IDOR |
| T-10 | Trial | Runtime one-device/replay scenarios | UNVERIFIED | MEDIUM | P2 | Requires disposable DB/runtime; SEC-01 is source-confirmed |
| T-11 | Start/session | All A–Z variants and duplicate/concurrent starts | UNVERIFIED | HIGH | P1 | Dedicated flow pass did not complete; source evidence in backend report |
| T-12 | Settlement | Online/offline/duplicate/wrong-owner settlement | UNVERIFIED | HIGH | P1 | Requires DB/API runtime |
| T-13 | Reservation | Double-booking/payment/cancellation races | UNVERIFIED | HIGH | P1 | Requires DB/API runtime |
| T-14 | Buffet/GN/LP | Full financial reconciliation and offline sync | UNVERIFIED | HIGH | P1 | Requires DB/API runtime; GN mismatch is source-confirmed GN-BE-06 |
| T-15 | Customer | CRUD/archive/sync/login after deletion | UNVERIFIED | HIGH | P1 | Full flow runtime not executed |
| T-16 | VPS | SHA = source = container = running API | UNVERIFIED | MEDIUM | P2 | Public time endpoint proves reachability only; CI F-16 |

## 8. Domain Audit Sections

The following appendices are the complete specialist reports, preserved verbatim so developers have the exact inventories, line references, snippets, tests, reproduction steps, fixes and verification plans requested. They are the authoritative detailed evidence for each completed domain.

### 8.1 Android Audit

# GameNexa Android Code Audit

## Audit identity and scope

| Field | Value |
|---|---|
| Audit ID | `GNX-ANDROID-2026-10-02-05348f2` |
| Repository | `/home/ubuntu/GameNexa-release-2026` |
| Requested ref / observed HEAD | `05348f2df8b1f6c30ee233071be13a899d969e25` / `05348f2df8b1f6c30ee233071be13a899d969e25` |
| Branch | `main` |
| Scope | **Android only**: Gradle/module config, manifest/resources, all Kotlin under `app/`, Room/DAO/entities, repository, networking/DTOs, ViewModels, Compose UI/actions, auth/token storage, offline/sync/retry/error/lifecycle, time/Jalali/locale, business defaults and tests. Backend was read only where necessary to prove an Android contract. |
| Source modifications | **None.** Working tree at the time of report generation: `(clean)`. |
| Method | Static source/contract audit. Every `app/**/*.kt` file was enumerated (58 files, including tests and the unconfigured applet source); targeted manual control-flow review was done for state, auth, persistence, network, billing, reservations, lifecycle, and UI actions. |
| Finding standard | Only evidence-backed, reproducible issues below are counted. Potential concerns that were not proven are separated under **Unverified items**. |

> **Important limitation:** Gradle compilation and unit tests were started but could not execute because this sandbox has no Android SDK location. There is no claim that the Android application builds or that tests pass.

## Executive summary

**Three confirmed findings** were identified:

1. **AND-001 / High / P1** — customer passwords are deliberately retained in a standard Room SQLite table, rendered in plaintext to staff and copied to the system clipboard.
2. **AND-002 / High / P1** — the customer VIP reservation-rules call uses manager/base headers rather than the customer bearer header; the backend route explicitly requires customer authentication. The customer full-hall screen therefore receives no rules for an ordinary customer session and has no selectable VIP duration.
3. **AND-003 / Medium / P2** — release lint checking, lint failure, and dependency lint checking are explicitly disabled, removing a release-time quality/security gate.

The client otherwise has several positive controls evidenced by code: `allowBackup="false"`, cleartext disabled, non-exported alarm receiver, AES-GCM/Android Keystore used for encrypted settings/tokens, customer session restoration uses the customer header, customer login refuses an offline fallback, network logging avoids retaining raw bodies, and customer reservation mutation/cancellation includes idempotency keys. These observations do **not** offset the confirmed issues above.

## Environment, build, and manifest inventory

### Gradle/module facts

- `settings.gradle.kts:22-23` names the project `GameNexa` and includes only `:app`; `app/applet/.../SubscriptionActivationScreen.kt` is present in the tree but no applet Gradle descriptor was found and it is **not** an included Android module.
- `app/build.gradle.kts:9-29`: namespace `com.example`; app id `com.MinmKhas.studio.GameNexa.wrtx`; compile SDK 36.1; min SDK 24; target SDK 36; version `2` / `1.0.1`; Java 11.
- Compose and KSP are enabled (`app/build.gradle.kts:1-6`, `95-99`); Room, Retrofit, Moshi, OkHttp, coroutines and test libraries are declared (`112-167`).
- Release signing credentials are loaded from properties/environment and release task fails if absent (`37-74`). Release minification is **off** (`79-85`).
- The configured test runner is `androidx.test.runner.AndroidJUnitRunner` (`28`), and unit tests include Android resources (`99`).

### Manifest, component, transport, and backup facts

`app/src/main/AndroidManifest.xml` was reviewed in full.

| Area | Confirmed implementation / evidence |
|---|---|
| Permissions | `INTERNET`, network-state, vibration, post notifications, exact alarms, contacts, boot complete and legacy external storage permissions are requested (`4-12`). |
| Backup/transport | Application sets `android:allowBackup="false"`, `usesCleartextTraffic="false"`, network config, data-extraction rules and backup rules (`14-25`). Network config independently sets `<base-config cleartextTrafficPermitted="false">` and system trust anchors (`res/xml/network_security_config.xml:2-8`). |
| Exposed entry point | `MainActivity` is exported (`27-32`) for launcher, verified HTTPS `/pay/callback` and browsable `gamenet://payment-callback` / `gamenet://verify` deep links (`33-50`). |
| Receiver | `AlarmReceiver` is declared `exported="false"` (`51-54`). It uses `goAsync()` then finishes its pending result in `finally` (`AlarmReceiver.kt:26-56`). |
| Resource caveat | `res/xml/backup_rules.xml` includes shared preferences (`2-4`), while the manifest disables backups. The effective behavior from the manifest is backup disabled; no finding is asserted from the unused-looking rule. |

## Source coverage inventory

The counts below come from a read-only enumeration of every Kotlin source under `app/`. “Declarations”, “Compose”, and “onClick” are simple inventory counts—not test coverage.

| Path | Lines | Package | Declarations | Compose | `onClick` |
|---|---:|---|---:|---:|---:|
| `app/applet/app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` | 335 | `com.example.ui` | 2 | 1 | 9 |
| `app/src/androidTest/java/com/example/ExampleInstrumentedTest.kt` | 22 | `com.example` | 3 | 0 | 0 |
| `app/src/main/java/com/example/MainActivity.kt` | 1035 | `com.example` | 14 | 8 | 18 |
| `app/src/main/java/com/example/data/AppDatabase.kt` | 182 | `com.example.data` | 50 | 0 | 0 |
| `app/src/main/java/com/example/data/CryptoManager.kt` | 100 | `com.example.data` | 4 | 0 | 0 |
| `app/src/main/java/com/example/data/Dao.kt` | 294 | `com.example.data` | 96 | 0 | 0 |
| `app/src/main/java/com/example/data/Entities.kt` | 363 | `com.example.data` | 32 | 0 | 0 |
| `app/src/main/java/com/example/data/GameNetRepository.kt` | 1086 | `com.example.data` | 86 | 0 | 0 |
| `app/src/main/java/com/example/data/LicenseCacheEntity.kt` | 31 | `com.example.data` | 5 | 0 | 0 |
| `app/src/main/java/com/example/data/PaymentModels.kt` | 23 | `com.example.data` | 3 | 0 | 0 |
| `app/src/main/java/com/example/data/SecurityUtils.kt` | 85 | `com.example.data` | 4 | 0 | 0 |
| `app/src/main/java/com/example/data/network/GameNetApi.kt` | 992 | `com.example.data.network` | 104 | 0 | 0 |
| `app/src/main/java/com/example/data/network/NetworkLogger.kt` | 188 | `com.example.data.network` | 9 | 0 | 0 |
| `app/src/main/java/com/example/data/network/RetryInterceptor.kt` | 57 | `com.example.data.network` | 2 | 0 | 0 |
| `app/src/main/java/com/example/data/network/SelfHostedManager.kt` | 1985 | `com.example.data.network` | 81 | 0 | 0 |
| `app/src/main/java/com/example/receiver/AlarmReceiver.kt` | 175 | `com.example.receiver` | 7 | 0 | 0 |
| `app/src/main/java/com/example/ui/AdminLoginScreen.kt` | 284 | `com.example.ui` | 2 | 1 | 2 |
| `app/src/main/java/com/example/ui/AdminNotificationComponents.kt` | 696 | `com.example.ui` | 4 | 4 | 14 |
| `app/src/main/java/com/example/ui/AuthDialog.kt` | 406 | `com.example.ui` | 1 | 1 | 7 |
| `app/src/main/java/com/example/ui/BehaviorDialog.kt` | 188 | `com.example.ui` | 1 | 1 | 5 |
| `app/src/main/java/com/example/ui/ContactUsScreen.kt` | 375 | `com.example.ui` | 6 | 2 | 4 |
| `app/src/main/java/com/example/ui/CustomerAppContent.kt` | 1793 | `com.example.ui` | 6 | 4 | 20 |
| `app/src/main/java/com/example/ui/CustomerClubScreen.kt` | 3618 | `com.example.ui` | 19 | 14 | 72 |
| `app/src/main/java/com/example/ui/CustomerDialogs.kt` | 1234 | `com.example.ui` | 7 | 6 | 27 |
| `app/src/main/java/com/example/ui/CustomerFullHallTab.kt` | 207 | `com.example.ui` | 1 | 1 | 4 |
| `app/src/main/java/com/example/ui/CustomerOnlinePaymentTab.kt` | 1028 | `com.example.ui` | 7 | 4 | 17 |
| `app/src/main/java/com/example/ui/CustomerTabs.kt` | 1651 | `com.example.ui` | 6 | 6 | 17 |
| `app/src/main/java/com/example/ui/CustomersReservationsScreen.kt` | 3506 | `com.example.ui` | 16 | 12 | 63 |
| `app/src/main/java/com/example/ui/FirstLaunchGuide.kt` | 319 | `com.example.ui` | 6 | 2 | 8 |
| `app/src/main/java/com/example/ui/GameNetViewModel.kt` | 6576 | `com.example.ui` | 236 | 0 | 0 |
| `app/src/main/java/com/example/ui/GameNexaWelcomeScreen.kt` | 638 | `com.example.ui` | 1 | 1 | 8 |
| `app/src/main/java/com/example/ui/HallWeatherEffects.kt` | 16 | `com.example.ui` | 1 | 2 | 0 |
| `app/src/main/java/com/example/ui/InvoiceCard.kt` | 179 | `com.example.ui` | 1 | 1 | 0 |
| `app/src/main/java/com/example/ui/LicenseViewModel.kt` | 126 | `com.example.ui` | 7 | 0 | 0 |
| `app/src/main/java/com/example/ui/Localization.kt` | 190 | `com.example.ui` | 2 | 0 | 0 |
| `app/src/main/java/com/example/ui/MainScreen.kt` | 2088 | `com.example.ui` | 9 | 5 | 21 |
| `app/src/main/java/com/example/ui/ManagerReservationsScreen.kt` | 200 | `com.example.ui` | 3 | 2 | 4 |
| `app/src/main/java/com/example/ui/NetworkDiagnosticsDialog.kt` | 674 | `com.example.ui` | 5 | 4 | 7 |
| `app/src/main/java/com/example/ui/ReservationSettingsScreen.kt` | 413 | `com.example.ui` | 16 | 5 | 6 |
| `app/src/main/java/com/example/ui/ServerConnectionTestDialog.kt` | 220 | `com.example.ui` | 3 | 2 | 4 |
| `app/src/main/java/com/example/ui/SettingsScreen.kt` | 3447 | `com.example.ui` | 18 | 14 | 72 |
| `app/src/main/java/com/example/ui/StatisticsScreen.kt` | 370 | `com.example.ui` | 2 | 2 | 1 |
| `app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` | 1115 | `com.example.ui` | 3 | 2 | 11 |
| `app/src/main/java/com/example/ui/SubscriptionLockScreen.kt` | 309 | `com.example.ui` | 1 | 1 | 7 |
| `app/src/main/java/com/example/ui/UnifiedEntryScreen.kt` | 626 | `com.example.ui` | 2 | 1 | 11 |
| `app/src/main/java/com/example/ui/theme/Color.kt` | 11 | `com.example.ui.theme` | 0 | 0 | 0 |
| `app/src/main/java/com/example/ui/theme/Theme.kt` | 76 | `com.example.ui.theme` | 1 | 2 | 0 |
| `app/src/main/java/com/example/ui/theme/Type.kt` | 36 | `com.example.ui.theme` | 0 | 0 | 0 |
| `app/src/main/java/com/example/util/ExactBilling.kt` | 17 | `com.example.util` | 3 | 0 | 0 |
| `app/src/main/java/com/example/util/JalaliCalendarHelper.kt` | 144 | `com.example.util` | 11 | 0 | 0 |
| `app/src/main/java/com/example/util/LocaleHelper.kt` | 30 | `com.example.util` | 3 | 0 | 0 |
| `app/src/main/java/com/example/util/VpnDetector.kt` | 43 | `com.example.util` | 5 | 0 | 0 |
| `app/src/main/java/com/example/utils/NumberConverter.kt` | 25 | `com.example.utils` | 3 | 0 | 0 |
| `app/src/test/java/com/example/ExampleRobolectricTest.kt` | 21 | `com.example` | 3 | 0 | 0 |
| `app/src/test/java/com/example/ExampleUnitTest.kt` | 16 | `com.example` | 2 | 0 | 0 |
| `app/src/test/java/com/example/GreetingScreenshotTest.kt` | 32 | `com.example` | 3 | 0 | 0 |
| `app/src/test/java/com/example/MainActivityTest.kt` | 31 | `com.example` | 4 | 0 | 0 |
| `app/src/test/java/com/example/TrialAndSubscriptionTest.kt` | 109 | `com.example` | 7 | 0 | 0 |

## Architecture and behavior inventory (proven facts)

### Persistence and repository

- `AppDatabase` registers the Room tables (including `Customer`, transactions, reservations, license cache and GN-related tables) and builds the ordinary Room database named `gamenet_manager_db` (`AppDatabase.kt:16-49`, `162-176`). No SQLCipher/encrypted Room setup is present in that builder.
- The `Customer` Room entity includes `val password: String = ""` (`Entities.kt:177-199`). Migration 8→9 creates that as `password TEXT NOT NULL DEFAULT ''` (`AppDatabase.kt:69-73`).
- Repository/database CRUD, default data, synchronization, station/session records, order data, settings, customer data and reservations were inspected in `GameNetRepository.kt`, `Dao.kt`, `Entities.kt`, `AppDatabase.kt`, `LicenseCacheEntity.kt`, and `PaymentModels.kt`.
- `CryptoManager` uses an Android Keystore AES-GCM key (`CryptoManager.kt:12-38`) and returns a Base64 `iv:ciphertext` pair (`54-72`). Its documented exception path throws rather than stores a plaintext fallback (`67-72`, `90-98`). ViewModel customer tokens are saved via encrypted settings after login (`GameNetViewModel.kt:1010-1014`). This is distinct from the unencrypted Room `Customer.password` field in AND-001.

### Network, auth, retry, error handling, and sync

- Base manager headers add `X-Manager-ID` and, only from `NetworkClient.authToken`, `Authorization: Bearer …` (`SelfHostedManager.kt:228-240`). Customer headers instead use only `NetworkClient.customerAuthToken` (`243-250`). This distinction proves AND-002.
- Customer login posts phone, manager id and password (`SelfHostedManager.kt:398-405`), then fetches `/api/v1/customer/profile`; it copies the submitted password into the parsed `Customer` (`436`) and `GameNetViewModel.loginCustomer` inserts that record into Room (`978-1014`). Registration also copies password into its parsed customer (`461-485`).
- Customer login is server-authoritative: a failed online result leads to the user-visible error **“offline login is not allowed”** and returns (`1029-1033`).
- Retry interceptor retries a request up to 3 times (5 for paths containing `sync`), uses 2-second exponential backoff and retries 5xx/408/429 plus `IOException`; it returns 4xx other than 408/429 without retry (`RetryInterceptor.kt:9-55`). It performs `Thread.sleep` in OkHttp interception (`19-45`). This is recorded behavior, not itself a defect claim.
- Network logger keeps at most 150 in-memory records (`NetworkLogger.kt:40-64`) and deliberately does not retain raw response bodies; for failed responses it extracts a bounded code/error string using `peekBody` (`139-162`).
- The customer VIP mutation and cancellation use the customer header plus idempotency key (`SelfHostedManager.kt:1410-1442`). The rules request is the exception captured in AND-002.

### Lifecycle, alarms, time, locale, UI, and business values

- Alarm receiver uses `goAsync`, performs DB work on `Dispatchers.IO`, and calls `pendingResult.finish()` in `finally` (`AlarmReceiver.kt:26-57`). Reservation notifications intentionally call vibration twice (`42-49`); this is behavior, not classified as a defect.
- Jalali display is calculated in `Asia/Tehran`, formats date/time with `Locale.US` digits, and treats `Long.MAX_VALUE`-like expiry as “active subscription” (`JalaliCalendarHelper.kt:10-34`, `126-141`). `CustomerFullHallTab` also presents reservation dates in Asia/Tehran, but explicitly with `SimpleDateFormat("yyyy/MM/dd - HH:mm", Locale.US)` (`CustomerFullHallTab.kt:72-75`).
- `LocaleHelper` changes the default locale/configuration and calls deprecated `resources.updateConfiguration` with suppression, then recreates the Activity in `applyLocale` (`LocaleHelper.kt:8-29`). This was observed; runtime locale behavior was not executed.
- Hard-coded seeds/defaults exist, including console prices (`GameNetRepository.kt:179-182`, `554-557`, `615-618`), product prices (`206-216` etc.), GN settings (`595-597`), and fallback GN price per 10 (`SelfHostedManager.kt:213`). They are reported as implementation facts, not defects; whether they are approved business defaults cannot be determined from code alone.
- Compose actions across all screens were enumerated. Financial/customer actions reviewed include manual GN adjustment, transfer, purchase, policy save, customer deletion, payment reports and reservation submission. UI controls often have role/feature gating, while server routes remain the authoritative boundary; no bypass was claimed without a complete runnable role scenario.

## Extracted API calls

### Retrofit declarations

| Kind | Android declaration | Endpoint | Evidence |
|---|---|---|---|
| Retrofit GET | `suspend fun getServerTime(): ServerClockResponse` | `/api/v1/time` | `GameNetApi.kt:41` |
| Retrofit GET | `suspend fun healthCheck(): retrofit2.Response<ServerClockResponse>` | `/api/v1/time` | `GameNetApi.kt:44` |
| Retrofit GET | `suspend fun getSuperManagers(): List<AdminManagerDto>` | `/api/v1/super-manager/managers` | `GameNetApi.kt:47` |
| Retrofit POST | `suspend fun addManagerContract(@Body request: Map<String, Any>): okhttp3.ResponseBody` | `/api/v1/super-manager/add-manager` | `GameNetApi.kt:50` |
| Retrofit POST | `suspend fun recoverSuperManager(@Body body: Map<String, String>): okhttp3.ResponseBody` | `/api/v1/super-manager/recover` | `GameNetApi.kt:54` |
| Retrofit POST | `suspend fun addManagerCustomer(@Body body: Map<String, Any>): okhttp3.ResponseBody` | `/api/v1/manager/customers` | `GameNetApi.kt:57` |
| Retrofit POST | `suspend fun checkTrialStatus(@Body body: CheckTrialRequest): CheckTrialResponse` | `/api/v1/trial/status` | `GameNetApi.kt:61` |
| Retrofit GET | `suspend fun getAllDeviceTrials(): okhttp3.ResponseBody` | `/api/v1/super-manager/trial-devices` | `GameNetApi.kt:64` |
| Retrofit DELETE | `suspend fun deleteDeviceTrial(@Path("id") id: String): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/trial-devices/{id}` | `GameNetApi.kt:67` |
| Retrofit POST | `suspend fun extendDeviceTrial(@Path("id") id: String): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/trial-devices/{id}/extend` | `GameNetApi.kt:70` |
| Retrofit POST | `suspend fun createSuperManager(@Body request: CreateManagerRequestDto): okhttp3.ResponseBody` | `/api/v1/super-manager/managers` | `GameNetApi.kt:75` |
| Retrofit PUT | `suspend fun updateManagerStatus(@Path("id") id: String, @Body request: UpdateManagerRequestDto): retrofit2.Response<okht` | `/api/v1/super-manager/managers/{id}` | `GameNetApi.kt:78` |
| Retrofit PUT | `suspend fun updateManager(@Path("id") id: String, @Body request: EditManagerRequestDto): retrofit2.Response<okhttp3.Resp` | `/api/v1/super-manager/managers/{id}` | `GameNetApi.kt:81` |
| Retrofit DELETE | `suspend fun deleteManager(@Path("id") id: String): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/managers/{id}` | `GameNetApi.kt:84` |
| Retrofit POST | `suspend fun createSuperManagerAlt(@Body request: CreateManagerRequestDto): okhttp3.ResponseBody` | `/api/v1/super-manager/create-manager` | `GameNetApi.kt:88` |
| Retrofit POST | `suspend fun createManager(@Body request: CreateManagerRequestDto): okhttp3.ResponseBody` | `/api/v1/super-manager/add-manager` | `GameNetApi.kt:91` |
| Retrofit GET | `suspend fun getStations(): List<StationState>` | `/api/v1/manager/stations` | `GameNetApi.kt:98` |
| Retrofit POST | `suspend fun saveStation(@Body state: StationState): StationState` | `/api/v1/manager/stations` | `GameNetApi.kt:101` |
| Retrofit GET | `suspend fun getConsoleTypes(): List<ConsoleType>` | `/api/v1/manager/console-types` | `GameNetApi.kt:108` |
| Retrofit POST | `suspend fun saveConsoleType(@Body console: ConsoleType): ConsoleType` | `/api/v1/manager/console-types` | `GameNetApi.kt:111` |
| Retrofit DELETE | `suspend fun deleteConsoleType(@Path("name") name: String): Response<Unit>` | `/api/v1/manager/console-types/{name}` | `GameNetApi.kt:114` |
| Retrofit GET | `suspend fun getProducts(): List<Product>` | `/api/v1/manager/products` | `GameNetApi.kt:117` |
| Retrofit POST | `suspend fun saveProduct(@Body product: Product): Product` | `/api/v1/manager/products` | `GameNetApi.kt:120` |
| Retrofit DELETE | `suspend fun deleteProduct(@Path("name") name: String): Response<Unit>` | `/api/v1/manager/products/{name}` | `GameNetApi.kt:123` |
| Retrofit GET | `suspend fun getOrders(@Path("stationId") stationId: Int): List<StationOrder>` | `/api/v1/manager/orders/{stationId}` | `GameNetApi.kt:126` |
| Retrofit POST | `suspend fun saveOrder(@Body order: StationOrder): StationOrder` | `/api/v1/manager/orders` | `GameNetApi.kt:129` |
| Retrofit DELETE | `suspend fun deleteOrder(@Path("id") id: String): Response<Unit>` | `/api/v1/manager/orders/{id}` | `GameNetApi.kt:132` |
| Retrofit DELETE | `suspend fun clearOrders(@Path("stationId") stationId: Int): Response<Unit>` | `/api/v1/manager/orders/station/{stationId}` | `GameNetApi.kt:135` |
| Retrofit GET | `suspend fun getSessionHistory(): List<SessionHistory>` | `/api/v1/manager/session-history` | `GameNetApi.kt:138` |
| Retrofit POST | `suspend fun addSessionHistory(@Body history: SessionHistory): SessionHistory` | `/api/v1/manager/session-history` | `GameNetApi.kt:141` |
| Retrofit DELETE | `suspend fun clearSessionHistory(): Response<Unit>` | `/api/v1/manager/session-history` | `GameNetApi.kt:144` |
| Retrofit GET | `suspend fun getCustomers(): List<Customer>` | `/api/v1/manager/customers` | `GameNetApi.kt:147` |
| Retrofit POST | `suspend fun saveCustomer(@Body customer: Customer): Customer` | `/api/v1/manager/customers` | `GameNetApi.kt:150` |
| Retrofit DELETE | `suspend fun deleteCustomer(@Path("id") id: Long): Response<Unit>` | `/api/v1/manager/customers/{id}` | `GameNetApi.kt:153` |
| Retrofit POST | `suspend fun deleteCustomerBatch(@Body body: Map<String, List<Long>>): Response<Map<String, Any>>` | `/api/v1/manager/customers/delete-batch` | `GameNetApi.kt:156` |
| Retrofit GET | `suspend fun getReservations(): List<Reservation>` | `/api/v1/manager/reservations` | `GameNetApi.kt:159` |
| Retrofit DELETE | `suspend fun deleteReservation(@Path("id") id: Long): Response<Unit>` | `/api/v1/manager/reservations/{id}` | `GameNetApi.kt:163` |
| Retrofit GET | `suspend fun getSetting(@Path("key") key: String): Map<String, String>` | `/api/v1/manager/settings/{key}` | `GameNetApi.kt:166` |
| Retrofit POST | `suspend fun saveSetting(@Body body: Map<String, String>): Response<Unit>` | `/api/v1/manager/settings` | `GameNetApi.kt:169` |
| Retrofit POST | `suspend fun registerUser(@Body body: UserRegisterRequest): UserAuthResponse` | `/api/auth/customer/register` | `GameNetApi.kt:174` |
| Retrofit POST | `suspend fun loginUser(@Body body: UserLoginRequest): UserAuthResponse` | `/api/auth/manager/login` | `GameNetApi.kt:177` |
| Retrofit GET | `suspend fun pingSuperManager(): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/ping` | `GameNetApi.kt:181` |
| Retrofit POST | `suspend fun pingSuperManagerPost(): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/ping` | `GameNetApi.kt:184` |
| Retrofit GET | `suspend fun checkAuth(): Map<String, Any>` | `/api/v1/auth/check` | `GameNetApi.kt:192` |
| Retrofit POST | `suspend fun addUserDevice(@Body body: Map<String, Any>): Map<String, Any>` | `/api/v1/manager/device` | `GameNetApi.kt:199` |
| Retrofit GET | `suspend fun checkSubscriptionStatus(` | `/api/v1/subscriptions/check` | `GameNetApi.kt:216` |
| Retrofit GET | `suspend fun checkSubscription(` | `/api/v1/subscriptions/check` | `GameNetApi.kt:223` |
| Retrofit POST | `suspend fun checkLicenseStatus(@Body body: LicenseCheckRequest): LicenseCheckResponse` | `/api/v1/subscriptions/status` | `GameNetApi.kt:230` |
| Retrofit POST | `suspend fun activateLicense(@Body body: LicenseActivateRequest): LicenseCheckResponse` | `/api/v1/subscriptions/activate` | `GameNetApi.kt:233` |
| Retrofit POST | `suspend fun startFreeTrial(@Body body: TrialStartRequest): TrialStartResponse` | `/api/v1/trial/start` | `GameNetApi.kt:236` |
| Retrofit POST | `suspend fun buySubscription(@Body body: LicenseBuyRequest): LicenseBuyResponse` | `/api/v1/subscriptions/buy` | `GameNetApi.kt:239` |
| Retrofit POST | `suspend fun setLicensePassword(@Body body: SetPasswordRequest): Boolean` | `/api/v1/subscriptions/set-password` | `GameNetApi.kt:242` |
| Retrofit POST | `suspend fun validateCoupon(@Body params: Map<String, String>): Map<String, Any>` | `/api/v1/coupons/validate` | `GameNetApi.kt:245` |
| Retrofit GET | `suspend fun getSubscriptionPlans(): List<SubscriptionPlanDto>` | `/api/v1/plans` | `GameNetApi.kt:248` |
| Retrofit GET | `suspend fun getSubscriptionPlansAdmin(): SubscriptionPlansAdminResponse` | `/api/v1/super-manager/subscription-plans` | `GameNetApi.kt:251` |
| Retrofit PUT | `suspend fun updateSubscriptionPlansAdmin(@Body body: Map<String, Any>): SubscriptionPlansAdminResponse` | `/api/v1/super-manager/subscription-plans` | `GameNetApi.kt:254` |

### Direct/self-hosted call strings

- `/api/v1/manager/club/point-logs` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:265`
- `/api/v1/manager/club/point-logs/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:280`
- `/api/v1/customer/profile` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:364`
- `/api/auth/customer/login` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:405`
- `/api/v1/customer/profile` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:425`
- `/api/auth/customer/register` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:471`
- `/api/v1/manager/customers` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:533`
- `/api/v1/manager/customers/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:543`
- `/api/v1/manager/customers/delete-batch` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:557`
- `/api/v1/manager/customers` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:581`
- `/api/v1/manager/live-stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:595`
- `/api/v1/manager/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:663`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:778`
- `/api/v1/manager/manual-payment-requests/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:800`
- `/api/v1/manager/configuration` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:827`
- `/api/v1/manager/configuration` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:859`
- `/api/v1/manager/configuration` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:877`
- `/api/v1/manager/settings/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:897`
- `/api/v1/manager/announcements` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:930`
- `/api/v1/manager/audit-logs` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:965`
- `/api/v1/manager/club/ledger` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:995`
- `/api/v1/customer/club/ledger` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1011`
- `/api/v1/auth/check` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1053`
- `/api/v1/manager/diagnostics` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1077`
- `/api/v1/manager/customer-transactions` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1172`
- `/api/v1/customer/transactions` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1187`
- `/api/v1/manager/stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1239`
- `/api/v1/manager/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1270`
- `/api/v1/manager/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1292`
- `/api/v1/customer/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1326`
- `/api/v1/customer/stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1358`
- `/api/v1/customer/reservations/rules` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1375`
- `/api/v1/customer/reservations/pricing-preview` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1398`
- `/api/v1/customer/reservations/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1416`
- `/api/v1/customer/reservations/atomic` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1441`
- `/api/v1/manager/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1469`
- `/api/v1/manager/reservations/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1496`
- `/api/v1/manager/reservation-payments/pending` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1510`
- `/api/v1/manager/reservation-payments/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1528`
- `/api/v1/manager/reservation-payments/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1545`
- `/api/v1/manager/reservations/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1561`
- `/api/v1/customer/club/transfer` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1598`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1646`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1677`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1713`
- `/api/v1/manager/live-stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1767`
- `/api/station/start` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1824`
- `/api/station/offline-start` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1883`
- `/api/station/order` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1919`
- `/api/station/event` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1952`
- `/api/station/settle` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1971`
- `/api/v1/manager/stations/purge-extra` — `app/src/main/java/com/example/ui/GameNetViewModel.kt:3807`

> This inventory is syntactic extraction of Retrofit annotations and literal direct URLs. It is not a proof that each declaration is reachable in the current UI. API DTOs and Retrofit setup were inspected in `GameNetApi.kt`; direct OkHttp flows were inspected in `SelfHostedManager.kt`.

## Confirmed findings

### AND-001 — Plaintext customer password is persisted, displayed, and copied

| Field | Detail |
|---|---|
| Severity / priority | **High / P1** |
| Confidence | **High** — direct entity, migration, login/register, insert, display and clipboard paths are all present. |
| Impact | Anyone able to access the app’s local Room database or an authorized staff view that can manage customer passwords can obtain reusable customer passwords. The UI can additionally place the phone number/password pair on the Android system clipboard, widening exposure to other clipboard readers or the user interface. |
| Preconditions | A customer account is created/logged in with a nonblank password; staff reaches a customer card with `canManagePasswords` true for display/copy. |
| Reproduction | 1. Log in a customer using `GameNetViewModel.loginCustomer(phone, pass)`; 2. observe `SelfHostedManager.loginCustomer` creates `Customer(...).copy(password = cleanPass)`; 3. observe the ViewModel calls `repository.insertCustomer(cloudCust)`; 4. inspect the Room `customers` table / expand that customer card in the staff UI; 5. press **کپی** (“Copy”) to set `username + password` as the primary clipboard clip. No source/runtime inference is needed for the storage/display flow. |
| Expected | Do not retain a login password in the normal business `Customer` record. Store only a server-issued token in protected storage where persistence is required; never render passwords or put them in the system clipboard. Use a reset/one-time credential workflow instead. |
| Actual | The password is a normal Room `TEXT` column and is directly interpolated into staff UI and `ClipData`. |
| Root cause | Credential material is modeled as a business/customer field and propagated through login, register, CRUD, sync and UI rather than being discarded after server authentication. |
| Fix | Remove `Customer.password` and its DAO/repository/sync serialization and migrate existing values away. Do not return/store customer plaintext passwords. Use password reset/one-time setup mechanisms; display only nonsecret account identifiers. If a temporary generated code is mandatory, make it short-lived, single-use, server-verified, protected, and prevent clipboard copying by default. Rotate/reset affected customer credentials. |
| Regression risk | **High:** schema migration and manager/customer sync contracts will change; legacy staff flows that show/share credentials need replacement. Ensure migration deletes the column/value and tests cover password reset, login and customer import/update. |
| Verification | Add a Room migration test asserting no password column/data, a repository serialization test asserting no `password` JSON field, Compose tests asserting no customer password/clipboard `ClipData`, and an authenticated end-to-end login/reset test. Inspect a post-migration database. |

**Evidence and caller/callee chain**

- Entity/callee: `app/src/main/java/com/example/data/Entities.kt:177-183` declares `@Entity(tableName = "customers")` and `val password: String = ""`.
- Migration: `app/src/main/java/com/example/data/AppDatabase.kt:69-73` runs **`ALTER TABLE customers ADD COLUMN password TEXT NOT NULL DEFAULT ''`**; normal Room builder at `162-176` has no database encryption configuration.
- Login source: `SelfHostedManager.kt:398-405` posts **`put("password", cleanPass)`**; `436` sets **`parseCustomerObject(...).copy(password = cleanPass)`**.
- Persistence caller: `GameNetViewModel.kt:997-1003` calls `SelfHostedManager.loginCustomer` then **`repository.insertCustomer(cloudCust)`**.
- Registration source: `SelfHostedManager.kt:461-485` posts the password and sets **`copy(password = passwordText.trim())`**.
- UI/action: `CustomersReservationsScreen.kt:1650-1688` gates on `customer.password.isNotBlank() && canManagePasswords`, displays **`"رمز اپلیکیشن مشتری: ${customer.password}"`**, and sends **`"نام کاربری: …\nرمز عبور: ${customer.password}…"`** to `clipboard.setPrimaryClip(clip)`.
- Additional propagation: `SelfHostedManager.kt:527-533` includes `put("password",customer.password)` in manager customer upsert.

### AND-002 — VIP reservation rules call omits customer Authorization

| Field | Detail |
|---|---|
| Severity / priority | **High / P1** |
| Confidence | **High** — Android header selection, UI caller/result behavior, and the backend route’s `requireCustomerAuth` middleware are all source-proven. |
| Impact | In an ordinary customer-only session, the VIP/full-hall tab cannot load its reservation rules. `rules` becomes `null`, the durations array is empty, and no request can be enabled/submitted. A stale manager token does not correct this: the endpoint requires a customer token. |
| Preconditions | Customer logged in, tier `GOLD` or `DIAMOND`, then navigates to the `CustomerFullHallTab`; no valid manager token has been placed in `NetworkClient.authToken` as a customer token. |
| Reproduction | 1. Customer logs in (the code assigns `NetworkClient.customerAuthToken`, not `authToken`); 2. open full-hall/VIP tab; 3. `LaunchedEffect` invokes `fetchReservationRules`; 4. its request uses `getBaseHeaders`; 5. `getBaseHeaders` has no `customerAuthToken`, while backend route requires customer auth; 6. non-2xx returns `null`; 7. UI sees no `vipDurations`, and the register button remains disabled. |
| Expected | `GET /api/v1/customer/reservations/rules` must send the customer bearer token and render server-supplied rules/durations to an authenticated customer. |
| Actual | It sends `getBaseHeaders()` rather than `getCustomerHeaders()`. The method converts any non-successful response to `null`, and UI then uses an empty JSON object/empty duration list. |
| Root cause | Header-builder confusion between manager/base and customer-authenticated direct calls. Neighboring pricing, cancellation and atomic routes use the correct builder; the rules route is the inconsistent call. |
| Fix | Change only `fetchReservationRules` to use `getCustomerHeaders()` (and preserve any required non-authority headers explicitly if contractually needed). Surface a distinguishable auth/configuration error instead of silently mapping all non-2xx responses to `null`. |
| Regression risk | **Medium:** header change may expose any untested backend requirement for `X-Manager-ID`; backend source shows the route derives manager scope from customer JWT. Verify no manager header is required. |
| Verification | Unit-test the OkHttp request header contains `Authorization: Bearer <customer token>` and no manager token; integration-test a Gold/Diamond customer receives 200/rules; Compose-test duration chips render and valid selections enable submission. Test 401/403 error UI separately. |

**Evidence and caller/callee chain**

- Customer header definition: `SelfHostedManager.kt:243-250` adds **`Authorization: Bearer $it`** only from `NetworkClient.customerAuthToken`.
- Incorrect base header definition: `SelfHostedManager.kt:228-240` adds authorization only from **`NetworkClient.authToken`** and may add `X-Manager-ID`.
- Defective caller/callee: `SelfHostedManager.kt:1372-1383` builds `GET "$SERVER_URL/api/v1/customer/reservations/rules…"` with **`.headers(getBaseHeaders())`** and returns `null` if `!response.isSuccessful`.
- Correct neighboring pattern: pricing uses **`.headers(getCustomerHeaders())`** (`1387-1406`); cancellation uses it (`1410-1424`); atomic reservation uses it plus `Idempotency-Key` (`1427-1442`).
- UI caller/result: `CustomerFullHallTab.kt:43-55` calls `fetchReservationRules`, then reads `rules?.optJSONObject("rules")?.optJSONArray("vipDurationsMinutes")`; `144-167` requires `vipDurations.contains(selectedDuration)` before submission.
- Backend contract used only to prove the Android call: `backend/canonical_routes.js:336` declares **`app.get('/api/v1/customer/reservations/rules', requireCustomerAuth, …)`**. It similarly requires customer auth for pricing/atomic at `384-385`.

### AND-003 — Release lint quality/security gate is disabled

| Field | Detail |
|---|---|
| Severity / priority | **Medium / P2** |
| Confidence | **High** — direct Gradle configuration. |
| Impact | Release builds do not enforce Android lint, do not fail on lint errors, and do not inspect dependencies through lint. Manifest/API/lifecycle/security regressions that lint could detect can reach release builds without blocking the build. This audit does not claim a particular lint error exists because lint could not run in this sandbox. |
| Reproduction | Inspect/run the configured release build: the Gradle Android `lint` block has `checkReleaseBuilds = false`, `abortOnError = false`, and `checkDependencies = false`. |
| Expected | CI/release builds should run lint for release variants, fail on actionable lint errors, and check dependencies (with narrowly documented suppressions/baselines for accepted findings). |
| Actual | All three safeguards are disabled. |
| Root cause | Explicit build configuration disables lint enforcement. |
| Fix | Set `checkReleaseBuilds = true`, `abortOnError = true`, `checkDependencies = true`; introduce a reviewed lint baseline only for known accepted debt and run `:app:lintRelease` in CI. |
| Regression risk | **Low–Medium:** enabling lint may initially fail on accumulated issues and/or require dependency updates; phase with baseline ownership rather than leaving the gate off. |
| Verification | In an Android-SDK-equipped CI runner, execute `./gradlew :app:lintRelease :app:assembleRelease`; confirm lint runs, reports dependencies, and fails the job on a deliberately introduced known lint violation. |

**Evidence**

- `app/build.gradle.kts:31-35` exactly states:

```kotlin
lint {
  checkReleaseBuilds = false
  abortOnError = false
  checkDependencies = false
}
```

- Release is a defined build type at `78-85`, so this is release configuration rather than a dead module setting.

## Items investigated but not elevated to findings

These are **not findings** because the audit did not establish a defect/impact beyond the fact stated:

1. **Applet duplicate source:** `app/applet/app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` and `app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` share a fully qualified name, but `settings.gradle.kts:22-23` includes only `:app`, and no applet build descriptor was found. The applet source is excluded from this build. Its intended inclusion/deployment is unverified.
2. **Hard-coded business values:** default prices, GN ratios, plan mappings and trial durations are present (examples above). Code does not establish whether those values conflict with approved business policy.
3. **Retry policy:** repeated non-idempotent direct requests could be operationally significant, but determining endpoint idempotency requires executing/contract-testing each route. Customer reservation calls do send idempotency keys; no generalized defect was asserted.
4. **Deep links/payment callbacks:** the exported deep links were traced in `MainActivity`, but the audit did not execute an end-to-end payment provider callback. No authorization bypass is claimed without that test and server interaction.
5. **Time/Jalali/locale correctness:** conversion/display code was inspected, including Asia/Tehran and Western-digit behavior. Boundary-date tests (leap years/DST/locales) were not runnable, so no calendar correctness claim was made.
6. **Role/permission enforcement:** UI and ViewModel actions were traced for representative high-impact actions. Complete server authorization is backend-owned and was not runtime tested under every role; absence of a reported bypass is not a certification.
7. **Room migrations and offline sync reconciliation:** static paths were reviewed; real database upgrade, conflict, network-loss, process-death, and server reconciliation scenarios were not executed.

## Tests and checks run

| Check | Command / scope | Outcome |
|---|---|---|
| Repository/ref check | `git rev-parse HEAD`, `git branch --show-current`, `git status --short` | Observed requested SHA on `main`; no source modification made by this audit. |
| Gradle compile + local unit test attempt | `./gradlew :app:compileDebugKotlin :app:testDebugUnitTest --stacktrace` | **Blocked before task execution.** Gradle says: `SDK location not found. Define a valid SDK location with an ANDROID_HOME environment variable or ... local.properties` (`gradle-compile-test.log:16-20`). No PASS claim. |
| Kotlin file inventory | Read-only enumeration of `app/**/*.kt` | **Completed:** 58 Kotlin files inventoried (table above). |
| Encoding sanity | Read all 58 Kotlin files as bytes/UTF-8 | **Completed:** no NUL bytes; UTF-8 decode passed. This is not compilation. |
| Delimiter heuristic | Comment/string-aware Python delimiter scan | **Inconclusive:** it reported `NetworkLogger.kt` an unclosed `[` at line 145, caused by regex syntax that the heuristic does not parse; it is not a compiler result and was not treated as a build finding. |
| Contract cross-reference | Read only the necessary backend route definitions | **Completed statically:** confirmed the reservation-rules route middleware required for AND-002. No HTTP request was sent to a deployed backend. |
| Unit/instrumented test source review | All 5 `src/test` Kotlin files and 1 `src/androidTest` file | **Completed statically; not executed.** |

## Recommended verification order

1. **Immediately remediate/contain AND-001:** stop new password persistence/display/copy, rotate/reset affected customer passwords, and plan a tested Room data-removal migration.
2. **Fix AND-002 and add a customer-auth request test:** this directly restores the VIP/full-hall flow for customer sessions.
3. **Enable lint gates (AND-003) in a CI environment with Android SDK 36 installed**, then run compile/unit/instrumentation tests and inspect resulting lint findings.
4. Execute manual and automated offline/sync, license/trial, Jalali boundary, deep-link/payment, Room migration and role-matrix test plans before treating the audit as release approval.

## Evidence artifacts retained for audit reproducibility

- Build attempt log: `/home/ubuntu/jobs/1748d5d62ac7_a0/gradle-compile-test.log`
- Source/definition scans: `/home/ubuntu/jobs/1748d5d62ac7_a0/kotlin-definition-inventory.md`, `/home/ubuntu/jobs/1748d5d62ac7_a0/kotlin-coverage-inventory.tsv`
- API extraction: `/home/ubuntu/jobs/1748d5d62ac7_a0/api-inventory-generated.md`, `/home/ubuntu/jobs/1748d5d62ac7_a0/android-endpoints.tsv`
- Contract and password-flow extracts: `/home/ubuntu/jobs/1748d5d62ac7_a0/backend-route-contracts-targeted.txt`, `/home/ubuntu/jobs/1748d5d62ac7_a0/customer-password-trace.txt`

---

**Audit conclusion:** The report establishes exactly three Android-side findings from code/configuration and clearly delineates the unexecuted/runtime-dependent areas. It is an engineering audit, **not a release certification**.


### 8.2 Backend/API/Database Audit

# GameNexa Backend/API/Database Audit

**Audit ID:** `GN-BE-2026-10-02-05348f2`  
**Target:** `/home/ubuntu/GameNexa-release-2026` at `05348f2df8b1f6c30ee233071be13a899d969e25` on `main`  
**Scope:** Backend/API/database only — `server.js`, `canonical_routes.js`, `reservationService.js`, `financialService.js`, `bootstrap.js`, `schema.sql`, test files, Docker/Nginx/runtime configuration, and Android callers only where they clarify an API contract.  
**Excluded:** Production systems, deployed databases, and source modifications. No production system was contacted.

## Executive summary

I extracted **108 Express routes** and reviewed their authorization gates, validation, tenancy scoping, SQL, transactions, idempotency, constraints, and the station/session, reservation, financial, GN/LP, customer, buffet/order, entitlement, subscription, and trial flows.

**Eight evidence-backed findings** were identified:

| Severity | ID | Finding |
|---|---|---|
| High | GN-BE-01 | The authenticated manager device-binding endpoint bypasses the entitlement device cap enforced at login. |
| High | GN-BE-02 | Guest participants are counted in session cost shares but skipped when invoices are written, losing revenue. |
| High | GN-BE-03 | Station-start prepayments are accepted and stored as snapshots but never applied to invoices, payment transactions, or settlement. |
| High | GN-BE-04 | Confirming a VIP reservation supersedes paid normal reservations without refunding them, then makes them non-cancellable. |
| High | GN-BE-05 | Generic manual-wallet approval permits an arbitrary, including negative, approved amount and bypasses the wallet ledger/audit trail. |
| High | GN-BE-06 | Android GN-purchase calls are silently treated as wallet top-ups, so approved purchases do not credit GN. |
| Medium | GN-BE-07 | Trial enforcement trusts caller-supplied device identifiers, allowing a caller to mint a new 24-hour trial with a new pair. |
| Medium | GN-BE-08 | A customer can directly reserve a disabled or non-reservable station despite the customer station-list policy. |

The application also has meaningful positive controls: parameterized SQL was used in the reviewed stateful flows; tenant predicates are common; session start has an advisory lock and active-session guard; session events and reservation/payment ledgers have unique keys; the reservation exclusion constraint protects ordinary station overlap; and invoice payment has amount, transaction, audit, and idempotency logic. Those controls do **not** compensate for the defects below.

> **Evidence standard:** A finding is based on source that establishes the behavior, not a claim about a deployed environment. Status codes and payloads are stated only where current code establishes them. Database-backed runtime proof was not possible because no local PostgreSQL/Docker runtime was installed or listening; this is detailed in [Testing](#testing-and-evidence-level).

---

## Scope and review method

1. Verified the specified Git commit and left source clean (`git diff --exit-code` passed).
2. Read all backend implementation files and `schema.sql`; no backend migrations directory or migration files were present.
3. Parsed both Express source files into the exhaustive 108-route inventory in Appendix A.
4. Traced authorization, ownership predicates, SQL parameterization, transaction boundaries, constraints, idempotency keys, and concurrency controls for reservation, manual payment, invoices, GN/LP, session, trial, and subscription flows.
5. Compared relevant Android request construction where it exposes a backend contract mismatch.
6. Ran source-only and mock tests locally; did not connect to production or start a service.

### Runtime/deployment observations

- `docker-compose.yml:3-57` defines PostgreSQL 16, API, and Nginx. PostgreSQL is not published to a host port; API is behind Nginx. Health checks wait for PostgreSQL and check `/api/v1/time` (`docker-compose.yml:28-37`).
- Nginx redirects HTTP to HTTPS, limits requests at `10r/s` with burst 20, enables TLS 1.2/1.3 and HSTS (`nginx.conf:8-47`).
- The Compose database volume is declared `external: true` (`docker-compose.yml:54-57`), so a fresh local Compose run would require a pre-existing named volume. Docker itself was unavailable in this sandbox.

---

## Findings

### GN-BE-01 — Manager device cap can be bypassed after login

**Severity:** High  
**Affected HTTP path:** `POST /api/v1/manager/device`  
**Authentication:** Manager bearer token plus active entitlement  
**Established response:** `200 {"success":true}` on an insert/update; `400 {"error":"device_id required"}` only if the body has no device ID; `500` on SQL error (`canonical_routes.js:471`).

**Evidence**

The login flow correctly computes the active entitlement's `max_devices`, obtains a transaction-scoped advisory lock, counts active bindings, and rejects a previously unknown device with `403 DEVICE_LIMIT_REACHED` when the cap is met (`server.js:211-239`). Its controlled insert is:

```js
if (Number(count.rows[0].count || 0) >= maxDevices) {
  return res.status(403).json({ error:'Maximum authorized devices reached', code:'DEVICE_LIMIT_REACHED', maxDevices });
}
await lock.query("INSERT INTO manager_device_bindings(manager_id,device_id) ...")
```

But the separate post-login binding route contains no entitlement lookup, count, or advisory lock:

```js
// canonical_routes.js:471
await pool.query(
  'INSERT INTO manager_device_bindings(manager_id,device_id,active,last_seen_at) '
  + 'VALUES($1,$2,TRUE,NOW()) ON CONFLICT(manager_id,device_id) '
  + 'DO UPDATE SET active=TRUE,last_seen_at=NOW()', [mid,did]
);
res.json({success:true});
```

The database only guarantees one row for a given `(manager_id, device_id)` (`schema.sql:1483-1488`); it has no maximum-active-device constraint. Once a device is written through this endpoint, a later login recognizes it as already active and skips the cap branch (`server.js:222-231`).

**Reproduction / exploit sequence (source-established)**

1. Obtain a valid manager bearer for an entitlement with `max_devices: 1`, after one device is already active.
2. Send `POST /api/v1/manager/device` with `{"deviceId":"second-device"}`.
3. The route writes a second active `manager_device_bindings` row and returns 200.
4. Login with that device ID is treated as known, avoiding the cap check.

**Root cause:** The entitlement invariant is implemented only in the login handler rather than centralized and reused by all writes to `manager_device_bindings`.

**Impact:** Subscription/device licensing can be exceeded by any holder of a valid manager session. It also weakens the intended ability to revoke a device as an effective login gate.

**Fix:** Remove this route unless it is necessary, or make it call one shared `bindManagerDevice()` routine which, in a single transaction, locks the manager, reads the active entitlement, counts active bindings, enforces the cap, and returns `403 DEVICE_LIMIT_REACHED`. Consider binding a token/session to a device and checking that binding on privileged requests.

**Regression / verification:** With a max-device entitlement of 1 and device A active, POSTing device B must return 403 and leave exactly one active binding. After a Super Manager revokes A, binding/logging in B must succeed. Run that test concurrently to prove the advisory lock prevents two device-B/device-C enrollments.

---

### GN-BE-02 — Guest shares disappear from session invoices

**Severity:** High  
**Affected HTTP paths:** `POST /api/station/start`, `POST /api/station/settle`  
**Authentication:** Manager bearer plus active entitlement  
**Established responses:** start returns `201` with `sessionId` (`server.js:1014-1015`); settlement returns `200` with `totalCost` and generated `invoices` (`server.js:1427-1429`).

**Evidence**

The start handler explicitly accepts a participant without a positive `customerId` as a guest (`server.js:850-857`) and stores every participant as a payer:

```js
// server.js:1001-1005
INSERT INTO session_participants(..., is_guest, is_payer)
VALUES(..., $6, TRUE)
```

Settlement divides game and buffet cost by *all* payers (`server.js:1372-1378`). It then skips every guest before invoice construction:

```js
// server.js:1390-1396
for (let payerIndex = 0; payerIndex < payers.length; payerIndex += 1) {
  const payer = payers[payerIndex];
  if (!payer.customer_id) continue;
  const shareGame = ...;
  const shareBuffet = ...;
```

Only non-guests reach the invoice insert (`server.js:1420-1423`), which writes `paid_amount` as `0`. The schema makes an invoice customer mandatory: `invoices.customer_id integer NOT NULL` (`schema.sql:378-404`). There is no anonymous/guest payable table or transfer of a guest share to a registered payer.

**Concrete behavior:** Start a session with one registered customer and one guest, both default payers, and no discounts. At settlement, `totalCost` is divided by two. The guest share is skipped; only the registered customer's half is invoiced. The response can therefore report a session total greater than the sum of its invoices.

**Root cause:** Allocation cardinality (`is_payer`) and billable-invoice cardinality (`customer_id`) differ without a reconciliation policy.

**Impact:** Underbilling and an unreconcilable financial record for every session containing billable guests. Any remainder assigned to a final guest is also skipped.

**Fix:** Require every guest to nominate a registered sponsor/payer before settlement, or add an explicit anonymous/walk-in payer/invoice model. Enforce the invariant before commit: `sum(invoice total_amount) + cash/guest receivable = game_sessions.total_cost` (with documented discounts separated). Reject settlement when no valid billing destination exists.

**Regression / verification:** Create a session with one registered customer and one guest at a deterministic hourly rate; settle it and assert conservation. Test both guest-first and guest-last ordering, a buffet line, an indivisible remainder, and a guest explicitly marked non-payer. The transaction should roll back if no sponsor is supplied.

---

### GN-BE-03 — Session prepayments are accepted but never reconciled financially

**Severity:** High  
**Affected HTTP paths:** `POST /api/station/start`, `POST /api/station/offline-start`, `POST /api/station/settle`  
**Authentication:** Manager bearer plus active entitlement.

**Evidence**

`/api/station/start` parses and validates an integer `prepaymentAmount` and `customerPrepayments` (`server.js:876-903`), records them in the immutable pricing snapshot (`server.js:978-990`), and writes per-customer values to `session_participants.prepayment_amount` (`server.js:1006-1011`). The offline-start endpoint accepts the same fields (`server.js:1025-1041`). Android also sends these values in its canonical start request, including a map keyed by customer ID (`SelfHostedManager.kt:1782-1828`).

Settlement, however, only computes game/buffet costs and updates `game_sessions` (`server.js:1359-1371`), then creates each invoice with a literal paid amount of zero:

```sql
-- server.js:1420-1422
INSERT INTO invoices(... total_amount, paid_amount, ...)
VALUES (..., ($6::numeric+$7::numeric), 0, ...)
```

That handler does not read `prepayment_amount`, `initialPrepaymentAmount`, or `customerPrepayments`, and it creates no `payment_transactions`, `wallet_transactions`, or `financial_audit_logs` for the start-time money. The later invoice-payment endpoint is separate and records only a fresh manager confirmation (`server.js:1230-1295`).

**Root cause:** “Prepayment” is represented as session metadata rather than a financial event, and settlement does not consume that metadata.

**Impact:** A manager can record cash/prepayment in the app but the invoice remains wholly unpaid. Staff may collect twice, lose traceability for money received, or disagree over balances after offline reconciliation.

**Fix:** Define prepayment semantics. If it is money received, create an exact, idempotent payment/ledger/audit event in the station-start transaction and allocate it to the eventual invoices at settlement (with a conservation rule). If it is only a UI estimate, remove it from the financial API contract and name it accordingly. The offline reconciliation route must follow the same ledger path.

**Regression / verification:** Start a two-customer session with both an aggregate and per-customer prepayment, settle it, and verify exact `payment_transactions`, invoice `paid_amount/status`, and financial audit totals. Replay the idempotency key and confirm no second payment. Test partial, full, overpayment rejection/credit policy, guest allocation, and offline-start reconciliation.

---

### GN-BE-04 — VIP priority can strand funds from a paid normal reservation

**Severity:** High  
**Affected HTTP paths:** `POST /api/v1/manager/reservation-payments/:id/approve`, `PUT /api/v1/manager/reservations/:id/status`, cancellation paths  
**Authentication:** Manager bearer plus active entitlement; customer cancellation is customer bearer.

**Evidence**

A manual reservation payment is persisted as a successful `payment_transactions` row before the reservation moves to `PENDING_APPROVAL`/`VIP_PAYMENT_PAID` (`canonical_routes.js:427-458`). A paid normal reservation can then be confirmed. When a VIP is confirmed, the status route locks ordinary overlapping reservations and changes each to `SUPERSEDED_BY_VIP_PRIORITY`:

```js
// canonical_routes.js:273-295
const overlaps = await c.query(`SELECT id,status,... FOR UPDATE`, ...);
for (const row of overlaps.rows) {
  await c.query("UPDATE reservations SET status='SUPERSEDED_BY_VIP_PRIORITY' ...", [row.id,mid]);
  await c.query('INSERT INTO reservation_audit_logs(...)', ...);
}
```

That loop does not insert a wallet refund, payment reversal, `financial_audit_logs` event, notification, or credit. The normal cancellation service explicitly refuses both superseded states:

```js
// financialService.js:195-197
if (['COMPLETED','EXPIRED','REJECTED','NO_SHOW',
     'SUPERSEDED_BY_VIP','SUPERSEDED_BY_VIP_PRIORITY'].includes(String(r.status))) {
  throw new Error('RESERVATION_CANNOT_BE_CANCELLED_IN_CURRENT_STATUS');
}
```

The customer and manager cancel route catch this and return `400 {error: ...}` (`canonical_routes.js:223-232`, `385-386`).

**Source-established sequence**

1. Approve the full manual payment for a normal booking; the dedicated approval endpoint inserts its successful payment.
2. Confirm that normal reservation.
3. Fully pay and confirm an overlapping VIP reservation.
4. The normal reservation is marked `SUPERSEDED_BY_VIP_PRIORITY`; no refund is created.
5. Calling either cancellation endpoint fails because that state is non-cancellable.

**Root cause:** VIP capacity arbitration changes reservation state but does not invoke the financial cancellation/refund workflow before rendering the old reservation terminal.

**Impact:** Customer money can remain associated with a reservation that cannot occur and cannot use the normal refund path. This creates a direct financial-loss/operational-dispute risk.

**Fix:** In the same transaction that supersedes each paid reservation, calculate paid funds from `payment_transactions`, create an idempotent wallet credit/refund and `financial_audit_logs` record, record a reason and notification, then change status. If business policy permits a penalty, snapshot and apply it explicitly rather than silently retaining the payment. A retry-safe unique key should be scoped to `vip-supersede-refund:<reservationId>`.

**Regression / verification:** Build a paid, confirmed normal booking that overlaps a paid VIP confirmation. Assert the normal booking becomes superseded, the sum of successful payment transactions is credited exactly once according to policy, a financial audit entry exists, the customer receives a notification, and a replay of the VIP confirmation cannot duplicate the credit.

---

### GN-BE-05 — Generic manual wallet approval has no amount guard or ledger/audit entry

**Severity:** High  
**Affected HTTP path:** `POST /api/v1/manager/manual-payment-requests/:id/approve`  
**Authentication:** Manager bearer plus active entitlement  
**Established responses:** 404 if no pending request; otherwise 200 `{success:true}`; catch-all 500 (`canonical_routes.js:400`).

**Evidence**

The generic approval handler locks only a pending request and applies `approvedAmount` directly, without checking finite/positive value, currency, or whether it is less than/equal to the requested amount:

```js
// canonical_routes.js:400
UPDATE manual_payment_requests SET status='APPROVED',approved_amount=$1, ...
  [Number(req.body?.approvedAmount||r.amount), ...]

if (r.purpose==='WALLET_TOPUP')
  UPDATE customers SET wallet_balance=wallet_balance+$1 ...
  [Number(req.body?.approvedAmount||r.amount), ...]
```

An authenticated manager can send `{"approvedAmount": -500}` for a pending `WALLET_TOPUP`: `-500` is truthy and is directly applied. It can also approve an amount greater than the original request. This path writes neither `wallet_transactions` nor `financial_audit_logs`; contrast the reservation payment path, which writes a `payment_transactions` row and audit trail (`canonical_routes.js:449-457`). `customers.wallet_balance` has no nonnegative check (`schema.sql:214-235`), whereas `wallet_transactions.amount` does require a nonnegative amount (`schema.sql:1142-1154`).

**Root cause:** The generic manual-review endpoint treats manager input as inherently valid and mutates a derived balance directly instead of using the wallet ledger as the source of truth.

**Impact:** A typo, malformed integration, or compromised manager bearer can credit any amount or push a wallet balance negative, without a wallet transaction or financial audit record that explains the change. Reconciliation of `customers.wallet_balance` to the wallet ledger is impossible for this path.

**Fix:** Validate `approvedAmount` with a decimal-string parser (positive, finite, approved currency) and bound it to the requested amount unless a separately authorized adjustment workflow is used. Insert an idempotent `wallet_transactions` credit/debit and `financial_audit_logs` row before updating the materialized balance; derive/reconcile balances from the ledger. Add database checks for nonnegative wallet balance if negative balances are disallowed.

**Regression / verification:** For a pending top-up, reject negative, zero, non-finite, and over-request amounts with 4xx and no mutation. A valid approval must produce one ledger row, one audit row, and one matching balance delta; a replay must not add another delta. Test an intentional adjustment only through a separately scoped/audited API.

---

### GN-BE-06 — Android “Buy GN” request silently becomes a wallet top-up

**Severity:** High  
**Affected HTTP path:** `POST /api/v1/customer/manual-payment-requests` followed by generic approval  
**Authentication:** Customer bearer for request; manager bearer plus active entitlement for approval  
**Established response:** accepted request returns `201 {success:true, request:...}` (`canonical_routes.js:397`).

**Evidence**

Android's GN purchase call sends `gnAmount` and `transactionType: "BUY_GN"`, but does not send a supported `purpose` or idempotency key:

```kotlin
// SelfHostedManager.kt:1694-1715
put("amount", totalToman.toLong())
put("gnAmount", gnAmount)
put("transactionType", "BUY_GN")
.url("$SERVER_URL/api/v1/customer/manual-payment-requests")
```

The backend ignores `gnAmount` for behavior and defaults any missing `purpose` to `WALLET_TOPUP`:

```js
// canonical_routes.js:397
const purpose = String(b.purpose || 'WALLET_TOPUP').toUpperCase();
if (!['WALLET_TOPUP','RESERVATION_PAYMENT'].includes(purpose)) return res.status(422)...
```

The generic manager approval only changes `wallet_balance` for that default purpose (`canonical_routes.js:400`). It neither inserts `gn_ledger` nor changes `customers.gn_balance`/`pending_gn`. The Android client immediately increments only its local `pendingGn` after a 2xx response (`SelfHostedManager.kt:1720-1723`), so its state diverges from the server after approval.

**Root cause:** The mobile payload's `transactionType`/`gnAmount` contract has no corresponding backend purpose or state transition, and the permissive default hides the mismatch instead of rejecting it.

**Impact:** A customer can submit and pay for a GN purchase that is approved as ordinary wallet credit, while the app displays pending GN. This causes customer entitlement loss and backend/client balance divergence.

**Fix:** Add an explicit, authenticated `GN_PURCHASE` purpose with strict fields and idempotency. On approval, atomically write a GN ledger credit, update `gn_balance` (and any intentionally used `pending_gn` transition), and audit the fiat receipt. Alternatively, reject `transactionType: BUY_GN` with 422 until GN purchase is supported. Do not silently default an unknown financial intent to wallet top-up.

**Regression / verification:** Submit the Android-equivalent payload and assert either a deterministic 422 with no state change or a request classified `GN_PURCHASE`. After approval, assert exactly one GN ledger credit and matching `gn_balance` increase, no unintended wallet credit, and a client reload that agrees with the server.

---

### GN-BE-07 — Trial identity can be reset by choosing a new caller-supplied pair

**Severity:** Medium  
**Affected HTTP path:** `POST /api/v1/trial/start`  
**Authentication:** Public  
**Established responses:** `400` if either identifier is blank, `403` if an authenticated Manager starts a trial or a matching identifier is blocked, `201` for a new pair, and `200` for an existing pair (`server.js:313-353`).

**Evidence**

The public endpoint takes both identifiers directly from JSON, merely trims/truncates them, and uses them as the whole eligibility boundary:

```js
// server.js:311-346
const deviceId = normalizeTrialIdentity(req.body?.deviceId || req.body?.device_id);
const deviceFingerprint = normalizeTrialIdentity(req.body?.deviceFingerprint || req.body?.device_fingerprint);
...
SELECT ... FROM trial_devices WHERE device_id = $1 OR device_fingerprint = $2
...
INSERT INTO trial_devices (device_id, device_fingerprint, ... expires_at, status)
VALUES ($1, $2, ..., $5, 'ACTIVE')
```

The route has an IP-keyed application rate limit of five per minute (`server.js:313`), but no user/account validation, app/device attestation, signed device claim, or server-maintained hardware identity. `trial_devices` uses `device_id` as primary key (`schema.sql:1126-1135`, `1723-1728`), so a caller choosing a previously unseen `deviceId` and `deviceFingerprint` gets a new 24-hour row and a 201 response.

**Root cause:** Client-provided, mutable identifiers are treated as proof of one physical device.

**Impact:** The one-trial-per-device commercial control is trivially bypassable by a direct API caller changing both strings. This can also fill the trial administration list with junk records.

**Fix:** Bind trial issuance to a verifiable identity appropriate to the Android threat model (e.g., Play Integrity/device attestation validated server-side plus an account/phone policy), retain a server-side risk/abuse record, and enforce broader WAF/IP/account quotas. Do not claim that an arbitrary string pair establishes a device identity.

**Regression / verification:** Use a mocked valid attestation for device A to obtain one trial; replay succeeds only as the same trial. A fabricated new device/fingerprint with no valid attestation must fail. Verify blocked identifiers, retry behavior, privacy handling, and administrative deletion/extension.

---

### GN-BE-08 — Direct reservation bypasses disabled/non-reservable station policy

**Severity:** Medium  
**Affected HTTP path:** `POST /api/v1/customer/reservations` and `/api/v1/customer/reservations/atomic`  
**Authentication:** Customer bearer  
**Established response:** a successful booking returns `201` (`canonical_routes.js:309-321`, `385`), subject to the service's other checks.

**Evidence**

The customer-facing station discovery endpoint deliberately exposes only active, reservable stations:

```sql
-- canonical_routes.js:323-334
WHERE s.manager_id=$1 AND s.active=TRUE AND s.reservable=TRUE
```

But normal reservation booking passes caller-controlled station IDs to `bookReservation`. Its ownership check is only:

```sql
-- reservationService.js:225-243
SELECT * FROM stations WHERE id = ANY($1) AND manager_id = $2
```

It checks controller capacity but never checks `stations.active` or `stations.reservable`. Thus a customer knowing an ID in their own tenant can submit it directly in `stationIds`/`stationId`, bypassing the list's policy. The service's VIP/full-hall branch filters `active = true` when it expands all stations (`reservationService.js:193-196`), but normal booking does not.

**Root cause:** Availability policy is enforced in a discovery/read route but not at the authoritative write boundary.

**Impact:** Reservations and payment requests can be created for maintenance-disabled or intentionally non-bookable stations, producing operational conflicts and possible invalid charges.

**Fix:** In the same write transaction and before price calculation, select stations `FOR UPDATE` with `manager_id`, `active=TRUE`, and `reservable=TRUE` for all customer-originated normal reservations. Return a clear 422/409 policy error. Keep manager-only overrides explicit and separately authorized if they are required.

**Regression / verification:** Create one active/reservable station, one inactive station, and one active/non-reservable station. Customer list returns only the first. Direct normal reservation attempts for the latter two must fail without creating reservations/idempotency records; the valid station must still create exactly one booking under concurrent retries.

---

## Cross-cutting authorization, consistency, and database observations

### Controls verified

- **Tenant scoping:** Reviewed financial, reservation, invoice, customer, GN transfer, and session queries commonly bind `manager_id`; static tenant-isolation checks passed. Examples include reservation ownership locks (`financialService.js:187-193`) and session station ownership (`server.js:933-938`).
- **Reservation overlap:** `reservations.no_overlap` is a PostgreSQL GiST exclusion constraint on manager, station, and time range for live blocking states (`schema.sql:1547-1552`). This is stronger than application-only overlap checks and guards concurrent ordinary bookings.
- **Reservation idempotency:** `reservation_request_idempotency` has a unique `(manager_id,idempotency_key)` constraint (`schema.sql:1603-1608`); the `/atomic` route also takes an advisory lock (`canonical_routes.js:385`).
- **Session concurrency:** Start takes an advisory lock keyed by manager/station and locks active sessions (`server.js:906-943`). Event IDs and per-session sequence numbers are uniquely constrained (`schema.sql:1626-1647`).
- **Invoice payments:** `/api/station/invoice/pay` requires an idempotency key, locks the invoice, rejects payments above remaining balance, inserts a successful payment, updates status, and writes a financial audit record (`server.js:1230-1295`).
- **GN transfer:** Customer transfer validates a positive finite amount and idempotency key, locks sender/recipient, checks funds, updates balances and inserts debit/credit ledger rows in one transaction (`canonical_routes.js:394`).
- **Subscription activation:** Activation locks the request, verifies the bcrypt activation secret, applies a manager-level advisory lock before device cap count, and uses a unique `(manager_id,device_id)` binding (`canonical_routes.js:559`; `schema.sql:1483-1488`). This does not cure GN-BE-01's separate bypass route.

### Constraints and design limitations reviewed

- `invoices` has a unique `(session_id,customer_id)` key (`schema.sql:1459-1464`), which prevents duplicated registered-customer invoices but has no representation for guest liabilities.
- `payment_transactions`, GN ledger, LP ledger, and wallet transactions have idempotency uniqueness (`schema.sql:1563-1568`, `1427-1439`, `1467-1479`, `1739-1751`). The generic manual wallet approval does not use them (GN-BE-05).
- Configuration revisions are unique per manager/version (`schema.sql:1363-1368`). Concurrent configuration initialization/update handling was reviewed; no standalone reportable data-integrity defect was asserted without a PostgreSQL runtime proof.
- No backend migration directory/file was present; `schema.sql` is the schema artifact reviewed.

---

## Android comparison

`tests/android_backend_contract_check.py` passed: it found **51 Retrofit contracts and 76 raw Android API paths** with a backend path match. This is an existence/static check, not proof that fields and financial semantics agree. GN-BE-06 is precisely such a semantic mismatch: the path exists and responds 201 while `gnAmount`/`BUY_GN` are not implemented by the backend.

Other useful Android observations:

- Station start sends `prepaymentAmount` and `customerPrepayments` (`SelfHostedManager.kt:1782-1828`), corroborating that GN-BE-03 is a real application contract rather than dead request parsing.
- The standard reservation-payment caller supplies an idempotency key (`SelfHostedManager.kt:1626-1649`), while the generic payment and GN purchase callers shown in GN-BE-06 do not (`SelfHostedManager.kt:759-803`, `1694-1715`). This reinforces the need to make idempotency mandatory for every financial request.
- Subscription purchase uses a UUID-bearing idempotency key and persists the pending activation secret (`GameNetViewModel.kt:4939-4960`).

---

## Testing and evidence level

| Check | Result | Evidence level / caveat |
|---|---|---|
| `npm ci --ignore-scripts` | Passed; 105 packages installed; npm reported 0 vulnerabilities | Local dependency installation only. |
| `npm run check` | Passed | Real Node syntax checks for `server.js`, `canonical_routes.js`, `reservationService.js`, and `financialService.js`. |
| `node backend/test_isolation_static.js` | Passed | Static source assertions; no HTTP/database runtime proof. |
| `node backend/test_financial_mock.js` | Passed | Mocked `query()` unit test only; it does not exercise PostgreSQL, HTTP, or the findings. |
| `python3 tests/android_backend_contract_check.py` | Passed | Static Android/backend path and token-contract inspection; it cannot detect semantic payload mismatches. |
| `python3 tests/deep_release_audit.py` | Passed (43 invariants) | Static text-pattern release checks; not an API/database execution. |
| `npm test` / `test:release` | Not completed | Initial static tests passed, then `test_reservation.js` attempted `DATABASE_URL` at localhost and failed `ECONNREFUSED` for `127.0.0.1:5432` / `::1:5432`. The suite was not continued. |
| Database-backed tests (`test_reservation.js`, `test_financial.js`, cancellation/security/subscription/DB-integrity tests) | Not run to completion | They require a local PostgreSQL database, and several also require an API at `127.0.0.1:3000` or an external hard-coded URL. Docker, PostgreSQL client/server, and a local listening PostgreSQL service were unavailable. No substitute/mock was represented as runtime proof. |
| Source integrity | Passed | `git diff --exit-code` passed after audit; no source modification was made. |

**Recommended verification order:** Fix GN-BE-02 through GN-BE-06 first, provision a disposable local PostgreSQL 16 instance initialized from `schema.sql`, then add targeted HTTP+database integration tests for every regression listed above. Run the full release suite only against that isolated database and local API, never an external production endpoint.

---

## Appendix A — Exhaustive route inventory

The following inventory is mechanically extracted from `server.js` and `canonical_routes.js`. “Public” means no Express auth middleware appears on that route declaration; it does not imply the payload is harmless. Line references are current at the audited commit.

| Source | Method | Path | Route gate |
|---|---|---|---|
| `server.js:196` | `POST` | `/api/auth/manager/login` | Public |
| `server.js:251` | `POST` | `/api/auth/customer/register` | Public |
| `server.js:278` | `POST` | `/api/auth/customer/login` | Public |
| `server.js:302` | `GET` | `/api/v1/time` | Public |
| `server.js:313` | `POST` | `/api/v1/trial/start` | Public |
| `server.js:356` | `POST` | `/api/v1/trial/status` | Public |
| `server.js:374` | `GET` | `/api/v1/super-manager/managers` | Super Manager |
| `server.js:418` | `POST` | `/api/v1/super-manager/add-manager` | Super Manager |
| `server.js:455` | `POST` | `/api/v1/super-manager/managers` | Super Manager |
| `server.js:475` | `POST` | `/api/v1/super-manager/create-manager` | Super Manager |
| `server.js:495` | `POST` | `/api/v1/super-manager/recover` | Super Manager |
| `server.js:509` | `GET` | `/api/v1/super-manager/trial-devices` | Super Manager |
| `server.js:525` | `DELETE` | `/api/v1/super-manager/trial-devices/:id` | Super Manager |
| `server.js:554` | `POST` | `/api/v1/super-manager/managers/:id/subscription/extend` | Super Manager |
| `server.js:592` | `DELETE` | `/api/v1/super-manager/managers/:id/devices/:deviceId` | Super Manager |
| `server.js:602` | `POST` | `/api/v1/super-manager/trial-devices/:id/extend` | Super Manager |
| `server.js:636` | `PUT` | `/api/v1/super-manager/managers/:id` | Super Manager |
| `server.js:674` | `DELETE` | `/api/v1/super-manager/managers/:id` | Super Manager |
| `server.js:688` | `POST` | `/api/v1/super-manager/managers/delete` | Super Manager |
| `server.js:703` | `POST` | `/api/v1/super-manager/managers/purge` | Super Manager |
| `server.js:720` | `GET` | `/api/v1/super-manager/ping` | Public |
| `server.js:721` | `POST` | `/api/v1/super-manager/ping` | Public |
| `server.js:737` | `GET` | `/api/v1/manager/diagnostics` | Manager bearer + active entitlement |
| `server.js:815` | `GET` | `/api/v1/manager/configuration` | Manager bearer + active entitlement |
| `server.js:825` | `PUT` | `/api/v1/manager/configuration` | Manager bearer + active entitlement |
| `server.js:876` | `POST` | `/api/station/start` | Manager bearer + active entitlement |
| `server.js:1025` | `POST` | `/api/station/offline-start` | Manager bearer + active entitlement |
| `server.js:1095` | `POST` | `/api/station/order` | Manager bearer + active entitlement |
| `server.js:1138` | `POST` | `/api/station/event` | Manager bearer + active entitlement |
| `server.js:1226` | `POST` | `/api/station/invoice/pay` | Manager bearer + active entitlement |
| `server.js:1303` | `GET` | `/api/station/active` | Manager bearer + active entitlement |
| `server.js:1317` | `GET` | `/api/customer/live-session` | Customer bearer |
| `server.js:1333` | `POST` | `/api/station/settle` | Manager bearer + active entitlement |
| `canonical_routes.js:103` | `GET` | `/api/v1/manager/customers` | Manager bearer + active entitlement |
| `canonical_routes.js:108` | `POST` | `/api/v1/manager/customers` | Manager bearer + active entitlement |
| `canonical_routes.js:121` | `DELETE` | `/api/v1/manager/customers/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:138` | `POST` | `/api/v1/manager/customers/delete-batch` | Manager bearer + active entitlement |
| `canonical_routes.js:158` | `GET` | `/api/v1/manager/settings/:key` | Manager bearer + active entitlement |
| `canonical_routes.js:159` | `GET` | `/api/v1/manager/reservation-configuration` | Manager bearer + active entitlement |
| `canonical_routes.js:160` | `PUT` | `/api/v1/manager/reservation-configuration` | Manager bearer + active entitlement |
| `canonical_routes.js:161` | `PUT` | `/api/v1/manager/settings/:key` | Manager bearer + active entitlement |
| `canonical_routes.js:164` | `GET` | `/api/v1/manager/console-types` | Manager bearer + active entitlement |
| `canonical_routes.js:165` | `POST` | `/api/v1/manager/console-types` | Manager bearer + active entitlement |
| `canonical_routes.js:166` | `DELETE` | `/api/v1/manager/console-types/:name` | Manager bearer + active entitlement |
| `canonical_routes.js:167` | `GET` | `/api/v1/manager/products` | Manager bearer + active entitlement |
| `canonical_routes.js:168` | `POST` | `/api/v1/manager/products` | Manager bearer + active entitlement |
| `canonical_routes.js:169` | `DELETE` | `/api/v1/manager/products/:name` | Manager bearer + active entitlement |
| `canonical_routes.js:172` | `GET` | `/api/v1/manager/stations` | Manager bearer + active entitlement |
| `canonical_routes.js:173` | `POST` | `/api/v1/manager/stations` | Manager bearer + active entitlement |
| `canonical_routes.js:176` | `GET` | `/api/v1/manager/live-stations` | Manager bearer + active entitlement |
| `canonical_routes.js:177` | `POST` | `/api/v1/manager/live-stations` | Manager bearer + active entitlement |
| `canonical_routes.js:180` | `GET` | `/api/v1/manager/orders/:stationId` | Manager bearer + active entitlement |
| `canonical_routes.js:181` | `POST` | `/api/v1/manager/orders` | Manager bearer + active entitlement |
| `canonical_routes.js:182` | `DELETE` | `/api/v1/manager/orders/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:183` | `DELETE` | `/api/v1/manager/orders/station/:stationId` | Manager bearer + active entitlement |
| `canonical_routes.js:184` | `GET` | `/api/v1/manager/session-history` | Manager bearer + active entitlement |
| `canonical_routes.js:187` | `GET` | `/api/v1/manager/reservations` | Manager bearer + active entitlement |
| `canonical_routes.js:188` | `POST` | `/api/v1/manager/reservations` | Manager bearer + active entitlement |
| `canonical_routes.js:222` | `DELETE` | `/api/v1/manager/reservations/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:223` | `POST` | `/api/v1/manager/reservations/:id/cancel` | Manager bearer + active entitlement |
| `canonical_routes.js:234` | `PUT` | `/api/v1/manager/reservations/:id/status` | Manager bearer + active entitlement |
| `canonical_routes.js:306` | `POST` | `/api/v1/manager/settings` | Manager bearer + active entitlement |
| `canonical_routes.js:307` | `POST` | `/api/v1/manager/session-history` | Manager bearer + active entitlement |
| `canonical_routes.js:308` | `DELETE` | `/api/v1/manager/session-history` | Manager bearer + active entitlement |
| `canonical_routes.js:309` | `POST` | `/api/v1/customer/reservations` | Customer bearer |
| `canonical_routes.js:323` | `GET` | `/api/v1/customer/stations` | Customer bearer |
| `canonical_routes.js:336` | `GET` | `/api/v1/customer/reservations/rules` | Customer bearer |
| `canonical_routes.js:384` | `POST` | `/api/v1/customer/reservations/pricing-preview` | Customer bearer |
| `canonical_routes.js:385` | `POST` | `/api/v1/customer/reservations/atomic` | Customer bearer |
| `canonical_routes.js:386` | `POST` | `/api/v1/customer/reservations/:id/cancel` | Customer bearer |
| `canonical_routes.js:390` | `GET` | `/api/v1/customer/profile` | Customer bearer |
| `canonical_routes.js:391` | `GET` | `/api/v1/customer/reservations` | Customer bearer |
| `canonical_routes.js:392` | `GET` | `/api/v1/customer/club/ledger` | Customer bearer |
| `canonical_routes.js:393` | `POST` | `/api/v1/manager/club/ledger` | Manager bearer + active entitlement |
| `canonical_routes.js:394` | `POST` | `/api/v1/customer/club/transfer` | Customer bearer |
| `canonical_routes.js:397` | `POST` | `/api/v1/customer/manual-payment-requests` | Customer bearer |
| `canonical_routes.js:398` | `GET` | `/api/v1/customer/manual-payment-requests` | Customer bearer |
| `canonical_routes.js:399` | `GET` | `/api/v1/manager/manual-payment-requests` | Manager bearer + active entitlement |
| `canonical_routes.js:400` | `POST` | `/api/v1/manager/manual-payment-requests/:id/approve` | Manager bearer + active entitlement |
| `canonical_routes.js:401` | `GET` | `/api/v1/manager/reservation-payments/pending` | Manager bearer + active entitlement |
| `canonical_routes.js:416` | `POST` | `/api/v1/manager/reservation-payments/:id/reject` | Manager bearer + active entitlement |
| `canonical_routes.js:427` | `POST` | `/api/v1/manager/reservation-payments/:id/approve` | Manager bearer + active entitlement |
| `canonical_routes.js:462` | `POST` | `/api/v1/manager/manual-payment-requests/:id/reject` | Manager bearer + active entitlement |
| `canonical_routes.js:464` | `POST` | `/api/v1/manager/club/point-logs` | Manager bearer + active entitlement |
| `canonical_routes.js:465` | `GET` | `/api/v1/manager/club/point-logs/:customerId` | Manager bearer + active entitlement |
| `canonical_routes.js:470` | `GET` | `/api/v1/auth/check` | Manager bearer + active entitlement |
| `canonical_routes.js:471` | `POST` | `/api/v1/manager/device` | Manager bearer + active entitlement |
| `canonical_routes.js:474` | `POST` | `/api/v1/manager/announcements` | Manager bearer + active entitlement |
| `canonical_routes.js:475` | `POST` | `/api/v1/manager/audit-logs` | Manager bearer + active entitlement |
| `canonical_routes.js:476` | `GET` | `/api/v1/manager/club/ledger/:customerId` | Manager bearer + active entitlement |
| `canonical_routes.js:477` | `DELETE` | `/api/v1/manager/reservations/by-phone/:phone` | Manager bearer + active entitlement |
| `canonical_routes.js:478` | `POST` | `/api/v1/manager/stations/purge-extra` | Manager bearer + active entitlement |
| `canonical_routes.js:506` | `POST` | `/api/v1/manager/customer-transactions` | Manager bearer + active entitlement |
| `canonical_routes.js:507` | `GET` | `/api/v1/manager/customer-transactions` | Manager bearer + active entitlement |
| `canonical_routes.js:508` | `GET` | `/api/v1/customer/transactions` | Customer bearer |
| `canonical_routes.js:509` | `GET` | `/api/v1/manager/payment-methods` | Manager bearer + active entitlement |
| `canonical_routes.js:510` | `POST` | `/api/v1/manager/payment-methods` | Manager bearer + active entitlement |
| `canonical_routes.js:511` | `PATCH` | `/api/v1/manager/payment-methods/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:512` | `GET` | `/api/v1/customer/payment-methods` | Customer bearer |
| `canonical_routes.js:524` | `GET` | `/api/v1/subscriptions/check` | Public |
| `canonical_routes.js:525` | `POST` | `/api/v1/subscriptions/status` | Public |
| `canonical_routes.js:526` | `GET` | `/api/v1/plans` | Public |
| `canonical_routes.js:530` | `GET` | `/api/v1/super-manager/subscription-plans` | Super Manager |
| `canonical_routes.js:534` | `PUT` | `/api/v1/super-manager/subscription-plans` | Super Manager |
| `canonical_routes.js:559` | `POST` | `/api/v1/subscriptions/activate` | Public |
| `canonical_routes.js:560` | `POST` | `/api/v1/subscriptions/buy` | Public |
| `canonical_routes.js:587` | `POST` | `/api/v1/subscriptions/set-password` | Public |
| `canonical_routes.js:588` | `POST` | `/api/v1/coupons/validate` | Public |

---

**End of audit report.**


### 8.3 Security Audit

# GameNexa White-Box Security Audit

## Audit identity and scope

| Item | Value |
|---|---|
| Target | `/home/ubuntu/GameNexa-release-2026` |
| Audited revision | `05348f2df8b1f6c30ee233071be13a899d969e25` on `main` |
| Repository state at review | Clean working tree; HEAD matched the requested commit |
| Method | Safe local white-box source review, deterministic static checks, syntax/contract tests, dependency audit. **No external endpoint or runtime was contacted.** |
| In-scope areas | Login/password/bcrypt/JWT/session/logout/expiry/roles; Manager A/B and Super Manager isolation; IDOR/BOLA; mass assignment; injection; CORS/CSRF; logging/PII; credentials/debug paths; trial/subscription; tests. |
| Findings | **4 confirmed findings**: 3 medium, 1 low |

## Executive summary

The server has several meaningful protections: JWT algorithm pinning to HS256, database-backed account-existence checks in auth middleware, tenant predicates on reviewed manager/customer operations, parameterized PostgreSQL calls, bcrypt usage, entitlement gates on manager mutations, Super Manager role/entitlement checks, origin allowlisting, and TLS/cleartext restrictions. I did **not** confirm a Manager A-to-B data IDOR, a Super-Manager authorization bypass, SQL injection, a committed production secret, a CSRF issue for bearer-authenticated APIs, or a subscription activation bypass in the reviewed revision.

However, four confirmed issues merit remediation:

1. The one-per-device trial rule can be bypassed because trial identities are caller-controlled and Manager blocking only executes when the caller volunteers a valid bearer token.
2. Logout is local-only: 24-hour bearer JWTs have no server-side revocation/session-version mechanism, including after password updates.
3. Customer password policy accepts four-character passwords, and the Super Manager manager-update path accepts any non-empty password.
4. Customer login exposes distinguishable account-state messages, enabling targeted account enumeration within a known manager tenant.

The most important corrective work is to replace the trial identity model with a server-verifiable enrollment/attestation design, and to add revocable server-side session state (or an account `auth_epoch`) to access-token verification.

## Severity method

Severity reflects observed impact in this codebase, not a generic scanner rating:

- **Medium**: security/business-control bypass or replay window affecting authenticated or licensed functionality, but no demonstrated cross-tenant read/write or privilege escalation.
- **Low**: limited information disclosure or a weakness that requires password guessing/other preconditions.
- **Confidence**: High = direct control-flow/data-flow proof in current code; Medium = direct behavior with a bounded deployment or business-policy dependency.

---

# Confirmed findings

## SEC-01 — Trial eligibility can be reset with caller-chosen identities and an omitted bearer token

| Property | Assessment |
|---|---|
| Severity | **Medium** |
| Confidence | **High** for bypass of the server's intended per-device / Manager-trial policy; **Medium** for broader commercial impact because protected manager API mutations still require `requireActiveEntitlement`. |
| CWE | CWE-602 (client-side enforcement of server-side security), CWE-639 (authorization based on user-controlled key) |
| Affected flow | Public `POST /api/v1/trial/start` → Android trial-active local state |

### Exact evidence

**1) The endpoint is public and accepts the device identity entirely from the request body.**

`backend/server.js:310-313, 323-325`

```js
const normalizeTrialIdentity = (value) => String(value || '').trim().slice(0, 255);

app.post('/api/v1/trial/start', rateLimit({ windowMs: 60_000, max: 5 }), async (req, res) => {
    ...
    const deviceId = normalizeTrialIdentity(req.body?.deviceId || req.body?.device_id);
    const deviceFingerprint = normalizeTrialIdentity(req.body?.deviceFingerprint || req.body?.device_fingerprint);
    const deviceName = normalizeTrialIdentity(req.body?.deviceName || req.body?.device_name || 'Android');
```

**2) The attempted Manager prohibition is conditional on the caller sending a valid `Authorization` header. Omitting that header skips the check.**

`backend/server.js:313-322`

```js
const authHeader = String(req.headers.authorization || '');
if (/^Bearer\s+\S+$/i.test(authHeader)) {
    try {
        const decoded = jwt.verify(authHeader.slice(7).trim(), JWT_SECRET, { algorithms: ['HS256'] });
        if (['MANAGER', 'SUPER_MANAGER'].includes(decoded?.role)) {
            return res.status(403).json({ ... code:'MANAGER_TRIAL_FORBIDDEN', ... });
        }
    } catch (_) {}
}
```

**3) The only server-side reuse check is equality on those two caller-supplied strings, followed by issuance for a previously unseen pair.**

`backend/server.js:331-348`

```js
const blocked = await client.query(
  'SELECT 1 FROM trial_device_blocks WHERE (device_id = $1 AND device_fingerprint = $2) OR device_id = $1 OR device_fingerprint = $2 LIMIT 1',
  [deviceId, deviceFingerprint]
);
...
const existing = await client.query(
  'SELECT device_id, device_fingerprint, started_at, expires_at, status FROM trial_devices WHERE device_id = $1 OR device_fingerprint = $2 ...',
  [deviceId, deviceFingerprint]
);
...
await client.query(
  'INSERT INTO trial_devices (device_id, device_fingerprint, device_name, started_at, expires_at, status) VALUES ($1, $2, $3, $4, $5, \'ACTIVE\')',
  [deviceId, deviceFingerprint, deviceName, startedAt, expiresAt]
);
return res.status(201).json({ success: true, trialActive: true, ... });
```

**4) The Android client treats a successful server trial response as active trial state.**

`app/src/main/java/com/example/ui/GameNetViewModel.kt:4773-4776` stores the successful trial response and reports success; `GameNetViewModel.kt:4653-4675` sets `_isSubscribed` true for a non-expired cached `TRIAL` plan and records role `TRIAL_USER`.

### Safe local reproduction / expected vs. actual

The following deterministic static test was run locally and passed; it verifies that the public route reads both identities from the body, inserts a trial record, performs the Manager check only inside the optional bearer-header branch, and has no server logout route:

```text
PASS static trial-identity/auth-header, no-server-logout, and 4-character-password-policy assertions
```

A safe integration reproduction (only in a disposable **local** test database/runtime) is:

```bash
# No Authorization header. Use a fresh arbitrary pair each time.
curl -i -X POST http://127.0.0.1:3000/api/v1/trial/start \
  -H 'Content-Type: application/json' \
  --data '{"deviceId":"audit-new-id-1","deviceFingerprint":"audit-new-fp-1","deviceName":"audit"}'
```

- **Expected:** a previously licensed/Manager-associated physical device cannot obtain another trial, and a physical device is limited to one trial.
- **Actual by current control flow:** any fresh pair of strings reaches the insert and `201` response. An authenticated Manager can omit `Authorization`; the `MANAGER_TRIAL_FORBIDDEN` branch is then not evaluated.

### Impact

An actor can repeatedly obtain server-issued 24-hour trial state by generating new `deviceId` and `deviceFingerprint` values. The five-per-minute IP limiter throttles only request rate; it does not make the identity trustworthy or bind it to a device. This defeats the intended one-trial-per-device control and the explicit Manager-trial check.

**Bounded impact:** manager data-changing endpoints reviewed here still carry `requireManagerAuth` plus `requireActiveEntitlement`; this finding does not itself prove access to those protected server routes without an entitlement.

### Root cause

The authoritative entitlement decision is based on a mutable client assertion (`deviceId`/`deviceFingerprint`). The optional bearer token is used as if it could establish that a request is not from a Manager; callers can simply not send it.

### Recommended fix

1. Do **not** treat client-provided device IDs/fingerprints as a durable entitlement identity.
2. Bind a trial to a server-issued installation credential stored in Android Keystore, and require a hardware/app-attestation signal appropriate to the deployment (for example Play Integrity) before first issuance. Verify it server-side.
3. If trial eligibility is account-based, require authenticated account creation/login and record one trial per verified account plus an attested installation.
4. Resolve the Manager policy from an authenticated credential or a server-side installation-to-account association; do not use absence of `Authorization` as evidence of non-Manager status.
5. Keep the database uniqueness/block controls, but make them enforce an identity the client cannot freely mint.

### Regression and verification

Add runtime tests against a disposable local PostgreSQL instance:

- A Manager token cannot start a trial; omitting its token from the same attested installation also cannot start one.
- Replaying a changed `deviceId`/`deviceFingerprint` with the same attested installation is rejected.
- A different installation cannot reuse an installation credential.
- A blocked/deleted trial remains blocked (existing coverage already exercises this last case).
- Verify protected manager routes remain entitlement-gated after trial changes.

---

## SEC-02 — Logout and password changes do not revoke issued 24-hour bearer JWTs

| Property | Assessment |
|---|---|
| Severity | **Medium** |
| Confidence | **High** |
| CWE | CWE-613 (insufficient session expiration), CWE-384 (session invalidation weakness) |
| Affected flow | Manager/customer login → bearer token use → local Android logout or password update |

### Exact evidence

**1) Manager and customer access tokens are signed for 24 hours.**

`backend/server.js:242-243`

```js
const token = jwt.sign(
  { id: manager.id, managerId: manager.id, role: manager.role || 'MANAGER' },
  JWT_SECRET,
  { expiresIn: '24h' }
);
```

`backend/server.js:270-271` and `backend/server.js:291-292` issue a 24-hour customer token after registration and login respectively.

**2) The authorization middleware verifies signature/expiry and current account existence, but has no session ID, deny-list, token version, logout state, password-change timestamp, or `auth_epoch` check.**

`backend/server.js:87-108` (`requireManagerAuth`):

```js
const decoded = jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] });
...
const account = await pool.query(
  "SELECT id, role FROM managers WHERE id = $1 AND role = ANY($2::text[]) LIMIT 1",
  [decoded.id, ['MANAGER', 'SUPER_MANAGER']]
);
if (!account.rows.length) return res.status(401).json({ error: 'Manager account no longer exists' });
req.user = decoded;
next();
```

`backend/server.js:112-129` uses the same model for customers: signature/expiry plus existence/tenant lookup only.

**3) Android logout only clears local state; it makes no server request.**

`app/src/main/java/com/example/ui/GameNetViewModel.kt:5585-5605`

```kotlin
fun logout(onComplete: (() -> Unit)? = null) {
    viewModelScope.launch(Dispatchers.IO) {
        encryptSetting("enc_auth_token", "")
        ...
        NetworkClient.managerAuthToken = null
        NetworkClient.customerAuthToken = null
        encryptSetting("enc_customer_auth_token", "")
        _authState.value = AuthState.Unauthenticated
```

A repository-wide route check found no backend `logout` route. Password-setting flows update `password_hash` (for example `backend/canonical_routes.js:587`) but do not update any token-version/revocation field; no such field/check is present in middleware.

### Safe local reproduction / expected vs. actual

The deterministic static test above passed and asserted both the absence of a backend logout route and the client-only token clearing.

A local-only integration verification sequence after adding disposable fixtures is:

```bash
# 1. Login and retain TOKEN outside the client.
# 2. Call Android logout (or clear its token storage).
# 3. Replay the retained token to a protected route.
curl -i http://127.0.0.1:3000/api/v1/manager/customers \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-Manager-ID: $MANAGER_ID"
```

- **Expected:** replay after logout or password reset returns `401`/`403`.
- **Actual by current middleware:** the token is accepted until its JWT `exp` (24 hours), provided its account still exists and, for manager routes, the matching `X-Manager-ID` is supplied.

### Impact

A copied/stolen bearer token remains usable for up to 24 hours after the user believes they logged out. Password updates do not force reauthentication, so a previously stolen token remains effective through a password change. For managers this window covers manager-scoped reads and mutations that also pass the entitlement gate; for customers it covers their self-service resources.

### Root cause

The design uses self-contained access JWTs as the entire session, while logout is only local credential deletion. There is no server-side session record or account-level invalidation state.

### Recommended fix

1. Introduce a `sessions`/refresh-token table with a random, hashed refresh credential, expiry, account ID, device metadata, revoked timestamp, and optionally a JWT `jti`.
2. Use short-lived access JWTs (for example 5–15 minutes) and a rotating refresh token. Revoke the refresh session at logout.
3. Add an `auth_epoch`/`token_version` (or `password_changed_at`) to `managers` and `customers`; place it in claims and compare it in middleware, or query it on verification. Increment it on logout-all, password change/reset, account archive/deletion, and sensitive role change.
4. Provide `POST /api/auth/logout` and `POST /api/auth/logout-all`, both idempotent and rate-limited. Do not log submitted tokens.
5. Preserve the existing algorithm pinning and account-existence checks.

### Regression and verification

- Token works before logout and fails immediately after logout.
- Two sessions for one account: logout-current revokes only one; logout-all revokes both.
- Password update invalidates all older tokens for manager and customer accounts.
- Expired access token is rejected; valid refresh rotates once and a replayed old refresh token is rejected.
- Account archival/deletion continues to reject prior tokens.

---

## SEC-03 — Password policy permits trivial customer passwords and a one-character Manager password update

| Property | Assessment |
|---|---|
| Severity | **Medium** |
| Confidence | **High** |
| CWE | CWE-521 (weak password requirements) |
| Affected flow | Public customer registration; manager-created/updated customers; Super Manager update of an existing Manager |

### Exact evidence

**1) Public customer registration accepts a four-character password and immediately issues a customer JWT.**

`backend/server.js:251-271`

```js
const password = typeof req.body?.password === 'string' ? req.body.password : '';
if (!phoneNumber || !managerId || !fullName || password.length < 4) {
    return res.status(400).json({ error: '... password of at least 4 characters are required' });
}
...
const hash = await bcrypt.hash(password, 12);
...
const token = jwt.sign({ id: customer.id, managerId, role: 'CUSTOMER' }, JWT_SECRET, { expiresIn: '24h' });
```

**2) The authenticated Manager customer-create/update path also hashes any password of length at least four.**

`backend/canonical_routes.js:108-116`

```js
if(typeof b.password==='string' && b.password.length>=4)
  await pool.query(
    'UPDATE customers SET password_hash=$1 WHERE id=$2 AND manager_id=$3',
    [await bcrypt.hash(b.password,12),q.rows[0].id,mid]
  );
```

**3) Super Manager update accepts any non-empty Manager password (including one character) and uses a lower bcrypt cost than the other paths.**

`backend/server.js:636-665`

```js
const password = req.body?.password ?? null;
...
if (password !== null && String(password).trim()) {
    const bcrypt = require('bcrypt');
    fields.push(`password_hash = $${i++}`);
    values.push(await bcrypt.hash(String(password), 10));
}
```

bcrypt is correctly used rather than plaintext storage, but bcrypt cannot compensate for accepting a trivially guessable secret.

### Safe local reproduction / expected vs. actual

The passed deterministic static test asserted the literal public registration minimum (`password.length < 4`) and Manager customer-path condition (`b.password.length>=4`).

For a disposable local runtime, the public path can be verified without attacking any account:

```bash
curl -i -X POST http://127.0.0.1:3000/api/auth/customer/register \
  -H 'Content-Type: application/json' \
  --data '{"phone_number":"audit-unique-phone","manager_id":"LOCAL_TEST_MANAGER","full_name":"Audit Fixture","password":"abcd"}'
```

- **Expected:** a centrally defined, strong policy rejects short/trivial passwords on every account creation and change path.
- **Actual by code:** `abcd` passes the public registration check and is bcrypt-hashed; an existing Manager can be set to any nonblank password through the Super Manager update route.

### Impact

Four-character customer passwords and one-character Manager passwords are susceptible to online guessing if credentials or identifiers become known. Per-IP login limiting (`10/minute` for customer login at `backend/server.js:278`) reduces but does not eliminate distributed guessing risk. The Manager update condition can downgrade an account that protects tenant data and financial operations.

### Root cause

Password validation is duplicated and inconsistent. It is based only on minimal string length, and the privileged account-update branch has no length requirement at all.

### Recommended fix

1. Implement one server-side `validatePassword()` used by registration, customer update, subscription setup, bootstrap, Manager create, and Manager update.
2. Require a length-based policy appropriate for the product (for example 12+ characters; permit long passphrases) and reject known-compromised passwords using a privacy-preserving breached-password check or local deny list.
3. Do not require arbitrary composition rules; normalize carefully and preserve Unicode/passphrases rather than silently trimming the credential.
4. Raise bcrypt cost consistently after performance calibration (the application already uses cost 12 in several interactive paths). Rehash at the stronger cost after successful login.
5. Pair this with the session invalidation fix in SEC-02 so password changes revoke old tokens.

### Regression and verification

- Test `abcd`, `1234`, and an empty/whitespace-only password against every creation/update/reset route; each must fail.
- Test a compliant passphrase for each route; each must succeed and store only a bcrypt hash.
- Assert no API response or log includes the plaintext password or its bcrypt hash.
- Assert a Manager update cannot reduce password strength below the shared policy.

---

## SEC-04 — Customer login reveals account state through distinct authentication errors

| Property | Assessment |
|---|---|
| Severity | **Low** |
| Confidence | **High** |
| CWE | CWE-204 (observable response discrepancy) |
| Affected flow | `POST /api/auth/customer/login` |

### Exact evidence

`backend/server.js:278-295`:

```js
const result = await pool.query(
  "SELECT * FROM customers WHERE phone_number = $1 AND manager_id = $2 AND COALESCE(description,'') NOT LIKE '[GAMENEX_ARCHIVED:%'",
  [phone_number, manager_id]
);
const customer = result.rows[0];
if (!customer) return res.status(401).json({ error: 'Customer not found' });
if (!customer.password_hash) return res.status(401).json({ error: 'Customer password is not configured' });
const valid = await bcrypt.compare(password, customer.password_hash);
if (!valid) return res.status(401).json({ error: 'Invalid credentials' });
```

The endpoint has a useful per-IP rate limit (`rateLimit({ windowMs: 60_000, max: 10 })` at line 278), but the three distinct 401 bodies remain observable.

### Safe local reproduction / expected vs. actual

With local disposable fixtures for a known `manager_id`, submit the same wrong password for:

1. a nonexistent phone number,
2. an existing customer with `password_hash IS NULL`, and
3. an existing customer with a password hash.

- **Expected:** each response has the same status/body, such as `401 {"error":"Invalid credentials"}`.
- **Actual:** the server returns `Customer not found`, `Customer password is not configured`, or `Invalid credentials`, revealing both account existence and provisioning status.

### Impact

An attacker who knows or guesses a Manager ID can enumerate which phone numbers correspond to customer accounts and identify accounts whose password has not yet been configured. This improves targeted phishing and password-guessing campaigns. No cross-tenant record read is demonstrated: the query is correctly scoped by `manager_id`.

### Root cause

Authentication failure handling exposes internal account state to the unauthenticated caller.

### Recommended fix

1. Return one generic `401` body for all credential failures, including nonexistent, archived, unprovisioned, and incorrect-password accounts.
2. Consider a dummy bcrypt comparison when no hash exists to reduce easily measurable timing differences.
3. Keep the existing rate limit; add account/phone-plus-tenant throttling and monitoring for distributed attempts.
4. Show password-provisioning instructions only after a separately authenticated or verified recovery/onboarding flow.

### Regression and verification

- For nonexistent, unprovisioned, archived, wrong-password, and valid accounts, assert all failures have exactly identical status, body, headers (aside from normal request IDs), and comparable timing envelope.
- Assert valid login still returns only the intended token/customer fields.

---

# Detailed review notes by requested area

## Authentication, bcrypt, JWT, roles, expiry, and logout

### Confirmed protections

- **JWT secret is mandatory:** `backend/server.js:62-64` loads `process.env.JWT_SECRET` and fails startup when absent. The committed `.env.example` contains placeholders; no tracked real credential was found in a local committed-file pattern scan.
- **Algorithm pinned:** `jwt.verify(..., { algorithms: ['HS256'] })` is used in Super Manager, Manager, and Customer middleware (`backend/server.js:68`, `94`, `119`), preventing algorithm-confusion acceptance.
- **Role and identity binding:** `requireManagerAuth` requires `MANAGER`/`SUPER_MANAGER`, nonempty `id` and `managerId`, equality of those two claims, and matching `X-Manager-ID` (`server.js:94-103`). `requireCustomerAuth` requires `CUSTOMER`, `id`, and `managerId` and verifies the customer row belongs to that tenant (`server.js:119-124`).
- **Database account existence checks:** deleted/archived customer rows and deleted manager rows do invalidate tokens through the middleware queries, even though normal logout and password change do not (SEC-02).
- **bcrypt:** customer registration/login uses `bcrypt.hash(..., 12)` and `bcrypt.compare` (`server.js:264`, `288`); subscription activation/password setup also validates a bcrypt-hashed random activation secret (`canonical_routes.js:559`, `587`).
- **Super Manager controls:** `requireSuperManagerAuth` checks the signed role, matching `id/managerId`, database role, and an active Super Manager entitlement (`server.js:45-85`). Super Manager routes reviewed carry this middleware.

### Required remediation

Apply SEC-02 and SEC-03. Ensure session revocation occurs on Manager deletion/archive, customer archive, credential reset, role changes, and device unbinding where product policy requires it.

## Manager A/B isolation, IDOR/BOLA, and Super Manager

No confirmed Manager A-to-B IDOR/BOLA was found in the reviewed current routes. Representative evidence:

- Manager customer listing derives tenant from authenticated claims and filters `manager_id=$1` (`backend/canonical_routes.js:102-105`).
- Customer archive validates `id=$1 AND manager_id=$2` before updating (`canonical_routes.js:121-135`).
- Customer reservation cancellation passes the authenticated `managerId` and `customerId` to `cancelReservation` (`canonical_routes.js:386`).
- Financial cancellation locks reservations with `WHERE id = $1 AND manager_id = $2` and adds `customer_id = $3` for customer cancellation (`backend/financialService.js:155-325`).
- Manager payment-method patch scopes its update by method ID **and** `manager_id` (`canonical_routes.js:511`).
- Super Manager Manager updates/delete routes explicitly reject target roles of `SUPER_MANAGER` (`backend/server.js:636-681`).

`backend/test_isolation_static.js` passed and asserts tenant claim use, Manager-scoped customer/reservation lookups, customer-cancellation owner passing, and required customer login password verification. The more comprehensive `test_security_isolated.js` has meaningful A/B fixtures and JWT-negative cases, but it requires a local PostgreSQL instance and an API on `127.0.0.1:3000`; neither was present in this safe sandbox review.

### Public registration note (not classified as a defect)

`POST /api/auth/customer/register` is unauthenticated and accepts a caller-supplied `manager_id` (`server.js:251-271`). It verifies only that a Manager exists before creating a customer and returning a customer token. This is a deliberate-looking public onboarding route, but the repository supplies no policy saying enrollment must be invite-only or manager-approved. It is therefore **not counted as a vulnerability** in this report. If enrollment should be restricted, bind it to an invitation/verified club identifier and add a test that arbitrary Manager IDs cannot be selected.

## Mass assignment and injection

- Reviewed PostgreSQL data access uses `$n` parameter binding for user-controlled values. The static scan found no query template with interpolated request values.
- The one dynamic SQL construction is `backend/server.js:650-665`, where `fields.join(', ')` is built only from a fixed local allowlist (`display_name`, `gamenet_name`, `plan_type`, `subscription_status`, `payment_status`, `password_hash`) and every value remains parameterized. This is **not** SQL injection as implemented.
- Manager/customer methods generally map explicit request fields rather than spreading entire bodies into persistence. The Super Manager route uses an allowlist. No confirmed mass-assignment privilege escalation was found.
- Continue to validate numeric ranges/types in configuration and financial routes; do not downgrade this code review into a claim of formal completeness.

## CORS and CSRF relevance

- CORS is an origin allowlist, not wildcard CORS: `backend/server.js:12-20` accepts requests with no browser Origin (native clients) or origins present in `CORS_ORIGINS`, and rejects other origins.
- The API uses explicit `Authorization: Bearer` headers rather than ambient cookies. Consequently, conventional browser CSRF is **not a primary applicable threat** to reviewed authenticated routes; browsers do not automatically attach the bearer token.
- Keep `CORS_ORIGINS` explicitly configured in production. Do not switch to `origin: true`/`*` when credentials or browser sessions are introduced.

## Sensitive data, PII, logging, and mobile storage

- Server logging reviewed does not log request bodies, bearer tokens, plaintext passwords, or bcrypt hashes. It logs operational error objects/messages in some catch blocks; retain the current practice and add a centralized redactor before any future structured request logging.
- `app/src/main/java/com/example/data/network/NetworkLogger.kt:126-186` records path-only URLs and explicitly says it does not retain raw response bodies; it limits failed-response extraction to safe `code`/`error` fields. The in-memory log has a 150-entry cap (`NetworkLogger.kt:40-64`). No persisted token/PII log was confirmed.
- `AndroidManifest.xml:14-20` sets `android:allowBackup="false"` and `android:usesCleartextTraffic="false"`; `network_security_config.xml:2-7` trusts system CAs and disallows cleartext. Debug HTTP logging is disabled in release (`SelfHostedManager.kt:150-155`).
- Android encrypted-setting helpers should remain Keystore-backed and should never fall back to plaintext. This review found no committed production secrets or signing keys; `.env`, `.jks`, `.keystore`, `.p12`, `.pem`, and `local.properties` are ignored and none were tracked.

## Trial/subscription

- **Trial:** SEC-01 is confirmed.
- **Subscription activation:** `canonical_routes.js:559` requires a `PENDING_<id>` code, the buyer-bound device ID, a bcrypt-validated random activation secret, a confirmed request, and active entitlement; it serializes device binding under a lock and enforces max device count. No subscription activation bypass was confirmed.
- **Set password:** `canonical_routes.js:587` requires a confirmed request, bcrypt secret comparison, phone-or-device match, and consumes (`NULL`s) the activation-secret hash on success. `backend/test_activation_runtime.js` includes checks for invalid secret rejection and secret consumption/replay rejection, but needs the absent local DB/API to execute.
- **Manual approval:** `backend/test_subscription_manual.js` checks the removed legacy automatic confirmation route and that a purchase stays pending; static source confirms that automatic confirmation is not present.

## Hardcoded credentials, debug/test bypasses, and dependencies

- `backend/bootstrap.js:9-40` requires `SUPER_ADMIN_PASSWORD` (minimum 16 chars on first bootstrap) and bcrypt-hashes it. The fallback username `superadmin` is predictable but is not a credential; retain a strong required password and consider requiring an explicit bootstrap username in production.
- No committed secret matching the local audit patterns was found. The only match was the safe `const JWT_SECRET = process.env.JWT_SECRET` reference.
- No `eval`, `child_process`, `exec`, `spawn`, test authentication bypass, or hardcoded authentication secret was found in the tracked application source searched.
- `npm audit --omit=dev --json` reported **0 vulnerabilities** across 105 production dependencies at audit time. This is point-in-time dependency metadata, not a replacement for continuous dependency monitoring.

# Tests and commands run

All were local-only and did not start or attack an external runtime.

| Command / activity | Result |
|---|---|
| `git -C /home/ubuntu/GameNexa-release-2026 rev-parse HEAD` and status check | PASS — exact requested commit; clean tree |
| `npm run check` | PASS — `server.js`, `canonical_routes.js`, `reservationService.js`, `financialService.js` parse |
| `node backend/test_isolation_static.js` | PASS — tenant-isolation static checks |
| `python3 tests/android_backend_contract_check.py` | PASS — server-authoritative session restoration and 51 Retrofit / 76 raw route contracts |
| `python3 tests/security_tests.py` | Exit 0, but file is only a placeholder comment; **no executable security assertion** |
| Deterministic local Node static assertions for SEC-01/02/03 | PASS — verified trial request identity/auth-header structure, no server logout route, local token clearing, and 4-character policy |
| Parameterization/dynamic-SQL static scan | No user-controlled dynamic SQL confirmed; one fixed field allowlist construction reviewed |
| `npm audit --omit=dev --json` | PASS — 0 reported vulnerabilities |
| `git fsck --no-reflogs --no-progress` | PASS |
| Committed secret-pattern scan | No real secret found; environment variable reference only |
| Local runtime availability check | No `DATABASE_URL`, Docker, PostgreSQL binaries, or listeners on 3000/5432; DB-backed integration suite intentionally not run |

## Test coverage gaps to close

1. Make `tests/security_tests.py` real or remove it; it currently only contains a placeholder comment.
2. Run `npm run test:release` in CI with an ephemeral PostgreSQL database and a locally started API. The scripts include valuable runtime coverage but cannot run without those fixtures.
3. Add regression tests for every finding above, especially trial identifier forgery/optional-bearer omission and post-logout/password-change token replay.
4. Add a route-level authorization matrix test that creates Manager A, Manager B, Customer A, Customer B, and Super Manager fixtures and tests every ID-bearing endpoint with foreign IDs.
5. Add secret-scanning and dependency-audit jobs to CI; fail release builds on tracked credentials or high/critical advisories after triage.

# Verification plan / remediation order

1. **First:** redesign trial identity and add server-verifiable proof (SEC-01).
2. **Second:** add revocable sessions/token epoch and revoke on logout/password change (SEC-02).
3. **Third:** centralize and enforce the password policy across all flows (SEC-03).
4. **Fourth:** make login failures generic and add account-plus-tenant rate limiting (SEC-04).
5. Run the full database-backed release suite, then specifically replay the regression tests listed under each finding.

> This report distinguishes observed defects from review observations. The absence of a finding in a category means no issue was confirmed from the audited revision and safe local analysis; it is not a claim of absence under unreviewed deployment configuration, database privileges, infrastructure, or future code changes.


### 8.4 CI/CD, Runtime and Test Audit

# GameNexa CI/CD, Build/Release, Runtime, and Test Audit

**Audited revision:** `05348f2df8b1f6c30ee233071be13a899d969e25` on `main`  
**Repository:** `mkhas1374/GameNexa-release-2026`  
**Audit date:** 2026-10-02  
**Scope:** all three tracked GitHub Actions workflows; Gradle and Android configuration; npm/package lock and backend test scripts; Docker/Compose/nginx; root and backend environment templates; README/status/package manifests; schema/bootstrap; and every tracked test source/assertion.

> **Release verdict: not release-ready.** The exact commit has a successful **debug** GitHub Actions run, but its **release-validation run failed before signed release artifacts were built**. Backend integration tests, Docker build/startup, migrations, and deployment are not part of CI. The locally executable passing checks are mostly syntax, source-text contracts, and a mocked arithmetic test—not evidence of a running backend or end-to-end release.

## 1. What was verified

### Revision and GitHub Actions execution

* Local `HEAD` and `origin/main` both resolve to the audited SHA. The worktree was clean when inspected.
* GitHub workflow definitions are active:
  * **GameNexa Android Debug Build** (workflow ID `370703256`)
  * **GameNexa Android Release Validation** (workflow ID `371735558`)
  * **Delete Old Workflow Runs** (workflow ID `372641750`)
* GitHub Actions records for this exact SHA:

| Workflow | Run | Event | Result | What this proves / does not prove |
|---|---:|---|---|---|
| Android Debug Build | [`36964812531`](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/36964812531) | push | **success** | The two Python source-contract scripts, `testDebugUnitTest`, and `assembleDebug` completed. Log shows `BUILD SUCCESSFUL` and uploaded `GameNexa-debug-apk` (30,018,909 bytes). It does **not** run `npm test`, Docker, Postgres, Android instrumentation, a real device, or deployment. |
| Android Release Validation | [`36964812496`](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/36964812496) | push | **failure** | The job failed at **“Require stable release signing credentials.”** The subsequent restore-keystore, signed `assembleRelease bundleRelease`, and release artifact upload steps were all skipped. The retrieved job metadata does not reveal which masked credential was absent; it establishes that the required-signing gate did not pass. |

The `RELEASE_AUDIT_STATUS.md` statement that `npm test` passed and Docker services were healthy is dated 2026-09-29 (lines 3, 6–16). It is not a current CI result for this commit and cannot override the failed release-validation run.

### Commands actually run locally

| Command | Result | Interpretation |
|---|---|---|
| `cd backend && npm run check` | **PASS** | Parses only `server.js`, `canonical_routes.js`, `reservationService.js`, and `financialService.js`. |
| `cd backend && node test_isolation_static.js` | **PASS** | String-presence assertions only. |
| `cd backend && node test_financial_mock.js` | **PASS** | Uses a hand-written mocked `query()` implementation; no DB or server. |
| `python3 tests/android_backend_contract_check.py` | **PASS**: 51 Retrofit + 76 raw paths | Static scanning of Android/backend source. |
| `python3 tests/deep_release_audit.py` | **PASS**: 43 invariants | Static substring checks of source files. |
| `node --check backend/test_*.js` | **PASS** | Syntax validation only. |
| `python3 -m py_compile tests/*.py` | **PASS** | Syntax compilation only; does not execute `cert_suite.py` side effects. |
| `env -i ... npm test` (no credentials; bounded to 45s) | **FAIL (expected prerequisite failure)** | First two static/mock tests passed; `test_reservation.js` then failed `ECONNREFUSED` on `::1:5432` / `127.0.0.1:5432`. No local Postgres or test server exists. |
| `./gradlew testDebugUnitTest --stacktrace --no-daemon` (bounded to 300s) | **FAIL** | Environment limitation: `SDK location not found`; no `ANDROID_HOME` or `local.properties` SDK path. This is not a source-test failure. The exact GitHub run did complete this task. |
| `docker compose -f backend/docker-compose.yml config` with placeholders | **NOT RUN** | Docker CLI is absent in the audit sandbox (`docker: command not found`). No containers were started. |
| `curl --fail ... https://api.gamenermayket.ir/api/v1/time` | **HTTP 200** | At audit time the public endpoint responded through nginx with `{"serverTime":1790920443840,"timezone":"Asia/Tehran"}` and HSTS. It cannot identify the deployed Git commit, schema version, container health, secret configuration, or functional correctness. |

The optional Python/Ruby/Node YAML parsing libraries were not installed in the sandbox. This is not a workflow defect: GitHub accepted and executed both Android workflow YAML files on this SHA.

## 2. Findings (16)

Severity reflects release and operational risk, not a claim of active exploitation.

### F-01 — Release gate is currently failing; no signed APK/AAB was produced

**Severity: Blocker**

**Evidence**

* The exact release run `36964812496` concluded `failure`; job metadata names the failed step **“Require stable release signing credentials.”** All signing, release-build, and release-upload steps were skipped.
* The gate explicitly refuses a release without all four secrets:

```yaml
# .github/workflows/android-release-validation.yml:66-92
66  - name: Require stable release signing credentials
68    GAMENEXA_RELEASE_KEYSTORE_BASE64: ${{ secrets.GAMENEXA_RELEASE_KEYSTORE_BASE64 }}
69    GAMENEXA_STORE_PASSWORD: ${{ secrets.GAMENEXA_STORE_PASSWORD }}
70    GAMENEXA_KEY_ALIAS: ${{ secrets.GAMENEXA_KEY_ALIAS }}
71    GAMENEXA_KEY_PASSWORD: ${{ secrets.GAMENEXA_KEY_PASSWORD }}
73    test -n "$GAMENEXA_RELEASE_KEYSTORE_BASE64" || { ...; exit 1; }
78  - name: Restore stable release keystore
82    echo "$GAMENEXA_RELEASE_KEYSTORE_BASE64" | base64 --decode > "$RUNNER_TEMP/gamenexa-release.jks"
85  - name: Build signed Release APK and AAB with stable signing
92    ./gradlew assembleRelease bundleRelease --stacktrace
```

**Impact:** There is no verified, signed release APK/AAB for this SHA. Debug APK success must not be described as release validation.

**Action:** Configure all four repository/environment secrets, preferably expose them only to a protected release environment and tag/manual-release workflow; rerun and retain the signed build result. Verify signing certificate fingerprint and app/version metadata in the generated artifacts.

---

### F-02 — CI does not execute backend tests, dependency installation, Docker build, database migration, or deployment

**Severity: High**

**Evidence**

Both Android workflows run the same narrow checks:

```yaml
# .github/workflows/android-debug.yml:46-56
46  - name: Check Android/backend API contract
47    run: python3 tests/android_backend_contract_check.py
49  - name: Run deep Android/backend/VPS contract audit
50    run: python3 tests/deep_release_audit.py
52  - name: Validate backend JavaScript syntax
53    run: node --check backend/server.js && node --check backend/canonical_routes.js && node --check backend/financialService.js
55  - name: Run JVM tests
56    run: ./gradlew testDebugUnitTest --stacktrace
```

`android-release-validation.yml:45-55` has the same checks. Neither workflow contains `npm ci`, `npm test`, `docker build`, `docker compose`, schema migration, image publication, SSH/VPS deployment, or post-deploy smoke test.

**Impact:** A green debug workflow is not evidence that the Node service starts, dependency lock resolves, PostgreSQL schema works, tests pass, an image builds, or the VPS is updated.

**Action:** Add an isolated backend job: `npm ci`, lint/test syntax, a disposable Postgres service, schema/migration application, local API startup, and integration tests. Add a separate container build with a digest/SBOM/provenance policy. Make deployment and post-deploy health/contract smoke tests explicit protected jobs rather than inferred from Android CI.

---

### F-03 — The declared backend test suite is integration/destructive and is not CI-safe as written

**Severity: High**

**Evidence**

* `backend/package.json:5-8` defines `npm test` as a chain of 12 scripts, most of which use `pg` and/or `fetch`; no CI job invokes it.
* `backend/test_reservation.js:5, 12-20` opens a `pg.Pool` from `DATABASE_URL` and inserts fixed `mgr_res`, customer IDs `888`, and station IDs `888/889`. It only deletes prior reservations at line 14 and does not clean the fixed manager/customer/station fixtures at completion.
* `backend/test_security_isolated.js:4-11` targets the public production-looking hostname while also mutating the database:

```js
const BASE='https://api.gamenermayket.ir';
const pool=new Pool({connectionString:process.env.DATABASE_URL});
// ... inserts managers/customers/reservations, then fetches BASE+path
```

* `test_financial.js`, `test_cancellation_boundaries.js`, `test_subscription_entitlement.js`, `test_activation_runtime.js`, `test_sm_auth.js`, and `test_sm_routes.js` also insert/delete rows and expect a separately running API at `127.0.0.1:3000` or the public host.
* The safe credential-free run confirmed the prerequisite coupling: after the static/mock tests, `test_reservation.js` aborted with `ECONNREFUSED` on port 5432.

**Impact:** These are not hermetic tests. Running them against a shared, staging, or production database can mutate data; running them in CI lacks the database/API setup and does not happen at all. Cleanup errors are often swallowed (`catch{}`), compounding residue risk.

**Action:** Split source-only tests from integration tests. Provision an ephemeral Postgres database per CI run, load migrations, start the API from the checked-out source on a randomized local port, use generated fixture namespaces, and clean up in `finally` with failures surfaced. Never point a test at the public production hostname.

---

### F-04 — Docker Compose cannot initialize a fresh database; the schema and volume are external assumptions

**Severity: High**

**Evidence**

```dockerfile
# backend/Dockerfile:1-6
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

```yaml
# backend/docker-compose.yml:54-57
volumes:
  pgdata:
    name: vps-backend_pgdata
    external: true
```

* `backend/schema.sql` exists but Compose contains no `schema`, `initdb`, `command`, or `entrypoint` reference, and it is not mounted into PostgreSQL’s initialization directory.
* `server.js:11` imports `bootstrap.js`, but `bootstrap.js:14-16` first queries the already-existing `managers` table; it does not create tables or load `schema.sql`.
* A fresh host also requires a pre-created external volume named `vps-backend_pgdata`.

**Impact:** `docker compose up` is not a reproducible fresh deployment. On a blank PostgreSQL volume, required tables will not exist. This is also inconsistent with the source-package claim that the package is self-hosted without a documented restore/migration procedure.

**Action:** Introduce versioned, idempotent migrations and run them in a one-shot migration job before API startup. Either create/manage the volume in Compose or document and validate the external-volume backup/restore prerequisite. Fail API readiness if the expected schema version is unavailable.

---

### F-05 — Runtime environment templates do not document all required variables

**Severity: High**

**Evidence**

* Compose interpolates both `${DB_PASSWORD}` and `${DATABASE_URL}` (`backend/docker-compose.yml:8,27`).
* The API creates its database pool only from `DATABASE_URL` (`backend/server.js:47-49`) and fails if `JWT_SECRET` is absent (`server.js:62-63`).
* Runtime also reads `CORS_ORIGINS`, `JSON_BODY_LIMIT`, `SUPER_ADMIN_PASSWORD`, and `SUPER_ADMIN_USERNAME` (`server.js:12,21`; `bootstrap.js:9-10`).
* Neither `.env.example` nor `backend/.env.example` documents `DATABASE_URL`, `CORS_ORIGINS`, `SUPER_ADMIN_PASSWORD`, `SUPER_ADMIN_USERNAME`, or `JSON_BODY_LIMIT`; the backend template instead lists disconnected `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, and `DB_NAME` values (lines 1-10).

**Impact:** Copying either supplied example will not produce a documented working Compose environment. An omitted `CORS_ORIGINS` rejects browser origins (while allowing requests with no `Origin` header), and a missing `DATABASE_URL` leaves the API unusable.

**Action:** Publish one non-secret, complete environment contract with required/optional status, accepted formats, a correctly URL-encoded `DATABASE_URL` example, the production CORS origins, bootstrap behavior, and secret provisioning method. Validate required configuration at process start with clear non-secret diagnostics.

---

### F-06 — API health can be green while the database/schema is unusable

**Severity: High**

**Evidence**

```yaml
# backend/docker-compose.yml:31-36
healthcheck:
  test: ["CMD-SHELL", "wget -q -O - http://127.0.0.1:3000/api/v1/time >/dev/null"]
  interval: 30s
```

```js
// backend/server.js:302-307
app.get('/api/v1/time', rateLimit({ windowMs: 60_000, max: 60 }), (req, res) => {
  res.set('Cache-Control', 'no-store, no-cache, must-revalidate');
  return res.json({ serverTime: Date.now(), timezone: 'Asia/Tehran' });
});
```

The endpoint does not query PostgreSQL. Furthermore, the initial expiry DB task catches query failures (`server.js:1480-1484`) and resolves; its `.then()` then starts the listener (`1488-1491`). Nginx only waits for this API health check (`docker-compose.yml:49-51`).

**Impact:** Compose can report the API healthy and start nginx even when the database schema is missing or runtime database queries are failing. The public 200 obtained in this audit establishes only reachability of that route.

**Action:** Add `/healthz` (process) and `/readyz` (bounded database query + schema version) endpoints. Point Compose/nginx/deployment readiness to `/readyz`; alert on DB connection/schema failures.

---

### F-07 — Android version metadata is contradictory; artifacts build as 2 / 1.0.1, not documented properties 4 / 1.0.2

**Severity: High**

**Evidence**

```kotlin
// app/build.gradle.kts:13-22
// Read versionCode and versionName dynamically from gradle.properties
val appVersionCode = 2
val appVersionName = "1.0.1"
...
versionCode = appVersionCode
versionName = appVersionName
```

```properties
# gradle.properties:30-32
app.versionCode=4
app.versionName=1.0.2
```

**Impact:** The comment is false and release versioning can regress or conflict with installed/market builds. This can block upgrade paths and makes artifact provenance ambiguous.

**Action:** Read and validate the Gradle properties (or remove them and maintain exactly one source of truth). Add a CI assertion that decoded APK/AAB package ID, `versionCode`, and `versionName` match the intended release manifest.

---

### F-08 — Release lint and shrink/obfuscation are disabled, with no CI lint task

**Severity: Medium**

**Evidence**

```kotlin
// app/build.gradle.kts:31-35, 79-84
lint {
  checkReleaseBuilds = false
  abortOnError = false
  checkDependencies = false
}
release {
  isDebuggable = false
  isMinifyEnabled = false
  proguardFiles(...)
}
```

No workflow runs `lint`, and the only release build (lines 85-92 of release workflow) did not execute because the credential gate failed.

**Impact:** Release-only Android correctness/security issues and dependency lint findings do not gate artifacts. `proguard-rules.pro` is effectively unused for shrinking because minification is off.

**Action:** Make `lintRelease` a required job with justified baselines only; enable R8/minification for production after testing or explicitly document the reason it is disabled.

---

### F-09 — Instrumented Android testing is absent from CI, and the only instrumented assertion is inconsistent with the application ID

**Severity: Medium**

**Evidence**

* CI invokes only `./gradlew testDebugUnitTest` (`android-debug.yml:55-56`; `android-release-validation.yml:54-55`). There is no emulator setup, `connected...AndroidTest`, `androidTest`, or device farm invocation.
* The declared package is `com.MinmKhas.studio.GameNexa.wrtx` (`app/build.gradle.kts:18`), but the device-only test asserts `com.example`:

```kotlin
// app/src/androidTest/java/com/example/ExampleInstrumentedTest.kt:14-21
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
  @Test fun useAppContext() {
    val appContext = InstrumentationRegistry.getInstrumentation().targetContext
    assertEquals("com.example", appContext.packageName)
  }
}
```

**Impact:** No actual installed-app, permission, networking, deep-link, or device behavior is tested in CI. If this sample test were run, the asserted package name is expected to disagree with the configured application ID.

**Action:** Correct/remove sample test, then add emulator instrumentation for launch, login/session restoration, protected API behavior, subscription/trial states, and payment/deep-link flows. Use a non-production backend fixture.

---

### F-10 — Android unit tests provide little behavioral assurance and include false-positive patterns

**Severity: Medium**

**Evidence**

* `ExampleUnitTest.kt:11-15` only asserts `4 == 2 + 2`.
* `ExampleRobolectricTest.kt:15-20` only reads the `GameNexa` string resource.
* `MainActivityTest.kt:15-29` starts the activity and prints; it contains no assertion.
* `GreetingScreenshotTest.kt:22-31` writes `src/test/screenshots/greeting.png` using `captureRoboImage`; it does not compare against a golden baseline and it renders only a literal `Text("GameNet Manager")`.
* `TrialAndSubscriptionTest.kt:25-108` performs UI interactions, sleeps, and prints callback results, but has no assertions about success, displayed error/success state, network request, or entitlement outcome.

**Impact:** A passing `testDebugUnitTest` mostly demonstrates that sample views can be constructed under Robolectric. It does not validate the claimed application workflows and can remain green when the trial/subscription behavior fails.

**Action:** Replace sample/print-only tests with deterministic assertions over ViewModel state, mocked HTTP responses, navigation, validation/error UI, and screenshot diffs against reviewed golden files. Avoid `Thread.sleep` in favor of idling/coroutine test dispatchers.

---

### F-11 — Several Python “test suites” are empty placeholders; the one large legacy suite targets another architecture

**Severity: Medium**

**Evidence**

Eight tracked files consist only of comments and exit successfully if invoked: `cancellation_boundary_tests.py`, `database_integrity_tests.py`, `gold_diamond_tests.py`, `payment_idempotency_tests.py`, `pricing_snapshot_tests.py` (one line each), plus `concurrency_tests.py`, `multi_worker_tests.py`, and `security_tests.py` (two lines each). Example:

```python
# tests/security_tests.py:1-2
# Placeholder for Security Tests (Manager IDOR, Auth bypass)
# Validated via Manager Isolation and Auth Token checks in cert_suite.py
```

`tests/cert_suite.py` is executable but not CI-invoked and uses a different system:

```python
# tests/cert_suite.py:8-10
BASE_URL = 'http://localhost:8080/api/selfhosted'
DB_PATH = '/var/www/gamenet-server/gamenet_central.db'
```

This conflicts with the current Node/PostgreSQL Compose API (`gamenet_api:3000`). It also directly mutates SQLite and an HTTP service, so it was not run during this safe audit.

**Impact:** File presence can be mistaken for coverage. These suites neither test the current runtime nor gate changes.

**Action:** Remove/archive stale tests or port them to the current API/Postgres architecture; implement real pytest tests with fixtures and include them in CI. Do not count comment-only files as test coverage.

---

### F-12 — Contract/audit scripts are useful regression tripwires but static-only; they do not verify the VPS or runtime contract

**Severity: Medium**

**Evidence**

* `tests/android_backend_contract_check.py:8-9` joins `.kt` and `.js` source into strings; route checks use regex/literal presence. Its own comment says raw paths intentionally do not infer HTTP method (`lines 17-18, 44-45`).
* `tests/deep_release_audit.py:8-20` reads source text, and `lines 22-75` evaluates 43 substring predicates.
* `backend/test_isolation_static.js:4-24` likewise asserts `includes(...)` snippets in source files.
* All three pass locally and ran in the successful debug workflow, but none starts Express, connects Postgres, authenticates a request, inspects a deployed VPS, or proves an Android call succeeds.

**Impact:** Source refactors can produce false negatives/positives; runtime route registration, middleware ordering, schema state, TLS, headers, container configuration, and deployment drift are untested.

**Action:** Keep these checks as fast static gates, but label them accurately and add contract/integration tests that start the actual service and client against versioned fixtures.

---

### F-13 — Container build is non-reproducible and can copy host `node_modules` into the Alpine image

**Severity: Medium**

**Evidence**

* `backend/Dockerfile:4` uses `npm install`, not lockfile-enforcing `npm ci`.
* `backend/.dockerignore` and root `.dockerignore` are absent. The physical `backend/node_modules` directory exists in the audit checkout, and `Dockerfile:5` uses `COPY . .` after installation.
* Therefore a Docker build context can overwrite container-installed dependencies with host `node_modules`; this is especially risky for native `bcrypt` when the host differs from Node 18 Alpine/musl.
* The image bases are mutable tags: `node:18-alpine` (`Dockerfile:1`), `postgres:16` (`docker-compose.yml:4`), and `nginx:latest` (`docker-compose.yml:40`).

**Impact:** Build output can depend on untracked host state, image tag movement, or incompatible native binaries. A production image cannot be reproduced from the commit alone.

**Action:** Add a backend `.dockerignore` at minimum for `node_modules`, `.env`, logs, test output, and VCS metadata; use `npm ci --omit=dev` (as appropriate) with the committed lockfile; pin base images by immutable digest; build/test the image in CI.

---

### F-14 — Supply-chain/reproducibility controls are incomplete

**Severity: Medium**

**Evidence**

* Workflow actions are version tags rather than commit SHAs: `actions/checkout@v7`, `actions/setup-java@v6`, `android-actions/setup-android@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`, and `Mattraks/delete-workflow-runs@v2` (all workflow files, e.g. debug lines 21-25 and delete-runs line 13).
* Gradle wrapper pins `gradle-9.3.1-bin.zip` but `gradle/wrapper/gradle-wrapper.properties:1-8` contains no `distributionSha256Sum`.
* No Gradle dependency-locking or verification metadata file exists. The version catalog pins many direct versions, but no dependency verification/lock is configured.
* Positive control: `package-lock.json` is lockfile v3 and includes integrity hashes; resolved production examples are Express 5.2.1, pg 8.23.1, dotenv 17.4.2, bcrypt 6.0.0, and jsonwebtoken 9.0.3.

**Impact:** Remote action, Gradle distribution, transitive dependency, and image resolution can drift from the source review. This is a supply-chain/rebuild risk; no vulnerability conclusion is implied because no current advisory scan was performed.

**Action:** SHA-pin actions (with Renovate/Dependabot management), add Gradle wrapper distribution checksum and dependency verification/locking, maintain dependency update/audit jobs, and publish SBOM/provenance for release artifacts/images.

---

### F-15 — No release publication, deployment, artifact provenance, or retention suitable for an operational release exists in workflow source

**Severity: Medium**

**Evidence**

* README only says the repository is for a self-hosted backend and Android release build and that no external APK upload service is part of release workflow (`README.md:3-7`).
* The release workflow only uploads GitHub workflow artifacts for three days (`android-release-validation.yml:94-116`); it does not create a GitHub Release, sign/attest/provenance-publish artifacts, publish a container, deploy a backend, or run post-deploy verification.
* Debug artifacts are retained for one day (`android-debug.yml:74-80`).
* The maintenance workflow may delete run history after zero days while retaining only four runs (`delete-runs.yml:12-18`).

**Impact:** There is no auditable promotion path from commit to an immutable Android release/backend deployment, and operational forensic evidence is aggressively short-lived.

**Action:** Define separate build, attest/sign, release, deploy, and verify stages; store release assets in a durable release registry; retain artifacts/logs according to incident and compliance needs; protect environments and require approval for production.

---

### F-16 — VPS/running-API consistency remains unverified despite a successful public time probe

**Severity: Medium**

**Confirmed limited evidence:** The configured hostname and TLS route were reachable during audit. `GET https://api.gamenermayket.ir/api/v1/time` returned 200, `server: nginx`, `Strict-Transport-Security: max-age=31536000`, and the expected Tehran-time JSON. Nginx source also enforces TLS 1.2/1.3 and HSTS (`backend/nginx.conf:24-36`) and proxies to `gamenet_api:3000` (`lines 38-47`).

**Not verified:** There was no VPS/SSH, Docker daemon, database, deployment credential, container digest, production environment, certificate-renewal, schema-version, backup/restore, or release-signing-secret access. A public time response cannot show that the VPS runs this commit, that `schema.sql` was applied, that Compose is healthy, that the database is backed up, or that privileged API paths work.

**Action:** Deploy an authenticated version/build-info endpoint containing a commit SHA and schema migration version (no secrets), record image digests, and perform protected post-deploy readiness/contract checks. Establish and test database backup/restore and certificate renewal procedures.

## 3. Security and operational positives / caveats

| Area | Confirmed positive | Caveat |
|---|---|---|
| Release signing | Gradle rejects release tasks without valid keystore path/password/alias/key password (`app/build.gradle.kts:37-73`), and the workflow refuses ephemeral replacement keys. | The gate currently fails; signing success and cert identity are unverified. Do not expose those secrets on every branch/PR run. |
| Secret handling in source | Root `.gitignore` excludes `.env`, keystores, JKS/P12/PEM, APK/AAB, and `node_modules`; no tracked `.env` or key/certificate filename was found. Templates contain placeholders rather than apparent values. | This is a filename/source review, not a historical secret scan or proof of secret hygiene in Git history, runner logs, VPS, caches, or GitHub settings. |
| Backend startup secrets | The server aborts at startup if `JWT_SECRET` is absent (`server.js:62-63`). | `DATABASE_URL` and supporting runtime environment are undocumented in templates; bootstrap configuration is incomplete. |
| Reverse proxy | TLS 1.2/1.3, HSTS, certificate paths, ACME location, and a basic nginx request limit are configured (`nginx.conf:14-46`). | `nginx:latest` is mutable; no certbot/renewal service or nginx health check is present in Compose. |
| API source controls | CORS allow-list logic, JSON body limit, `X-Content-Type-Options`, `Referrer-Policy`, JWT algorithm restriction, and auth gates are present in source (`server.js:10-27, 62-75`). | No live/authenticated test proves actual deployed configuration or multi-instance rate-limit behavior; application rate limits are in-memory (`server.js:30-45`). |

## 4. Test quality classification

| Class | Files / commands | Quality conclusion |
|---|---|---|
| **Real build executed for exact commit** | GitHub debug `testDebugUnitTest`, `assembleDebug` | Confirms the Android debug build/unit task executed in GitHub. Coverage quality remains weak due to sample/print-only tests. |
| **Release build attempted** | GitHub release workflow | Did not reach signed build; credentials gate failed. |
| **Source-only static contracts** | `android_backend_contract_check.py`, `deep_release_audit.py`, `test_isolation_static.js`, `npm run check` | Useful fast safeguards; **not** runtime, API, database, or VPS evidence. |
| **Mock-only logic** | `test_financial_mock.js` | Exercises production functions with SQL-string-sensitive mocked responses; not a PostgreSQL transaction/behavior test. |
| **Potentially meaningful integration tests, but not run by CI** | Most `backend/test_*.js` | Could exercise actual Postgres/API behavior only in an isolated environment. Present implementation is unsafe/non-hermetic and assumes live services. |
| **Placeholders / stale architecture** | Eight short Python comment files; `cert_suite.py` | Not meaningful coverage of current Node/Postgres runtime. |
| **No device/E2E** | Android `androidTest` absent from workflow | No evidence for real-device/emulator install, deep links, network security, auth, subscription, billing, offline/reconnect, or lifecycle flows. |

## 5. Priority remediation plan

1. **Block release promotion now:** configure protected release-signing secrets; rerun the exact release workflow; verify signed APK/AAB certificate, application ID, and corrected version values. Restrict signing to tags/protected environments instead of all branches and PRs.
2. **Create hermetic backend CI:** `npm ci`; ephemeral Postgres; apply versioned migrations; start API locally; run test suite against only that environment; eliminate public-host and fixed-data test targets.
3. **Make Docker deployable from zero:** migration job/schema version, non-external-or-documented volume lifecycle, complete `.env.example`, `/readyz` DB/schema health, Docker `.dockerignore`, digest-pinned images, and CI image build.
4. **Repair test signal:** replace Android sample tests and print/sleep tests with asserted behavior; enable emulator instrumentation; replace Python placeholders/stale SQLite suite; preserve static checks as a separate labeled job.
5. **Harden reproducibility and provenance:** use `npm ci`, SHA-pin GitHub Actions, add Gradle checksum/dependency verification, digest-pin containers, generate SBOM/provenance, and retain immutable release assets/logs appropriately.
6. **Close operational verification:** deploy commit/schema/image-digest build info, add protected post-deploy smoke tests, monitor readiness, and test DB backup/restore plus certificate renewal.

## 6. Explicit unverified items

The following cannot be inferred from the repository or public unauthenticated endpoint and were **not claimed as verified**:

* Presence/correctness of GitHub release-signing secrets, keystore alias, certificate fingerprint, or Play/Myket signing identity.
* VPS container state, Compose invocation, image digest, current code SHA, database state/schema, migrations, persistent volume contents, logs, backups/restores, firewall, certificate renewal, or system resource limits.
* Live behavior of authenticated/financial/subscription endpoints, database constraints under concurrency, Android real-device behavior, and cross-version upgrade/install behavior.
* Current dependency CVEs/advisories (no advisory database scan was performed); only file-visible versioning/reproducibility risks are reported.
* Whether any action-version tag, Docker base tag, or remote dependency has changed since review; tags are identified as a risk because they are mutable.

## 7. Inspected file groups

* **Actions:** `.github/workflows/android-debug.yml`, `android-release-validation.yml`, `delete-runs.yml`.
* **Android/Gradle:** root/app Gradle Kotlin scripts, settings, properties, version catalog, wrapper properties/JAR presence, manifest, network/backup/extraction XML, ProGuard rules, Android test sources.
* **Backend/npm:** `package.json`, `package-lock.json`, server/canonical/financial/reservation/bootstrap sources, all twelve `backend/test_*.js` files, schema.
* **Runtime/deployment:** `Dockerfile`, `docker-compose.yml`, `nginx.conf`, root/backend env examples, ignore files.
* **Documentation/manifests:** `README.md`, `RELEASE_AUDIT_STATUS.md`, `SOURCE_PACKAGE_MANIFEST.md`, `metadata.json`.
* **Python QA:** every file under `tests/`, including comment-only/stale suites.


## 9. API Contract Audit

The backend appendix contains the exhaustive 108-route inventory and the Android appendix contains extracted Retrofit/direct API calls. Confirmed cross-layer mismatch: `AND-002` (VIP reservation rules auth header) and `GN-BE-06` (GN purchase semantics). Static contract scripts passed, but their own evidence classification is source-text/static only; they are not HTTP runtime contract tests. The full Android↔backend request/response comparison for every route remains **UNVERIFIED** without a generated runtime contract harness and running API/database.

## 10. Database, Transaction and Concurrency Audit

Source review confirmed meaningful controls such as parameterized SQL, tenant predicates, advisory lock/session guards, unique keys and reservation exclusion constraints. It also confirmed the financial/data-integrity defects `GN-BE-02` through `GN-BE-05`, and the fresh-schema/runtime defects `F-04` and `F-06`. Actual concurrent executions of start, settle, approval, reservation and payment were **UNVERIFIED** because PostgreSQL/API runtime was unavailable. Required verification is listed per finding in the preserved backend report.

## 11. Offline/Online, Time, UI, Logging and Performance

- Offline/online reconciliation, replay, process-death recovery and full station/session UI chains were not runtime-tested; classify as **UNVERIFIED**.
- Android source review did confirm encrypted token settings, local Room customer password exposure (`AND-001`), retry behavior, 150-entry in-memory network diagnostics and Tehran/Jalali formatting. Exact evidence is in the Android report.
- No additional confirmed UI/notch/time/performance defect is asserted solely from static observations. This is not a claim that runtime behavior is correct.

## 12. Recommended Fix Plan

1. **P0:** restore protected release signing credentials and produce/verify a signed artifact for this exact SHA.
2. **P1 financial integrity:** fix guest billing conservation, prepayment reconciliation, VIP supersession refunds, manual wallet approval bounds/ledgering, and GN purchase semantics. Add transactional/idempotent regression tests.
3. **P1 privacy/auth:** remove Room password persistence/display/clipboard use; use server-side password verification only. Fix customer VIP rules header. Centralize device-cap enforcement.
4. **P1 runtime/release:** add disposable PostgreSQL CI, migrations/schema initialization, API startup/readiness checks, Docker build validation, backend integration tests and post-deploy contract smoke tests.
5. **P1 configuration/version:** unify version source of truth and validate decoded artifact metadata; complete env templates and fail fast on required config.
6. **P2 security:** replace caller-minted trial identity with server-verifiable installation/account attestation; add server-side token revocation/auth epoch; strengthen password policy; use uniform customer login errors.
7. **P2 quality:** enable release lint and dependency checks, add instrumented tests, replace placeholder/static-only tests with behavior assertions, and make CI provenance/reproducibility explicit.
8. **Before sign-off:** run the complete scenario matrix against disposable runtime, then repeat concurrency/idempotency tests and verify SHA/source/container/API identity.

## 13. Regression Test Plan

- Start two concurrent sessions for one station and one customer; assert one committed session and no orphan claims.
- Settle twice and retry after timeout; assert one invoice/payment/ledger outcome.
- Start with guests, prepayments, multiple customers, buffet and custom durations; assert invoice/payment conservation.
- Supersede a paid normal reservation with VIP; assert idempotent refund/credit, audit log, notification and terminal state.
- Approve wallet/payment/GN operations with negative, oversized, duplicate and replayed requests; assert rejection and no ledger corruption.
- Attempt manager device B beyond entitlement cap concurrently; assert 403 and unchanged active count.
- Attempt trial with omitted bearer and changed caller identities; assert attestation/account binding rejects replay/reset.
- Login, logout, password-change, archive and role-change; assert old JWTs immediately fail.
- Store/update/delete customer; assert no plaintext password at rest, no clipboard exposure, correct archive/sync behavior and ownership isolation.
- Run Android API contract tests against the real local API and PostgreSQL, including 400/401/403/404/409/422/429/5xx mapping.
- Build debug and signed release; run lint, unit, instrumentation, backend integration, schema migration, Docker and post-deploy readiness checks in CI.

## 14. Final Release Readiness Assessment

**Assessment: FAIL / not release-ready.** The failed signed-release gate alone prevents release certification. Independent source evidence additionally proves unresolved High/P1 financial, authorization, data-protection, API-contract and runtime-pipeline defects. The absence of a local runtime means many scenarios cannot be declared PASS; they remain unverified and require a disposable integrated test environment.

## WHAT MUST BE FIXED BEFORE I WOULD TRUST THIS BUILD

- Produce and verify a signed release artifact for SHA `05348f2df8b1f6c30ee233071be13a899d969e25`.
- Fix `AND-001`, `AND-002`, `GN-BE-01` through `GN-BE-06`, and `F-02` through `F-07` with regression tests.
- Reconcile all money flows atomically and idempotently, including guests, prepayments, reservations, wallet approvals and GN purchases.
- Remove plaintext customer password persistence/exposure and enforce strong server-side password/session lifecycle controls.
- Make schema initialization/migrations and readiness checks reproducible; run backend/Postgres/Docker/integration tests in CI.
- Resolve all unverified station/session, settlement, customer, reservation, buffet, offline-sync and concurrency scenarios in a disposable runtime before release sign-off.
- Prove GitHub SHA = deployed source = running container/API, or explicitly block release until that evidence exists.

## WHAT IS ACTUALLY PROVEN

- The repository and `main` SHA under audit are exactly `05348f2df8b1f6c30ee233071be13a899d969e25` and the local source tree remained clean.
- The Android, backend/API/database, security, and CI/runtime reports completed read-only, evidence-based inspections with exact current line references.
- The listed confirmed findings are established by current source/configuration or exact GitHub run/test output.
- GitHub debug run `36964812531` succeeded for its configured Android/static scope; release run `36964812496` failed before signed artifacts.
- Static checks, syntax checks, selected mocked checks, dependency audit and contract scripts produced the PASS results recorded in the appendices.
- No confirmed Manager A→B IDOR, Super Manager authorization bypass, SQL injection, committed production secret, or bearer-CSRF issue was established in the reviewed code; this is bounded negative evidence, not a universal security guarantee.

## WHAT COULD NOT BE VERIFIED

- Full Android local compile/test/instrumentation in this sandbox.
- PostgreSQL-backed integration behavior, real HTTP end-to-end flows, transaction isolation and concurrency races.
- Docker Compose startup, schema migration, container image and nginx deployment behavior.
- Full Android↔backend API response/error contract at runtime.
- Offline/reconnect/reconciliation, process death, rotation, duplicate taps and all requested station/session/settlement/customer/reservation/buffet/GN/LP scenarios.
- VPS source/container/API commit identity.
- Any behavior of a different commit, branch, database, environment or deployment.

## Appendix Traceability

Specialist evidence files retained alongside this report:

- `GameNexa-Audit-android.md` — Android inventory, API extraction and AND-001..003.
- `GameNexa-Audit-backend.md` — 108-route inventory and GN-BE-01..08.
- `GameNexa-Audit-security.md` — SEC-01..04 and security control review.
- `GameNexa-Audit-ci.md` — CI/runtime/test inventory and F-01..16.



---

## خروجی ۲ — ممیزی Android، UI، State و Offline

> فایل منبع این بخش: `GameNexa-Audit-android.md`

# GameNexa Android Code Audit

## Audit identity and scope

| Field | Value |
|---|---|
| Audit ID | `GNX-ANDROID-2026-10-02-05348f2` |
| Repository | `/home/ubuntu/GameNexa-release-2026` |
| Requested ref / observed HEAD | `05348f2df8b1f6c30ee233071be13a899d969e25` / `05348f2df8b1f6c30ee233071be13a899d969e25` |
| Branch | `main` |
| Scope | **Android only**: Gradle/module config, manifest/resources, all Kotlin under `app/`, Room/DAO/entities, repository, networking/DTOs, ViewModels, Compose UI/actions, auth/token storage, offline/sync/retry/error/lifecycle, time/Jalali/locale, business defaults and tests. Backend was read only where necessary to prove an Android contract. |
| Source modifications | **None.** Working tree at the time of report generation: `(clean)`. |
| Method | Static source/contract audit. Every `app/**/*.kt` file was enumerated (58 files, including tests and the unconfigured applet source); targeted manual control-flow review was done for state, auth, persistence, network, billing, reservations, lifecycle, and UI actions. |
| Finding standard | Only evidence-backed, reproducible issues below are counted. Potential concerns that were not proven are separated under **Unverified items**. |

> **Important limitation:** Gradle compilation and unit tests were started but could not execute because this sandbox has no Android SDK location. There is no claim that the Android application builds or that tests pass.

## Executive summary

**Three confirmed findings** were identified:

1. **AND-001 / High / P1** — customer passwords are deliberately retained in a standard Room SQLite table, rendered in plaintext to staff and copied to the system clipboard.
2. **AND-002 / High / P1** — the customer VIP reservation-rules call uses manager/base headers rather than the customer bearer header; the backend route explicitly requires customer authentication. The customer full-hall screen therefore receives no rules for an ordinary customer session and has no selectable VIP duration.
3. **AND-003 / Medium / P2** — release lint checking, lint failure, and dependency lint checking are explicitly disabled, removing a release-time quality/security gate.

The client otherwise has several positive controls evidenced by code: `allowBackup="false"`, cleartext disabled, non-exported alarm receiver, AES-GCM/Android Keystore used for encrypted settings/tokens, customer session restoration uses the customer header, customer login refuses an offline fallback, network logging avoids retaining raw bodies, and customer reservation mutation/cancellation includes idempotency keys. These observations do **not** offset the confirmed issues above.

## Environment, build, and manifest inventory

### Gradle/module facts

- `settings.gradle.kts:22-23` names the project `GameNexa` and includes only `:app`; `app/applet/.../SubscriptionActivationScreen.kt` is present in the tree but no applet Gradle descriptor was found and it is **not** an included Android module.
- `app/build.gradle.kts:9-29`: namespace `com.example`; app id `com.MinmKhas.studio.GameNexa.wrtx`; compile SDK 36.1; min SDK 24; target SDK 36; version `2` / `1.0.1`; Java 11.
- Compose and KSP are enabled (`app/build.gradle.kts:1-6`, `95-99`); Room, Retrofit, Moshi, OkHttp, coroutines and test libraries are declared (`112-167`).
- Release signing credentials are loaded from properties/environment and release task fails if absent (`37-74`). Release minification is **off** (`79-85`).
- The configured test runner is `androidx.test.runner.AndroidJUnitRunner` (`28`), and unit tests include Android resources (`99`).

### Manifest, component, transport, and backup facts

`app/src/main/AndroidManifest.xml` was reviewed in full.

| Area | Confirmed implementation / evidence |
|---|---|
| Permissions | `INTERNET`, network-state, vibration, post notifications, exact alarms, contacts, boot complete and legacy external storage permissions are requested (`4-12`). |
| Backup/transport | Application sets `android:allowBackup="false"`, `usesCleartextTraffic="false"`, network config, data-extraction rules and backup rules (`14-25`). Network config independently sets `<base-config cleartextTrafficPermitted="false">` and system trust anchors (`res/xml/network_security_config.xml:2-8`). |
| Exposed entry point | `MainActivity` is exported (`27-32`) for launcher, verified HTTPS `/pay/callback` and browsable `gamenet://payment-callback` / `gamenet://verify` deep links (`33-50`). |
| Receiver | `AlarmReceiver` is declared `exported="false"` (`51-54`). It uses `goAsync()` then finishes its pending result in `finally` (`AlarmReceiver.kt:26-56`). |
| Resource caveat | `res/xml/backup_rules.xml` includes shared preferences (`2-4`), while the manifest disables backups. The effective behavior from the manifest is backup disabled; no finding is asserted from the unused-looking rule. |

## Source coverage inventory

The counts below come from a read-only enumeration of every Kotlin source under `app/`. “Declarations”, “Compose”, and “onClick” are simple inventory counts—not test coverage.

| Path | Lines | Package | Declarations | Compose | `onClick` |
|---|---:|---|---:|---:|---:|
| `app/applet/app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` | 335 | `com.example.ui` | 2 | 1 | 9 |
| `app/src/androidTest/java/com/example/ExampleInstrumentedTest.kt` | 22 | `com.example` | 3 | 0 | 0 |
| `app/src/main/java/com/example/MainActivity.kt` | 1035 | `com.example` | 14 | 8 | 18 |
| `app/src/main/java/com/example/data/AppDatabase.kt` | 182 | `com.example.data` | 50 | 0 | 0 |
| `app/src/main/java/com/example/data/CryptoManager.kt` | 100 | `com.example.data` | 4 | 0 | 0 |
| `app/src/main/java/com/example/data/Dao.kt` | 294 | `com.example.data` | 96 | 0 | 0 |
| `app/src/main/java/com/example/data/Entities.kt` | 363 | `com.example.data` | 32 | 0 | 0 |
| `app/src/main/java/com/example/data/GameNetRepository.kt` | 1086 | `com.example.data` | 86 | 0 | 0 |
| `app/src/main/java/com/example/data/LicenseCacheEntity.kt` | 31 | `com.example.data` | 5 | 0 | 0 |
| `app/src/main/java/com/example/data/PaymentModels.kt` | 23 | `com.example.data` | 3 | 0 | 0 |
| `app/src/main/java/com/example/data/SecurityUtils.kt` | 85 | `com.example.data` | 4 | 0 | 0 |
| `app/src/main/java/com/example/data/network/GameNetApi.kt` | 992 | `com.example.data.network` | 104 | 0 | 0 |
| `app/src/main/java/com/example/data/network/NetworkLogger.kt` | 188 | `com.example.data.network` | 9 | 0 | 0 |
| `app/src/main/java/com/example/data/network/RetryInterceptor.kt` | 57 | `com.example.data.network` | 2 | 0 | 0 |
| `app/src/main/java/com/example/data/network/SelfHostedManager.kt` | 1985 | `com.example.data.network` | 81 | 0 | 0 |
| `app/src/main/java/com/example/receiver/AlarmReceiver.kt` | 175 | `com.example.receiver` | 7 | 0 | 0 |
| `app/src/main/java/com/example/ui/AdminLoginScreen.kt` | 284 | `com.example.ui` | 2 | 1 | 2 |
| `app/src/main/java/com/example/ui/AdminNotificationComponents.kt` | 696 | `com.example.ui` | 4 | 4 | 14 |
| `app/src/main/java/com/example/ui/AuthDialog.kt` | 406 | `com.example.ui` | 1 | 1 | 7 |
| `app/src/main/java/com/example/ui/BehaviorDialog.kt` | 188 | `com.example.ui` | 1 | 1 | 5 |
| `app/src/main/java/com/example/ui/ContactUsScreen.kt` | 375 | `com.example.ui` | 6 | 2 | 4 |
| `app/src/main/java/com/example/ui/CustomerAppContent.kt` | 1793 | `com.example.ui` | 6 | 4 | 20 |
| `app/src/main/java/com/example/ui/CustomerClubScreen.kt` | 3618 | `com.example.ui` | 19 | 14 | 72 |
| `app/src/main/java/com/example/ui/CustomerDialogs.kt` | 1234 | `com.example.ui` | 7 | 6 | 27 |
| `app/src/main/java/com/example/ui/CustomerFullHallTab.kt` | 207 | `com.example.ui` | 1 | 1 | 4 |
| `app/src/main/java/com/example/ui/CustomerOnlinePaymentTab.kt` | 1028 | `com.example.ui` | 7 | 4 | 17 |
| `app/src/main/java/com/example/ui/CustomerTabs.kt` | 1651 | `com.example.ui` | 6 | 6 | 17 |
| `app/src/main/java/com/example/ui/CustomersReservationsScreen.kt` | 3506 | `com.example.ui` | 16 | 12 | 63 |
| `app/src/main/java/com/example/ui/FirstLaunchGuide.kt` | 319 | `com.example.ui` | 6 | 2 | 8 |
| `app/src/main/java/com/example/ui/GameNetViewModel.kt` | 6576 | `com.example.ui` | 236 | 0 | 0 |
| `app/src/main/java/com/example/ui/GameNexaWelcomeScreen.kt` | 638 | `com.example.ui` | 1 | 1 | 8 |
| `app/src/main/java/com/example/ui/HallWeatherEffects.kt` | 16 | `com.example.ui` | 1 | 2 | 0 |
| `app/src/main/java/com/example/ui/InvoiceCard.kt` | 179 | `com.example.ui` | 1 | 1 | 0 |
| `app/src/main/java/com/example/ui/LicenseViewModel.kt` | 126 | `com.example.ui` | 7 | 0 | 0 |
| `app/src/main/java/com/example/ui/Localization.kt` | 190 | `com.example.ui` | 2 | 0 | 0 |
| `app/src/main/java/com/example/ui/MainScreen.kt` | 2088 | `com.example.ui` | 9 | 5 | 21 |
| `app/src/main/java/com/example/ui/ManagerReservationsScreen.kt` | 200 | `com.example.ui` | 3 | 2 | 4 |
| `app/src/main/java/com/example/ui/NetworkDiagnosticsDialog.kt` | 674 | `com.example.ui` | 5 | 4 | 7 |
| `app/src/main/java/com/example/ui/ReservationSettingsScreen.kt` | 413 | `com.example.ui` | 16 | 5 | 6 |
| `app/src/main/java/com/example/ui/ServerConnectionTestDialog.kt` | 220 | `com.example.ui` | 3 | 2 | 4 |
| `app/src/main/java/com/example/ui/SettingsScreen.kt` | 3447 | `com.example.ui` | 18 | 14 | 72 |
| `app/src/main/java/com/example/ui/StatisticsScreen.kt` | 370 | `com.example.ui` | 2 | 2 | 1 |
| `app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` | 1115 | `com.example.ui` | 3 | 2 | 11 |
| `app/src/main/java/com/example/ui/SubscriptionLockScreen.kt` | 309 | `com.example.ui` | 1 | 1 | 7 |
| `app/src/main/java/com/example/ui/UnifiedEntryScreen.kt` | 626 | `com.example.ui` | 2 | 1 | 11 |
| `app/src/main/java/com/example/ui/theme/Color.kt` | 11 | `com.example.ui.theme` | 0 | 0 | 0 |
| `app/src/main/java/com/example/ui/theme/Theme.kt` | 76 | `com.example.ui.theme` | 1 | 2 | 0 |
| `app/src/main/java/com/example/ui/theme/Type.kt` | 36 | `com.example.ui.theme` | 0 | 0 | 0 |
| `app/src/main/java/com/example/util/ExactBilling.kt` | 17 | `com.example.util` | 3 | 0 | 0 |
| `app/src/main/java/com/example/util/JalaliCalendarHelper.kt` | 144 | `com.example.util` | 11 | 0 | 0 |
| `app/src/main/java/com/example/util/LocaleHelper.kt` | 30 | `com.example.util` | 3 | 0 | 0 |
| `app/src/main/java/com/example/util/VpnDetector.kt` | 43 | `com.example.util` | 5 | 0 | 0 |
| `app/src/main/java/com/example/utils/NumberConverter.kt` | 25 | `com.example.utils` | 3 | 0 | 0 |
| `app/src/test/java/com/example/ExampleRobolectricTest.kt` | 21 | `com.example` | 3 | 0 | 0 |
| `app/src/test/java/com/example/ExampleUnitTest.kt` | 16 | `com.example` | 2 | 0 | 0 |
| `app/src/test/java/com/example/GreetingScreenshotTest.kt` | 32 | `com.example` | 3 | 0 | 0 |
| `app/src/test/java/com/example/MainActivityTest.kt` | 31 | `com.example` | 4 | 0 | 0 |
| `app/src/test/java/com/example/TrialAndSubscriptionTest.kt` | 109 | `com.example` | 7 | 0 | 0 |

## Architecture and behavior inventory (proven facts)

### Persistence and repository

- `AppDatabase` registers the Room tables (including `Customer`, transactions, reservations, license cache and GN-related tables) and builds the ordinary Room database named `gamenet_manager_db` (`AppDatabase.kt:16-49`, `162-176`). No SQLCipher/encrypted Room setup is present in that builder.
- The `Customer` Room entity includes `val password: String = ""` (`Entities.kt:177-199`). Migration 8→9 creates that as `password TEXT NOT NULL DEFAULT ''` (`AppDatabase.kt:69-73`).
- Repository/database CRUD, default data, synchronization, station/session records, order data, settings, customer data and reservations were inspected in `GameNetRepository.kt`, `Dao.kt`, `Entities.kt`, `AppDatabase.kt`, `LicenseCacheEntity.kt`, and `PaymentModels.kt`.
- `CryptoManager` uses an Android Keystore AES-GCM key (`CryptoManager.kt:12-38`) and returns a Base64 `iv:ciphertext` pair (`54-72`). Its documented exception path throws rather than stores a plaintext fallback (`67-72`, `90-98`). ViewModel customer tokens are saved via encrypted settings after login (`GameNetViewModel.kt:1010-1014`). This is distinct from the unencrypted Room `Customer.password` field in AND-001.

### Network, auth, retry, error handling, and sync

- Base manager headers add `X-Manager-ID` and, only from `NetworkClient.authToken`, `Authorization: Bearer …` (`SelfHostedManager.kt:228-240`). Customer headers instead use only `NetworkClient.customerAuthToken` (`243-250`). This distinction proves AND-002.
- Customer login posts phone, manager id and password (`SelfHostedManager.kt:398-405`), then fetches `/api/v1/customer/profile`; it copies the submitted password into the parsed `Customer` (`436`) and `GameNetViewModel.loginCustomer` inserts that record into Room (`978-1014`). Registration also copies password into its parsed customer (`461-485`).
- Customer login is server-authoritative: a failed online result leads to the user-visible error **“offline login is not allowed”** and returns (`1029-1033`).
- Retry interceptor retries a request up to 3 times (5 for paths containing `sync`), uses 2-second exponential backoff and retries 5xx/408/429 plus `IOException`; it returns 4xx other than 408/429 without retry (`RetryInterceptor.kt:9-55`). It performs `Thread.sleep` in OkHttp interception (`19-45`). This is recorded behavior, not itself a defect claim.
- Network logger keeps at most 150 in-memory records (`NetworkLogger.kt:40-64`) and deliberately does not retain raw response bodies; for failed responses it extracts a bounded code/error string using `peekBody` (`139-162`).
- The customer VIP mutation and cancellation use the customer header plus idempotency key (`SelfHostedManager.kt:1410-1442`). The rules request is the exception captured in AND-002.

### Lifecycle, alarms, time, locale, UI, and business values

- Alarm receiver uses `goAsync`, performs DB work on `Dispatchers.IO`, and calls `pendingResult.finish()` in `finally` (`AlarmReceiver.kt:26-57`). Reservation notifications intentionally call vibration twice (`42-49`); this is behavior, not classified as a defect.
- Jalali display is calculated in `Asia/Tehran`, formats date/time with `Locale.US` digits, and treats `Long.MAX_VALUE`-like expiry as “active subscription” (`JalaliCalendarHelper.kt:10-34`, `126-141`). `CustomerFullHallTab` also presents reservation dates in Asia/Tehran, but explicitly with `SimpleDateFormat("yyyy/MM/dd - HH:mm", Locale.US)` (`CustomerFullHallTab.kt:72-75`).
- `LocaleHelper` changes the default locale/configuration and calls deprecated `resources.updateConfiguration` with suppression, then recreates the Activity in `applyLocale` (`LocaleHelper.kt:8-29`). This was observed; runtime locale behavior was not executed.
- Hard-coded seeds/defaults exist, including console prices (`GameNetRepository.kt:179-182`, `554-557`, `615-618`), product prices (`206-216` etc.), GN settings (`595-597`), and fallback GN price per 10 (`SelfHostedManager.kt:213`). They are reported as implementation facts, not defects; whether they are approved business defaults cannot be determined from code alone.
- Compose actions across all screens were enumerated. Financial/customer actions reviewed include manual GN adjustment, transfer, purchase, policy save, customer deletion, payment reports and reservation submission. UI controls often have role/feature gating, while server routes remain the authoritative boundary; no bypass was claimed without a complete runnable role scenario.

## Extracted API calls

### Retrofit declarations

| Kind | Android declaration | Endpoint | Evidence |
|---|---|---|---|
| Retrofit GET | `suspend fun getServerTime(): ServerClockResponse` | `/api/v1/time` | `GameNetApi.kt:41` |
| Retrofit GET | `suspend fun healthCheck(): retrofit2.Response<ServerClockResponse>` | `/api/v1/time` | `GameNetApi.kt:44` |
| Retrofit GET | `suspend fun getSuperManagers(): List<AdminManagerDto>` | `/api/v1/super-manager/managers` | `GameNetApi.kt:47` |
| Retrofit POST | `suspend fun addManagerContract(@Body request: Map<String, Any>): okhttp3.ResponseBody` | `/api/v1/super-manager/add-manager` | `GameNetApi.kt:50` |
| Retrofit POST | `suspend fun recoverSuperManager(@Body body: Map<String, String>): okhttp3.ResponseBody` | `/api/v1/super-manager/recover` | `GameNetApi.kt:54` |
| Retrofit POST | `suspend fun addManagerCustomer(@Body body: Map<String, Any>): okhttp3.ResponseBody` | `/api/v1/manager/customers` | `GameNetApi.kt:57` |
| Retrofit POST | `suspend fun checkTrialStatus(@Body body: CheckTrialRequest): CheckTrialResponse` | `/api/v1/trial/status` | `GameNetApi.kt:61` |
| Retrofit GET | `suspend fun getAllDeviceTrials(): okhttp3.ResponseBody` | `/api/v1/super-manager/trial-devices` | `GameNetApi.kt:64` |
| Retrofit DELETE | `suspend fun deleteDeviceTrial(@Path("id") id: String): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/trial-devices/{id}` | `GameNetApi.kt:67` |
| Retrofit POST | `suspend fun extendDeviceTrial(@Path("id") id: String): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/trial-devices/{id}/extend` | `GameNetApi.kt:70` |
| Retrofit POST | `suspend fun createSuperManager(@Body request: CreateManagerRequestDto): okhttp3.ResponseBody` | `/api/v1/super-manager/managers` | `GameNetApi.kt:75` |
| Retrofit PUT | `suspend fun updateManagerStatus(@Path("id") id: String, @Body request: UpdateManagerRequestDto): retrofit2.Response<okht` | `/api/v1/super-manager/managers/{id}` | `GameNetApi.kt:78` |
| Retrofit PUT | `suspend fun updateManager(@Path("id") id: String, @Body request: EditManagerRequestDto): retrofit2.Response<okhttp3.Resp` | `/api/v1/super-manager/managers/{id}` | `GameNetApi.kt:81` |
| Retrofit DELETE | `suspend fun deleteManager(@Path("id") id: String): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/managers/{id}` | `GameNetApi.kt:84` |
| Retrofit POST | `suspend fun createSuperManagerAlt(@Body request: CreateManagerRequestDto): okhttp3.ResponseBody` | `/api/v1/super-manager/create-manager` | `GameNetApi.kt:88` |
| Retrofit POST | `suspend fun createManager(@Body request: CreateManagerRequestDto): okhttp3.ResponseBody` | `/api/v1/super-manager/add-manager` | `GameNetApi.kt:91` |
| Retrofit GET | `suspend fun getStations(): List<StationState>` | `/api/v1/manager/stations` | `GameNetApi.kt:98` |
| Retrofit POST | `suspend fun saveStation(@Body state: StationState): StationState` | `/api/v1/manager/stations` | `GameNetApi.kt:101` |
| Retrofit GET | `suspend fun getConsoleTypes(): List<ConsoleType>` | `/api/v1/manager/console-types` | `GameNetApi.kt:108` |
| Retrofit POST | `suspend fun saveConsoleType(@Body console: ConsoleType): ConsoleType` | `/api/v1/manager/console-types` | `GameNetApi.kt:111` |
| Retrofit DELETE | `suspend fun deleteConsoleType(@Path("name") name: String): Response<Unit>` | `/api/v1/manager/console-types/{name}` | `GameNetApi.kt:114` |
| Retrofit GET | `suspend fun getProducts(): List<Product>` | `/api/v1/manager/products` | `GameNetApi.kt:117` |
| Retrofit POST | `suspend fun saveProduct(@Body product: Product): Product` | `/api/v1/manager/products` | `GameNetApi.kt:120` |
| Retrofit DELETE | `suspend fun deleteProduct(@Path("name") name: String): Response<Unit>` | `/api/v1/manager/products/{name}` | `GameNetApi.kt:123` |
| Retrofit GET | `suspend fun getOrders(@Path("stationId") stationId: Int): List<StationOrder>` | `/api/v1/manager/orders/{stationId}` | `GameNetApi.kt:126` |
| Retrofit POST | `suspend fun saveOrder(@Body order: StationOrder): StationOrder` | `/api/v1/manager/orders` | `GameNetApi.kt:129` |
| Retrofit DELETE | `suspend fun deleteOrder(@Path("id") id: String): Response<Unit>` | `/api/v1/manager/orders/{id}` | `GameNetApi.kt:132` |
| Retrofit DELETE | `suspend fun clearOrders(@Path("stationId") stationId: Int): Response<Unit>` | `/api/v1/manager/orders/station/{stationId}` | `GameNetApi.kt:135` |
| Retrofit GET | `suspend fun getSessionHistory(): List<SessionHistory>` | `/api/v1/manager/session-history` | `GameNetApi.kt:138` |
| Retrofit POST | `suspend fun addSessionHistory(@Body history: SessionHistory): SessionHistory` | `/api/v1/manager/session-history` | `GameNetApi.kt:141` |
| Retrofit DELETE | `suspend fun clearSessionHistory(): Response<Unit>` | `/api/v1/manager/session-history` | `GameNetApi.kt:144` |
| Retrofit GET | `suspend fun getCustomers(): List<Customer>` | `/api/v1/manager/customers` | `GameNetApi.kt:147` |
| Retrofit POST | `suspend fun saveCustomer(@Body customer: Customer): Customer` | `/api/v1/manager/customers` | `GameNetApi.kt:150` |
| Retrofit DELETE | `suspend fun deleteCustomer(@Path("id") id: Long): Response<Unit>` | `/api/v1/manager/customers/{id}` | `GameNetApi.kt:153` |
| Retrofit POST | `suspend fun deleteCustomerBatch(@Body body: Map<String, List<Long>>): Response<Map<String, Any>>` | `/api/v1/manager/customers/delete-batch` | `GameNetApi.kt:156` |
| Retrofit GET | `suspend fun getReservations(): List<Reservation>` | `/api/v1/manager/reservations` | `GameNetApi.kt:159` |
| Retrofit DELETE | `suspend fun deleteReservation(@Path("id") id: Long): Response<Unit>` | `/api/v1/manager/reservations/{id}` | `GameNetApi.kt:163` |
| Retrofit GET | `suspend fun getSetting(@Path("key") key: String): Map<String, String>` | `/api/v1/manager/settings/{key}` | `GameNetApi.kt:166` |
| Retrofit POST | `suspend fun saveSetting(@Body body: Map<String, String>): Response<Unit>` | `/api/v1/manager/settings` | `GameNetApi.kt:169` |
| Retrofit POST | `suspend fun registerUser(@Body body: UserRegisterRequest): UserAuthResponse` | `/api/auth/customer/register` | `GameNetApi.kt:174` |
| Retrofit POST | `suspend fun loginUser(@Body body: UserLoginRequest): UserAuthResponse` | `/api/auth/manager/login` | `GameNetApi.kt:177` |
| Retrofit GET | `suspend fun pingSuperManager(): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/ping` | `GameNetApi.kt:181` |
| Retrofit POST | `suspend fun pingSuperManagerPost(): retrofit2.Response<okhttp3.ResponseBody>` | `/api/v1/super-manager/ping` | `GameNetApi.kt:184` |
| Retrofit GET | `suspend fun checkAuth(): Map<String, Any>` | `/api/v1/auth/check` | `GameNetApi.kt:192` |
| Retrofit POST | `suspend fun addUserDevice(@Body body: Map<String, Any>): Map<String, Any>` | `/api/v1/manager/device` | `GameNetApi.kt:199` |
| Retrofit GET | `suspend fun checkSubscriptionStatus(` | `/api/v1/subscriptions/check` | `GameNetApi.kt:216` |
| Retrofit GET | `suspend fun checkSubscription(` | `/api/v1/subscriptions/check` | `GameNetApi.kt:223` |
| Retrofit POST | `suspend fun checkLicenseStatus(@Body body: LicenseCheckRequest): LicenseCheckResponse` | `/api/v1/subscriptions/status` | `GameNetApi.kt:230` |
| Retrofit POST | `suspend fun activateLicense(@Body body: LicenseActivateRequest): LicenseCheckResponse` | `/api/v1/subscriptions/activate` | `GameNetApi.kt:233` |
| Retrofit POST | `suspend fun startFreeTrial(@Body body: TrialStartRequest): TrialStartResponse` | `/api/v1/trial/start` | `GameNetApi.kt:236` |
| Retrofit POST | `suspend fun buySubscription(@Body body: LicenseBuyRequest): LicenseBuyResponse` | `/api/v1/subscriptions/buy` | `GameNetApi.kt:239` |
| Retrofit POST | `suspend fun setLicensePassword(@Body body: SetPasswordRequest): Boolean` | `/api/v1/subscriptions/set-password` | `GameNetApi.kt:242` |
| Retrofit POST | `suspend fun validateCoupon(@Body params: Map<String, String>): Map<String, Any>` | `/api/v1/coupons/validate` | `GameNetApi.kt:245` |
| Retrofit GET | `suspend fun getSubscriptionPlans(): List<SubscriptionPlanDto>` | `/api/v1/plans` | `GameNetApi.kt:248` |
| Retrofit GET | `suspend fun getSubscriptionPlansAdmin(): SubscriptionPlansAdminResponse` | `/api/v1/super-manager/subscription-plans` | `GameNetApi.kt:251` |
| Retrofit PUT | `suspend fun updateSubscriptionPlansAdmin(@Body body: Map<String, Any>): SubscriptionPlansAdminResponse` | `/api/v1/super-manager/subscription-plans` | `GameNetApi.kt:254` |

### Direct/self-hosted call strings

- `/api/v1/manager/club/point-logs` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:265`
- `/api/v1/manager/club/point-logs/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:280`
- `/api/v1/customer/profile` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:364`
- `/api/auth/customer/login` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:405`
- `/api/v1/customer/profile` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:425`
- `/api/auth/customer/register` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:471`
- `/api/v1/manager/customers` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:533`
- `/api/v1/manager/customers/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:543`
- `/api/v1/manager/customers/delete-batch` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:557`
- `/api/v1/manager/customers` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:581`
- `/api/v1/manager/live-stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:595`
- `/api/v1/manager/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:663`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:778`
- `/api/v1/manager/manual-payment-requests/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:800`
- `/api/v1/manager/configuration` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:827`
- `/api/v1/manager/configuration` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:859`
- `/api/v1/manager/configuration` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:877`
- `/api/v1/manager/settings/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:897`
- `/api/v1/manager/announcements` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:930`
- `/api/v1/manager/audit-logs` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:965`
- `/api/v1/manager/club/ledger` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:995`
- `/api/v1/customer/club/ledger` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1011`
- `/api/v1/auth/check` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1053`
- `/api/v1/manager/diagnostics` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1077`
- `/api/v1/manager/customer-transactions` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1172`
- `/api/v1/customer/transactions` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1187`
- `/api/v1/manager/stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1239`
- `/api/v1/manager/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1270`
- `/api/v1/manager/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1292`
- `/api/v1/customer/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1326`
- `/api/v1/customer/stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1358`
- `/api/v1/customer/reservations/rules` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1375`
- `/api/v1/customer/reservations/pricing-preview` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1398`
- `/api/v1/customer/reservations/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1416`
- `/api/v1/customer/reservations/atomic` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1441`
- `/api/v1/manager/reservations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1469`
- `/api/v1/manager/reservations/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1496`
- `/api/v1/manager/reservation-payments/pending` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1510`
- `/api/v1/manager/reservation-payments/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1528`
- `/api/v1/manager/reservation-payments/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1545`
- `/api/v1/manager/reservations/` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1561`
- `/api/v1/customer/club/transfer` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1598`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1646`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1677`
- `/api/v1/customer/manual-payment-requests` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1713`
- `/api/v1/manager/live-stations` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1767`
- `/api/station/start` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1824`
- `/api/station/offline-start` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1883`
- `/api/station/order` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1919`
- `/api/station/event` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1952`
- `/api/station/settle` — `app/src/main/java/com/example/data/network/SelfHostedManager.kt:1971`
- `/api/v1/manager/stations/purge-extra` — `app/src/main/java/com/example/ui/GameNetViewModel.kt:3807`

> This inventory is syntactic extraction of Retrofit annotations and literal direct URLs. It is not a proof that each declaration is reachable in the current UI. API DTOs and Retrofit setup were inspected in `GameNetApi.kt`; direct OkHttp flows were inspected in `SelfHostedManager.kt`.

## Confirmed findings

### AND-001 — Plaintext customer password is persisted, displayed, and copied

| Field | Detail |
|---|---|
| Severity / priority | **High / P1** |
| Confidence | **High** — direct entity, migration, login/register, insert, display and clipboard paths are all present. |
| Impact | Anyone able to access the app’s local Room database or an authorized staff view that can manage customer passwords can obtain reusable customer passwords. The UI can additionally place the phone number/password pair on the Android system clipboard, widening exposure to other clipboard readers or the user interface. |
| Preconditions | A customer account is created/logged in with a nonblank password; staff reaches a customer card with `canManagePasswords` true for display/copy. |
| Reproduction | 1. Log in a customer using `GameNetViewModel.loginCustomer(phone, pass)`; 2. observe `SelfHostedManager.loginCustomer` creates `Customer(...).copy(password = cleanPass)`; 3. observe the ViewModel calls `repository.insertCustomer(cloudCust)`; 4. inspect the Room `customers` table / expand that customer card in the staff UI; 5. press **کپی** (“Copy”) to set `username + password` as the primary clipboard clip. No source/runtime inference is needed for the storage/display flow. |
| Expected | Do not retain a login password in the normal business `Customer` record. Store only a server-issued token in protected storage where persistence is required; never render passwords or put them in the system clipboard. Use a reset/one-time credential workflow instead. |
| Actual | The password is a normal Room `TEXT` column and is directly interpolated into staff UI and `ClipData`. |
| Root cause | Credential material is modeled as a business/customer field and propagated through login, register, CRUD, sync and UI rather than being discarded after server authentication. |
| Fix | Remove `Customer.password` and its DAO/repository/sync serialization and migrate existing values away. Do not return/store customer plaintext passwords. Use password reset/one-time setup mechanisms; display only nonsecret account identifiers. If a temporary generated code is mandatory, make it short-lived, single-use, server-verified, protected, and prevent clipboard copying by default. Rotate/reset affected customer credentials. |
| Regression risk | **High:** schema migration and manager/customer sync contracts will change; legacy staff flows that show/share credentials need replacement. Ensure migration deletes the column/value and tests cover password reset, login and customer import/update. |
| Verification | Add a Room migration test asserting no password column/data, a repository serialization test asserting no `password` JSON field, Compose tests asserting no customer password/clipboard `ClipData`, and an authenticated end-to-end login/reset test. Inspect a post-migration database. |

**Evidence and caller/callee chain**

- Entity/callee: `app/src/main/java/com/example/data/Entities.kt:177-183` declares `@Entity(tableName = "customers")` and `val password: String = ""`.
- Migration: `app/src/main/java/com/example/data/AppDatabase.kt:69-73` runs **`ALTER TABLE customers ADD COLUMN password TEXT NOT NULL DEFAULT ''`**; normal Room builder at `162-176` has no database encryption configuration.
- Login source: `SelfHostedManager.kt:398-405` posts **`put("password", cleanPass)`**; `436` sets **`parseCustomerObject(...).copy(password = cleanPass)`**.
- Persistence caller: `GameNetViewModel.kt:997-1003` calls `SelfHostedManager.loginCustomer` then **`repository.insertCustomer(cloudCust)`**.
- Registration source: `SelfHostedManager.kt:461-485` posts the password and sets **`copy(password = passwordText.trim())`**.
- UI/action: `CustomersReservationsScreen.kt:1650-1688` gates on `customer.password.isNotBlank() && canManagePasswords`, displays **`"رمز اپلیکیشن مشتری: ${customer.password}"`**, and sends **`"نام کاربری: …\nرمز عبور: ${customer.password}…"`** to `clipboard.setPrimaryClip(clip)`.
- Additional propagation: `SelfHostedManager.kt:527-533` includes `put("password",customer.password)` in manager customer upsert.

### AND-002 — VIP reservation rules call omits customer Authorization

| Field | Detail |
|---|---|
| Severity / priority | **High / P1** |
| Confidence | **High** — Android header selection, UI caller/result behavior, and the backend route’s `requireCustomerAuth` middleware are all source-proven. |
| Impact | In an ordinary customer-only session, the VIP/full-hall tab cannot load its reservation rules. `rules` becomes `null`, the durations array is empty, and no request can be enabled/submitted. A stale manager token does not correct this: the endpoint requires a customer token. |
| Preconditions | Customer logged in, tier `GOLD` or `DIAMOND`, then navigates to the `CustomerFullHallTab`; no valid manager token has been placed in `NetworkClient.authToken` as a customer token. |
| Reproduction | 1. Customer logs in (the code assigns `NetworkClient.customerAuthToken`, not `authToken`); 2. open full-hall/VIP tab; 3. `LaunchedEffect` invokes `fetchReservationRules`; 4. its request uses `getBaseHeaders`; 5. `getBaseHeaders` has no `customerAuthToken`, while backend route requires customer auth; 6. non-2xx returns `null`; 7. UI sees no `vipDurations`, and the register button remains disabled. |
| Expected | `GET /api/v1/customer/reservations/rules` must send the customer bearer token and render server-supplied rules/durations to an authenticated customer. |
| Actual | It sends `getBaseHeaders()` rather than `getCustomerHeaders()`. The method converts any non-successful response to `null`, and UI then uses an empty JSON object/empty duration list. |
| Root cause | Header-builder confusion between manager/base and customer-authenticated direct calls. Neighboring pricing, cancellation and atomic routes use the correct builder; the rules route is the inconsistent call. |
| Fix | Change only `fetchReservationRules` to use `getCustomerHeaders()` (and preserve any required non-authority headers explicitly if contractually needed). Surface a distinguishable auth/configuration error instead of silently mapping all non-2xx responses to `null`. |
| Regression risk | **Medium:** header change may expose any untested backend requirement for `X-Manager-ID`; backend source shows the route derives manager scope from customer JWT. Verify no manager header is required. |
| Verification | Unit-test the OkHttp request header contains `Authorization: Bearer <customer token>` and no manager token; integration-test a Gold/Diamond customer receives 200/rules; Compose-test duration chips render and valid selections enable submission. Test 401/403 error UI separately. |

**Evidence and caller/callee chain**

- Customer header definition: `SelfHostedManager.kt:243-250` adds **`Authorization: Bearer $it`** only from `NetworkClient.customerAuthToken`.
- Incorrect base header definition: `SelfHostedManager.kt:228-240` adds authorization only from **`NetworkClient.authToken`** and may add `X-Manager-ID`.
- Defective caller/callee: `SelfHostedManager.kt:1372-1383` builds `GET "$SERVER_URL/api/v1/customer/reservations/rules…"` with **`.headers(getBaseHeaders())`** and returns `null` if `!response.isSuccessful`.
- Correct neighboring pattern: pricing uses **`.headers(getCustomerHeaders())`** (`1387-1406`); cancellation uses it (`1410-1424`); atomic reservation uses it plus `Idempotency-Key` (`1427-1442`).
- UI caller/result: `CustomerFullHallTab.kt:43-55` calls `fetchReservationRules`, then reads `rules?.optJSONObject("rules")?.optJSONArray("vipDurationsMinutes")`; `144-167` requires `vipDurations.contains(selectedDuration)` before submission.
- Backend contract used only to prove the Android call: `backend/canonical_routes.js:336` declares **`app.get('/api/v1/customer/reservations/rules', requireCustomerAuth, …)`**. It similarly requires customer auth for pricing/atomic at `384-385`.

### AND-003 — Release lint quality/security gate is disabled

| Field | Detail |
|---|---|
| Severity / priority | **Medium / P2** |
| Confidence | **High** — direct Gradle configuration. |
| Impact | Release builds do not enforce Android lint, do not fail on lint errors, and do not inspect dependencies through lint. Manifest/API/lifecycle/security regressions that lint could detect can reach release builds without blocking the build. This audit does not claim a particular lint error exists because lint could not run in this sandbox. |
| Reproduction | Inspect/run the configured release build: the Gradle Android `lint` block has `checkReleaseBuilds = false`, `abortOnError = false`, and `checkDependencies = false`. |
| Expected | CI/release builds should run lint for release variants, fail on actionable lint errors, and check dependencies (with narrowly documented suppressions/baselines for accepted findings). |
| Actual | All three safeguards are disabled. |
| Root cause | Explicit build configuration disables lint enforcement. |
| Fix | Set `checkReleaseBuilds = true`, `abortOnError = true`, `checkDependencies = true`; introduce a reviewed lint baseline only for known accepted debt and run `:app:lintRelease` in CI. |
| Regression risk | **Low–Medium:** enabling lint may initially fail on accumulated issues and/or require dependency updates; phase with baseline ownership rather than leaving the gate off. |
| Verification | In an Android-SDK-equipped CI runner, execute `./gradlew :app:lintRelease :app:assembleRelease`; confirm lint runs, reports dependencies, and fails the job on a deliberately introduced known lint violation. |

**Evidence**

- `app/build.gradle.kts:31-35` exactly states:

```kotlin
lint {
  checkReleaseBuilds = false
  abortOnError = false
  checkDependencies = false
}
```

- Release is a defined build type at `78-85`, so this is release configuration rather than a dead module setting.

## Items investigated but not elevated to findings

These are **not findings** because the audit did not establish a defect/impact beyond the fact stated:

1. **Applet duplicate source:** `app/applet/app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` and `app/src/main/java/com/example/ui/SubscriptionActivationScreen.kt` share a fully qualified name, but `settings.gradle.kts:22-23` includes only `:app`, and no applet build descriptor was found. The applet source is excluded from this build. Its intended inclusion/deployment is unverified.
2. **Hard-coded business values:** default prices, GN ratios, plan mappings and trial durations are present (examples above). Code does not establish whether those values conflict with approved business policy.
3. **Retry policy:** repeated non-idempotent direct requests could be operationally significant, but determining endpoint idempotency requires executing/contract-testing each route. Customer reservation calls do send idempotency keys; no generalized defect was asserted.
4. **Deep links/payment callbacks:** the exported deep links were traced in `MainActivity`, but the audit did not execute an end-to-end payment provider callback. No authorization bypass is claimed without that test and server interaction.
5. **Time/Jalali/locale correctness:** conversion/display code was inspected, including Asia/Tehran and Western-digit behavior. Boundary-date tests (leap years/DST/locales) were not runnable, so no calendar correctness claim was made.
6. **Role/permission enforcement:** UI and ViewModel actions were traced for representative high-impact actions. Complete server authorization is backend-owned and was not runtime tested under every role; absence of a reported bypass is not a certification.
7. **Room migrations and offline sync reconciliation:** static paths were reviewed; real database upgrade, conflict, network-loss, process-death, and server reconciliation scenarios were not executed.

## Tests and checks run

| Check | Command / scope | Outcome |
|---|---|---|
| Repository/ref check | `git rev-parse HEAD`, `git branch --show-current`, `git status --short` | Observed requested SHA on `main`; no source modification made by this audit. |
| Gradle compile + local unit test attempt | `./gradlew :app:compileDebugKotlin :app:testDebugUnitTest --stacktrace` | **Blocked before task execution.** Gradle says: `SDK location not found. Define a valid SDK location with an ANDROID_HOME environment variable or ... local.properties` (`gradle-compile-test.log:16-20`). No PASS claim. |
| Kotlin file inventory | Read-only enumeration of `app/**/*.kt` | **Completed:** 58 Kotlin files inventoried (table above). |
| Encoding sanity | Read all 58 Kotlin files as bytes/UTF-8 | **Completed:** no NUL bytes; UTF-8 decode passed. This is not compilation. |
| Delimiter heuristic | Comment/string-aware Python delimiter scan | **Inconclusive:** it reported `NetworkLogger.kt` an unclosed `[` at line 145, caused by regex syntax that the heuristic does not parse; it is not a compiler result and was not treated as a build finding. |
| Contract cross-reference | Read only the necessary backend route definitions | **Completed statically:** confirmed the reservation-rules route middleware required for AND-002. No HTTP request was sent to a deployed backend. |
| Unit/instrumented test source review | All 5 `src/test` Kotlin files and 1 `src/androidTest` file | **Completed statically; not executed.** |

## Recommended verification order

1. **Immediately remediate/contain AND-001:** stop new password persistence/display/copy, rotate/reset affected customer passwords, and plan a tested Room data-removal migration.
2. **Fix AND-002 and add a customer-auth request test:** this directly restores the VIP/full-hall flow for customer sessions.
3. **Enable lint gates (AND-003) in a CI environment with Android SDK 36 installed**, then run compile/unit/instrumentation tests and inspect resulting lint findings.
4. Execute manual and automated offline/sync, license/trial, Jalali boundary, deep-link/payment, Room migration and role-matrix test plans before treating the audit as release approval.

## Evidence artifacts retained for audit reproducibility

- Build attempt log: `/home/ubuntu/jobs/1748d5d62ac7_a0/gradle-compile-test.log`
- Source/definition scans: `/home/ubuntu/jobs/1748d5d62ac7_a0/kotlin-definition-inventory.md`, `/home/ubuntu/jobs/1748d5d62ac7_a0/kotlin-coverage-inventory.tsv`
- API extraction: `/home/ubuntu/jobs/1748d5d62ac7_a0/api-inventory-generated.md`, `/home/ubuntu/jobs/1748d5d62ac7_a0/android-endpoints.tsv`
- Contract and password-flow extracts: `/home/ubuntu/jobs/1748d5d62ac7_a0/backend-route-contracts-targeted.txt`, `/home/ubuntu/jobs/1748d5d62ac7_a0/customer-password-trace.txt`

---

**Audit conclusion:** The report establishes exactly three Android-side findings from code/configuration and clearly delineates the unexecuted/runtime-dependent areas. It is an engineering audit, **not a release certification**.


---

## خروجی ۳ — ممیزی Backend، API، Database و Transactions

> فایل منبع این بخش: `GameNexa-Audit-backend.md`

# GameNexa Backend/API/Database Audit

**Audit ID:** `GN-BE-2026-10-02-05348f2`  
**Target:** `/home/ubuntu/GameNexa-release-2026` at `05348f2df8b1f6c30ee233071be13a899d969e25` on `main`  
**Scope:** Backend/API/database only — `server.js`, `canonical_routes.js`, `reservationService.js`, `financialService.js`, `bootstrap.js`, `schema.sql`, test files, Docker/Nginx/runtime configuration, and Android callers only where they clarify an API contract.  
**Excluded:** Production systems, deployed databases, and source modifications. No production system was contacted.

## Executive summary

I extracted **108 Express routes** and reviewed their authorization gates, validation, tenancy scoping, SQL, transactions, idempotency, constraints, and the station/session, reservation, financial, GN/LP, customer, buffet/order, entitlement, subscription, and trial flows.

**Eight evidence-backed findings** were identified:

| Severity | ID | Finding |
|---|---|---|
| High | GN-BE-01 | The authenticated manager device-binding endpoint bypasses the entitlement device cap enforced at login. |
| High | GN-BE-02 | Guest participants are counted in session cost shares but skipped when invoices are written, losing revenue. |
| High | GN-BE-03 | Station-start prepayments are accepted and stored as snapshots but never applied to invoices, payment transactions, or settlement. |
| High | GN-BE-04 | Confirming a VIP reservation supersedes paid normal reservations without refunding them, then makes them non-cancellable. |
| High | GN-BE-05 | Generic manual-wallet approval permits an arbitrary, including negative, approved amount and bypasses the wallet ledger/audit trail. |
| High | GN-BE-06 | Android GN-purchase calls are silently treated as wallet top-ups, so approved purchases do not credit GN. |
| Medium | GN-BE-07 | Trial enforcement trusts caller-supplied device identifiers, allowing a caller to mint a new 24-hour trial with a new pair. |
| Medium | GN-BE-08 | A customer can directly reserve a disabled or non-reservable station despite the customer station-list policy. |

The application also has meaningful positive controls: parameterized SQL was used in the reviewed stateful flows; tenant predicates are common; session start has an advisory lock and active-session guard; session events and reservation/payment ledgers have unique keys; the reservation exclusion constraint protects ordinary station overlap; and invoice payment has amount, transaction, audit, and idempotency logic. Those controls do **not** compensate for the defects below.

> **Evidence standard:** A finding is based on source that establishes the behavior, not a claim about a deployed environment. Status codes and payloads are stated only where current code establishes them. Database-backed runtime proof was not possible because no local PostgreSQL/Docker runtime was installed or listening; this is detailed in [Testing](#testing-and-evidence-level).

---

## Scope and review method

1. Verified the specified Git commit and left source clean (`git diff --exit-code` passed).
2. Read all backend implementation files and `schema.sql`; no backend migrations directory or migration files were present.
3. Parsed both Express source files into the exhaustive 108-route inventory in Appendix A.
4. Traced authorization, ownership predicates, SQL parameterization, transaction boundaries, constraints, idempotency keys, and concurrency controls for reservation, manual payment, invoices, GN/LP, session, trial, and subscription flows.
5. Compared relevant Android request construction where it exposes a backend contract mismatch.
6. Ran source-only and mock tests locally; did not connect to production or start a service.

### Runtime/deployment observations

- `docker-compose.yml:3-57` defines PostgreSQL 16, API, and Nginx. PostgreSQL is not published to a host port; API is behind Nginx. Health checks wait for PostgreSQL and check `/api/v1/time` (`docker-compose.yml:28-37`).
- Nginx redirects HTTP to HTTPS, limits requests at `10r/s` with burst 20, enables TLS 1.2/1.3 and HSTS (`nginx.conf:8-47`).
- The Compose database volume is declared `external: true` (`docker-compose.yml:54-57`), so a fresh local Compose run would require a pre-existing named volume. Docker itself was unavailable in this sandbox.

---

## Findings

### GN-BE-01 — Manager device cap can be bypassed after login

**Severity:** High  
**Affected HTTP path:** `POST /api/v1/manager/device`  
**Authentication:** Manager bearer token plus active entitlement  
**Established response:** `200 {"success":true}` on an insert/update; `400 {"error":"device_id required"}` only if the body has no device ID; `500` on SQL error (`canonical_routes.js:471`).

**Evidence**

The login flow correctly computes the active entitlement's `max_devices`, obtains a transaction-scoped advisory lock, counts active bindings, and rejects a previously unknown device with `403 DEVICE_LIMIT_REACHED` when the cap is met (`server.js:211-239`). Its controlled insert is:

```js
if (Number(count.rows[0].count || 0) >= maxDevices) {
  return res.status(403).json({ error:'Maximum authorized devices reached', code:'DEVICE_LIMIT_REACHED', maxDevices });
}
await lock.query("INSERT INTO manager_device_bindings(manager_id,device_id) ...")
```

But the separate post-login binding route contains no entitlement lookup, count, or advisory lock:

```js
// canonical_routes.js:471
await pool.query(
  'INSERT INTO manager_device_bindings(manager_id,device_id,active,last_seen_at) '
  + 'VALUES($1,$2,TRUE,NOW()) ON CONFLICT(manager_id,device_id) '
  + 'DO UPDATE SET active=TRUE,last_seen_at=NOW()', [mid,did]
);
res.json({success:true});
```

The database only guarantees one row for a given `(manager_id, device_id)` (`schema.sql:1483-1488`); it has no maximum-active-device constraint. Once a device is written through this endpoint, a later login recognizes it as already active and skips the cap branch (`server.js:222-231`).

**Reproduction / exploit sequence (source-established)**

1. Obtain a valid manager bearer for an entitlement with `max_devices: 1`, after one device is already active.
2. Send `POST /api/v1/manager/device` with `{"deviceId":"second-device"}`.
3. The route writes a second active `manager_device_bindings` row and returns 200.
4. Login with that device ID is treated as known, avoiding the cap check.

**Root cause:** The entitlement invariant is implemented only in the login handler rather than centralized and reused by all writes to `manager_device_bindings`.

**Impact:** Subscription/device licensing can be exceeded by any holder of a valid manager session. It also weakens the intended ability to revoke a device as an effective login gate.

**Fix:** Remove this route unless it is necessary, or make it call one shared `bindManagerDevice()` routine which, in a single transaction, locks the manager, reads the active entitlement, counts active bindings, enforces the cap, and returns `403 DEVICE_LIMIT_REACHED`. Consider binding a token/session to a device and checking that binding on privileged requests.

**Regression / verification:** With a max-device entitlement of 1 and device A active, POSTing device B must return 403 and leave exactly one active binding. After a Super Manager revokes A, binding/logging in B must succeed. Run that test concurrently to prove the advisory lock prevents two device-B/device-C enrollments.

---

### GN-BE-02 — Guest shares disappear from session invoices

**Severity:** High  
**Affected HTTP paths:** `POST /api/station/start`, `POST /api/station/settle`  
**Authentication:** Manager bearer plus active entitlement  
**Established responses:** start returns `201` with `sessionId` (`server.js:1014-1015`); settlement returns `200` with `totalCost` and generated `invoices` (`server.js:1427-1429`).

**Evidence**

The start handler explicitly accepts a participant without a positive `customerId` as a guest (`server.js:850-857`) and stores every participant as a payer:

```js
// server.js:1001-1005
INSERT INTO session_participants(..., is_guest, is_payer)
VALUES(..., $6, TRUE)
```

Settlement divides game and buffet cost by *all* payers (`server.js:1372-1378`). It then skips every guest before invoice construction:

```js
// server.js:1390-1396
for (let payerIndex = 0; payerIndex < payers.length; payerIndex += 1) {
  const payer = payers[payerIndex];
  if (!payer.customer_id) continue;
  const shareGame = ...;
  const shareBuffet = ...;
```

Only non-guests reach the invoice insert (`server.js:1420-1423`), which writes `paid_amount` as `0`. The schema makes an invoice customer mandatory: `invoices.customer_id integer NOT NULL` (`schema.sql:378-404`). There is no anonymous/guest payable table or transfer of a guest share to a registered payer.

**Concrete behavior:** Start a session with one registered customer and one guest, both default payers, and no discounts. At settlement, `totalCost` is divided by two. The guest share is skipped; only the registered customer's half is invoiced. The response can therefore report a session total greater than the sum of its invoices.

**Root cause:** Allocation cardinality (`is_payer`) and billable-invoice cardinality (`customer_id`) differ without a reconciliation policy.

**Impact:** Underbilling and an unreconcilable financial record for every session containing billable guests. Any remainder assigned to a final guest is also skipped.

**Fix:** Require every guest to nominate a registered sponsor/payer before settlement, or add an explicit anonymous/walk-in payer/invoice model. Enforce the invariant before commit: `sum(invoice total_amount) + cash/guest receivable = game_sessions.total_cost` (with documented discounts separated). Reject settlement when no valid billing destination exists.

**Regression / verification:** Create a session with one registered customer and one guest at a deterministic hourly rate; settle it and assert conservation. Test both guest-first and guest-last ordering, a buffet line, an indivisible remainder, and a guest explicitly marked non-payer. The transaction should roll back if no sponsor is supplied.

---

### GN-BE-03 — Session prepayments are accepted but never reconciled financially

**Severity:** High  
**Affected HTTP paths:** `POST /api/station/start`, `POST /api/station/offline-start`, `POST /api/station/settle`  
**Authentication:** Manager bearer plus active entitlement.

**Evidence**

`/api/station/start` parses and validates an integer `prepaymentAmount` and `customerPrepayments` (`server.js:876-903`), records them in the immutable pricing snapshot (`server.js:978-990`), and writes per-customer values to `session_participants.prepayment_amount` (`server.js:1006-1011`). The offline-start endpoint accepts the same fields (`server.js:1025-1041`). Android also sends these values in its canonical start request, including a map keyed by customer ID (`SelfHostedManager.kt:1782-1828`).

Settlement, however, only computes game/buffet costs and updates `game_sessions` (`server.js:1359-1371`), then creates each invoice with a literal paid amount of zero:

```sql
-- server.js:1420-1422
INSERT INTO invoices(... total_amount, paid_amount, ...)
VALUES (..., ($6::numeric+$7::numeric), 0, ...)
```

That handler does not read `prepayment_amount`, `initialPrepaymentAmount`, or `customerPrepayments`, and it creates no `payment_transactions`, `wallet_transactions`, or `financial_audit_logs` for the start-time money. The later invoice-payment endpoint is separate and records only a fresh manager confirmation (`server.js:1230-1295`).

**Root cause:** “Prepayment” is represented as session metadata rather than a financial event, and settlement does not consume that metadata.

**Impact:** A manager can record cash/prepayment in the app but the invoice remains wholly unpaid. Staff may collect twice, lose traceability for money received, or disagree over balances after offline reconciliation.

**Fix:** Define prepayment semantics. If it is money received, create an exact, idempotent payment/ledger/audit event in the station-start transaction and allocate it to the eventual invoices at settlement (with a conservation rule). If it is only a UI estimate, remove it from the financial API contract and name it accordingly. The offline reconciliation route must follow the same ledger path.

**Regression / verification:** Start a two-customer session with both an aggregate and per-customer prepayment, settle it, and verify exact `payment_transactions`, invoice `paid_amount/status`, and financial audit totals. Replay the idempotency key and confirm no second payment. Test partial, full, overpayment rejection/credit policy, guest allocation, and offline-start reconciliation.

---

### GN-BE-04 — VIP priority can strand funds from a paid normal reservation

**Severity:** High  
**Affected HTTP paths:** `POST /api/v1/manager/reservation-payments/:id/approve`, `PUT /api/v1/manager/reservations/:id/status`, cancellation paths  
**Authentication:** Manager bearer plus active entitlement; customer cancellation is customer bearer.

**Evidence**

A manual reservation payment is persisted as a successful `payment_transactions` row before the reservation moves to `PENDING_APPROVAL`/`VIP_PAYMENT_PAID` (`canonical_routes.js:427-458`). A paid normal reservation can then be confirmed. When a VIP is confirmed, the status route locks ordinary overlapping reservations and changes each to `SUPERSEDED_BY_VIP_PRIORITY`:

```js
// canonical_routes.js:273-295
const overlaps = await c.query(`SELECT id,status,... FOR UPDATE`, ...);
for (const row of overlaps.rows) {
  await c.query("UPDATE reservations SET status='SUPERSEDED_BY_VIP_PRIORITY' ...", [row.id,mid]);
  await c.query('INSERT INTO reservation_audit_logs(...)', ...);
}
```

That loop does not insert a wallet refund, payment reversal, `financial_audit_logs` event, notification, or credit. The normal cancellation service explicitly refuses both superseded states:

```js
// financialService.js:195-197
if (['COMPLETED','EXPIRED','REJECTED','NO_SHOW',
     'SUPERSEDED_BY_VIP','SUPERSEDED_BY_VIP_PRIORITY'].includes(String(r.status))) {
  throw new Error('RESERVATION_CANNOT_BE_CANCELLED_IN_CURRENT_STATUS');
}
```

The customer and manager cancel route catch this and return `400 {error: ...}` (`canonical_routes.js:223-232`, `385-386`).

**Source-established sequence**

1. Approve the full manual payment for a normal booking; the dedicated approval endpoint inserts its successful payment.
2. Confirm that normal reservation.
3. Fully pay and confirm an overlapping VIP reservation.
4. The normal reservation is marked `SUPERSEDED_BY_VIP_PRIORITY`; no refund is created.
5. Calling either cancellation endpoint fails because that state is non-cancellable.

**Root cause:** VIP capacity arbitration changes reservation state but does not invoke the financial cancellation/refund workflow before rendering the old reservation terminal.

**Impact:** Customer money can remain associated with a reservation that cannot occur and cannot use the normal refund path. This creates a direct financial-loss/operational-dispute risk.

**Fix:** In the same transaction that supersedes each paid reservation, calculate paid funds from `payment_transactions`, create an idempotent wallet credit/refund and `financial_audit_logs` record, record a reason and notification, then change status. If business policy permits a penalty, snapshot and apply it explicitly rather than silently retaining the payment. A retry-safe unique key should be scoped to `vip-supersede-refund:<reservationId>`.

**Regression / verification:** Build a paid, confirmed normal booking that overlaps a paid VIP confirmation. Assert the normal booking becomes superseded, the sum of successful payment transactions is credited exactly once according to policy, a financial audit entry exists, the customer receives a notification, and a replay of the VIP confirmation cannot duplicate the credit.

---

### GN-BE-05 — Generic manual wallet approval has no amount guard or ledger/audit entry

**Severity:** High  
**Affected HTTP path:** `POST /api/v1/manager/manual-payment-requests/:id/approve`  
**Authentication:** Manager bearer plus active entitlement  
**Established responses:** 404 if no pending request; otherwise 200 `{success:true}`; catch-all 500 (`canonical_routes.js:400`).

**Evidence**

The generic approval handler locks only a pending request and applies `approvedAmount` directly, without checking finite/positive value, currency, or whether it is less than/equal to the requested amount:

```js
// canonical_routes.js:400
UPDATE manual_payment_requests SET status='APPROVED',approved_amount=$1, ...
  [Number(req.body?.approvedAmount||r.amount), ...]

if (r.purpose==='WALLET_TOPUP')
  UPDATE customers SET wallet_balance=wallet_balance+$1 ...
  [Number(req.body?.approvedAmount||r.amount), ...]
```

An authenticated manager can send `{"approvedAmount": -500}` for a pending `WALLET_TOPUP`: `-500` is truthy and is directly applied. It can also approve an amount greater than the original request. This path writes neither `wallet_transactions` nor `financial_audit_logs`; contrast the reservation payment path, which writes a `payment_transactions` row and audit trail (`canonical_routes.js:449-457`). `customers.wallet_balance` has no nonnegative check (`schema.sql:214-235`), whereas `wallet_transactions.amount` does require a nonnegative amount (`schema.sql:1142-1154`).

**Root cause:** The generic manual-review endpoint treats manager input as inherently valid and mutates a derived balance directly instead of using the wallet ledger as the source of truth.

**Impact:** A typo, malformed integration, or compromised manager bearer can credit any amount or push a wallet balance negative, without a wallet transaction or financial audit record that explains the change. Reconciliation of `customers.wallet_balance` to the wallet ledger is impossible for this path.

**Fix:** Validate `approvedAmount` with a decimal-string parser (positive, finite, approved currency) and bound it to the requested amount unless a separately authorized adjustment workflow is used. Insert an idempotent `wallet_transactions` credit/debit and `financial_audit_logs` row before updating the materialized balance; derive/reconcile balances from the ledger. Add database checks for nonnegative wallet balance if negative balances are disallowed.

**Regression / verification:** For a pending top-up, reject negative, zero, non-finite, and over-request amounts with 4xx and no mutation. A valid approval must produce one ledger row, one audit row, and one matching balance delta; a replay must not add another delta. Test an intentional adjustment only through a separately scoped/audited API.

---

### GN-BE-06 — Android “Buy GN” request silently becomes a wallet top-up

**Severity:** High  
**Affected HTTP path:** `POST /api/v1/customer/manual-payment-requests` followed by generic approval  
**Authentication:** Customer bearer for request; manager bearer plus active entitlement for approval  
**Established response:** accepted request returns `201 {success:true, request:...}` (`canonical_routes.js:397`).

**Evidence**

Android's GN purchase call sends `gnAmount` and `transactionType: "BUY_GN"`, but does not send a supported `purpose` or idempotency key:

```kotlin
// SelfHostedManager.kt:1694-1715
put("amount", totalToman.toLong())
put("gnAmount", gnAmount)
put("transactionType", "BUY_GN")
.url("$SERVER_URL/api/v1/customer/manual-payment-requests")
```

The backend ignores `gnAmount` for behavior and defaults any missing `purpose` to `WALLET_TOPUP`:

```js
// canonical_routes.js:397
const purpose = String(b.purpose || 'WALLET_TOPUP').toUpperCase();
if (!['WALLET_TOPUP','RESERVATION_PAYMENT'].includes(purpose)) return res.status(422)...
```

The generic manager approval only changes `wallet_balance` for that default purpose (`canonical_routes.js:400`). It neither inserts `gn_ledger` nor changes `customers.gn_balance`/`pending_gn`. The Android client immediately increments only its local `pendingGn` after a 2xx response (`SelfHostedManager.kt:1720-1723`), so its state diverges from the server after approval.

**Root cause:** The mobile payload's `transactionType`/`gnAmount` contract has no corresponding backend purpose or state transition, and the permissive default hides the mismatch instead of rejecting it.

**Impact:** A customer can submit and pay for a GN purchase that is approved as ordinary wallet credit, while the app displays pending GN. This causes customer entitlement loss and backend/client balance divergence.

**Fix:** Add an explicit, authenticated `GN_PURCHASE` purpose with strict fields and idempotency. On approval, atomically write a GN ledger credit, update `gn_balance` (and any intentionally used `pending_gn` transition), and audit the fiat receipt. Alternatively, reject `transactionType: BUY_GN` with 422 until GN purchase is supported. Do not silently default an unknown financial intent to wallet top-up.

**Regression / verification:** Submit the Android-equivalent payload and assert either a deterministic 422 with no state change or a request classified `GN_PURCHASE`. After approval, assert exactly one GN ledger credit and matching `gn_balance` increase, no unintended wallet credit, and a client reload that agrees with the server.

---

### GN-BE-07 — Trial identity can be reset by choosing a new caller-supplied pair

**Severity:** Medium  
**Affected HTTP path:** `POST /api/v1/trial/start`  
**Authentication:** Public  
**Established responses:** `400` if either identifier is blank, `403` if an authenticated Manager starts a trial or a matching identifier is blocked, `201` for a new pair, and `200` for an existing pair (`server.js:313-353`).

**Evidence**

The public endpoint takes both identifiers directly from JSON, merely trims/truncates them, and uses them as the whole eligibility boundary:

```js
// server.js:311-346
const deviceId = normalizeTrialIdentity(req.body?.deviceId || req.body?.device_id);
const deviceFingerprint = normalizeTrialIdentity(req.body?.deviceFingerprint || req.body?.device_fingerprint);
...
SELECT ... FROM trial_devices WHERE device_id = $1 OR device_fingerprint = $2
...
INSERT INTO trial_devices (device_id, device_fingerprint, ... expires_at, status)
VALUES ($1, $2, ..., $5, 'ACTIVE')
```

The route has an IP-keyed application rate limit of five per minute (`server.js:313`), but no user/account validation, app/device attestation, signed device claim, or server-maintained hardware identity. `trial_devices` uses `device_id` as primary key (`schema.sql:1126-1135`, `1723-1728`), so a caller choosing a previously unseen `deviceId` and `deviceFingerprint` gets a new 24-hour row and a 201 response.

**Root cause:** Client-provided, mutable identifiers are treated as proof of one physical device.

**Impact:** The one-trial-per-device commercial control is trivially bypassable by a direct API caller changing both strings. This can also fill the trial administration list with junk records.

**Fix:** Bind trial issuance to a verifiable identity appropriate to the Android threat model (e.g., Play Integrity/device attestation validated server-side plus an account/phone policy), retain a server-side risk/abuse record, and enforce broader WAF/IP/account quotas. Do not claim that an arbitrary string pair establishes a device identity.

**Regression / verification:** Use a mocked valid attestation for device A to obtain one trial; replay succeeds only as the same trial. A fabricated new device/fingerprint with no valid attestation must fail. Verify blocked identifiers, retry behavior, privacy handling, and administrative deletion/extension.

---

### GN-BE-08 — Direct reservation bypasses disabled/non-reservable station policy

**Severity:** Medium  
**Affected HTTP path:** `POST /api/v1/customer/reservations` and `/api/v1/customer/reservations/atomic`  
**Authentication:** Customer bearer  
**Established response:** a successful booking returns `201` (`canonical_routes.js:309-321`, `385`), subject to the service's other checks.

**Evidence**

The customer-facing station discovery endpoint deliberately exposes only active, reservable stations:

```sql
-- canonical_routes.js:323-334
WHERE s.manager_id=$1 AND s.active=TRUE AND s.reservable=TRUE
```

But normal reservation booking passes caller-controlled station IDs to `bookReservation`. Its ownership check is only:

```sql
-- reservationService.js:225-243
SELECT * FROM stations WHERE id = ANY($1) AND manager_id = $2
```

It checks controller capacity but never checks `stations.active` or `stations.reservable`. Thus a customer knowing an ID in their own tenant can submit it directly in `stationIds`/`stationId`, bypassing the list's policy. The service's VIP/full-hall branch filters `active = true` when it expands all stations (`reservationService.js:193-196`), but normal booking does not.

**Root cause:** Availability policy is enforced in a discovery/read route but not at the authoritative write boundary.

**Impact:** Reservations and payment requests can be created for maintenance-disabled or intentionally non-bookable stations, producing operational conflicts and possible invalid charges.

**Fix:** In the same write transaction and before price calculation, select stations `FOR UPDATE` with `manager_id`, `active=TRUE`, and `reservable=TRUE` for all customer-originated normal reservations. Return a clear 422/409 policy error. Keep manager-only overrides explicit and separately authorized if they are required.

**Regression / verification:** Create one active/reservable station, one inactive station, and one active/non-reservable station. Customer list returns only the first. Direct normal reservation attempts for the latter two must fail without creating reservations/idempotency records; the valid station must still create exactly one booking under concurrent retries.

---

## Cross-cutting authorization, consistency, and database observations

### Controls verified

- **Tenant scoping:** Reviewed financial, reservation, invoice, customer, GN transfer, and session queries commonly bind `manager_id`; static tenant-isolation checks passed. Examples include reservation ownership locks (`financialService.js:187-193`) and session station ownership (`server.js:933-938`).
- **Reservation overlap:** `reservations.no_overlap` is a PostgreSQL GiST exclusion constraint on manager, station, and time range for live blocking states (`schema.sql:1547-1552`). This is stronger than application-only overlap checks and guards concurrent ordinary bookings.
- **Reservation idempotency:** `reservation_request_idempotency` has a unique `(manager_id,idempotency_key)` constraint (`schema.sql:1603-1608`); the `/atomic` route also takes an advisory lock (`canonical_routes.js:385`).
- **Session concurrency:** Start takes an advisory lock keyed by manager/station and locks active sessions (`server.js:906-943`). Event IDs and per-session sequence numbers are uniquely constrained (`schema.sql:1626-1647`).
- **Invoice payments:** `/api/station/invoice/pay` requires an idempotency key, locks the invoice, rejects payments above remaining balance, inserts a successful payment, updates status, and writes a financial audit record (`server.js:1230-1295`).
- **GN transfer:** Customer transfer validates a positive finite amount and idempotency key, locks sender/recipient, checks funds, updates balances and inserts debit/credit ledger rows in one transaction (`canonical_routes.js:394`).
- **Subscription activation:** Activation locks the request, verifies the bcrypt activation secret, applies a manager-level advisory lock before device cap count, and uses a unique `(manager_id,device_id)` binding (`canonical_routes.js:559`; `schema.sql:1483-1488`). This does not cure GN-BE-01's separate bypass route.

### Constraints and design limitations reviewed

- `invoices` has a unique `(session_id,customer_id)` key (`schema.sql:1459-1464`), which prevents duplicated registered-customer invoices but has no representation for guest liabilities.
- `payment_transactions`, GN ledger, LP ledger, and wallet transactions have idempotency uniqueness (`schema.sql:1563-1568`, `1427-1439`, `1467-1479`, `1739-1751`). The generic manual wallet approval does not use them (GN-BE-05).
- Configuration revisions are unique per manager/version (`schema.sql:1363-1368`). Concurrent configuration initialization/update handling was reviewed; no standalone reportable data-integrity defect was asserted without a PostgreSQL runtime proof.
- No backend migration directory/file was present; `schema.sql` is the schema artifact reviewed.

---

## Android comparison

`tests/android_backend_contract_check.py` passed: it found **51 Retrofit contracts and 76 raw Android API paths** with a backend path match. This is an existence/static check, not proof that fields and financial semantics agree. GN-BE-06 is precisely such a semantic mismatch: the path exists and responds 201 while `gnAmount`/`BUY_GN` are not implemented by the backend.

Other useful Android observations:

- Station start sends `prepaymentAmount` and `customerPrepayments` (`SelfHostedManager.kt:1782-1828`), corroborating that GN-BE-03 is a real application contract rather than dead request parsing.
- The standard reservation-payment caller supplies an idempotency key (`SelfHostedManager.kt:1626-1649`), while the generic payment and GN purchase callers shown in GN-BE-06 do not (`SelfHostedManager.kt:759-803`, `1694-1715`). This reinforces the need to make idempotency mandatory for every financial request.
- Subscription purchase uses a UUID-bearing idempotency key and persists the pending activation secret (`GameNetViewModel.kt:4939-4960`).

---

## Testing and evidence level

| Check | Result | Evidence level / caveat |
|---|---|---|
| `npm ci --ignore-scripts` | Passed; 105 packages installed; npm reported 0 vulnerabilities | Local dependency installation only. |
| `npm run check` | Passed | Real Node syntax checks for `server.js`, `canonical_routes.js`, `reservationService.js`, and `financialService.js`. |
| `node backend/test_isolation_static.js` | Passed | Static source assertions; no HTTP/database runtime proof. |
| `node backend/test_financial_mock.js` | Passed | Mocked `query()` unit test only; it does not exercise PostgreSQL, HTTP, or the findings. |
| `python3 tests/android_backend_contract_check.py` | Passed | Static Android/backend path and token-contract inspection; it cannot detect semantic payload mismatches. |
| `python3 tests/deep_release_audit.py` | Passed (43 invariants) | Static text-pattern release checks; not an API/database execution. |
| `npm test` / `test:release` | Not completed | Initial static tests passed, then `test_reservation.js` attempted `DATABASE_URL` at localhost and failed `ECONNREFUSED` for `127.0.0.1:5432` / `::1:5432`. The suite was not continued. |
| Database-backed tests (`test_reservation.js`, `test_financial.js`, cancellation/security/subscription/DB-integrity tests) | Not run to completion | They require a local PostgreSQL database, and several also require an API at `127.0.0.1:3000` or an external hard-coded URL. Docker, PostgreSQL client/server, and a local listening PostgreSQL service were unavailable. No substitute/mock was represented as runtime proof. |
| Source integrity | Passed | `git diff --exit-code` passed after audit; no source modification was made. |

**Recommended verification order:** Fix GN-BE-02 through GN-BE-06 first, provision a disposable local PostgreSQL 16 instance initialized from `schema.sql`, then add targeted HTTP+database integration tests for every regression listed above. Run the full release suite only against that isolated database and local API, never an external production endpoint.

---

## Appendix A — Exhaustive route inventory

The following inventory is mechanically extracted from `server.js` and `canonical_routes.js`. “Public” means no Express auth middleware appears on that route declaration; it does not imply the payload is harmless. Line references are current at the audited commit.

| Source | Method | Path | Route gate |
|---|---|---|---|
| `server.js:196` | `POST` | `/api/auth/manager/login` | Public |
| `server.js:251` | `POST` | `/api/auth/customer/register` | Public |
| `server.js:278` | `POST` | `/api/auth/customer/login` | Public |
| `server.js:302` | `GET` | `/api/v1/time` | Public |
| `server.js:313` | `POST` | `/api/v1/trial/start` | Public |
| `server.js:356` | `POST` | `/api/v1/trial/status` | Public |
| `server.js:374` | `GET` | `/api/v1/super-manager/managers` | Super Manager |
| `server.js:418` | `POST` | `/api/v1/super-manager/add-manager` | Super Manager |
| `server.js:455` | `POST` | `/api/v1/super-manager/managers` | Super Manager |
| `server.js:475` | `POST` | `/api/v1/super-manager/create-manager` | Super Manager |
| `server.js:495` | `POST` | `/api/v1/super-manager/recover` | Super Manager |
| `server.js:509` | `GET` | `/api/v1/super-manager/trial-devices` | Super Manager |
| `server.js:525` | `DELETE` | `/api/v1/super-manager/trial-devices/:id` | Super Manager |
| `server.js:554` | `POST` | `/api/v1/super-manager/managers/:id/subscription/extend` | Super Manager |
| `server.js:592` | `DELETE` | `/api/v1/super-manager/managers/:id/devices/:deviceId` | Super Manager |
| `server.js:602` | `POST` | `/api/v1/super-manager/trial-devices/:id/extend` | Super Manager |
| `server.js:636` | `PUT` | `/api/v1/super-manager/managers/:id` | Super Manager |
| `server.js:674` | `DELETE` | `/api/v1/super-manager/managers/:id` | Super Manager |
| `server.js:688` | `POST` | `/api/v1/super-manager/managers/delete` | Super Manager |
| `server.js:703` | `POST` | `/api/v1/super-manager/managers/purge` | Super Manager |
| `server.js:720` | `GET` | `/api/v1/super-manager/ping` | Public |
| `server.js:721` | `POST` | `/api/v1/super-manager/ping` | Public |
| `server.js:737` | `GET` | `/api/v1/manager/diagnostics` | Manager bearer + active entitlement |
| `server.js:815` | `GET` | `/api/v1/manager/configuration` | Manager bearer + active entitlement |
| `server.js:825` | `PUT` | `/api/v1/manager/configuration` | Manager bearer + active entitlement |
| `server.js:876` | `POST` | `/api/station/start` | Manager bearer + active entitlement |
| `server.js:1025` | `POST` | `/api/station/offline-start` | Manager bearer + active entitlement |
| `server.js:1095` | `POST` | `/api/station/order` | Manager bearer + active entitlement |
| `server.js:1138` | `POST` | `/api/station/event` | Manager bearer + active entitlement |
| `server.js:1226` | `POST` | `/api/station/invoice/pay` | Manager bearer + active entitlement |
| `server.js:1303` | `GET` | `/api/station/active` | Manager bearer + active entitlement |
| `server.js:1317` | `GET` | `/api/customer/live-session` | Customer bearer |
| `server.js:1333` | `POST` | `/api/station/settle` | Manager bearer + active entitlement |
| `canonical_routes.js:103` | `GET` | `/api/v1/manager/customers` | Manager bearer + active entitlement |
| `canonical_routes.js:108` | `POST` | `/api/v1/manager/customers` | Manager bearer + active entitlement |
| `canonical_routes.js:121` | `DELETE` | `/api/v1/manager/customers/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:138` | `POST` | `/api/v1/manager/customers/delete-batch` | Manager bearer + active entitlement |
| `canonical_routes.js:158` | `GET` | `/api/v1/manager/settings/:key` | Manager bearer + active entitlement |
| `canonical_routes.js:159` | `GET` | `/api/v1/manager/reservation-configuration` | Manager bearer + active entitlement |
| `canonical_routes.js:160` | `PUT` | `/api/v1/manager/reservation-configuration` | Manager bearer + active entitlement |
| `canonical_routes.js:161` | `PUT` | `/api/v1/manager/settings/:key` | Manager bearer + active entitlement |
| `canonical_routes.js:164` | `GET` | `/api/v1/manager/console-types` | Manager bearer + active entitlement |
| `canonical_routes.js:165` | `POST` | `/api/v1/manager/console-types` | Manager bearer + active entitlement |
| `canonical_routes.js:166` | `DELETE` | `/api/v1/manager/console-types/:name` | Manager bearer + active entitlement |
| `canonical_routes.js:167` | `GET` | `/api/v1/manager/products` | Manager bearer + active entitlement |
| `canonical_routes.js:168` | `POST` | `/api/v1/manager/products` | Manager bearer + active entitlement |
| `canonical_routes.js:169` | `DELETE` | `/api/v1/manager/products/:name` | Manager bearer + active entitlement |
| `canonical_routes.js:172` | `GET` | `/api/v1/manager/stations` | Manager bearer + active entitlement |
| `canonical_routes.js:173` | `POST` | `/api/v1/manager/stations` | Manager bearer + active entitlement |
| `canonical_routes.js:176` | `GET` | `/api/v1/manager/live-stations` | Manager bearer + active entitlement |
| `canonical_routes.js:177` | `POST` | `/api/v1/manager/live-stations` | Manager bearer + active entitlement |
| `canonical_routes.js:180` | `GET` | `/api/v1/manager/orders/:stationId` | Manager bearer + active entitlement |
| `canonical_routes.js:181` | `POST` | `/api/v1/manager/orders` | Manager bearer + active entitlement |
| `canonical_routes.js:182` | `DELETE` | `/api/v1/manager/orders/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:183` | `DELETE` | `/api/v1/manager/orders/station/:stationId` | Manager bearer + active entitlement |
| `canonical_routes.js:184` | `GET` | `/api/v1/manager/session-history` | Manager bearer + active entitlement |
| `canonical_routes.js:187` | `GET` | `/api/v1/manager/reservations` | Manager bearer + active entitlement |
| `canonical_routes.js:188` | `POST` | `/api/v1/manager/reservations` | Manager bearer + active entitlement |
| `canonical_routes.js:222` | `DELETE` | `/api/v1/manager/reservations/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:223` | `POST` | `/api/v1/manager/reservations/:id/cancel` | Manager bearer + active entitlement |
| `canonical_routes.js:234` | `PUT` | `/api/v1/manager/reservations/:id/status` | Manager bearer + active entitlement |
| `canonical_routes.js:306` | `POST` | `/api/v1/manager/settings` | Manager bearer + active entitlement |
| `canonical_routes.js:307` | `POST` | `/api/v1/manager/session-history` | Manager bearer + active entitlement |
| `canonical_routes.js:308` | `DELETE` | `/api/v1/manager/session-history` | Manager bearer + active entitlement |
| `canonical_routes.js:309` | `POST` | `/api/v1/customer/reservations` | Customer bearer |
| `canonical_routes.js:323` | `GET` | `/api/v1/customer/stations` | Customer bearer |
| `canonical_routes.js:336` | `GET` | `/api/v1/customer/reservations/rules` | Customer bearer |
| `canonical_routes.js:384` | `POST` | `/api/v1/customer/reservations/pricing-preview` | Customer bearer |
| `canonical_routes.js:385` | `POST` | `/api/v1/customer/reservations/atomic` | Customer bearer |
| `canonical_routes.js:386` | `POST` | `/api/v1/customer/reservations/:id/cancel` | Customer bearer |
| `canonical_routes.js:390` | `GET` | `/api/v1/customer/profile` | Customer bearer |
| `canonical_routes.js:391` | `GET` | `/api/v1/customer/reservations` | Customer bearer |
| `canonical_routes.js:392` | `GET` | `/api/v1/customer/club/ledger` | Customer bearer |
| `canonical_routes.js:393` | `POST` | `/api/v1/manager/club/ledger` | Manager bearer + active entitlement |
| `canonical_routes.js:394` | `POST` | `/api/v1/customer/club/transfer` | Customer bearer |
| `canonical_routes.js:397` | `POST` | `/api/v1/customer/manual-payment-requests` | Customer bearer |
| `canonical_routes.js:398` | `GET` | `/api/v1/customer/manual-payment-requests` | Customer bearer |
| `canonical_routes.js:399` | `GET` | `/api/v1/manager/manual-payment-requests` | Manager bearer + active entitlement |
| `canonical_routes.js:400` | `POST` | `/api/v1/manager/manual-payment-requests/:id/approve` | Manager bearer + active entitlement |
| `canonical_routes.js:401` | `GET` | `/api/v1/manager/reservation-payments/pending` | Manager bearer + active entitlement |
| `canonical_routes.js:416` | `POST` | `/api/v1/manager/reservation-payments/:id/reject` | Manager bearer + active entitlement |
| `canonical_routes.js:427` | `POST` | `/api/v1/manager/reservation-payments/:id/approve` | Manager bearer + active entitlement |
| `canonical_routes.js:462` | `POST` | `/api/v1/manager/manual-payment-requests/:id/reject` | Manager bearer + active entitlement |
| `canonical_routes.js:464` | `POST` | `/api/v1/manager/club/point-logs` | Manager bearer + active entitlement |
| `canonical_routes.js:465` | `GET` | `/api/v1/manager/club/point-logs/:customerId` | Manager bearer + active entitlement |
| `canonical_routes.js:470` | `GET` | `/api/v1/auth/check` | Manager bearer + active entitlement |
| `canonical_routes.js:471` | `POST` | `/api/v1/manager/device` | Manager bearer + active entitlement |
| `canonical_routes.js:474` | `POST` | `/api/v1/manager/announcements` | Manager bearer + active entitlement |
| `canonical_routes.js:475` | `POST` | `/api/v1/manager/audit-logs` | Manager bearer + active entitlement |
| `canonical_routes.js:476` | `GET` | `/api/v1/manager/club/ledger/:customerId` | Manager bearer + active entitlement |
| `canonical_routes.js:477` | `DELETE` | `/api/v1/manager/reservations/by-phone/:phone` | Manager bearer + active entitlement |
| `canonical_routes.js:478` | `POST` | `/api/v1/manager/stations/purge-extra` | Manager bearer + active entitlement |
| `canonical_routes.js:506` | `POST` | `/api/v1/manager/customer-transactions` | Manager bearer + active entitlement |
| `canonical_routes.js:507` | `GET` | `/api/v1/manager/customer-transactions` | Manager bearer + active entitlement |
| `canonical_routes.js:508` | `GET` | `/api/v1/customer/transactions` | Customer bearer |
| `canonical_routes.js:509` | `GET` | `/api/v1/manager/payment-methods` | Manager bearer + active entitlement |
| `canonical_routes.js:510` | `POST` | `/api/v1/manager/payment-methods` | Manager bearer + active entitlement |
| `canonical_routes.js:511` | `PATCH` | `/api/v1/manager/payment-methods/:id` | Manager bearer + active entitlement |
| `canonical_routes.js:512` | `GET` | `/api/v1/customer/payment-methods` | Customer bearer |
| `canonical_routes.js:524` | `GET` | `/api/v1/subscriptions/check` | Public |
| `canonical_routes.js:525` | `POST` | `/api/v1/subscriptions/status` | Public |
| `canonical_routes.js:526` | `GET` | `/api/v1/plans` | Public |
| `canonical_routes.js:530` | `GET` | `/api/v1/super-manager/subscription-plans` | Super Manager |
| `canonical_routes.js:534` | `PUT` | `/api/v1/super-manager/subscription-plans` | Super Manager |
| `canonical_routes.js:559` | `POST` | `/api/v1/subscriptions/activate` | Public |
| `canonical_routes.js:560` | `POST` | `/api/v1/subscriptions/buy` | Public |
| `canonical_routes.js:587` | `POST` | `/api/v1/subscriptions/set-password` | Public |
| `canonical_routes.js:588` | `POST` | `/api/v1/coupons/validate` | Public |

---

**End of audit report.**


---

## خروجی ۴ — ممیزی Security، Authentication و Authorization

> فایل منبع این بخش: `GameNexa-Audit-security.md`

# GameNexa White-Box Security Audit

## Audit identity and scope

| Item | Value |
|---|---|
| Target | `/home/ubuntu/GameNexa-release-2026` |
| Audited revision | `05348f2df8b1f6c30ee233071be13a899d969e25` on `main` |
| Repository state at review | Clean working tree; HEAD matched the requested commit |
| Method | Safe local white-box source review, deterministic static checks, syntax/contract tests, dependency audit. **No external endpoint or runtime was contacted.** |
| In-scope areas | Login/password/bcrypt/JWT/session/logout/expiry/roles; Manager A/B and Super Manager isolation; IDOR/BOLA; mass assignment; injection; CORS/CSRF; logging/PII; credentials/debug paths; trial/subscription; tests. |
| Findings | **4 confirmed findings**: 3 medium, 1 low |

## Executive summary

The server has several meaningful protections: JWT algorithm pinning to HS256, database-backed account-existence checks in auth middleware, tenant predicates on reviewed manager/customer operations, parameterized PostgreSQL calls, bcrypt usage, entitlement gates on manager mutations, Super Manager role/entitlement checks, origin allowlisting, and TLS/cleartext restrictions. I did **not** confirm a Manager A-to-B data IDOR, a Super-Manager authorization bypass, SQL injection, a committed production secret, a CSRF issue for bearer-authenticated APIs, or a subscription activation bypass in the reviewed revision.

However, four confirmed issues merit remediation:

1. The one-per-device trial rule can be bypassed because trial identities are caller-controlled and Manager blocking only executes when the caller volunteers a valid bearer token.
2. Logout is local-only: 24-hour bearer JWTs have no server-side revocation/session-version mechanism, including after password updates.
3. Customer password policy accepts four-character passwords, and the Super Manager manager-update path accepts any non-empty password.
4. Customer login exposes distinguishable account-state messages, enabling targeted account enumeration within a known manager tenant.

The most important corrective work is to replace the trial identity model with a server-verifiable enrollment/attestation design, and to add revocable server-side session state (or an account `auth_epoch`) to access-token verification.

## Severity method

Severity reflects observed impact in this codebase, not a generic scanner rating:

- **Medium**: security/business-control bypass or replay window affecting authenticated or licensed functionality, but no demonstrated cross-tenant read/write or privilege escalation.
- **Low**: limited information disclosure or a weakness that requires password guessing/other preconditions.
- **Confidence**: High = direct control-flow/data-flow proof in current code; Medium = direct behavior with a bounded deployment or business-policy dependency.

---

# Confirmed findings

## SEC-01 — Trial eligibility can be reset with caller-chosen identities and an omitted bearer token

| Property | Assessment |
|---|---|
| Severity | **Medium** |
| Confidence | **High** for bypass of the server's intended per-device / Manager-trial policy; **Medium** for broader commercial impact because protected manager API mutations still require `requireActiveEntitlement`. |
| CWE | CWE-602 (client-side enforcement of server-side security), CWE-639 (authorization based on user-controlled key) |
| Affected flow | Public `POST /api/v1/trial/start` → Android trial-active local state |

### Exact evidence

**1) The endpoint is public and accepts the device identity entirely from the request body.**

`backend/server.js:310-313, 323-325`

```js
const normalizeTrialIdentity = (value) => String(value || '').trim().slice(0, 255);

app.post('/api/v1/trial/start', rateLimit({ windowMs: 60_000, max: 5 }), async (req, res) => {
    ...
    const deviceId = normalizeTrialIdentity(req.body?.deviceId || req.body?.device_id);
    const deviceFingerprint = normalizeTrialIdentity(req.body?.deviceFingerprint || req.body?.device_fingerprint);
    const deviceName = normalizeTrialIdentity(req.body?.deviceName || req.body?.device_name || 'Android');
```

**2) The attempted Manager prohibition is conditional on the caller sending a valid `Authorization` header. Omitting that header skips the check.**

`backend/server.js:313-322`

```js
const authHeader = String(req.headers.authorization || '');
if (/^Bearer\s+\S+$/i.test(authHeader)) {
    try {
        const decoded = jwt.verify(authHeader.slice(7).trim(), JWT_SECRET, { algorithms: ['HS256'] });
        if (['MANAGER', 'SUPER_MANAGER'].includes(decoded?.role)) {
            return res.status(403).json({ ... code:'MANAGER_TRIAL_FORBIDDEN', ... });
        }
    } catch (_) {}
}
```

**3) The only server-side reuse check is equality on those two caller-supplied strings, followed by issuance for a previously unseen pair.**

`backend/server.js:331-348`

```js
const blocked = await client.query(
  'SELECT 1 FROM trial_device_blocks WHERE (device_id = $1 AND device_fingerprint = $2) OR device_id = $1 OR device_fingerprint = $2 LIMIT 1',
  [deviceId, deviceFingerprint]
);
...
const existing = await client.query(
  'SELECT device_id, device_fingerprint, started_at, expires_at, status FROM trial_devices WHERE device_id = $1 OR device_fingerprint = $2 ...',
  [deviceId, deviceFingerprint]
);
...
await client.query(
  'INSERT INTO trial_devices (device_id, device_fingerprint, device_name, started_at, expires_at, status) VALUES ($1, $2, $3, $4, $5, \'ACTIVE\')',
  [deviceId, deviceFingerprint, deviceName, startedAt, expiresAt]
);
return res.status(201).json({ success: true, trialActive: true, ... });
```

**4) The Android client treats a successful server trial response as active trial state.**

`app/src/main/java/com/example/ui/GameNetViewModel.kt:4773-4776` stores the successful trial response and reports success; `GameNetViewModel.kt:4653-4675` sets `_isSubscribed` true for a non-expired cached `TRIAL` plan and records role `TRIAL_USER`.

### Safe local reproduction / expected vs. actual

The following deterministic static test was run locally and passed; it verifies that the public route reads both identities from the body, inserts a trial record, performs the Manager check only inside the optional bearer-header branch, and has no server logout route:

```text
PASS static trial-identity/auth-header, no-server-logout, and 4-character-password-policy assertions
```

A safe integration reproduction (only in a disposable **local** test database/runtime) is:

```bash
# No Authorization header. Use a fresh arbitrary pair each time.
curl -i -X POST http://127.0.0.1:3000/api/v1/trial/start \
  -H 'Content-Type: application/json' \
  --data '{"deviceId":"audit-new-id-1","deviceFingerprint":"audit-new-fp-1","deviceName":"audit"}'
```

- **Expected:** a previously licensed/Manager-associated physical device cannot obtain another trial, and a physical device is limited to one trial.
- **Actual by current control flow:** any fresh pair of strings reaches the insert and `201` response. An authenticated Manager can omit `Authorization`; the `MANAGER_TRIAL_FORBIDDEN` branch is then not evaluated.

### Impact

An actor can repeatedly obtain server-issued 24-hour trial state by generating new `deviceId` and `deviceFingerprint` values. The five-per-minute IP limiter throttles only request rate; it does not make the identity trustworthy or bind it to a device. This defeats the intended one-trial-per-device control and the explicit Manager-trial check.

**Bounded impact:** manager data-changing endpoints reviewed here still carry `requireManagerAuth` plus `requireActiveEntitlement`; this finding does not itself prove access to those protected server routes without an entitlement.

### Root cause

The authoritative entitlement decision is based on a mutable client assertion (`deviceId`/`deviceFingerprint`). The optional bearer token is used as if it could establish that a request is not from a Manager; callers can simply not send it.

### Recommended fix

1. Do **not** treat client-provided device IDs/fingerprints as a durable entitlement identity.
2. Bind a trial to a server-issued installation credential stored in Android Keystore, and require a hardware/app-attestation signal appropriate to the deployment (for example Play Integrity) before first issuance. Verify it server-side.
3. If trial eligibility is account-based, require authenticated account creation/login and record one trial per verified account plus an attested installation.
4. Resolve the Manager policy from an authenticated credential or a server-side installation-to-account association; do not use absence of `Authorization` as evidence of non-Manager status.
5. Keep the database uniqueness/block controls, but make them enforce an identity the client cannot freely mint.

### Regression and verification

Add runtime tests against a disposable local PostgreSQL instance:

- A Manager token cannot start a trial; omitting its token from the same attested installation also cannot start one.
- Replaying a changed `deviceId`/`deviceFingerprint` with the same attested installation is rejected.
- A different installation cannot reuse an installation credential.
- A blocked/deleted trial remains blocked (existing coverage already exercises this last case).
- Verify protected manager routes remain entitlement-gated after trial changes.

---

## SEC-02 — Logout and password changes do not revoke issued 24-hour bearer JWTs

| Property | Assessment |
|---|---|
| Severity | **Medium** |
| Confidence | **High** |
| CWE | CWE-613 (insufficient session expiration), CWE-384 (session invalidation weakness) |
| Affected flow | Manager/customer login → bearer token use → local Android logout or password update |

### Exact evidence

**1) Manager and customer access tokens are signed for 24 hours.**

`backend/server.js:242-243`

```js
const token = jwt.sign(
  { id: manager.id, managerId: manager.id, role: manager.role || 'MANAGER' },
  JWT_SECRET,
  { expiresIn: '24h' }
);
```

`backend/server.js:270-271` and `backend/server.js:291-292` issue a 24-hour customer token after registration and login respectively.

**2) The authorization middleware verifies signature/expiry and current account existence, but has no session ID, deny-list, token version, logout state, password-change timestamp, or `auth_epoch` check.**

`backend/server.js:87-108` (`requireManagerAuth`):

```js
const decoded = jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] });
...
const account = await pool.query(
  "SELECT id, role FROM managers WHERE id = $1 AND role = ANY($2::text[]) LIMIT 1",
  [decoded.id, ['MANAGER', 'SUPER_MANAGER']]
);
if (!account.rows.length) return res.status(401).json({ error: 'Manager account no longer exists' });
req.user = decoded;
next();
```

`backend/server.js:112-129` uses the same model for customers: signature/expiry plus existence/tenant lookup only.

**3) Android logout only clears local state; it makes no server request.**

`app/src/main/java/com/example/ui/GameNetViewModel.kt:5585-5605`

```kotlin
fun logout(onComplete: (() -> Unit)? = null) {
    viewModelScope.launch(Dispatchers.IO) {
        encryptSetting("enc_auth_token", "")
        ...
        NetworkClient.managerAuthToken = null
        NetworkClient.customerAuthToken = null
        encryptSetting("enc_customer_auth_token", "")
        _authState.value = AuthState.Unauthenticated
```

A repository-wide route check found no backend `logout` route. Password-setting flows update `password_hash` (for example `backend/canonical_routes.js:587`) but do not update any token-version/revocation field; no such field/check is present in middleware.

### Safe local reproduction / expected vs. actual

The deterministic static test above passed and asserted both the absence of a backend logout route and the client-only token clearing.

A local-only integration verification sequence after adding disposable fixtures is:

```bash
# 1. Login and retain TOKEN outside the client.
# 2. Call Android logout (or clear its token storage).
# 3. Replay the retained token to a protected route.
curl -i http://127.0.0.1:3000/api/v1/manager/customers \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-Manager-ID: $MANAGER_ID"
```

- **Expected:** replay after logout or password reset returns `401`/`403`.
- **Actual by current middleware:** the token is accepted until its JWT `exp` (24 hours), provided its account still exists and, for manager routes, the matching `X-Manager-ID` is supplied.

### Impact

A copied/stolen bearer token remains usable for up to 24 hours after the user believes they logged out. Password updates do not force reauthentication, so a previously stolen token remains effective through a password change. For managers this window covers manager-scoped reads and mutations that also pass the entitlement gate; for customers it covers their self-service resources.

### Root cause

The design uses self-contained access JWTs as the entire session, while logout is only local credential deletion. There is no server-side session record or account-level invalidation state.

### Recommended fix

1. Introduce a `sessions`/refresh-token table with a random, hashed refresh credential, expiry, account ID, device metadata, revoked timestamp, and optionally a JWT `jti`.
2. Use short-lived access JWTs (for example 5–15 minutes) and a rotating refresh token. Revoke the refresh session at logout.
3. Add an `auth_epoch`/`token_version` (or `password_changed_at`) to `managers` and `customers`; place it in claims and compare it in middleware, or query it on verification. Increment it on logout-all, password change/reset, account archive/deletion, and sensitive role change.
4. Provide `POST /api/auth/logout` and `POST /api/auth/logout-all`, both idempotent and rate-limited. Do not log submitted tokens.
5. Preserve the existing algorithm pinning and account-existence checks.

### Regression and verification

- Token works before logout and fails immediately after logout.
- Two sessions for one account: logout-current revokes only one; logout-all revokes both.
- Password update invalidates all older tokens for manager and customer accounts.
- Expired access token is rejected; valid refresh rotates once and a replayed old refresh token is rejected.
- Account archival/deletion continues to reject prior tokens.

---

## SEC-03 — Password policy permits trivial customer passwords and a one-character Manager password update

| Property | Assessment |
|---|---|
| Severity | **Medium** |
| Confidence | **High** |
| CWE | CWE-521 (weak password requirements) |
| Affected flow | Public customer registration; manager-created/updated customers; Super Manager update of an existing Manager |

### Exact evidence

**1) Public customer registration accepts a four-character password and immediately issues a customer JWT.**

`backend/server.js:251-271`

```js
const password = typeof req.body?.password === 'string' ? req.body.password : '';
if (!phoneNumber || !managerId || !fullName || password.length < 4) {
    return res.status(400).json({ error: '... password of at least 4 characters are required' });
}
...
const hash = await bcrypt.hash(password, 12);
...
const token = jwt.sign({ id: customer.id, managerId, role: 'CUSTOMER' }, JWT_SECRET, { expiresIn: '24h' });
```

**2) The authenticated Manager customer-create/update path also hashes any password of length at least four.**

`backend/canonical_routes.js:108-116`

```js
if(typeof b.password==='string' && b.password.length>=4)
  await pool.query(
    'UPDATE customers SET password_hash=$1 WHERE id=$2 AND manager_id=$3',
    [await bcrypt.hash(b.password,12),q.rows[0].id,mid]
  );
```

**3) Super Manager update accepts any non-empty Manager password (including one character) and uses a lower bcrypt cost than the other paths.**

`backend/server.js:636-665`

```js
const password = req.body?.password ?? null;
...
if (password !== null && String(password).trim()) {
    const bcrypt = require('bcrypt');
    fields.push(`password_hash = $${i++}`);
    values.push(await bcrypt.hash(String(password), 10));
}
```

bcrypt is correctly used rather than plaintext storage, but bcrypt cannot compensate for accepting a trivially guessable secret.

### Safe local reproduction / expected vs. actual

The passed deterministic static test asserted the literal public registration minimum (`password.length < 4`) and Manager customer-path condition (`b.password.length>=4`).

For a disposable local runtime, the public path can be verified without attacking any account:

```bash
curl -i -X POST http://127.0.0.1:3000/api/auth/customer/register \
  -H 'Content-Type: application/json' \
  --data '{"phone_number":"audit-unique-phone","manager_id":"LOCAL_TEST_MANAGER","full_name":"Audit Fixture","password":"abcd"}'
```

- **Expected:** a centrally defined, strong policy rejects short/trivial passwords on every account creation and change path.
- **Actual by code:** `abcd` passes the public registration check and is bcrypt-hashed; an existing Manager can be set to any nonblank password through the Super Manager update route.

### Impact

Four-character customer passwords and one-character Manager passwords are susceptible to online guessing if credentials or identifiers become known. Per-IP login limiting (`10/minute` for customer login at `backend/server.js:278`) reduces but does not eliminate distributed guessing risk. The Manager update condition can downgrade an account that protects tenant data and financial operations.

### Root cause

Password validation is duplicated and inconsistent. It is based only on minimal string length, and the privileged account-update branch has no length requirement at all.

### Recommended fix

1. Implement one server-side `validatePassword()` used by registration, customer update, subscription setup, bootstrap, Manager create, and Manager update.
2. Require a length-based policy appropriate for the product (for example 12+ characters; permit long passphrases) and reject known-compromised passwords using a privacy-preserving breached-password check or local deny list.
3. Do not require arbitrary composition rules; normalize carefully and preserve Unicode/passphrases rather than silently trimming the credential.
4. Raise bcrypt cost consistently after performance calibration (the application already uses cost 12 in several interactive paths). Rehash at the stronger cost after successful login.
5. Pair this with the session invalidation fix in SEC-02 so password changes revoke old tokens.

### Regression and verification

- Test `abcd`, `1234`, and an empty/whitespace-only password against every creation/update/reset route; each must fail.
- Test a compliant passphrase for each route; each must succeed and store only a bcrypt hash.
- Assert no API response or log includes the plaintext password or its bcrypt hash.
- Assert a Manager update cannot reduce password strength below the shared policy.

---

## SEC-04 — Customer login reveals account state through distinct authentication errors

| Property | Assessment |
|---|---|
| Severity | **Low** |
| Confidence | **High** |
| CWE | CWE-204 (observable response discrepancy) |
| Affected flow | `POST /api/auth/customer/login` |

### Exact evidence

`backend/server.js:278-295`:

```js
const result = await pool.query(
  "SELECT * FROM customers WHERE phone_number = $1 AND manager_id = $2 AND COALESCE(description,'') NOT LIKE '[GAMENEX_ARCHIVED:%'",
  [phone_number, manager_id]
);
const customer = result.rows[0];
if (!customer) return res.status(401).json({ error: 'Customer not found' });
if (!customer.password_hash) return res.status(401).json({ error: 'Customer password is not configured' });
const valid = await bcrypt.compare(password, customer.password_hash);
if (!valid) return res.status(401).json({ error: 'Invalid credentials' });
```

The endpoint has a useful per-IP rate limit (`rateLimit({ windowMs: 60_000, max: 10 })` at line 278), but the three distinct 401 bodies remain observable.

### Safe local reproduction / expected vs. actual

With local disposable fixtures for a known `manager_id`, submit the same wrong password for:

1. a nonexistent phone number,
2. an existing customer with `password_hash IS NULL`, and
3. an existing customer with a password hash.

- **Expected:** each response has the same status/body, such as `401 {"error":"Invalid credentials"}`.
- **Actual:** the server returns `Customer not found`, `Customer password is not configured`, or `Invalid credentials`, revealing both account existence and provisioning status.

### Impact

An attacker who knows or guesses a Manager ID can enumerate which phone numbers correspond to customer accounts and identify accounts whose password has not yet been configured. This improves targeted phishing and password-guessing campaigns. No cross-tenant record read is demonstrated: the query is correctly scoped by `manager_id`.

### Root cause

Authentication failure handling exposes internal account state to the unauthenticated caller.

### Recommended fix

1. Return one generic `401` body for all credential failures, including nonexistent, archived, unprovisioned, and incorrect-password accounts.
2. Consider a dummy bcrypt comparison when no hash exists to reduce easily measurable timing differences.
3. Keep the existing rate limit; add account/phone-plus-tenant throttling and monitoring for distributed attempts.
4. Show password-provisioning instructions only after a separately authenticated or verified recovery/onboarding flow.

### Regression and verification

- For nonexistent, unprovisioned, archived, wrong-password, and valid accounts, assert all failures have exactly identical status, body, headers (aside from normal request IDs), and comparable timing envelope.
- Assert valid login still returns only the intended token/customer fields.

---

# Detailed review notes by requested area

## Authentication, bcrypt, JWT, roles, expiry, and logout

### Confirmed protections

- **JWT secret is mandatory:** `backend/server.js:62-64` loads `process.env.JWT_SECRET` and fails startup when absent. The committed `.env.example` contains placeholders; no tracked real credential was found in a local committed-file pattern scan.
- **Algorithm pinned:** `jwt.verify(..., { algorithms: ['HS256'] })` is used in Super Manager, Manager, and Customer middleware (`backend/server.js:68`, `94`, `119`), preventing algorithm-confusion acceptance.
- **Role and identity binding:** `requireManagerAuth` requires `MANAGER`/`SUPER_MANAGER`, nonempty `id` and `managerId`, equality of those two claims, and matching `X-Manager-ID` (`server.js:94-103`). `requireCustomerAuth` requires `CUSTOMER`, `id`, and `managerId` and verifies the customer row belongs to that tenant (`server.js:119-124`).
- **Database account existence checks:** deleted/archived customer rows and deleted manager rows do invalidate tokens through the middleware queries, even though normal logout and password change do not (SEC-02).
- **bcrypt:** customer registration/login uses `bcrypt.hash(..., 12)` and `bcrypt.compare` (`server.js:264`, `288`); subscription activation/password setup also validates a bcrypt-hashed random activation secret (`canonical_routes.js:559`, `587`).
- **Super Manager controls:** `requireSuperManagerAuth` checks the signed role, matching `id/managerId`, database role, and an active Super Manager entitlement (`server.js:45-85`). Super Manager routes reviewed carry this middleware.

### Required remediation

Apply SEC-02 and SEC-03. Ensure session revocation occurs on Manager deletion/archive, customer archive, credential reset, role changes, and device unbinding where product policy requires it.

## Manager A/B isolation, IDOR/BOLA, and Super Manager

No confirmed Manager A-to-B IDOR/BOLA was found in the reviewed current routes. Representative evidence:

- Manager customer listing derives tenant from authenticated claims and filters `manager_id=$1` (`backend/canonical_routes.js:102-105`).
- Customer archive validates `id=$1 AND manager_id=$2` before updating (`canonical_routes.js:121-135`).
- Customer reservation cancellation passes the authenticated `managerId` and `customerId` to `cancelReservation` (`canonical_routes.js:386`).
- Financial cancellation locks reservations with `WHERE id = $1 AND manager_id = $2` and adds `customer_id = $3` for customer cancellation (`backend/financialService.js:155-325`).
- Manager payment-method patch scopes its update by method ID **and** `manager_id` (`canonical_routes.js:511`).
- Super Manager Manager updates/delete routes explicitly reject target roles of `SUPER_MANAGER` (`backend/server.js:636-681`).

`backend/test_isolation_static.js` passed and asserts tenant claim use, Manager-scoped customer/reservation lookups, customer-cancellation owner passing, and required customer login password verification. The more comprehensive `test_security_isolated.js` has meaningful A/B fixtures and JWT-negative cases, but it requires a local PostgreSQL instance and an API on `127.0.0.1:3000`; neither was present in this safe sandbox review.

### Public registration note (not classified as a defect)

`POST /api/auth/customer/register` is unauthenticated and accepts a caller-supplied `manager_id` (`server.js:251-271`). It verifies only that a Manager exists before creating a customer and returning a customer token. This is a deliberate-looking public onboarding route, but the repository supplies no policy saying enrollment must be invite-only or manager-approved. It is therefore **not counted as a vulnerability** in this report. If enrollment should be restricted, bind it to an invitation/verified club identifier and add a test that arbitrary Manager IDs cannot be selected.

## Mass assignment and injection

- Reviewed PostgreSQL data access uses `$n` parameter binding for user-controlled values. The static scan found no query template with interpolated request values.
- The one dynamic SQL construction is `backend/server.js:650-665`, where `fields.join(', ')` is built only from a fixed local allowlist (`display_name`, `gamenet_name`, `plan_type`, `subscription_status`, `payment_status`, `password_hash`) and every value remains parameterized. This is **not** SQL injection as implemented.
- Manager/customer methods generally map explicit request fields rather than spreading entire bodies into persistence. The Super Manager route uses an allowlist. No confirmed mass-assignment privilege escalation was found.
- Continue to validate numeric ranges/types in configuration and financial routes; do not downgrade this code review into a claim of formal completeness.

## CORS and CSRF relevance

- CORS is an origin allowlist, not wildcard CORS: `backend/server.js:12-20` accepts requests with no browser Origin (native clients) or origins present in `CORS_ORIGINS`, and rejects other origins.
- The API uses explicit `Authorization: Bearer` headers rather than ambient cookies. Consequently, conventional browser CSRF is **not a primary applicable threat** to reviewed authenticated routes; browsers do not automatically attach the bearer token.
- Keep `CORS_ORIGINS` explicitly configured in production. Do not switch to `origin: true`/`*` when credentials or browser sessions are introduced.

## Sensitive data, PII, logging, and mobile storage

- Server logging reviewed does not log request bodies, bearer tokens, plaintext passwords, or bcrypt hashes. It logs operational error objects/messages in some catch blocks; retain the current practice and add a centralized redactor before any future structured request logging.
- `app/src/main/java/com/example/data/network/NetworkLogger.kt:126-186` records path-only URLs and explicitly says it does not retain raw response bodies; it limits failed-response extraction to safe `code`/`error` fields. The in-memory log has a 150-entry cap (`NetworkLogger.kt:40-64`). No persisted token/PII log was confirmed.
- `AndroidManifest.xml:14-20` sets `android:allowBackup="false"` and `android:usesCleartextTraffic="false"`; `network_security_config.xml:2-7` trusts system CAs and disallows cleartext. Debug HTTP logging is disabled in release (`SelfHostedManager.kt:150-155`).
- Android encrypted-setting helpers should remain Keystore-backed and should never fall back to plaintext. This review found no committed production secrets or signing keys; `.env`, `.jks`, `.keystore`, `.p12`, `.pem`, and `local.properties` are ignored and none were tracked.

## Trial/subscription

- **Trial:** SEC-01 is confirmed.
- **Subscription activation:** `canonical_routes.js:559` requires a `PENDING_<id>` code, the buyer-bound device ID, a bcrypt-validated random activation secret, a confirmed request, and active entitlement; it serializes device binding under a lock and enforces max device count. No subscription activation bypass was confirmed.
- **Set password:** `canonical_routes.js:587` requires a confirmed request, bcrypt secret comparison, phone-or-device match, and consumes (`NULL`s) the activation-secret hash on success. `backend/test_activation_runtime.js` includes checks for invalid secret rejection and secret consumption/replay rejection, but needs the absent local DB/API to execute.
- **Manual approval:** `backend/test_subscription_manual.js` checks the removed legacy automatic confirmation route and that a purchase stays pending; static source confirms that automatic confirmation is not present.

## Hardcoded credentials, debug/test bypasses, and dependencies

- `backend/bootstrap.js:9-40` requires `SUPER_ADMIN_PASSWORD` (minimum 16 chars on first bootstrap) and bcrypt-hashes it. The fallback username `superadmin` is predictable but is not a credential; retain a strong required password and consider requiring an explicit bootstrap username in production.
- No committed secret matching the local audit patterns was found. The only match was the safe `const JWT_SECRET = process.env.JWT_SECRET` reference.
- No `eval`, `child_process`, `exec`, `spawn`, test authentication bypass, or hardcoded authentication secret was found in the tracked application source searched.
- `npm audit --omit=dev --json` reported **0 vulnerabilities** across 105 production dependencies at audit time. This is point-in-time dependency metadata, not a replacement for continuous dependency monitoring.

# Tests and commands run

All were local-only and did not start or attack an external runtime.

| Command / activity | Result |
|---|---|
| `git -C /home/ubuntu/GameNexa-release-2026 rev-parse HEAD` and status check | PASS — exact requested commit; clean tree |
| `npm run check` | PASS — `server.js`, `canonical_routes.js`, `reservationService.js`, `financialService.js` parse |
| `node backend/test_isolation_static.js` | PASS — tenant-isolation static checks |
| `python3 tests/android_backend_contract_check.py` | PASS — server-authoritative session restoration and 51 Retrofit / 76 raw route contracts |
| `python3 tests/security_tests.py` | Exit 0, but file is only a placeholder comment; **no executable security assertion** |
| Deterministic local Node static assertions for SEC-01/02/03 | PASS — verified trial request identity/auth-header structure, no server logout route, local token clearing, and 4-character policy |
| Parameterization/dynamic-SQL static scan | No user-controlled dynamic SQL confirmed; one fixed field allowlist construction reviewed |
| `npm audit --omit=dev --json` | PASS — 0 reported vulnerabilities |
| `git fsck --no-reflogs --no-progress` | PASS |
| Committed secret-pattern scan | No real secret found; environment variable reference only |
| Local runtime availability check | No `DATABASE_URL`, Docker, PostgreSQL binaries, or listeners on 3000/5432; DB-backed integration suite intentionally not run |

## Test coverage gaps to close

1. Make `tests/security_tests.py` real or remove it; it currently only contains a placeholder comment.
2. Run `npm run test:release` in CI with an ephemeral PostgreSQL database and a locally started API. The scripts include valuable runtime coverage but cannot run without those fixtures.
3. Add regression tests for every finding above, especially trial identifier forgery/optional-bearer omission and post-logout/password-change token replay.
4. Add a route-level authorization matrix test that creates Manager A, Manager B, Customer A, Customer B, and Super Manager fixtures and tests every ID-bearing endpoint with foreign IDs.
5. Add secret-scanning and dependency-audit jobs to CI; fail release builds on tracked credentials or high/critical advisories after triage.

# Verification plan / remediation order

1. **First:** redesign trial identity and add server-verifiable proof (SEC-01).
2. **Second:** add revocable sessions/token epoch and revoke on logout/password change (SEC-02).
3. **Third:** centralize and enforce the password policy across all flows (SEC-03).
4. **Fourth:** make login failures generic and add account-plus-tenant rate limiting (SEC-04).
5. Run the full database-backed release suite, then specifically replay the regression tests listed under each finding.

> This report distinguishes observed defects from review observations. The absence of a finding in a category means no issue was confirmed from the audited revision and safe local analysis; it is not a claim of absence under unreviewed deployment configuration, database privileges, infrastructure, or future code changes.


---

## خروجی ۵ — ممیزی CI/CD، Runtime، Docker و Test Quality

> فایل منبع این بخش: `GameNexa-Audit-ci.md`

# GameNexa CI/CD, Build/Release, Runtime, and Test Audit

**Audited revision:** `05348f2df8b1f6c30ee233071be13a899d969e25` on `main`  
**Repository:** `mkhas1374/GameNexa-release-2026`  
**Audit date:** 2026-10-02  
**Scope:** all three tracked GitHub Actions workflows; Gradle and Android configuration; npm/package lock and backend test scripts; Docker/Compose/nginx; root and backend environment templates; README/status/package manifests; schema/bootstrap; and every tracked test source/assertion.

> **Release verdict: not release-ready.** The exact commit has a successful **debug** GitHub Actions run, but its **release-validation run failed before signed release artifacts were built**. Backend integration tests, Docker build/startup, migrations, and deployment are not part of CI. The locally executable passing checks are mostly syntax, source-text contracts, and a mocked arithmetic test—not evidence of a running backend or end-to-end release.

## 1. What was verified

### Revision and GitHub Actions execution

* Local `HEAD` and `origin/main` both resolve to the audited SHA. The worktree was clean when inspected.
* GitHub workflow definitions are active:
  * **GameNexa Android Debug Build** (workflow ID `370703256`)
  * **GameNexa Android Release Validation** (workflow ID `371735558`)
  * **Delete Old Workflow Runs** (workflow ID `372641750`)
* GitHub Actions records for this exact SHA:

| Workflow | Run | Event | Result | What this proves / does not prove |
|---|---:|---|---|---|
| Android Debug Build | [`36964812531`](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/36964812531) | push | **success** | The two Python source-contract scripts, `testDebugUnitTest`, and `assembleDebug` completed. Log shows `BUILD SUCCESSFUL` and uploaded `GameNexa-debug-apk` (30,018,909 bytes). It does **not** run `npm test`, Docker, Postgres, Android instrumentation, a real device, or deployment. |
| Android Release Validation | [`36964812496`](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/36964812496) | push | **failure** | The job failed at **“Require stable release signing credentials.”** The subsequent restore-keystore, signed `assembleRelease bundleRelease`, and release artifact upload steps were all skipped. The retrieved job metadata does not reveal which masked credential was absent; it establishes that the required-signing gate did not pass. |

The `RELEASE_AUDIT_STATUS.md` statement that `npm test` passed and Docker services were healthy is dated 2026-09-29 (lines 3, 6–16). It is not a current CI result for this commit and cannot override the failed release-validation run.

### Commands actually run locally

| Command | Result | Interpretation |
|---|---|---|
| `cd backend && npm run check` | **PASS** | Parses only `server.js`, `canonical_routes.js`, `reservationService.js`, and `financialService.js`. |
| `cd backend && node test_isolation_static.js` | **PASS** | String-presence assertions only. |
| `cd backend && node test_financial_mock.js` | **PASS** | Uses a hand-written mocked `query()` implementation; no DB or server. |
| `python3 tests/android_backend_contract_check.py` | **PASS**: 51 Retrofit + 76 raw paths | Static scanning of Android/backend source. |
| `python3 tests/deep_release_audit.py` | **PASS**: 43 invariants | Static substring checks of source files. |
| `node --check backend/test_*.js` | **PASS** | Syntax validation only. |
| `python3 -m py_compile tests/*.py` | **PASS** | Syntax compilation only; does not execute `cert_suite.py` side effects. |
| `env -i ... npm test` (no credentials; bounded to 45s) | **FAIL (expected prerequisite failure)** | First two static/mock tests passed; `test_reservation.js` then failed `ECONNREFUSED` on `::1:5432` / `127.0.0.1:5432`. No local Postgres or test server exists. |
| `./gradlew testDebugUnitTest --stacktrace --no-daemon` (bounded to 300s) | **FAIL** | Environment limitation: `SDK location not found`; no `ANDROID_HOME` or `local.properties` SDK path. This is not a source-test failure. The exact GitHub run did complete this task. |
| `docker compose -f backend/docker-compose.yml config` with placeholders | **NOT RUN** | Docker CLI is absent in the audit sandbox (`docker: command not found`). No containers were started. |
| `curl --fail ... https://api.gamenermayket.ir/api/v1/time` | **HTTP 200** | At audit time the public endpoint responded through nginx with `{"serverTime":1790920443840,"timezone":"Asia/Tehran"}` and HSTS. It cannot identify the deployed Git commit, schema version, container health, secret configuration, or functional correctness. |

The optional Python/Ruby/Node YAML parsing libraries were not installed in the sandbox. This is not a workflow defect: GitHub accepted and executed both Android workflow YAML files on this SHA.

## 2. Findings (16)

Severity reflects release and operational risk, not a claim of active exploitation.

### F-01 — Release gate is currently failing; no signed APK/AAB was produced

**Severity: Blocker**

**Evidence**

* The exact release run `36964812496` concluded `failure`; job metadata names the failed step **“Require stable release signing credentials.”** All signing, release-build, and release-upload steps were skipped.
* The gate explicitly refuses a release without all four secrets:

```yaml
# .github/workflows/android-release-validation.yml:66-92
66  - name: Require stable release signing credentials
68    GAMENEXA_RELEASE_KEYSTORE_BASE64: ${{ secrets.GAMENEXA_RELEASE_KEYSTORE_BASE64 }}
69    GAMENEXA_STORE_PASSWORD: ${{ secrets.GAMENEXA_STORE_PASSWORD }}
70    GAMENEXA_KEY_ALIAS: ${{ secrets.GAMENEXA_KEY_ALIAS }}
71    GAMENEXA_KEY_PASSWORD: ${{ secrets.GAMENEXA_KEY_PASSWORD }}
73    test -n "$GAMENEXA_RELEASE_KEYSTORE_BASE64" || { ...; exit 1; }
78  - name: Restore stable release keystore
82    echo "$GAMENEXA_RELEASE_KEYSTORE_BASE64" | base64 --decode > "$RUNNER_TEMP/gamenexa-release.jks"
85  - name: Build signed Release APK and AAB with stable signing
92    ./gradlew assembleRelease bundleRelease --stacktrace
```

**Impact:** There is no verified, signed release APK/AAB for this SHA. Debug APK success must not be described as release validation.

**Action:** Configure all four repository/environment secrets, preferably expose them only to a protected release environment and tag/manual-release workflow; rerun and retain the signed build result. Verify signing certificate fingerprint and app/version metadata in the generated artifacts.

---

### F-02 — CI does not execute backend tests, dependency installation, Docker build, database migration, or deployment

**Severity: High**

**Evidence**

Both Android workflows run the same narrow checks:

```yaml
# .github/workflows/android-debug.yml:46-56
46  - name: Check Android/backend API contract
47    run: python3 tests/android_backend_contract_check.py
49  - name: Run deep Android/backend/VPS contract audit
50    run: python3 tests/deep_release_audit.py
52  - name: Validate backend JavaScript syntax
53    run: node --check backend/server.js && node --check backend/canonical_routes.js && node --check backend/financialService.js
55  - name: Run JVM tests
56    run: ./gradlew testDebugUnitTest --stacktrace
```

`android-release-validation.yml:45-55` has the same checks. Neither workflow contains `npm ci`, `npm test`, `docker build`, `docker compose`, schema migration, image publication, SSH/VPS deployment, or post-deploy smoke test.

**Impact:** A green debug workflow is not evidence that the Node service starts, dependency lock resolves, PostgreSQL schema works, tests pass, an image builds, or the VPS is updated.

**Action:** Add an isolated backend job: `npm ci`, lint/test syntax, a disposable Postgres service, schema/migration application, local API startup, and integration tests. Add a separate container build with a digest/SBOM/provenance policy. Make deployment and post-deploy health/contract smoke tests explicit protected jobs rather than inferred from Android CI.

---

### F-03 — The declared backend test suite is integration/destructive and is not CI-safe as written

**Severity: High**

**Evidence**

* `backend/package.json:5-8` defines `npm test` as a chain of 12 scripts, most of which use `pg` and/or `fetch`; no CI job invokes it.
* `backend/test_reservation.js:5, 12-20` opens a `pg.Pool` from `DATABASE_URL` and inserts fixed `mgr_res`, customer IDs `888`, and station IDs `888/889`. It only deletes prior reservations at line 14 and does not clean the fixed manager/customer/station fixtures at completion.
* `backend/test_security_isolated.js:4-11` targets the public production-looking hostname while also mutating the database:

```js
const BASE='https://api.gamenermayket.ir';
const pool=new Pool({connectionString:process.env.DATABASE_URL});
// ... inserts managers/customers/reservations, then fetches BASE+path
```

* `test_financial.js`, `test_cancellation_boundaries.js`, `test_subscription_entitlement.js`, `test_activation_runtime.js`, `test_sm_auth.js`, and `test_sm_routes.js` also insert/delete rows and expect a separately running API at `127.0.0.1:3000` or the public host.
* The safe credential-free run confirmed the prerequisite coupling: after the static/mock tests, `test_reservation.js` aborted with `ECONNREFUSED` on port 5432.

**Impact:** These are not hermetic tests. Running them against a shared, staging, or production database can mutate data; running them in CI lacks the database/API setup and does not happen at all. Cleanup errors are often swallowed (`catch{}`), compounding residue risk.

**Action:** Split source-only tests from integration tests. Provision an ephemeral Postgres database per CI run, load migrations, start the API from the checked-out source on a randomized local port, use generated fixture namespaces, and clean up in `finally` with failures surfaced. Never point a test at the public production hostname.

---

### F-04 — Docker Compose cannot initialize a fresh database; the schema and volume are external assumptions

**Severity: High**

**Evidence**

```dockerfile
# backend/Dockerfile:1-6
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

```yaml
# backend/docker-compose.yml:54-57
volumes:
  pgdata:
    name: vps-backend_pgdata
    external: true
```

* `backend/schema.sql` exists but Compose contains no `schema`, `initdb`, `command`, or `entrypoint` reference, and it is not mounted into PostgreSQL’s initialization directory.
* `server.js:11` imports `bootstrap.js`, but `bootstrap.js:14-16` first queries the already-existing `managers` table; it does not create tables or load `schema.sql`.
* A fresh host also requires a pre-created external volume named `vps-backend_pgdata`.

**Impact:** `docker compose up` is not a reproducible fresh deployment. On a blank PostgreSQL volume, required tables will not exist. This is also inconsistent with the source-package claim that the package is self-hosted without a documented restore/migration procedure.

**Action:** Introduce versioned, idempotent migrations and run them in a one-shot migration job before API startup. Either create/manage the volume in Compose or document and validate the external-volume backup/restore prerequisite. Fail API readiness if the expected schema version is unavailable.

---

### F-05 — Runtime environment templates do not document all required variables

**Severity: High**

**Evidence**

* Compose interpolates both `${DB_PASSWORD}` and `${DATABASE_URL}` (`backend/docker-compose.yml:8,27`).
* The API creates its database pool only from `DATABASE_URL` (`backend/server.js:47-49`) and fails if `JWT_SECRET` is absent (`server.js:62-63`).
* Runtime also reads `CORS_ORIGINS`, `JSON_BODY_LIMIT`, `SUPER_ADMIN_PASSWORD`, and `SUPER_ADMIN_USERNAME` (`server.js:12,21`; `bootstrap.js:9-10`).
* Neither `.env.example` nor `backend/.env.example` documents `DATABASE_URL`, `CORS_ORIGINS`, `SUPER_ADMIN_PASSWORD`, `SUPER_ADMIN_USERNAME`, or `JSON_BODY_LIMIT`; the backend template instead lists disconnected `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, and `DB_NAME` values (lines 1-10).

**Impact:** Copying either supplied example will not produce a documented working Compose environment. An omitted `CORS_ORIGINS` rejects browser origins (while allowing requests with no `Origin` header), and a missing `DATABASE_URL` leaves the API unusable.

**Action:** Publish one non-secret, complete environment contract with required/optional status, accepted formats, a correctly URL-encoded `DATABASE_URL` example, the production CORS origins, bootstrap behavior, and secret provisioning method. Validate required configuration at process start with clear non-secret diagnostics.

---

### F-06 — API health can be green while the database/schema is unusable

**Severity: High**

**Evidence**

```yaml
# backend/docker-compose.yml:31-36
healthcheck:
  test: ["CMD-SHELL", "wget -q -O - http://127.0.0.1:3000/api/v1/time >/dev/null"]
  interval: 30s
```

```js
// backend/server.js:302-307
app.get('/api/v1/time', rateLimit({ windowMs: 60_000, max: 60 }), (req, res) => {
  res.set('Cache-Control', 'no-store, no-cache, must-revalidate');
  return res.json({ serverTime: Date.now(), timezone: 'Asia/Tehran' });
});
```

The endpoint does not query PostgreSQL. Furthermore, the initial expiry DB task catches query failures (`server.js:1480-1484`) and resolves; its `.then()` then starts the listener (`1488-1491`). Nginx only waits for this API health check (`docker-compose.yml:49-51`).

**Impact:** Compose can report the API healthy and start nginx even when the database schema is missing or runtime database queries are failing. The public 200 obtained in this audit establishes only reachability of that route.

**Action:** Add `/healthz` (process) and `/readyz` (bounded database query + schema version) endpoints. Point Compose/nginx/deployment readiness to `/readyz`; alert on DB connection/schema failures.

---

### F-07 — Android version metadata is contradictory; artifacts build as 2 / 1.0.1, not documented properties 4 / 1.0.2

**Severity: High**

**Evidence**

```kotlin
// app/build.gradle.kts:13-22
// Read versionCode and versionName dynamically from gradle.properties
val appVersionCode = 2
val appVersionName = "1.0.1"
...
versionCode = appVersionCode
versionName = appVersionName
```

```properties
# gradle.properties:30-32
app.versionCode=4
app.versionName=1.0.2
```

**Impact:** The comment is false and release versioning can regress or conflict with installed/market builds. This can block upgrade paths and makes artifact provenance ambiguous.

**Action:** Read and validate the Gradle properties (or remove them and maintain exactly one source of truth). Add a CI assertion that decoded APK/AAB package ID, `versionCode`, and `versionName` match the intended release manifest.

---

### F-08 — Release lint and shrink/obfuscation are disabled, with no CI lint task

**Severity: Medium**

**Evidence**

```kotlin
// app/build.gradle.kts:31-35, 79-84
lint {
  checkReleaseBuilds = false
  abortOnError = false
  checkDependencies = false
}
release {
  isDebuggable = false
  isMinifyEnabled = false
  proguardFiles(...)
}
```

No workflow runs `lint`, and the only release build (lines 85-92 of release workflow) did not execute because the credential gate failed.

**Impact:** Release-only Android correctness/security issues and dependency lint findings do not gate artifacts. `proguard-rules.pro` is effectively unused for shrinking because minification is off.

**Action:** Make `lintRelease` a required job with justified baselines only; enable R8/minification for production after testing or explicitly document the reason it is disabled.

---

### F-09 — Instrumented Android testing is absent from CI, and the only instrumented assertion is inconsistent with the application ID

**Severity: Medium**

**Evidence**

* CI invokes only `./gradlew testDebugUnitTest` (`android-debug.yml:55-56`; `android-release-validation.yml:54-55`). There is no emulator setup, `connected...AndroidTest`, `androidTest`, or device farm invocation.
* The declared package is `com.MinmKhas.studio.GameNexa.wrtx` (`app/build.gradle.kts:18`), but the device-only test asserts `com.example`:

```kotlin
// app/src/androidTest/java/com/example/ExampleInstrumentedTest.kt:14-21
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {
  @Test fun useAppContext() {
    val appContext = InstrumentationRegistry.getInstrumentation().targetContext
    assertEquals("com.example", appContext.packageName)
  }
}
```

**Impact:** No actual installed-app, permission, networking, deep-link, or device behavior is tested in CI. If this sample test were run, the asserted package name is expected to disagree with the configured application ID.

**Action:** Correct/remove sample test, then add emulator instrumentation for launch, login/session restoration, protected API behavior, subscription/trial states, and payment/deep-link flows. Use a non-production backend fixture.

---

### F-10 — Android unit tests provide little behavioral assurance and include false-positive patterns

**Severity: Medium**

**Evidence**

* `ExampleUnitTest.kt:11-15` only asserts `4 == 2 + 2`.
* `ExampleRobolectricTest.kt:15-20` only reads the `GameNexa` string resource.
* `MainActivityTest.kt:15-29` starts the activity and prints; it contains no assertion.
* `GreetingScreenshotTest.kt:22-31` writes `src/test/screenshots/greeting.png` using `captureRoboImage`; it does not compare against a golden baseline and it renders only a literal `Text("GameNet Manager")`.
* `TrialAndSubscriptionTest.kt:25-108` performs UI interactions, sleeps, and prints callback results, but has no assertions about success, displayed error/success state, network request, or entitlement outcome.

**Impact:** A passing `testDebugUnitTest` mostly demonstrates that sample views can be constructed under Robolectric. It does not validate the claimed application workflows and can remain green when the trial/subscription behavior fails.

**Action:** Replace sample/print-only tests with deterministic assertions over ViewModel state, mocked HTTP responses, navigation, validation/error UI, and screenshot diffs against reviewed golden files. Avoid `Thread.sleep` in favor of idling/coroutine test dispatchers.

---

### F-11 — Several Python “test suites” are empty placeholders; the one large legacy suite targets another architecture

**Severity: Medium**

**Evidence**

Eight tracked files consist only of comments and exit successfully if invoked: `cancellation_boundary_tests.py`, `database_integrity_tests.py`, `gold_diamond_tests.py`, `payment_idempotency_tests.py`, `pricing_snapshot_tests.py` (one line each), plus `concurrency_tests.py`, `multi_worker_tests.py`, and `security_tests.py` (two lines each). Example:

```python
# tests/security_tests.py:1-2
# Placeholder for Security Tests (Manager IDOR, Auth bypass)
# Validated via Manager Isolation and Auth Token checks in cert_suite.py
```

`tests/cert_suite.py` is executable but not CI-invoked and uses a different system:

```python
# tests/cert_suite.py:8-10
BASE_URL = 'http://localhost:8080/api/selfhosted'
DB_PATH = '/var/www/gamenet-server/gamenet_central.db'
```

This conflicts with the current Node/PostgreSQL Compose API (`gamenet_api:3000`). It also directly mutates SQLite and an HTTP service, so it was not run during this safe audit.

**Impact:** File presence can be mistaken for coverage. These suites neither test the current runtime nor gate changes.

**Action:** Remove/archive stale tests or port them to the current API/Postgres architecture; implement real pytest tests with fixtures and include them in CI. Do not count comment-only files as test coverage.

---

### F-12 — Contract/audit scripts are useful regression tripwires but static-only; they do not verify the VPS or runtime contract

**Severity: Medium**

**Evidence**

* `tests/android_backend_contract_check.py:8-9` joins `.kt` and `.js` source into strings; route checks use regex/literal presence. Its own comment says raw paths intentionally do not infer HTTP method (`lines 17-18, 44-45`).
* `tests/deep_release_audit.py:8-20` reads source text, and `lines 22-75` evaluates 43 substring predicates.
* `backend/test_isolation_static.js:4-24` likewise asserts `includes(...)` snippets in source files.
* All three pass locally and ran in the successful debug workflow, but none starts Express, connects Postgres, authenticates a request, inspects a deployed VPS, or proves an Android call succeeds.

**Impact:** Source refactors can produce false negatives/positives; runtime route registration, middleware ordering, schema state, TLS, headers, container configuration, and deployment drift are untested.

**Action:** Keep these checks as fast static gates, but label them accurately and add contract/integration tests that start the actual service and client against versioned fixtures.

---

### F-13 — Container build is non-reproducible and can copy host `node_modules` into the Alpine image

**Severity: Medium**

**Evidence**

* `backend/Dockerfile:4` uses `npm install`, not lockfile-enforcing `npm ci`.
* `backend/.dockerignore` and root `.dockerignore` are absent. The physical `backend/node_modules` directory exists in the audit checkout, and `Dockerfile:5` uses `COPY . .` after installation.
* Therefore a Docker build context can overwrite container-installed dependencies with host `node_modules`; this is especially risky for native `bcrypt` when the host differs from Node 18 Alpine/musl.
* The image bases are mutable tags: `node:18-alpine` (`Dockerfile:1`), `postgres:16` (`docker-compose.yml:4`), and `nginx:latest` (`docker-compose.yml:40`).

**Impact:** Build output can depend on untracked host state, image tag movement, or incompatible native binaries. A production image cannot be reproduced from the commit alone.

**Action:** Add a backend `.dockerignore` at minimum for `node_modules`, `.env`, logs, test output, and VCS metadata; use `npm ci --omit=dev` (as appropriate) with the committed lockfile; pin base images by immutable digest; build/test the image in CI.

---

### F-14 — Supply-chain/reproducibility controls are incomplete

**Severity: Medium**

**Evidence**

* Workflow actions are version tags rather than commit SHAs: `actions/checkout@v7`, `actions/setup-java@v6`, `android-actions/setup-android@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`, and `Mattraks/delete-workflow-runs@v2` (all workflow files, e.g. debug lines 21-25 and delete-runs line 13).
* Gradle wrapper pins `gradle-9.3.1-bin.zip` but `gradle/wrapper/gradle-wrapper.properties:1-8` contains no `distributionSha256Sum`.
* No Gradle dependency-locking or verification metadata file exists. The version catalog pins many direct versions, but no dependency verification/lock is configured.
* Positive control: `package-lock.json` is lockfile v3 and includes integrity hashes; resolved production examples are Express 5.2.1, pg 8.23.1, dotenv 17.4.2, bcrypt 6.0.0, and jsonwebtoken 9.0.3.

**Impact:** Remote action, Gradle distribution, transitive dependency, and image resolution can drift from the source review. This is a supply-chain/rebuild risk; no vulnerability conclusion is implied because no current advisory scan was performed.

**Action:** SHA-pin actions (with Renovate/Dependabot management), add Gradle wrapper distribution checksum and dependency verification/locking, maintain dependency update/audit jobs, and publish SBOM/provenance for release artifacts/images.

---

### F-15 — No release publication, deployment, artifact provenance, or retention suitable for an operational release exists in workflow source

**Severity: Medium**

**Evidence**

* README only says the repository is for a self-hosted backend and Android release build and that no external APK upload service is part of release workflow (`README.md:3-7`).
* The release workflow only uploads GitHub workflow artifacts for three days (`android-release-validation.yml:94-116`); it does not create a GitHub Release, sign/attest/provenance-publish artifacts, publish a container, deploy a backend, or run post-deploy verification.
* Debug artifacts are retained for one day (`android-debug.yml:74-80`).
* The maintenance workflow may delete run history after zero days while retaining only four runs (`delete-runs.yml:12-18`).

**Impact:** There is no auditable promotion path from commit to an immutable Android release/backend deployment, and operational forensic evidence is aggressively short-lived.

**Action:** Define separate build, attest/sign, release, deploy, and verify stages; store release assets in a durable release registry; retain artifacts/logs according to incident and compliance needs; protect environments and require approval for production.

---

### F-16 — VPS/running-API consistency remains unverified despite a successful public time probe

**Severity: Medium**

**Confirmed limited evidence:** The configured hostname and TLS route were reachable during audit. `GET https://api.gamenermayket.ir/api/v1/time` returned 200, `server: nginx`, `Strict-Transport-Security: max-age=31536000`, and the expected Tehran-time JSON. Nginx source also enforces TLS 1.2/1.3 and HSTS (`backend/nginx.conf:24-36`) and proxies to `gamenet_api:3000` (`lines 38-47`).

**Not verified:** There was no VPS/SSH, Docker daemon, database, deployment credential, container digest, production environment, certificate-renewal, schema-version, backup/restore, or release-signing-secret access. A public time response cannot show that the VPS runs this commit, that `schema.sql` was applied, that Compose is healthy, that the database is backed up, or that privileged API paths work.

**Action:** Deploy an authenticated version/build-info endpoint containing a commit SHA and schema migration version (no secrets), record image digests, and perform protected post-deploy readiness/contract checks. Establish and test database backup/restore and certificate renewal procedures.

## 3. Security and operational positives / caveats

| Area | Confirmed positive | Caveat |
|---|---|---|
| Release signing | Gradle rejects release tasks without valid keystore path/password/alias/key password (`app/build.gradle.kts:37-73`), and the workflow refuses ephemeral replacement keys. | The gate currently fails; signing success and cert identity are unverified. Do not expose those secrets on every branch/PR run. |
| Secret handling in source | Root `.gitignore` excludes `.env`, keystores, JKS/P12/PEM, APK/AAB, and `node_modules`; no tracked `.env` or key/certificate filename was found. Templates contain placeholders rather than apparent values. | This is a filename/source review, not a historical secret scan or proof of secret hygiene in Git history, runner logs, VPS, caches, or GitHub settings. |
| Backend startup secrets | The server aborts at startup if `JWT_SECRET` is absent (`server.js:62-63`). | `DATABASE_URL` and supporting runtime environment are undocumented in templates; bootstrap configuration is incomplete. |
| Reverse proxy | TLS 1.2/1.3, HSTS, certificate paths, ACME location, and a basic nginx request limit are configured (`nginx.conf:14-46`). | `nginx:latest` is mutable; no certbot/renewal service or nginx health check is present in Compose. |
| API source controls | CORS allow-list logic, JSON body limit, `X-Content-Type-Options`, `Referrer-Policy`, JWT algorithm restriction, and auth gates are present in source (`server.js:10-27, 62-75`). | No live/authenticated test proves actual deployed configuration or multi-instance rate-limit behavior; application rate limits are in-memory (`server.js:30-45`). |

## 4. Test quality classification

| Class | Files / commands | Quality conclusion |
|---|---|---|
| **Real build executed for exact commit** | GitHub debug `testDebugUnitTest`, `assembleDebug` | Confirms the Android debug build/unit task executed in GitHub. Coverage quality remains weak due to sample/print-only tests. |
| **Release build attempted** | GitHub release workflow | Did not reach signed build; credentials gate failed. |
| **Source-only static contracts** | `android_backend_contract_check.py`, `deep_release_audit.py`, `test_isolation_static.js`, `npm run check` | Useful fast safeguards; **not** runtime, API, database, or VPS evidence. |
| **Mock-only logic** | `test_financial_mock.js` | Exercises production functions with SQL-string-sensitive mocked responses; not a PostgreSQL transaction/behavior test. |
| **Potentially meaningful integration tests, but not run by CI** | Most `backend/test_*.js` | Could exercise actual Postgres/API behavior only in an isolated environment. Present implementation is unsafe/non-hermetic and assumes live services. |
| **Placeholders / stale architecture** | Eight short Python comment files; `cert_suite.py` | Not meaningful coverage of current Node/Postgres runtime. |
| **No device/E2E** | Android `androidTest` absent from workflow | No evidence for real-device/emulator install, deep links, network security, auth, subscription, billing, offline/reconnect, or lifecycle flows. |

## 5. Priority remediation plan

1. **Block release promotion now:** configure protected release-signing secrets; rerun the exact release workflow; verify signed APK/AAB certificate, application ID, and corrected version values. Restrict signing to tags/protected environments instead of all branches and PRs.
2. **Create hermetic backend CI:** `npm ci`; ephemeral Postgres; apply versioned migrations; start API locally; run test suite against only that environment; eliminate public-host and fixed-data test targets.
3. **Make Docker deployable from zero:** migration job/schema version, non-external-or-documented volume lifecycle, complete `.env.example`, `/readyz` DB/schema health, Docker `.dockerignore`, digest-pinned images, and CI image build.
4. **Repair test signal:** replace Android sample tests and print/sleep tests with asserted behavior; enable emulator instrumentation; replace Python placeholders/stale SQLite suite; preserve static checks as a separate labeled job.
5. **Harden reproducibility and provenance:** use `npm ci`, SHA-pin GitHub Actions, add Gradle checksum/dependency verification, digest-pin containers, generate SBOM/provenance, and retain immutable release assets/logs appropriately.
6. **Close operational verification:** deploy commit/schema/image-digest build info, add protected post-deploy smoke tests, monitor readiness, and test DB backup/restore plus certificate renewal.

## 6. Explicit unverified items

The following cannot be inferred from the repository or public unauthenticated endpoint and were **not claimed as verified**:

* Presence/correctness of GitHub release-signing secrets, keystore alias, certificate fingerprint, or Play/Myket signing identity.
* VPS container state, Compose invocation, image digest, current code SHA, database state/schema, migrations, persistent volume contents, logs, backups/restores, firewall, certificate renewal, or system resource limits.
* Live behavior of authenticated/financial/subscription endpoints, database constraints under concurrency, Android real-device behavior, and cross-version upgrade/install behavior.
* Current dependency CVEs/advisories (no advisory database scan was performed); only file-visible versioning/reproducibility risks are reported.
* Whether any action-version tag, Docker base tag, or remote dependency has changed since review; tags are identified as a risk because they are mutable.

## 7. Inspected file groups

* **Actions:** `.github/workflows/android-debug.yml`, `android-release-validation.yml`, `delete-runs.yml`.
* **Android/Gradle:** root/app Gradle Kotlin scripts, settings, properties, version catalog, wrapper properties/JAR presence, manifest, network/backup/extraction XML, ProGuard rules, Android test sources.
* **Backend/npm:** `package.json`, `package-lock.json`, server/canonical/financial/reservation/bootstrap sources, all twelve `backend/test_*.js` files, schema.
* **Runtime/deployment:** `Dockerfile`, `docker-compose.yml`, `nginx.conf`, root/backend env examples, ignore files.
* **Documentation/manifests:** `README.md`, `RELEASE_AUDIT_STATUS.md`, `SOURCE_PACKAGE_MANIFEST.md`, `metadata.json`.
* **Python QA:** every file under `tests/`, including comment-only/stale suites.


---

