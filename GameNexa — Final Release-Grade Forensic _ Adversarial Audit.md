# GameNexa — Final Release-Grade Forensic / Adversarial Audit

**Audit date/time:** 2026-10-02, approximately 23:21–23:23 +03:30  
**Tester:** Manus  
**Test environment:** Ubuntu 24.04 sandbox; repository clone; no Android SDK; no PostgreSQL service; no emulator/device; no VPS/runtime access  
**Repository:** `mkhas1374/GameNexa-release-2026`  
**Branch:** `main`  
**Exact commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`  
**Commit date:** `2026-10-02T22:26:34+03:30`  
**Commit subject:** `Release hardening: financial settlement and Android security`

> **Scope rule:** Only source-proven defects are marked `CONFIRMED`. Runtime behavior that could not be executed is marked `UNVERIFIED` or `BLOCKED`; it is not presented as a passing result.

## Version Freeze

| Item | Recorded value | Evidence / status |
|---|---|---|
| Repository | `mkhas1374/GameNexa-release-2026` | GitHub clone |
| Branch | `main` | `git branch --show-current` |
| Commit SHA | `a845c2e35b4678bf11b7ee746bd4d8d941602551` | `git rev-parse HEAD` |
| Commit date | `2026-10-02T22:26:34+03:30` | `git show -s --format=%cI` |
| Android applicationId | `com.MinmKhas.studio.GameNexa.wrtx` | `app/build.gradle.kts:18` |
| Android namespace | `com.example` | `app/build.gradle.kts:10` |
| versionCode | `4` | `gradle.properties:31` |
| versionName | `1.0.2` | `gradle.properties:32` |
| Backend package version | `1.0.0` | `backend/package.json` |
| Database schema version | PostgreSQL schema has no explicit migration/version marker; Android Room version `13` | `AppDatabase.kt:11-32`; PostgreSQL version **not declared** |
| Docker API image | `node:18-alpine` | `backend/Dockerfile:1` |
| Docker database image | `postgres:16` | `backend/docker-compose.yml:3` |
| Docker proxy image | `nginx:latest` | `backend/docker-compose.yml:22` |
| Docker Compose version | Compose Specification; no top-level `version` field | `backend/docker-compose.yml` |
| VPS runtime commit | **UNVERIFIED** | No VPS/runtime access |
| Test APK SHA256 | **UNVERIFIED / unavailable** | No APK produced; Android SDK unavailable |
| Working tree | Not clean in clone after audit | Only untracked audit artifact `GAMENEXA_RELEASE_AUDIT.md`; tracked source was not changed |

### Commit correspondence

The tracked Android, backend, test, Docker, schema, and configuration files inspected are from the exact commit above. No source fix was applied during this audit. The untracked prior report is an audit artifact, not part of the audited commit. GitHub-to-VPS/container/running-API correspondence could not be checked.

## Architecture Map

```text
Compose UI
  -> GameNetViewModel / LicenseViewModel
  -> GameNetRepository or SelfHostedManager
  -> Retrofit/OkHttp or raw OkHttp
  -> Express middleware and route
  -> PostgreSQL transaction/query
  -> JSON response
  -> Room / StateFlow / in-memory auth state
  -> Compose UI

GitHub commit
  -> GitHub Actions
  -> APK/AAB or Docker build
  -> Docker Compose
  -> VPS containers
  -> Nginx/TLS
  -> Running API
