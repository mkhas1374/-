# GameNexa Release Audit

**Audit date:** 2026-10-03  
**Tester:** Manus (target-locked source audit; exact-target CI evidence review)  
**Audit status:** Target validation **PASS**; product/release audit **NOT RELEASE READY**  
**Scope note:** Source review was performed only after verifying the required repository, branch, and exact SHA. No production source was changed. Source-derived defects below have not all been reproduced on a live Android device/backend; runtime gaps are explicitly identified.

## Executive summary

The required target was obtained and verified exactly: `mkhas1374/GameNexa-release-2026`, branch `hardening-release-20261002`, commit `e800b92f9ff75a05d912a23911ef2cbb5e63eb78`; the working tree was clean at the start and end of the audit.

Current-target CI provides positive but incomplete evidence: the backend integration workflow and Android JVM tests/debug build passed for this SHA. The debug APK was downloaded from that exact-target CI run and hashed. **Both Release Validation runs for this exact SHA failed** before release signing/build because the stable release keystore and signing secrets are not configured. No signed release APK/AAB or release artifact integrity evidence exists. This alone blocks release certification.

The independent read-only source review found material defects across registration, password handling, customer/station synchronization, reservations, payments, subscriptions, authorization, database integrity, error handling, Docker hardening, and Persian date/time presentation. Key examples include a live-stations POST that accepts writes but discards them while returning success; customer logout that selects the manager token and therefore does not revoke a customer token; unauthenticated subscription-status routes returning account-specific state; non-serializable reservation races; financial workflow/idempotency mismatches; and plaintext manager passwords displayed in the Android UI.

**Final verdict: NOT RELEASE READY.** This report does not certify operational behavior that was not executed. Release signing, full Android/runtime coverage, live backend/database verification, device tests, performance measurements, and several concurrency/security scenarios remain unverified.

## Audit target

| Field | Required | Actual | Result |
|---|---|---|---|
| Repository | `mkhas1374/GameNexa-release-2026` | `mkhas1374/GameNexa-release-2026` (`https://github.com/mkhas1374/GameNexa-release-2026.git`) | PASS |
| Branch | `hardening-release-20261002` | `hardening-release-20261002` | PASS |
| Commit | `e800b92f9ff75a05d912a23911ef2cbb5e63eb78` | `e800b92f9ff75a05d912a23911ef2cbb5e63eb78` | PASS |
| Working tree | Clean | Clean; `git status --short` empty; `git diff` and `git diff --cached` empty | PASS |
| Source origin | Target branch | Fetched from `origin`, checked out required branch and fast-forwarded; HEAD matches target | PASS |
| Commit date | — | `2026-10-03T04:48:04Z` | Recorded |
| Commit message | `Merge remote-tracking branch 'origin/main' into hardening-release-20261002` | Exact match | PASS |
| Tester source modifications | None | None | PASS |

### Target verification output

```text
git rev-parse --abbrev-ref HEAD
hardening-release-20261002

git rev-parse HEAD
e800b92f9ff75a05d912a23911ef2cbb5e63eb78

git status --short
[no output]

git log -1 --oneline
e800b92 (HEAD -> hardening-release-20261002, origin/hardening-release-20261002) Merge remote-tracking branch 'origin/main' into hardening-release-20261002
```

The repository's `RELEASE_AUDIT_STATUS.md` is dated 2026-09-29 and contains historical claims. Those claims were **not** reused as evidence for this audit.

## Android artifact provenance

| Field | Result |
|---|---|
| Release APK/AAB | **Not produced / not available**; release validation halted at required stable signing credentials |
| Debug artifact | `app-debug.apk`, downloaded from GitHub Actions run `37097754171` for exact target SHA |
| Build variant | Debug |
| Application ID | `com.MinmKhas.studio.GameNexa.wrtx` (from target source) |
| versionCode / versionName | `4` / `1.0.2` (from target `gradle.properties`) |
| CI build interval | 2026-10-03, approximately 04:52:39–04:54:48 UTC |
| APK SHA-256 | `80545a9d4edbca2ff77f7039510482a1f5bc7952855141ce92a1ef7b83d6cd09` |
| Uploaded artifact ZIP SHA-256 | `d936be433132da2fbe4f322c2927ad399f184e03f477d5a0e77bd8cff71f9944` |
| CI artifact ID | `11265240960` (run `37097754171`) |
| Device installation / runtime | Not performed; this debug APK is not represented as a release APK or as device-tested |

The build variant and app version values are source/configuration evidence. The artifact run's head SHA is the required SHA, but no signed release artifact exists for certification.

## Backend target and database

| Field | Result |
|---|---|
| Backend source SHA | `e800b92f9ff75a05d912a23911ef2cbb5e63eb78` (same verified repository checkout) |
| Backend version | No separately verified runtime version; target source only |
| Database/schema | No local database or live target database was connected for this audit |
| Migration/schema version | PostgreSQL schema source reviewed; deployed schema/migration identity not verified |
| Docker image/digest | No target image built or inspected; Docker CLI unavailable in the sandbox |
| Backend deployment identity | No backend deployment was started or contacted |

## CI/CD evidence for the exact SHA

All run records below were queried by the required commit and their `headSha` was verified as `e800b92f9ff75a05d912a23911ef2cbb5e63eb78`.

| Workflow / run | Result | Evidence |
|---|---|---|
| Backend Integration — [37097754127](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/37097754127) | PASS | Completed successfully; isolated PostgreSQL, `npm ci`, API readiness sequence, and `npm test` / `test:release` chain completed. Logged test suites include tenant isolation, reservation, cancellation, security, financial, subscription, activation, Super Manager, and database integrity checks. |
| Backend Integration — [37097751172](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/37097751172) | PASS | Second completed backend run on the same SHA. |
| Android Debug Build — [37097754171](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/37097754171) | PASS | `testDebugUnitTest` succeeded; `assembleDebug` succeeded; one debug APK uploaded. No device/instrumentation run is evidenced. |
| Android Release Validation — [37097754157](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/37097754157) | **FAIL — release blocker** | Failed at “Require stable release signing credentials”: `GAMENEXA_RELEASE_KEYSTORE_BASE64` and associated signing secrets were empty/missing. Workflow explicitly refused to create an ephemeral signing key. Release artifact not produced. |
| Android Release Validation — [37097751180](https://github.com/mkhas1374/GameNexa-release-2026/actions/runs/37097751180) | **FAIL — release blocker** | Same missing stable release keystore/secrets failure for the exact target SHA. |

Static workflow review additionally found no explicit expected-SHA assertion/ref lock, no dedicated Android lint task, no instrumented/device test invocation, and no post-build release signature/checksum verification step. The exact-SHA run records establish what these particular workflows processed; the workflow source does not itself enforce that future runs validate only the mandated SHA.

## Previous findings retested against the current target

| Finding | Result | Current-target conclusion |
|---|---|---|
| F-001 — Customer registration | **FAIL** | Normal request fields and success parsing are broadly aligned, but required email is collected and discarded; registration and login normalize numeral characters differently; duplicate/error responses are poorly surfaced and a race can become HTTP 500. Source evidence: `AuthDialog.kt:297-307,356-375`; `GameNexaWelcomeScreen.kt:487-510`; `GameNetViewModel.kt:5301-5319`; `SelfHostedManager.kt:383-402,466-523`; `backend/server.js:254-294`; `backend/schema.sql:1400-1401`. No live registration test was performed. |
| F-002 — Password exposure | **FAIL** | Manager create/edit UI uses unmasked password fields; plaintext is displayed after creation and a manager-list UI sink can display/copy a returned `password` field. Canonical server projection omits the field, so that particular list-to-sink path is exposure-capable but not shown to be populated by the canonical response. Source: `SettingsScreen.kt:2133-2178,2553-2577,2682-2686`; `GameNetApi.kt:687-696`; `backend/server.js:389-427`. Backend password hashing and Room removal of customer plaintext are positive controls. No UI interaction was executed. |
| F-003 — Customer synchronization | **FAIL** | Offline customer mutations have no durable CRUD outbox; later server reconciliation can delete them. POST upsert response identity is discarded and local IDs may be retained instead of canonical server IDs. Source: `GameNetRepository.kt:271-294,788-802`; `SelfHostedManager.kt:545-568`; `canonical_routes.js:112-139`. No offline/device scenario was run. |
| F-004 — Station synchronization | **FAIL** | Station configuration sync can overwrite operational state; successful empty responses are treated as bootstrap signals; live-station GET/Android DTO fields disagree; the POST write is a no-op that returns success; buffet decrements/removals do not reach the server's charge ledger. Source: `GameNetRepository.kt:235-269`; `SelfHostedManager.kt:610-675,1748-1791`; `canonical_routes.js:175-181`; `server.js:1117-1156,1217-1231,1381-1387`. No runtime synchronization/settlement test was performed. |
| F-005 — Docker hardening | **PARTIAL** | Lockfile plus `npm ci --omit=dev` and environment-file exclusions exist. Base image tags are mutable, non-root/least-privilege runtime is not source-enforced, ignore rules are incomplete, and no target image was built/inspected. Source: `backend/Dockerfile:1-6`; `backend/.dockerignore:1-6`; `backend/docker-compose.yml:4,20-54`; `backend/package-lock.json`. |
| F-006 — Release validation | **FAIL** | Exact-SHA Release Validation failed twice due missing stable signing secrets. No release APK/AAB exists. Workflow-source gaps include no exact-SHA assertion, no dedicated lint/device-test task, and no post-build artifact signing/checksum verification. See CI table and `.github/workflows/android-release-validation.yml:39-126`. |
| F-007 — Silent exception handling | **FAIL** | Current source ignores or swallows errors for customer upsert, manager configuration, reservation-payment decisions/cancellation, station sync, payment UI parsing/external actions, and customer session restoration. Better durable retry controls exist for selected station/session/settlement flows but are inconsistent. Examples: `GameNetViewModel.kt:4082-4085,4229-4238,6465-6500`; `SelfHostedManager.kt:830-854,1541-1570`; `ManagerReservationsScreen.kt:66-85`; `CustomerOnlinePaymentTab.kt:29-90,275,476-484`. No UI/network failure injection was run. |
| F-008 — Legacy no-op endpoint | **FAIL** | `POST /api/v1/manager/live-stations` authenticates and then returns `{success:true,canonical:true}` without reading or persisting the payload or invoking the session engine; Android actively posts to it and treats 2xx as success. Source: `backend/canonical_routes.js:179-181`; `SelfHostedManager.kt:1748-1788`; callers in `GameNetViewModel.kt:6428-6457,6465-6497`. No live HTTP request was made. |

## Full release audit matrix

**FAIL** means current-target source or target-SHA CI provides concrete failure evidence. **PARTIAL** means some controls/evidence exist but coverage is incomplete. **UNVERIFIED** means no defensible pass/fail conclusion was established without runtime measurement.

| Area | Status | Summary |
|---|---|---|
| Architecture / Android-backend contract | FAIL | Several ordinary contracts align, but critical registration, live-station, and manager-payment paths have concrete mismatches/no-op behavior. |
| Authentication | FAIL | Customer logout selects the manager bearer token; customer server revocation therefore does not occur through this client path. |
| Authorization | FAIL | Public subscription-status routes disclose account-specific entitlement details based on caller-provided identity; other role/tenant controls exist but were not runtime-matrix-tested. |
| Customer management | FAIL | Offline create/update can be lost; local/server customer identity may diverge; failure reporting is incomplete. |
| Stations | FAIL | Operational-state synchronization can be overwritten; live-stations write is a no-op; buffet removals are not reconciled. |
| Reservations | FAIL | Concurrent capacity/VIP races, non-VIP full-hall charge behavior, snapshot-linkage mismatch, and other lifecycle issues are present in source. |
| Payments / wallet / GN / LP | FAIL | Generic approval can strand reservation proof; BUY_GN conflicts with the schema check; retries can create duplicate requests; wallet-top-up approval omits ledger/audit records. |
| Subscriptions | FAIL | Extension can retire current entitlement before successor start; GET loses revision number; no runtime persistence test. |
| 24-hour trial | FAIL | Trial identity depends on caller-supplied device/fingerprint values without an attested binding; paid-expiry scheduling uses mutable device wall-clock. |
| Offline / online | FAIL | Selected session flows have durable outboxes, but customer/config/station/order mutation paths are not consistently durable/replayed. |
| Time / calendar | FAIL | Solar-Hijri helper exists, but multiple Persian-facing reservation/notification views use Gregorian `Locale.US` and device timezone; digit behavior is inconsistent. |
| UI / UX | FAIL | Plaintext manager password presentation and silent action failures are source-visible. Full fresh-install/screen/lifecycle inspection not performed. |
| Settings / manager configuration | FAIL | Optimistic configuration is not rolled back/retried after failed sync; subscription store GET does not return stored revision. |
| Database integrity | FAIL | Source has substantive PostgreSQL constraints and transactions, but Android migration coverage gaps, restore/deletion non-atomicity, and backend transaction/race defects remain. Deployed schema not verified. |
| Security | FAIL | Customer token revocation and unauthenticated subscription data exposure are concrete source issues; comprehensive endpoint probes were not run. |
| Concurrency | FAIL | Reservation race conditions and unkeyed payment retry paths are evident in source; no concurrent runtime test was executed. |
| Performance | UNVERIFIED | No startup/API/database/sync/settlement latency or query measurements were collected. |
| Crash / stability | UNVERIFIED | No Android device/emulator, lifecycle stress, process-kill, or crash-injection run was available. |
| CI/CD | FAIL | Backend and debug gates pass, but both target-SHA release validations fail for signing configuration; artifact integrity and signed release are absent. |

## New findings

All entries below apply to **SHA `e800b92f9ff75a05d912a23911ef2cbb5e63eb78` only**. For source-derived findings, “actual” describes the path established by current source; it is not a claim that the flow was executed on a live system.

### F-009 — Release signing gate blocks signed release output

- **Severity:** Release blocker
- **Status:** Reproduced in target-SHA CI
- **Affected file/workflow:** `.github/workflows/android-release-validation.yml`; run `37097754157` and run `37097751180`
- **Reproduction:** Open either linked run and inspect `validate-release` → “Require stable release signing credentials”.
- **Expected:** Stable release keystore and associated credentials are configured; release signing/build and artifact validation complete.
- **Actual:** `GAMENEXA_RELEASE_KEYSTORE_BASE64` is empty; the workflow exits 1 with “Stable release keystore secret is not configured. Refusing to create another ephemeral signing key…”. Store password, alias and key password are also empty. No release APK/AAB was produced.
- **Evidence / impact:** Both exact-SHA runs fail before release output. Release cannot be certified or distributed from this evidence.
- **Recommended fix:** Provision and validate the approved stable signing identity through protected CI secrets; rerun Release Validation on the exact SHA; verify signer identity, APK/AAB integrity and checksums.

### F-010 — Customer logout does not revoke customer token

- **Severity:** High
- **Status:** Source-verified; runtime unverified
- **Affected files:** `app/src/main/java/com/example/data/network/GameNetApi.kt:965-972`; `GameNetViewModel.kt:5509-5522`; `backend/server.js:301-312`
- **Reproduction:** Trace customer logout → `logoutSession()` → interceptor route test. `/api/auth/logout` does not match either customer path prefix, so the interceptor selects `managerAuthToken`; the logout view-model then clears both local tokens. The backend requires a valid bearer before incrementing `token_version`.
- **Expected:** Customer bearer is sent and the server revokes/increments the customer token version before local credentials are cleared.
- **Actual:** Customer-only session selects manager token (normally absent), server rejects/misses revocation, and the customer JWT remains usable until expiry unless revoked elsewhere.
- **Evidence / impact:** Customer login issues a 24-hour token (`backend/server.js:281-294`); stale credentials may remain valid after apparent logout.
- **Recommended fix:** Make logout token selection session-role-aware, await/confirm revocation, and test customer and manager logout/replay separately.

### F-011 — Public subscription status reveals account-specific state

- **Severity:** High
- **Status:** Source-verified; runtime unverified
- **Affected files:** `backend/canonical_routes.js:531-542,551-573`
- **Reproduction:** Trace `subscriptionState` and its `/api/v1/subscriptions/check` and `/api/v1/subscriptions/status` callers; neither has auth middleware. Submit a caller-controlled phone/user identifier and inspect returned `active`, plan/expiry, and `st.licenseCode` fields.
- **Expected:** Sensitive manager entitlement/license status requires an authenticated, authorized principal or an unguessable narrowly scoped proof.
- **Actual:** Public route derives manager identity from supplied phone/device values and returns subscription state and a derived `MGR_{managerId}` license code.
- **Evidence / impact:** Enables subscription/account enumeration and exposes identifiers/state. Rate-limit coverage is not established at these route definitions.
- **Recommended fix:** Authenticate/authorize, minimize response fields, avoid deriving public license identifiers, and add rate-limited negative tests.

### F-012 — Required registration email is silently discarded; locale-digit passwords are inconsistent

- **Severity:** High (registration/auth contract)
- **Status:** Source-verified; runtime unverified
- **Affected files:** `AuthDialog.kt:297-307,356-375`; `GameNexaWelcomeScreen.kt:487-510`; `GameNetViewModel.kt:5301-5319`; `SelfHostedManager.kt:383-402,466-486`; `backend/server.js:254-294`
- **Reproduction:** Follow the registration form email through `registerUser` to `registerCustomer`; inspect the JSON assembled by `SelfHostedManager`. Compare password conversion at registration versus login.
- **Expected:** Required validated fields are sent and persisted; identical password text is interpreted identically during registration and login.
- **Actual:** UI validates/collects email but ViewModel/HTTP payload omits it; backend returns `user.email: null`. Registration trims but does not normalize Persian/Arabic numeral glyphs while login runs `toEnglishDigits`; password accepted at registration can be changed before bcrypt comparison at login.
- **Evidence / impact:** Invoice/contact email is lost; users with non-ASCII numeral passwords can be locked out. Duplicate registration is also a SELECT-then-INSERT race mapped to generic 500 (`server.js:265-277`; schema uniqueness at `schema.sql:1400-1401`), and Android ignores non-success error bodies (`SelfHostedManager.kt:494-523`).
- **Recommended fix:** Define the canonical registration contract, persist/return email if required, use one password canonicalization policy or no transformation on either path, map unique violations to 409, and parse/display safe error responses.

### F-013 — Manager passwords are visibly rendered in Android UI

- **Severity:** High
- **Status:** Source-verified; UI runtime unverified
- **Affected files:** `app/src/main/java/com/example/ui/SettingsScreen.kt:2133-2178,2553-2577,2682-2686`; DTO `GameNetApi.kt:687-696`
- **Reproduction:** Inspect create-manager and edit-manager `OutlinedTextField` definitions, post-create confirmation rendering, and manager detail clipboard code.
- **Expected:** Password entry is masked; passwords are not displayed, copied, or retrievable in manager lists.
- **Actual:** Create/edit inputs lack password visual transformation; the created password is rendered in confirmation UI. Manager detail renders/copies a nonblank DTO `password` value. Canonical server projection currently omits password, so the latter is an unsafe sink rather than evidence that the canonical response currently populates it.
- **Evidence / impact:** Shoulder-surfing, screenshots, clipboard retention or an alternate/fallback response can disclose credentials.
- **Recommended fix:** Mask fields, remove plaintext confirmation/display/copy behavior, remove password from DTO/list responses, rotate/reset rather than retrieve passwords, and add UI/API leakage tests.

### F-014 — Station live-state write is a success-acknowledged no-op

- **Severity:** Critical (state integrity)
- **Status:** Source-verified; runtime unverified
- **Affected files:** `backend/canonical_routes.js:179-181`; `SelfHostedManager.kt:1748-1788`; `GameNetViewModel.kt:6428-6457,6465-6497`
- **Reproduction:** Trace Android `syncStationToCloud` JSON to `POST /api/v1/manager/live-stations`; inspect the registered handler. It performs middleware then responds without reading body or mutating state.
- **Expected:** Persist/reconcile server-authoritative live station/session/order state, or explicitly reject/deprecate the operation.
- **Actual:** Handler returns `{success:true,canonical:true}` while discarding the submitted payload; Android treats any 2xx as successful sync.
- **Evidence / impact:** Offline recovery and station/order sync may believe state is stored when it is not. There is no explicit unsupported response or endpoint-specific test.
- **Recommended fix:** Route writes through the canonical session/event engine with idempotency and tests, or return an explicit unsupported status and remove the client call sites.

### F-015 — Station synchronization replaces runtime state with configuration defaults

- **Severity:** High
- **Status:** Source-verified; runtime unverified
- **Affected files:** `GameNetRepository.kt:235-269`; `backend/canonical_routes.js:175-181`; `SelfHostedManager.kt:610-675`
- **Reproduction:** Follow `getStations()` into Room refresh, then compare backend configuration response fields to Android live-state DTO fields.
- **Expected:** Active server session/state wins online; stale offline fallback is clearly marked; an authoritative empty list remains empty.
- **Actual:** Configuration refresh clears local station rows and re-inserts configured stations as `FREE`, irrespective of active sessions; Android live parser reads `status` while endpoint uses `session_status`, defaulting missing state to FREE. Empty list can trigger local re-upload/bootstrap behavior instead of authoritative reconciliation.
- **Evidence / impact:** Active stations can appear available locally, stale station states can be displayed, and duplicate/recreated stations are possible. No live-state sync test was performed.
- **Recommended fix:** Separate station configuration from live session state; define versioned/explicit empty-state semantics; parse the exact DTO; add two-client online/offline reconciliation tests.

### F-016 — Offline customer mutation can be lost and customer IDs can diverge

- **Severity:** High (data integrity)
- **Status:** Source-verified; runtime unverified
- **Affected files:** `GameNetRepository.kt:271-294,788-802`; `SelfHostedManager.kt:545-568`; `backend/canonical_routes.js:112-139`
- **Reproduction:** Follow an offline local customer upsert, failed cloud call, and later non-empty/empty authoritative response. Separately compare local Customer ID with server upsert response handling.
- **Expected:** Offline mutations remain queued until acknowledged; server-generated identity is reconciled before later ID-keyed operations.
- **Actual:** Failed upsert is caught with no durable customer CRUD outbox; reconciliation can delete local-only rows. POST response identity is discarded and phone reconciliation can preserve the local ID even where server resolves a different database row.
- **Evidence / impact:** Lost customers and later archive/session/transaction calls using stale or wrong IDs.
- **Recommended fix:** Persist CRUD operations in an idempotent outbox, reconcile IDs from server responses, and test server-empty/removal and multi-device cases.

### F-017 — Reservation conflicts and capacity are not fully concurrency-safe

- **Severity:** Critical (booking/financial integrity)
- **Status:** Source-verified; runtime concurrency unverified
- **Affected files:** `backend/reservationService.js:101-105,225-244,262-274,289-342`; `backend/canonical_routes.js:272-295`; `backend/schema.sql:1932-1935`
- **Reproduction:** Inspect locks for overlapping reservations. Different-customer requests lock only customer rows; station/time conflicts are read before insert, and schema has no exclusion constraint for those ranges. For VIP approvals, inspect the predicate and absence of a shared time-slot lock.
- **Expected:** Concurrent bookings/confirmations cannot exceed capacity or create overlapping full-hall reservations.
- **Actual:** Two requests can observe no conflicting row and both insert; two VIP confirmations can both pass a predicate before either commits its status transition.
- **Evidence / impact:** Overbooking and double allocation. Existing test definitions are sequential, not a concurrent race.
- **Recommended fix:** Serialize per station/time resource or use PostgreSQL exclusion constraints; lock a shared VIP slot and add parallel database integration tests.

### F-018 — Reservation/station charge and snapshot defects

- **Severity:** High (financial/state integrity)
- **Status:** Source-verified; runtime unverified
- **Affected files:** `backend/financialService.js:150-159`; `backend/reservationService.js:193-199,289-342`; `backend/schema.sql:824-848`; `backend/server.js:898-911,1029-1033,1093-1105,1393-1395`
- **Reproduction:** Compare generated `configurationRevisionId/configurationVersion` names with fields inserted by reservation creation; inspect station start capacity lookup; compare stored `prepayment_amount` with settlement SELECT; inspect offline guest default key.
- **Expected:** Bookings retain correct immutable price-revision linkage; station controller count honors configured capacity; settlement applies stored prepayments; all offline guests receive distinct identities.
- **Actual:** Reservation insert reads different/undefined revision/version property names; start permits up to 4 without checking station `controller_capacity`; settlement query omits `prepayment_amount`; offline guests without keys default to `guest:0` and later rows conflict/do nothing. Additionally, non-VIP FULL_HALL creates per-station reservation rows priced from one VIP price (`reservationService.js:193-199,289-342`; `financialService.js:111-114`).
- **Evidence / impact:** Missing price provenance, excess controllers, overcharge/unapplied payment, dropped participants, or duplicate full-hall charging.
- **Recommended fix:** Align snapshot fields, enforce capacity at server start, select/apply prepayments transactionally, assign stable unique participant keys, and define one allocation/charge model for full hall.

### F-019 — Manual payment routes permit duplicate/stranded financial requests

- **Severity:** Critical (financial integrity)
- **Status:** Source-verified; runtime/database execution unverified
- **Affected files:** `backend/canonical_routes.js:397,408,411,444-475`; `backend/schema.sql:591-618`; `SelfHostedManager.kt:781-800,1653-1665,1683-1699,1719-1731`; `financialService.js:255-266`
- **Reproduction:** Compare generic approval branches with reservation-specific approval; trace Android BUY_GN payload to table CHECK constraint; inspect missing idempotency key at top-up/BUY_GN calls; compare wallet balance update with ledger writes.
- **Expected:** Every accepted request purpose has a valid schema value; duplicate submissions replay safely; reservation payments advance the reservation; wallet mutation has a corresponding ledger/audit entry.
- **Actual:** Generic approval can mark a RESERVATION_PAYMENT request approved without creating its payment transaction/reservation transition, after which the dedicated route requires it to still be pending. API accepts BUY_GN but schema CHECK excludes it. Keyless client calls get `Date.now()` keys, so retries can create separate approval candidates. WALLET_TOPUP approval increments balance without wallet transaction/audit record.
- **Evidence / impact:** Stranded proof, blocked GN purchase, duplicate credits, and unreconciled wallet balance.
- **Recommended fix:** Use one purpose enum end-to-end, require idempotency keys and return canonical replay results, constrain generic approval branches, and update wallet ledger/audit atomically with balance.

### F-020 — Subscription extension and revision reporting are inconsistent

- **Severity:** High
- **Status:** Source-verified; runtime unverified
- **Affected files:** `backend/server.js:569-610`; `backend/canonical_routes.js:28-34,547-571`; `schema.sql:1128-1137,2058-2075`
- **Reproduction:** Follow extension of a still-active entitlement through retirement and successor creation, then apply the access gate requiring `starts_at <= NOW()`. Compare settings PUT revision increment with GET return object.
- **Expected:** Extension leaves continuous entitlement; GET returns persisted current configuration revision; device/trial identity cannot be reset by supplying arbitrary caller values.
- **Actual:** Current entitlement is marked EXPIRED before successor starts at the old expiry, creating a gap. Subscription-store GET omits `version_number` and callers default to 1. Trial start accepts untrusted caller-supplied device/fingerprint strings as identity, enabling a fresh pair to avoid the one-trial-per-device check. Paid expiry scheduling also uses mutable device wall-clock (`GameNetViewModel.kt:1747-1773`).
- **Evidence / impact:** Premature manager access loss, stale configuration revision reporting, repeat trials, or extended local access after clock rollback.
- **Recommended fix:** Make extensions immediately continuous or atomically preserve the active entitlement; return the stored revision; bind trial entitlement to an attested/server-issued device identity; derive expiry from server time/monotonic anchor.

### F-021 — Backend exception and rollback paths are inconsistent

- **Severity:** High
- **Status:** Source-verified; runtime unverified
- **Affected files:** `GameNetViewModel.kt:4082-4085,4229-4238,6465-6500`; `ManagerReservationsScreen.kt:66-85`; `CustomerOnlinePaymentTab.kt:29-90,275,476-484`; `backend/reservationService.js:59-85`
- **Reproduction:** Follow each client action return/catch path after forced network failure in a future test; inspect current handling. Inspect `rejectVipReservation` transaction boundaries.
- **Expected:** Failure is shown to the relevant user, state is rolled back or persisted for retry, and multi-write financial rejection is atomic.
- **Actual:** Several Boolean results are ignored and exceptions swallowed; customer update/config/station sync lacks consistent durable replay; VIP rejection does dependent status/refund/balance changes without explicit transaction/row lock.
- **Evidence / impact:** UI can appear successful when remote state failed; reminders/state can diverge; partial refund mutation is possible on error/concurrency.
- **Recommended fix:** Return typed outcomes, persist retries, reflect failure in UI, and wrap multi-table financial effects in locked DB transactions.

### F-022 — Android database restore/delete and retry paths are unsafe

- **Severity:** High (data integrity)
- **Status:** Source-verified; runtime unverified
- **Affected files:** `AppDatabase.kt:31,57-115,189-205`; `GameNetViewModel.kt:6372-6404`; `GameNetRepository.kt:805-831`; `Dao.kt:128-138,191-192,209-210,248-249`; `RetryInterceptor.kt:16-29,37-55`; `GameNetApi.kt:926-936`
- **Reproduction:** Map registered Room migration paths from versions 1–14; inspect asynchronous restore return timing and customer deletion transactionality; inspect retry interceptor eligibility for state-changing POSTs.
- **Expected:** All supported DB versions migrate; restore succeeds only after a complete atomic write; local deletion is atomic; unsafe non-idempotent mutations are not automatically replayed.
- **Actual:** Migration routes are only 4→5 and 7→14, with gaps and no destructive fallback; restore reports success immediately after launching asynchronous writes, without a Room transaction; multi-table customer deletion is not transactional; interceptor retries all methods on I/O/5xx without requiring idempotency keys.
- **Evidence / impact:** Upgrade failure, partial/corrupt restore/deletion, and duplicate mutations on ambiguous retries.
- **Recommended fix:** Define contiguous supported migrations, transactionally apply/validate backup data, make related deletes atomic, and retry writes only with idempotent server keys.

### F-023 — Docker runtime provenance/hardening is insufficiently fixed

- **Severity:** Medium (release hardening)
- **Status:** Partial source finding; no image built
- **Affected files:** `backend/Dockerfile:1-6`; `backend/.dockerignore:1-6`; `backend/docker-compose.yml:4,20-54`
- **Reproduction:** Inspect base image tags, runtime `USER`, Compose privilege controls, and `COPY . .` context exclusions.
- **Expected:** Immutable reviewed base images, least-privilege runtime, deterministic dependency inputs, and context/image contents that exclude secrets and repository metadata.
- **Actual:** `node:22-alpine`, `postgres:16`, and `nginx:1.29-alpine` are mutable tags; API runtime does not source-enforce a non-root user or least-privilege settings; `.dockerignore` is not an allowlist and does not exclude several common secret/metadata patterns. Lockfile plus `npm ci` is a positive control.
- **Evidence / impact:** Weaker reproducibility and increased chance of future context secrets/metadata inclusion. This is a source policy gap, not evidence that the current built image contains a secret.
- **Recommended fix:** Pin image digests, define non-root/read-only/capability policy, use narrow allowlist context, then build and inspect image layers/digests.

### F-024 — Persian date/time presentation is inconsistent

- **Severity:** Medium
- **Status:** Source-verified; device presentation unverified
- **Affected files:** `JalaliCalendarHelper.kt:13-33,105-108`; `ManagerReservationsScreen.kt:134-135`; `CustomersReservationsScreen.kt:2219-2224`; `GameNetViewModel.kt:521-523`
- **Reproduction:** Trace date rendering in the listed reservation/notification paths and compare with the Solar-Hijri helper and timezone selection.
- **Expected:** Persian-facing dates use Tehran timezone and Solar Hijri where required; displayed numerals follow the stated English-digit policy.
- **Actual:** Several views format Gregorian dates with `Locale.US` and inherit device timezone; helper's `toPersianDigits` converts Persian/Arabic digits to Western digits, making behavior inconsistent.
- **Evidence / impact:** Incorrect/confusing dates across timezones and inconsistent calendar/digit presentation.
- **Recommended fix:** Centralize date/time formatting, explicitly bind Asia/Tehran and Solar Hijri, define digit policy, and add boundary tests for timezone/year transitions.

## Test execution and evidence limitations

- **Executed/inspected:** Target identity commands, clean-tree/diff checks, exact-SHA GitHub Actions metadata/logs, debug artifact download and hash, and read-only current-source inspection.
- **Target-CI tests with passing result:** Backend integration (`npm test` release chain) and Android `testDebugUnitTest`; Android `assembleDebug` succeeded.
- **Target-CI failure:** Both release-validation attempts stopped at required signing secrets; signing, release assembly, AAB creation, and release-artifact integrity were not achieved.
- **Not run locally:** Backend test suite, Gradle builds/tests, Docker build/image inspection, endpoint tests, PostgreSQL migration/constraint tests, Android install/UI/instrumentation, concurrency/race tests, performance measurements, crash/lifecycle tests, and security probes. The sandbox has no `adb`, no Docker CLI, no local backend dependencies or Gradle cache, and no connected test database/device.
- The current Android test definitions and backend CI suites do not close all findings above. Tests were not fabricated or inferred from source-only checks. No old branch, old APK, old database, old Docker image, or historical CI result was used as proof.

## Final certification

**TARGET STATUS: PASS** — the exact required repository, branch, commit, and clean source tree were verified.

**FINAL VERDICT: NOT RELEASE READY.**

Reason: (1) release validation is red twice on the exact target due missing stable signing credentials, with no signed release artifact; and (2) current-target source review identifies unresolved release-blocking security, data-integrity, reservation, financial, synchronization, and subscription defects. Runtime/device/backend verification remains incomplete and is explicitly **UNVERIFIED** where not executed. No release certification is issued.