```

Relevant components are `GameNetViewModel.kt`, `GameNetRepository.kt`, `SelfHostedManager.kt`, `GameNetApi.kt`, `AppDatabase.kt`, `backend/server.js`, `backend/canonical_routes.js`, `backend/reservationService.js`, `backend/financialService.js`, `backend/schema.sql`, `backend/Dockerfile`, `backend/docker-compose.yml`, and `.github/workflows/*`.

## Test Execution Record

| Test / inspection | Result | Exact evidence |
|---|---|---|
| Git commit/branch/date | PASS | SHA and commit date above |
| Android/backend static contract check | PASS | `51 Retrofit contracts and 76 raw API paths have backend matches` |
| Deep static release audit | PASS | `deep release audit (53 invariants)` |
| Tenant-isolation static test | PASS | `tenant-isolation static checks passed` |
| Financial mock test | PASS | `Arithmetic is exact without floats, refund is exact` |
| Backend `npm test` | BLOCKED | `ECONNREFUSED ::1:5432` and `127.0.0.1:5432` in `backend/test_reservation.js` |
| `tests/cert_suite.py` | BLOCKED | `sqlite3.OperationalError: unable to open database file` |
| Android `./gradlew test` | BLOCKED | `SDK location not found`; no `ANDROID_HOME`/`local.properties` SDK path |
| Android emulator/device E2E | UNVERIFIED | No emulator/device available |
| PostgreSQL-backed integration | UNVERIFIED | No local PostgreSQL runtime available |
| VPS/GitHub/container consistency | UNVERIFIED | No VPS/runtime access |
| Performance/load | UNVERIFIED | No load environment or measurements |
| Migration/data-upgrade test | UNVERIFIED | No old/new database fixtures or live PostgreSQL |

Static tests that only inspect source are not treated as substitutes for runtime integration tests.

# A — RELEASE BLOCKERS

## RB-001 — Customer registration cannot use one compatible Android/backend contract

* **Finding ID:** BUG-001
* **Title:** Android Customer registration request/response contract is incompatible with the backend
* **Status:** `CONFIRMED`
* **Severity:** `HIGH`
* **Priority:** `P1`
* **Component:** Android / Backend / API contract
* **Environment:** Exact commit listed in Version Freeze; static source proof, runtime unavailable
* **Exact Commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`
* **File:** `app/src/main/java/com/example/data/network/GameNetApi.kt`; `app/src/main/java/com/example/ui/GameNetViewModel.kt`; `backend/server.js`
* **Exact Line:** `GameNetApi.kt:174-175, 260-266`; `GameNetViewModel.kt:5303-5345`; `server.js:251-271`
* **Function:** `registerUser(...)`
* **Endpoint:** `POST /api/auth/customer/register`

**Description:** Android production UI calls `GameNetApi.registerUser()` with `UserRegisterRequest`, containing `username`, `password`, `phone`, `email`, and `role`. The backend route requires `phone_number`/`phoneNumber`, `manager_id`/`managerId`, `full_name`/`fullName`, and `password`. The backend response is `{success, token, customerId, customer}`, while Android expects a `UserAuthResponse` containing `token` and `user`, and the ViewModel requires a non-null parsed user with an ID.

**Expected:** Android request serialization, backend validation, backend response, and Android response parsing must be the same contract.

**Actual:** The Android request omits required backend fields and uses a different registration shape. A request from the shown DTO fails backend validation with HTTP 400. A backend-success-shaped response still does not match the Android success condition.

**Reproduction:** Reach the registration flow at `GameNetViewModel.kt:5303`; serialize `UserRegisterRequest`; send it to the shown endpoint. The body has no `manager_id` or `full_name`, so the validation at `server.js:255-258` rejects it. If manually corrected, the response still contains `customerId/customer`, not the expected `user` object.

**Evidence:** Exact DTO and call at `GameNetApi.kt:174-175, 260-266` and `GameNetViewModel.kt:5338-5349`; exact required backend fields and response at `server.js:251-271`.

**Root Cause:** A generic/operator registration DTO remains wired to the customer-registration endpoint while a separate `SelfHostedManager.registerCustomer()` flow uses the backend-shaped fields.

**Security Impact:** Registration ownership and manager binding are inconsistent across code paths. No backend authorization bypass is claimed.

**Business Impact:** Customer registration fails or is interpreted as failed; retries and support/manual recovery may create duplicate attempts.

**Data Integrity Impact:** A backend-created customer could be treated as unsuccessful by Android if response parsing does not recognize it.

**Affected Users:** Customers using the `GameNetViewModel.registerUser()` UI path and managers relying on that registration flow.

**Recommended Fix:** Choose one canonical DTO and response. Either route Android through `SelfHostedManager.registerCustomer()` or change the Retrofit DTO/backend together. Add a runtime serialization test asserting HTTP 201, token, customer ID, manager ID, and login.

**Regression Risk:** Customer login, manager scoping, password setup, and session restoration.

**Verification Test:** Isolated PostgreSQL API; register via exact Android serialization; assert 201 and parsed response; duplicate phone returns 409; login succeeds.

**Confidence:** High; source-proven. Runtime execution was blocked.

## RB-002 — Required runtime release evidence is unavailable

* **Finding ID:** BUG-004 / RELEASE-EVIDENCE-001
* **Title:** Backend and Android release gates were not executable in the audit environment
* **Status:** `CONFIRMED` as an evidence/release-gate blocker; not a product defect by itself
* **Severity:** `HIGH`
* **Priority:** `P1`
* **Component:** CI/CD / Release validation
* **Environment:** Audit sandbox
* **Exact Commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`
* **File:** `backend/test_reservation.js`; `.github/workflows/backend-integration.yml`; `app/build.gradle.kts` / Gradle execution environment
* **Exact Line:** Runtime command output recorded above; workflow `.github/workflows/backend-integration.yml:17-69`
* **Function:** Backend release suite and Android Gradle test task
* **Endpoint:** API runtime not started

**Description:** The repository test suite cannot be treated as a release pass from this environment: backend runtime tests attempted PostgreSQL and failed to connect; Gradle failed before tests because Android SDK location was unavailable. No emulator/device or VPS comparison was available.

**Expected:** Release evidence should include an isolated PostgreSQL API run, Android compile/test, device or emulator E2E, and artifact identity.

**Actual:** Static checks passed, but runtime and device evidence are absent.

**Reproduction:** Run `backend/npm test`: first database-backed reservation test fails with `ECONNREFUSED` on port 5432. Run `./gradlew test`: Gradle reports `SDK location not found`.

**Evidence:** Exact command outputs from the audit execution; workflow provisions PostgreSQL in CI but that workflow execution was not available here.

**Root Cause:** Missing audit dependencies/runtime access, plus CI coverage does not include a full Android-to-live-API E2E matrix.

**Security Impact:** Expired-token, IDOR, replay, rate-limit, and production configuration behavior are not runtime-proven here.

**Business Impact:** Financial, reservation, offline, and lifecycle claims cannot be released on static evidence alone.

**Data Integrity Impact:** Database transaction and concurrency outcomes remain unverified.

**Affected Users:** All production users if an untested integration or deployment defect exists.

**Recommended Fix:** Run the exact commit in isolated CI with PostgreSQL, Android SDK, emulator/device, API startup, database assertions, concurrency tests, and retained APK/AAB hashes. Compare runtime commit hashes with GitHub.

**Regression Risk:** Any fix to contracts, migrations, transaction ordering, or offline behavior.

**Verification Test:** Full matrix in the final release matrix below, with all artifacts tied to the exact SHA.

**Confidence:** Certain as a release-evidence blocker; it does not assert that every unexecuted behavior fails.

# B — CONFIRMED BUGS

## BUG-002 — Customer archive is not reconciled out of local Room state

* **Finding ID:** BUG-002
* **Title:** Server-absent/archived customers remain in Android local active data
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P1`
* **Component:** Android sync / Offline / Data integrity
* **Environment:** Exact commit; source proof, runtime unavailable
* **Exact Commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`
* **File:** `app/src/main/java/com/example/data/GameNetRepository.kt`; `backend/canonical_routes.js`
* **Exact Line:** `GameNetRepository.kt:271-287`; `canonical_routes.js:106-109`
* **Function:** `syncAllWithServer()`
* **Endpoint:** `GET /api/v1/manager/customers`

**Description:** The server excludes archived customers. Android only upserts customers returned by the server and does not reconcile local server-backed customers missing from a non-empty authoritative result.

**Expected:** A customer archived on the server becomes locally hidden/tombstoned and cannot be selected for new active operations, while unsynced local records remain in a separate pending state.

**Actual:** The local row remains in Room and remains available to code reading `getAllCustomersLocal()`.

**Reproduction:** Keep customer A in Room; archive A through `DELETE /api/v1/manager/customers/:id`; run `syncAllWithServer()`; the response omits A and the shown loop never processes A.

**Evidence:** `GameNetRepository.kt:274-287` processes only `remoteCustomers`; `canonical_routes.js:106-109` filters archived records.

**Root Cause:** Upsert-only sync has no tombstone or set-difference phase.

**Security Impact:** Backend canonical routes still reject archived customers; a backend authorization bypass is not proven. Stale local state increases the risk of sending invalid or unintended operations.

**Business Impact:** Archived customers may remain visible/selectable and create support and operational confusion.

**Data Integrity Impact:** Android and server customer sets diverge; later local-to-server sync may attempt to resurrect stale data.

**Affected Users:** Managers operating offline or after reconnect; archived customers.

**Recommended Fix:** Add server-backed tombstones or authoritative sync generations. Mark missing rows archived/hidden and keep genuinely unsynced local records in an outbox.

**Regression Risk:** Customer deletion, history retention, local pending creates, and manager switching.

**Verification Test:** Archive remotely, sync, assert active local query excludes the customer, history remains, and start rejects locally and server-side.

**Confidence:** High; source-proven. Runtime execution blocked.

## BUG-003 — Online station read returns stale Room state without revalidation

* **Finding ID:** BUG-003
* **Title:** Local station state wins even when sync mode is online
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P2`
* **Component:** Android station state / Offline-online behavior
* **Environment:** Exact commit; source proof, runtime unavailable
* **Exact Commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`
* **File:** `app/src/main/java/com/example/data/GameNetRepository.kt`
* **Exact Line:** `316-330`
* **Function:** `getStationStateById(id: Int)`
* **Endpoint:** `GET /api/v1/manager/stations` only when local row is absent

**Description:** The function returns the local Room row immediately. It only queries the server if no local row exists.

**Expected:** In online/sync mode, authoritative station state should be refreshed according to a freshness/ETag policy; local data should be an explicit offline fallback.

**Actual:** Disabled, changed, or otherwise stale server station state can be hidden by an existing local row until another broad sync occurs.

**Reproduction:** Have a local station row; change its server state; call `getStationStateById()` in sync mode. The first `if (local != null) return local` prevents network validation.

**Evidence:** `GameNetRepository.kt:316-330`.

**Root Cause:** Local-first lookup lacks freshness semantics.

**Security Impact:** Backend start still rechecks ownership/status/pricing, so a backend authorization bypass is not proven.

**Business Impact:** UI may show stale availability/configuration and allow a flow that later fails with a server conflict.

**Data Integrity Impact:** Local operational state diverges from server state.

**Affected Users:** Managers using online mode after station configuration changes.

**Recommended Fix:** Use server-authoritative reads when online, or attach explicit freshness and stale-state UI. Keep local-first only in verified offline mode.

**Regression Risk:** Offline operation, UI responsiveness, network load, and station configuration refresh.

**Verification Test:** Change station server state, invoke the read path online and offline, assert online freshness and offline fallback behavior.

**Confidence:** High; source-proven. Runtime impact unverified.

## BUG-005 — Customer password is retained in in-memory Customer/UI state after authentication

* **Finding ID:** BUG-005
* **Title:** Customer password copied into the authenticated Customer object and state
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P1`
* **Component:** Android privacy / authentication state
* **Environment:** Exact commit; source proof, runtime unavailable
* **Exact Commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`
* **File:** `app/src/main/java/com/example/data/network/SelfHostedManager.kt`; `app/src/main/java/com/example/data/Entities.kt`; `app/src/main/java/com/example/data/AppDatabase.kt`
* **Exact Line:** `SelfHostedManager.kt:383-440, 466-512`; `Entities.kt:177-199`; `AppDatabase.kt:88-93`
* **Function:** `loginCustomer(...)`, `registerCustomer(...)`
* **Endpoint:** `POST /api/auth/customer/login`; `POST /api/auth/customer/register`

**Description:** The code correctly clears the Room password column in migration 12→13 and usually inserts local customers with `password = ""`, but the authentication paths explicitly copy the plaintext password into the returned `Customer` object:

```kotlin
val cust = parseCustomerObject(JSONObject(profileBody)).copy(password = cleanPass)
...
val newCust = parseCustomerObject(cObj).copy(password = passwordText.trim())
```

`Customer` is used as application/state data and `_currentLoggedInCustomer` is assigned these objects.

**Expected:** Passwords should be used only to construct the authentication request and then immediately discarded; authenticated profile/customer state should never contain the plaintext password.

**Actual:** Plaintext customer passwords remain in the in-memory `Customer` model and current-customer state after login/registration. The model also has a password field, and some manager upsert paths conditionally send it.

**Reproduction:** Call `loginCustomer()` or `registerCustomer()` with a password; inspect the returned `Customer` and `_currentLoggedInCustomer`; `password` equals the supplied plaintext value.

**Evidence:** `SelfHostedManager.kt:436`, `SelfHostedManager.kt:501`, `Entities.kt:182`, and the state assignments at `SelfHostedManager.kt:438-439` and `506-510`.

**Root Cause:** Authentication DTO/state and customer profile/entity were combined; the code only removed password persistence from Room, not from memory/state.

**Security Impact:** Any memory inspection, heap dump, crash/debug inspection, or accidental logging/serialization of the Customer object can expose the password. This audit did not claim that a crash report or production log definitely contains it.

**Business Impact:** Credential reuse risk for customers and increased incident severity if the app process or diagnostics are compromised.

**Data Integrity Impact:** No direct financial mutation is proven; credential exposure can enable unauthorized login where the password is reused.

**Affected Users:** Customers authenticating through the Android app.

**Recommended Fix:** Remove `password` from the domain `Customer` entity and all authenticated profile objects. Use dedicated `CustomerLoginRequest` and `CustomerRegistrationRequest` DTOs, never copy request passwords into response/state objects, and add a static/runtime assertion that customer profile serialization contains no password field.

**Regression Risk:** Customer creation/edit flows, local Room migration, login UI state, and manager-created customer password setup.

**Verification Test:** Login/register, inspect returned/state objects and network logs, force a crash/debug dump in a controlled test, and assert no plaintext password remains outside the request-local variable.

**Confidence:** High; source-proven. Runtime exposure path not executed.

## BUG-006 — Docker deployment is not reproducible from the audited commit

* **Finding ID:** BUG-006
* **Title:** Mutable proxy image and non-deterministic dependency installation weaken release identity
* **Status:** `CONFIRMED` as a deployment/reproducibility defect
* **Severity:** `MEDIUM`
* **Priority:** `P1`
* **Component:** Docker / Deployment / Supply-chain reproducibility
* **Environment:** Exact commit; source proof, deployment runtime unavailable
* **Exact Commit:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`
* **File:** `backend/Dockerfile`; `backend/docker-compose.yml`
* **Exact Line:** `Dockerfile:1-6`; `docker-compose.yml:22`
* **Function:** Docker build / Compose service startup
* **Endpoint:** Running API deployment

**Description:** The API image uses `RUN npm install` rather than `npm ci`, and Compose uses `nginx:latest`. The exact Git commit therefore does not uniquely determine the dependency/proxy image set.

**Expected:** Release deployment should use a lock-enforcing install and immutable, reviewed image digests or fixed versions.

**Actual:** `npm install` may resolve/update dependency state at build time, while `nginx:latest` can change independently of the audited commit.

**Reproduction:** Build the same commit at different times or on different builders; dependency resolution and the `latest` proxy image are not pinned by the commit.

**Evidence:** `backend/Dockerfile:1-6` contains `FROM node:18-alpine`, `COPY package*.json ./`, `RUN npm install`; `backend/docker-compose.yml:22` contains `image: nginx:latest`.

**Root Cause:** Deployment files do not pin all build/runtime inputs.

**Security Impact:** Supply-chain drift can introduce unreviewed dependency or proxy changes.

**Business Impact:** A tested artifact may not be the artifact deployed later.

**Data Integrity Impact:** No direct data mutation proven; runtime behavior can differ between builds.

**Affected Users:** All production users if a changed dependency/proxy affects API behavior.

**Recommended Fix:** Use `npm ci --omit=dev`, pin Node/Postgres/Nginx versions or digests, generate SBOM/checksums, and deploy immutable image tags tied to the Git SHA.

**Regression Risk:** Native bcrypt build, Node compatibility, TLS/proxy configuration, and CI cache behavior.

**Verification Test:** Build twice from the same SHA, compare image digests and dependency tree, then verify running container labels and file hashes.

**Confidence:** High; deployment-file defect proven. Actual production divergence unverified.

# C — SECURITY FINDINGS

## Confirmed security findings

1. **BUG-005 — MEDIUM/P1:** Plaintext Customer password retained in in-memory domain/state objects after login/register.
2. **Authorization runtime coverage is not a confirmed weakness:** static source contains manager/customer JWT separation and tenant checks, but live IDOR/replay tests were unavailable. Classified `UNVERIFIED`, not `CONFIRMED`.
3. **Trial identity forgery is not a confirmed weakness:** client-supplied device fields exist, but runtime abuse impact was not proven. Classified `UNVERIFIED`.
4. **No hardcoded production JWT secret or production password was proven in the inspected tracked source.** CI-only test secrets are present in workflow configuration and are scoped to isolated CI; this is not reported as a production secret leak.

## Source-proven positive controls

* Manager/customer token separation and interceptor routing are covered by the deep static audit.
* JWT verification and bcrypt use are present in `backend/server.js`.
* Customer and manager middleware check role/identity/manager scope.
* Production HTTP logging redacts Authorization, signature, and manager headers in `GameNetApi.kt:917-922`.
* Canonical station/session routes use manager-derived scope and transactional/advisory locking.

# D — UNVERIFIED

The following were not proven because runtime dependencies or external environments were unavailable:

1. VPS commit, running container commit, and running API source consistency.
2. Test APK SHA256, release artifact identity, and release-signing output.
3. Android compile, JVM tests, emulator/device UI, rotation, process death, ANR, and crash recovery.
4. Live PostgreSQL schema creation, constraints, indexes, triggers, migrations, and query behavior.
5. Manager A/B and Customer A/B runtime IDOR tests across GET/POST/PUT/PATCH/DELETE.
6. Expired, malformed, replayed, revoked, and cross-user JWT behavior at runtime.
7. Station start combinations, concurrent starts with 2/5/10/50 requests, and active-customer races.
8. Settlement retry, duplicate settlement, server restart, app kill, and financial idempotency.
9. Offline start/reconcile/settle when a request is received but the client sees failure.
10. Reservation, VIP supersession, payment approval, refund, wallet, GN, and LP concurrent behavior.
11. Device-limit concurrency at 10–100 requests.
12. Trial reinstall/device identity forgery and clock manipulation.
13. HTTP 400/401/403/404/409/422/429/500/502/503/timeout classification in Android UI.
14. Password presence in crash output, clipboard, heap dump, or production diagnostics beyond the source-proven state retention.
15. Persian calendar/number formatting across every screen and device size.
16. Docker fresh deployment, restart behavior, TLS/Nginx runtime, health checks, and volume recovery.
17. Old database → new backend and new database → new backend migration behavior.
18. Performance at 1/10/50/100 users, lock contention, slow queries, CPU, memory, and connection pool behavior.
19. Actual GitHub Actions execution for this exact commit.

# E — FALSE POSITIVES

No previous finding was demonstrably false based on the second-pass evidence.

The following earlier claims were **not** treated as false positives:

* Registration contract mismatch remains confirmed.
* Customer archive reconciliation gap remains confirmed.
* Local-first station state remains confirmed as a stale-state design defect.
* Runtime evidence gap remains real, although it is classified as a release/evidence blocker rather than a product behavior bug.

# F — FIXED FINDINGS

No previous finding was proven `CONFIRMED FIXED` in this audit because no runtime comparison or post-fix commit was available.

### Partial/related hardening observed

* Android Room migration `12→13` clears the Customer password column: `AppDatabase.kt:88-93`. This is a partial mitigation, but it does not fix BUG-005 because login/register still copy plaintext passwords into in-memory Customer/state objects.
* Static tests assert separate manager/customer tokens and server-authoritative station start. These are positive controls, not proof that all runtime paths pass.

# G — TOP 10 NEXT ATTACK VECTORS

These are prioritized attack/test targets, not confirmed defects:

1. Run Android registration through the exact Retrofit path and compare serialized request/response bytes.
2. Execute Manager A/B and Customer A/B IDOR tests against every canonical and legacy-looking route.
3. Fire 50–100 concurrent station starts, settlement calls, payment approvals, device registrations, and trial activations.
4. Simulate lost-response requests: server commits, client times out, reconnect replay, and settlement.
5. Test customer archive/delete while the Android client is offline, then reconnect and attempt selection/start/upsert.
6. Inspect Customer object, ViewModel state, logs, crash output, and heap after login/register for password retention.
7. Test old Room versions and old PostgreSQL data against current migrations and schema.
8. Build Docker images repeatedly and compare dependency trees, image digests, and runtime file hashes.
9. Change device/phone clocks across trial, entitlement, reservation, payment, and settlement boundaries.
10. Run the full UI matrix: process death, rotation, cutout, RTL/LTR, Persian digits, keyboard, network errors, and rapid taps.

# Final Release Matrix

| Area | Tested | Passed | Failed | Unverified |
|---|---:|---:|---:|---:|
| Android | Static only | Static contract/deep audit | None runtime-proven | Build, device, UI, lifecycle, crash recovery |
| Authentication | Static only | Token separation assertions | None runtime-proven | Login/logout/expiry/replay/password flows |
| Authorization | Static only | Tenant-scope source checks | None runtime-proven | Full IDOR matrix |
| Station | Static only | Server-side ownership/pricing/lock code | BUG-003 stale local read | Runtime station matrix |
| Session | Static only | Transaction/claim code | None runtime-proven | Start/stop/recovery/concurrency |
| Settlement | Static/mock only | Financial mock; source locks | None runtime-proven | Retry/double settlement/DB state |
| Invoice | Static only | Schema uniqueness/source paths | None runtime-proven | Guest/retry/amount validation |
| Reservation | Static only | Static checks/schema exclusion | None runtime-proven | Live PostgreSQL and concurrency |
| VIP | Static only | Static supersession assertions | None runtime-proven | Concurrent refund/retry |
| Wallet | Static only | Source/idempotency indicators | None runtime-proven | Live ledger/balance behavior |
| Payment | Static/mock only | Arithmetic mock | None runtime-proven | Approval bounds/replay |
| GN/LP | Static only | Source ledger structures | None runtime-proven | Exact end-to-end contract/ledger |
| Trial | Static only | Static source assertions | None runtime-proven | Reinstall/device forgery/clock |
| Device | Static only | Transactional source code | None runtime-proven | 10–100 concurrent requests |
| Super Manager | Static only | Route/static checks | None runtime-proven | Unauthorized operation matrix |
| Security | Static only | JWT/bcrypt/header redaction source | BUG-005 password state retention | Runtime attack and privacy inspection |
| Database | Schema inspection only | Constraints/source reviewed | None runtime-proven | Load, migration, locks, cascades |
| API Contract | Static | 51/51 Retrofit, 76 raw path matches | BUG-001 payload/response mismatch | Full field/type/status runtime |
| Offline | Static only | Offline-start source protections | BUG-002/003 stale-state risks | Reconnect/replay runtime |
| Concurrency | No live run | Static advisory locks present | None runtime-proven | 2–100 request tests |
| CI/CD | Workflow inspection | Workflow definitions present | RB-002 evidence unavailable here; BUG-006 reproducibility | Actual run/artifact/signing hashes |
| VPS | No | None | None proven | GitHub/VPS/container/API equality |
| UI | Source inventory | Cutout/static assertions | None runtime-proven | Every screen/button/form |
| Performance | No | None | None proven | 1/10/50/100-user measurements |

# Final Summary

## A — RELEASE BLOCKERS

1. **BUG-001 / RB-001 — HIGH/P1:** Android Customer registration contract is incompatible with the backend.
2. **RB-002 — HIGH/P1:** Required backend runtime, Android build/test, device E2E, artifact hash, and VPS consistency evidence is absent in this audit environment.
3. **BUG-006 — MEDIUM/P1:** Docker deployment inputs are not fully reproducible because of `npm install` and `nginx:latest`.

## B — CONFIRMED BUGS

1. **BUG-001 — HIGH/P1:** Registration request/response mismatch.
2. **BUG-002 — MEDIUM/P1:** Archived/server-absent Customers remain in local active data.
3. **BUG-003 — MEDIUM/P2:** Online station lookup returns local state without revalidation.
4. **BUG-005 — MEDIUM/P1:** Plaintext Customer password retained in Customer/state objects.
5. **BUG-006 — MEDIUM/P1:** Non-reproducible Docker dependency/proxy inputs.

## C — SECURITY FINDINGS

1. **BUG-005 — CONFIRMED, MEDIUM/P1:** In-memory plaintext Customer password retention.
2. No confirmed backend IDOR, JWT forgery, trial bypass, SQL injection, or production-secret exposure was established in this audit. Those areas remain partly unverified at runtime.

## D — UNVERIFIED

VPS/runtime consistency; Android build/device behavior; PostgreSQL runtime; live API error/auth behavior; concurrency; offline replay; financial/reservation/ledger behavior; migrations; performance; full UI and crash recovery; actual CI execution and artifact hashes.

## E — FALSE POSITIVES

None demonstrated.

## F — FIXED FINDINGS

None proven fully fixed. Room password-column clearing is a partial mitigation only and does not fix BUG-005.

## G — TOP 10 NEXT ATTACK VECTORS

See the prioritized list above.

# Final Release Decision

## NOT READY

The build is **NOT READY** for production release.

### Confirmed Critical/High Findings

* One confirmed HIGH/P1 product integration defect: BUG-001.
* One HIGH/P1 release-evidence blocker: RB-002.
* No confirmed CRITICAL finding.

### Confirmed Medium/Low Findings

* BUG-002 — MEDIUM/P1 customer sync/tombstone gap.
* BUG-003 — MEDIUM/P2 stale online station read.
* BUG-005 — MEDIUM/P1 plaintext password in memory/state.
* BUG-006 — MEDIUM/P1 Docker reproducibility weakness.

### Unverified Areas

All live database, emulator/device, VPS, performance, migration, concurrency, replay, full financial, and full UI matrices listed in Section D.

### Security Blockers

BUG-005 must be fixed before release. Runtime IDOR/replay/device/trial attack evidence is also required before claiming security readiness.

### Financial Integrity Blockers

No financial defect was confirmed from the available source evidence, but PostgreSQL-backed settlement/payment/refund/wallet/GN/LP runtime and concurrency tests were not executed. Financial readiness is therefore unverified, not passed.

### Data Integrity Blockers

BUG-002 is confirmed. PostgreSQL schema load, migration, transaction, constraint, and recovery behavior remain unverified.

### Android/Backend Contract Blockers

BUG-001 is confirmed and blocks release until one canonical customer-registration contract is implemented and runtime-tested.

### CI/CD Blockers

RB-002 blocks release evidence in this audit run. BUG-006 prevents deterministic deployment identity until images and dependency installation are pinned.

### VPS/GitHub Integrity Blockers

VPS/runtime commit and container file hashes were unavailable; production consistency is unverified and must be checked before release.

## Required pre-release actions

1. Fix BUG-001 and add a serialized Android-to-backend registration contract test.
2. Remove password from Customer entities and all authenticated profile/state objects; retain it only in request-local DTOs.
3. Implement customer tombstone/authoritative reconciliation for BUG-002.
4. Add explicit online freshness/offline fallback semantics for BUG-003.
5. Pin Docker images and use lock-enforcing dependency installation for BUG-006.
6. Execute the isolated PostgreSQL backend suite and Android build/tests.
7. Run emulator/device E2E, concurrency, offline replay, financial idempotency, and cross-tenant authorization tests.
8. Compare exact hashes across GitHub, build artifacts, Docker images, VPS source, running container, and running API.

**Final verdict: NOT READY.**
