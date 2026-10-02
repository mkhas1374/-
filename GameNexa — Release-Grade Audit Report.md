# GameNexa — Release-Grade Audit Report

**Audit mode:** White-box source audit + repository-provided static checks + attempted test execution  
**Repository:** `mkhas1374/GameNexa-release-2026`  
**Branch:** `main`  
**Commit tested:** `a845c2e35b4678bf11b7ee746bd4d8d941602551`  
**Working tree:** clean after clone; no project files were changed, committed, pushed, merged, or deployed.  
**Audit date:** 2026-10-02

> This report intentionally does not claim that unexecuted runtime, database, emulator, device, VPS, or production flows passed. Every FAIL below has source or command evidence; unavailable runtime scenarios are listed separately as `UNVERIFIED` or `BLOCKED`.

## 1. Executive Summary

The inspected commit contains a substantial Android/Node/PostgreSQL implementation with transaction-oriented station/session and reservation code, tenant-scoped SQL queries, JWT/bcrypt authentication, idempotency keys, database constraints, Docker configuration, GitHub Actions, and a large repository test suite.

The current build is **not release-trusted** for two independent reasons:

1. **Confirmed production integration defect:** the Android registration flow sends a request and expects a response contract that does not match the backend customer-registration endpoint. The UI calls this path, so registration cannot complete against the backend as currently implemented.
2. **Release evidence is incomplete:** backend runtime tests could not run because no PostgreSQL service was available in the audit environment, and Android JVM tests could not run because the Android SDK location was unavailable. No real-device/emulator E2E run or VPS/runtime-to-commit comparison was possible.

Additional confirmed risks concern stale local customer state after server-side archival, local station state being returned without online revalidation, and CI coverage being materially narrower than the requested release-grade E2E scope.

## 2. Exact GitHub Commit Tested

```text
a845c2e35b4678bf11b7ee746bd4d8d941602551
```

Obtained directly with `git rev-parse HEAD` immediately after cloning `main` from GitHub.

## 3. Project Architecture / Inventory

### Repository structure

| Area | Evidence / inventory |
|---|---|
| Android module | `app/` |
| Kotlin source | 58 Kotlin files, including `MainActivity`, Compose UI, ViewModels, Room entities/DAOs, repository, network client |
| Android tests | `app/src/test`, `app/src/androidTest`; 6 test source files plus screenshot fixture |
| Backend | `backend/server.js`, `canonical_routes.js`, `financialService.js`, `reservationService.js`, `bootstrap.js` |
| Database | `backend/schema.sql` — PostgreSQL dump/schema, 2,789 lines |
| Backend tests | 12 JavaScript test files under `backend/` |
| Python audit tests | 11 files under `tests/` |
| Runtime | `backend/Dockerfile`, `backend/docker-compose.yml`, `backend/nginx.conf` |
| CI/CD | 4 workflows under `.github/workflows/` |
| Configuration | `.env.example`, `backend/.env.example`, Gradle version catalog, Gradle properties |
| Secrets references | `JWT_SECRET`, `SUPER_ADMIN_PASSWORD`, release keystore/password/alias secrets, payment/SMS/Bale variables |

### Android components identified

* Compose screens include manager/admin login, main/station screen, customer management, reservations, customer club, settings, statistics, subscription/trial, diagnostics, customer application, and welcome/auth flows.
* Main state/control layer: `GameNetViewModel.kt` and `LicenseViewModel.kt`.
* Persistence: `AppDatabase.kt`, `Dao.kt`, `Entities.kt`, Room entities for stations, products, orders, customers, history, reservations, ledgers, logs, settings, and referral/behavior data.
* Network: `GameNetApi.kt`, `NetworkClient`, `SelfHostedManager`, `NetworkLogger`, `RetryInterceptor`.
* Crypto/settings: `CryptoManager.kt`, encrypted settings calls from `GameNetViewModel`.

### Backend components identified

* Express app with CORS, JSON body limit, rate limiting, JWT/bcrypt authentication middleware, entitlement gate, diagnostics, authentication endpoints, session/station endpoints, invoices/payments, customer endpoints, reservation endpoints, subscription/trial routes, and canonical route registration.
* PostgreSQL schema includes managers, customers, stations, sessions, session participants/events/orders, invoices, payments, reservations, ledgers, wallet transactions, entitlement/device/trial tables, configuration revisions, audit logs, and constraints/indexes/triggers.

### Dependency flow

```text
Compose UI
  -> GameNetViewModel / LicenseViewModel
  -> GameNetRepository or SelfHostedManager
  -> Retrofit/OkHttp or raw OkHttp
  -> Express routes and auth middleware
  -> PostgreSQL transaction/query
  -> JSON response
  -> Room/StateFlow/local state
  -> Compose UI
```

## 4. Test Execution Record

| Check | Result | Evidence |
|---|---|---|
| GitHub clone / commit capture | PASS | `a845c2e35b4678bf11b7ee746bd4d8d941602551` |
| Backend static isolation test | PASS | `tenant-isolation static checks passed` |
| Backend financial mock test | PASS | `PASS: Arithmetic is exact without floats, refund is exact` |
| `tests/android_backend_contract_check.py` | PASS | `PASS: 51 Retrofit contracts and 76 raw API paths have backend matches` |
| `tests/deep_release_audit.py` | PASS | `PASS: deep release audit (53 invariants)` |
| Other lightweight Python scripts | Exit 0 | Several scripts have no output and were not treated as behavioral proof |
| `backend/npm test` | **BLOCKED/FAIL TO EXECUTE** | `ECONNREFUSED ::1:5432` and `127.0.0.1:5432` in `backend/test_reservation.js` |
| `tests/cert_suite.py` | **BLOCKED** | `sqlite3.OperationalError: unable to open database file` |
| `./gradlew test` | **BLOCKED** | `SDK location not found`; no `ANDROID_HOME` or `local.properties` SDK path |
| Real Android emulator/device E2E | UNVERIFIED | No emulator/device was available |
| VPS/container/API consistency | UNVERIFIED | No VPS/runtime access was available |

## 5. Confirmed Findings

## BUG-001 — Android customer registration contract is incompatible with backend

* **Severity:** HIGH
* **Priority:** P1
* **Component:** Android, API, Backend, Integration
* **Confidence:** Confirmed

### What is wrong

The Android registration UI calls `GameNetApi.registerUser()` with `UserRegisterRequest`, but the backend route `/api/auth/customer/register` requires a different payload and returns a different response shape.

### Evidence — Android request and call site

**FILE:** `app/src/main/java/com/example/data/network/GameNetApi.kt`  
**LINES:** 174-175, 260-266  
**FUNCTION:** `registerUser`, `UserRegisterRequest`  

```kotlin
@POST("api/auth/customer/register")
suspend fun registerUser(@Body body: UserRegisterRequest): UserAuthResponse

 data class UserRegisterRequest(
    @Json(name = "username") val username: String,
    @Json(name = "password") val password: String,
    @Json(name = "phone") val phone: String? = null,
    @Json(name = "email") val email: String? = null,
    @Json(name = "role") val role: String = "OPERATOR"
)
```

**FILE:** `app/src/main/java/com/example/ui/GameNetViewModel.kt`  
**LINES:** 5303-5345  
**FUNCTION:** `registerUser(...)`

```kotlin
val req = UserRegisterRequest(
    username = cleanUsername,
    password = cleanPassword,
    phone = cleanPhone,
    email = cleanEmail,
    role = "OPERATOR"
)
val response = api.registerUser(req)
```

### Evidence — backend expectation and response

**FILE:** `backend/server.js`  
**LINES:** 251-271  
**ROUTE:** `POST /api/auth/customer/register`

```javascript
const phoneNumber = String(req.body?.phone_number || req.body?.phoneNumber || '').trim();
const managerId = String(req.body?.manager_id || req.body?.managerId || '').trim();
const fullName = String(req.body?.full_name || req.body?.fullName || '').trim();
const password = typeof req.body?.password === 'string' ? req.body.password : '';
if (!phoneNumber || !managerId || !fullName || password.length < 4) {
    return res.status(400).json({ error: 'phone_number, manager_id, full_name and a password of at least 4 characters are required' });
}
...
res.status(201).json({ success: true, token, customerId: customer.id, customer });
```

### Reproduction

1. Open the Android registration flow, which reaches `GameNetViewModel.registerUser()`.
2. Send the serialized request produced by `UserRegisterRequest` to `POST /api/auth/customer/register`.
3. The body contains `username`, `password`, `phone`, `email`, and `role`; it does not contain `manager_id` or `full_name`.
4. Backend validation returns HTTP **400**.
5. Even if the request is manually changed to satisfy backend validation, the backend returns `customerId` and `customer`, while `UserAuthResponse` expects `token` and `user`; the success condition at `GameNetViewModel.kt:5349` requires a non-null `user` with a real ID.

### Expected behavior

The Android and backend must share one customer-registration contract: required fields, field names, ownership/manager binding, status codes, and response JSON must match.

### Actual behavior

The production Android registration path cannot satisfy the backend's required request contract and cannot parse the backend's successful response as the expected `user` object.

### Impact

Customer self-registration fails or is treated as a failed authentication even if the backend creates the customer. This can create support failures and, depending on retries/manual recovery, duplicate-registration attempts.

### Root cause

A generic/operator-oriented DTO and flow were left wired to a customer-registration backend route.

### Recommended fix

Choose one canonical contract and apply it to both sides. For the existing backend route, the Android DTO should carry at least `phone_number`, `manager_id`, `full_name`, and `password`; the response model should parse `customerId` and `customer` (or the backend should be changed to return the Android contract). Add a runtime contract test that sends the exact serialized Android JSON and asserts HTTP 201 plus the exact parsed response.

### Regression risk

Customer login, manager ownership, customer password setup, and customer session restoration may be affected. Test both manager-created and self-registration flows.

### Verification test

* Start an isolated PostgreSQL-backed API.
* Register a customer through the Android serialization path.
* Assert HTTP 201, token, customer ID, manager ID, and customer fields.
* Re-register the same phone within the same manager and assert HTTP 409.
* Log in with the returned customer credentials and assert HTTP 200.

## BUG-002 — Customer sync never removes or archives local customers absent from the server result

* **Severity:** MEDIUM
* **Priority:** P1
* **Component:** Android, Offline, Sync, Data Integrity
* **Confidence:** Confirmed (static code; runtime reproduction blocked)

### Evidence

**FILE:** `app/src/main/java/com/example/data/GameNetRepository.kt`  
**LINES:** 271-287  
**FUNCTION:** `syncAllWithServer()`

```kotlin
val remoteCustomers = api.getCustomers()
if (remoteCustomers.isNotEmpty()) {
    val localCustomers = customerDao.getAllList()
    for (remote in remoteCustomers) {
        val existing = localCustomers.find { it.phoneNumber == remote.phoneNumber || it.id == remote.id }
        if (existing != null) {
            customerDao.insert(remote.copy(id = existing.id))
        } else {
            customerDao.insert(remote)
        }
    }
}
```

**FILE:** `backend/canonical_routes.js`  
**LINES:** 106-109  
**ROUTE:** `GET /api/v1/manager/customers`

```javascript
SELECT * FROM customers
WHERE manager_id=$1
  AND COALESCE(description,'') NOT LIKE '[GAMENEX_ARCHIVED:%'
```

### Why this is a defect

The server intentionally excludes archived customers, but the Android reconciliation loop only upserts records returned by the server. It never marks, deletes, or archives local customers that are missing from a non-empty authoritative server list.

### Reproduction

1. Put customer A in Room.
2. Archive customer A on the server through `DELETE /api/v1/manager/customers/:id`.
3. Reconnect and run `syncAllWithServer()`.
4. The server response excludes A.
5. The Android loop processes only returned rows; A remains in Room.
6. Any UI reading `getAllCustomersLocal()` can continue to display or use A until another path removes it.

### Impact

Stale or archived customer data can remain visible locally, may be selected for new operations, and can be resurrected by later local-to-server upsert code. The server still rejects unauthorized/archived use in the canonical start path, so the primary proven impact is Android/server state divergence and stale UI, not a proven backend isolation bypass.

### Recommended fix

Use a server-authoritative reconciliation algorithm with a tenant/session generation or tombstone set. For the returned authoritative customer list, mark missing local server-backed records as archived/hidden, while preserving explicitly unsynced local records in a separate pending queue. Never use a normal upsert to resurrect a server-archived customer.

### Verification

Archive a customer remotely, reconnect, assert it disappears from active local queries, assert local history remains, and assert a new start attempt is rejected both locally and by the backend.

## BUG-003 — Online station lookup returns Room state before checking server authority

* **Severity:** MEDIUM
* **Priority:** P2
* **Component:** Android, Offline, Sync, UI, Data Integrity
* **Confidence:** Confirmed (static code; runtime impact unverified)

### Evidence

**FILE:** `app/src/main/java/com/example/data/GameNetRepository.kt`  
**LINES:** 316-330  
**FUNCTION:** `getStationStateById(...)`

```kotlin
suspend fun getStationStateById(id: Int): StationState? {
    val local = stationStateDao.getById(id)
    if (local != null) return local
    if (isSyncModeEnabled()) {
        try {
            val remote = getApi()?.getStations()?.find { it.id == id }
            if (remote != null) {
                stationStateDao.insert(remote)
                return remote
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    return null
}
```

### Why this is a defect

When a local row exists, the function exits without any online revalidation. A disabled station, changed console type, or server state change is therefore not reflected in this read path until a broader sync replaces the local row.

### Impact

The UI can display stale station availability/configuration and permit an operator to begin a flow that the server later rejects. This is not by itself a backend authorization bypass because the canonical backend start route rechecks station ownership, active status, console type, pricing, and active-session conflicts.

### Recommended fix

When sync mode is enabled and authentication is valid, fetch the authoritative station or use a freshness/ETag policy before returning local state. Keep local data as an explicit offline fallback only when the app is known to be offline, and expose stale/fallback state to the UI.

## BUG-004 — Release gate does not prove the requested runtime/E2E coverage

* **Severity:** HIGH
* **Priority:** P1
* **Component:** CI/CD, Integration, Android, Backend, Offline, Security
* **Confidence:** Confirmed as an evidence gap

### Evidence — CI scope

**FILE:** `.github/workflows/android-release-validation.yml`  
**LINES:** 45-55, 63-92

The release workflow runs the static contract audit, deep static audit, JS syntax checks, JVM unit tests, and release builds. It does not run a real Android emulator/device E2E suite, authenticated API integration against a PostgreSQL service, concurrency tests against a running backend, or offline/reconnect flows.

**FILE:** `.github/workflows/backend-integration.yml`  
**LINES:** 17-69

The backend integration workflow does start PostgreSQL and the API, but it runs the backend `npm test` suite only; it does not install or run the Android application against that API.

**FILE:** `RELEASE_AUDIT_STATUS.md`  
**LINES:** 18-34

The repository itself records that local Android compilation was not passed and that a real-device/emulator E2E run remains a prerequisite.

### Impact

A green static audit or JVM test result can coexist with broken Retrofit serialization, Room/UI state divergence, Android lifecycle failures, HTTP error handling problems, and backend/Android behavior mismatches.

### Recommended fix

Add a release-gated pipeline with:

1. PostgreSQL schema initialization.
2. API startup with isolated test secrets.
3. Android emulator/device execution.
4. Manager/customer login.
5. Start/participant/prepayment/order/settle/invoice/history.
6. Customer archival and login rejection.
7. Reservation/payment/approval/cancellation.
8. Offline start/reconnect/reconcile/settle.
9. Trial/subscription expiry and device-limit cases.
10. Manager A/B isolation and duplicate/concurrent request tests.

## 6. Backend Audit

### Positive, source-proven controls

* Manager JWT verification restricts algorithms to HS256 and checks role/id/manager identity: `backend/server.js:67-109`.
* Customer JWT checks role, customer ID, manager ID, and archived status: `backend/server.js:111-130`.
* Manager routes commonly derive tenant scope from `req.user`, not from a client-supplied manager ID: `backend/canonical_routes.js:6-8, 106-121`.
* Station starts are transaction-wrapped and serialized by manager/station advisory lock: `backend/server.js:910-915`.
* Online starts check station ownership, active status, customer ownership, active customer claims, canonical console type, and server-side pricing: `backend/server.js:939-998`.
* Settlement locks the session row and clamps the client end time to server time: `backend/server.js:1344-1377`.
* PostgreSQL has tenant-aware composite foreign keys for many financial/session relationships and idempotency uniqueness constraints: `backend/schema.sql:2225-2357`, `1427-1471`, `1563-1568`.
* Reservation overlap has a PostgreSQL exclusion constraint: `backend/schema.sql:1547-1551`.

### Remaining runtime limitations

No live API was started in this audit environment because PostgreSQL was unavailable. Therefore authentication/authorization, HTTP status behavior, concurrent requests, settlement outputs, and database transaction behavior were not runtime-proven here.

## 7. API Contract Audit

The repository contract checker reported:

```text
PASS: 51 Retrofit contracts and 76 raw API paths have backend matches
```

This proves path/method coverage as implemented by that repository checker; it does not prove payload, nullability, response-deserialization, status-code, or runtime semantics for every method. BUG-001 is a concrete counterexample in the registration request/response contract.

## 8. Database Audit

The schema contains substantial defensive structure:

* Primary keys, manager-scoped unique keys, payment/ledger idempotency keys, invoice uniqueness, and reservation overlap exclusion constraints.
* Session/customer/station/invoice composite foreign keys in the schema.
* `release_session_customer_claims()` trigger for settled/closed/cancelled sessions: `backend/schema.sql:50-64`.

The schema was not loaded into a live PostgreSQL instance in this environment, so actual migration/load success, constraint behavior, query plans, lock waits, and concurrent transaction outcomes remain unverified.

## 9. Security Audit

### Source-proven observations

* JWT and bcrypt are used in manager/customer authentication: `backend/server.js:196-243, 278-292`.
* Production Android logging redacts Authorization, signature, and manager headers in the Retrofit client: `GameNetApi.kt:917-922`.
* CORS is allow-list based and rejects unknown origins: `backend/server.js:12-20`.
* No hardcoded JWT secret or password was proven from the inspected source; secrets are referenced through environment variables and CI secrets.
* The server derives manager scope from the authenticated token in canonical routes and rejects mismatched `X-Manager-ID`: `backend/server.js:94-105`.

No destructive or unauthorized attack was run against any external/production system.

## 10. Offline / Online Audit

The source contains server-authoritative offline reconciliation for station starts with an idempotency key, advisory lock, active-station check, participant validation, and a bounded start-time window: `backend/server.js:1031-1098`.

However, full offline confidence is not proven because:

* Android runtime tests could not run.
* The Android repository has partial-sync behavior with broad catches and `printStackTrace()` in multiple paths, e.g. `GameNetRepository.kt:159-310`.
* Customer tombstone reconciliation is incomplete (BUG-002).
* Local reads can precede server reads (BUG-003).

## 11. Station / Session Audit

The canonical online start route has strong static protections: per-station advisory lock, active-session conflict detection, customer claims, transaction rollback, server-side pricing snapshot, and participant/customer ownership checks (`server.js:912-1021`).

Settlement similarly locks and handles duplicate settled sessions (`server.js:1344-1377`). Runtime proof of double tap, two stations, process death, reconnect, and concurrent settle was unavailable.

## 12. Customer Audit

* Server deletion is archival and transaction-wrapped: `canonical_routes.js:123-140`.
* Active-session deletion is rejected with HTTP 409: `canonical_routes.js:131-139`.
* Archived customers are excluded from canonical manager listing and customer login: `canonical_routes.js:107-109`; `server.js:284-292`.
* Android local reconciliation does not remove missing archived rows: BUG-002.
* Customer registration is broken at the Android/backend boundary: BUG-001.

## 13. Reservation Audit

Static source shows:

* Tenant-scoped customer/station checks and active entitlement checks in `reservationService.js:101-145, 225-243`.
* Duration, VIP tier, VIP time window, station capacity, and trial limits in `reservationService.js:148-223`.
* Database overlap exclusion constraint in `schema.sql:1547-1551`.
* Idempotency and payment approval logic in canonical routes.

Database-backed reservation tests could not execute because PostgreSQL was unavailable; no runtime PASS is claimed.

## 14. Buffet / Food Audit

The backend has session-order insertion with active-session locking, idempotency event detection, server-side product price lookup, target-customer membership validation, and transaction wrapping: `server.js:1101-1141`.

The Android repository also clears stale station orders during server station sync: `GameNetRepository.kt:235-251`.

The full UI/offline/reconnect/order-resurrection flow remains unverified.

## 15. GN / LP / Customer Club Audit

The source contains tenant-scoped ledgers and idempotency keys in the schema and invoice settlement loyalty issuance in `server.js:1276-1297`. The actual rate configuration, refunds, transfer behavior, tier transitions, referral cooldown/decay, and UI consistency require live database and behavioral tests; they are not claimed as PASS here.

## 16. Subscription / Trial Audit

Static source includes:

* Active entitlement gate: `server.js:133-192`.
* Manager device binding and max-device transaction lock at login: `server.js:209-240`.
* Trial start/status and subscription activation routes in `server.js` and `canonical_routes.js`.
* Android trial-local operation guard returning a synthetic HTTP 409 for non-authoritative operational endpoints: `GameNetApi.kt:935-951`.

Device reinstall, clear-data, multiple-device, clock manipulation, real subscription activation, and expiry were not runtime-proven.

## 17. Time / Clock Audit

* Server clock endpoint: `backend/server.js:300-308`.
* Settlement clamps supplied end time to `Date.now()`: `backend/server.js:1355-1356`.
* Offline start accepts only a bounded window around server time: `backend/server.js:1071-1074`.
* Reservation uses server-side `Date`/Tehran rules in the backend reservation engine: `reservationService.js:148-175`.

Phone clock manipulation and all billing boundary cases require runtime execution; no PASS is claimed.

## 18. UI Audit

Compose screens and buttons were inventoried at source level, but a complete click-by-click UI audit across screen sizes, rotations, process death, cutouts, landscape, Android versions, and accessibility was not executable without an Android runtime. The repository does explicitly configure cutout/edge-to-edge behavior in the deep static audit, but that is not a device rendering proof.

## 19. Logging / Diagnostics Audit

The deep static audit passed its assertions for a 150-entry log buffer, safe diagnostic route codes, server diagnostic authentication, body/header non-disclosure, and deduplicated visible counters. These are source/test assertions, not a live load or production logging test.

## 20. Performance Audit

No production-scale load, query-plan, memory, Compose recomposition, battery, or coroutine-leak test was run. The code includes a simple in-process route/IP rate limiter (`server.js:30-45`) and broad synchronous-looking database loops in some routes; production capacity remains unverified.

## 21. CI/CD Audit

### Proven from workflow files

* Android debug and release workflows use JDK 17 and Android SDK setup.
* Release workflow refuses missing release signing credentials: `.github/workflows/android-release-validation.yml:66-92`.
* Backend integration workflow provisions PostgreSQL 16, loads `schema.sql`, starts the API, and runs `npm test`: `.github/workflows/backend-integration.yml:17-69`.
* Workflow permissions are restricted to read-only contents except the explicit workflow-run deletion job.

### Gap

CI does not provide the requested full Android/backend/device E2E matrix. See BUG-004.

## 22. Complete Test Matrix

| ID | Feature | Scenario | Result | Severity | Priority | Evidence |
|---|---|---|---|---|---|---|
| TM-001 | Repository | Exact `main` commit captured | PASS | INFO | P3 | SHA recorded above |
| TM-002 | API | Android/backend path matching | PASS | INFO | P3 | Repository checker: 51/51, 76 raw paths |
| TM-003 | Static security/isolation | Repository static invariants | PASS | INFO | P3 | `deep_release_audit.py`: 53 invariants |
| TM-004 | Customer registration | Android request to backend route | FAIL | HIGH | P1 | BUG-001; exact DTO/route evidence |
| TM-005 | Customer archive sync | Archived server row disappears locally | FAIL | MEDIUM | P1 | BUG-002; repository only upserts returned rows |
| TM-006 | Station read | Online local state revalidation | FAIL | MEDIUM | P2 | BUG-003; early local return |
| TM-007 | Backend reservation runtime | PostgreSQL-backed suite | BLOCKED | HIGH | P1 | `ECONNREFUSED 127.0.0.1:5432` |
| TM-008 | Android JVM tests | Gradle test | BLOCKED | HIGH | P1 | Android SDK location unavailable |
| TM-009 | Android device E2E | Auth/session/offline/trial/UI | UNVERIFIED | HIGH | P1 | No emulator/device |
| TM-010 | VPS/runtime consistency | GitHub = VPS = container = API | UNVERIFIED | HIGH | P1 | No runtime access |
| TM-011 | Concurrent station start | Same station / customer | UNVERIFIED | HIGH | P1 | Static locks exist; no live execution |
| TM-012 | Duplicate settle/payment | Retry and idempotency runtime | UNVERIFIED | HIGH | P1 | Static code only |
| TM-013 | Reservation double booking | Concurrent reservation requests | UNVERIFIED | HIGH | P1 | DB constraint present; not loaded/executed |
| TM-014 | Customer login after archive | Server rejection | UNVERIFIED | MEDIUM | P2 | Static predicate present; runtime blocked |
| TM-015 | Subscription/trial | Expiry/reinstall/device limits | UNVERIFIED | HIGH | P1 | No runtime/device execution |

## 23. Complete Bug List

### CRITICAL

None confirmed from the available source/runtime evidence.

### HIGH

* **BUG-001** — Android customer registration request/response contract incompatible with backend.
* **BUG-004** — Release gate lacks proof of the required runtime/E2E matrix; backend and Android execution were blocked in this audit environment.

### MEDIUM

* **BUG-002** — Customer sync does not reconcile server-absent/archived customers locally.
* **BUG-003** — Online station lookup returns stale Room state without revalidation.

### LOW

No additional low-severity defect was promoted without stronger evidence.

## 24. Release Blockers

### BUG-001 — Customer registration is broken

* **Reason:** The UI sends fields the backend does not require/consume and omits required backend fields; the success response model also differs.
* **Evidence:** `GameNetApi.kt:174-175, 260-266`; `GameNetViewModel.kt:5303-5345`; `server.js:251-271`.
* **Impact:** Customer registration cannot complete reliably.
* **Required fix:** Canonicalize request and response DTOs and add a runtime contract test.
* **Verification:** Isolated API registration + login test through Android serialization.

### BUG-004 — No release-grade runtime evidence

* **Reason:** PostgreSQL-backed backend tests and Android tests did not execute in this audit environment; no device/E2E or VPS consistency test was available.
* **Evidence:** Exact command output in Section 4.
* **Impact:** Security, lifecycle, offline, concurrency, UI, and financial behavior remain unproven.
* **Required fix:** Run the isolated CI/device matrix described in BUG-004.
* **Verification:** All required jobs green with retained reports and exact commit SHA.

## 25. Unverified / Needs Manual Verification

1. Current VPS source/container/API commit equality.
2. Live PostgreSQL schema load and all backend tests.
3. Real Android compile/JVM tests with SDK installed.
4. Emulator/device installation and release-signed APK/AAB execution.
5. Manager and customer authentication against a running API.
6. Android rotation, process death, reconnect, background/foreground, and memory-pressure behavior.
7. All station/session combinations listed in the requested audit specification.
8. Double-tap/concurrent station start, concurrent customer start, duplicate settle, duplicate payment, and reservation race runtime behavior.
9. Full invoice, buffet, GN/LP, referral, reservation cancellation, subscription, trial, and device-limit behaviors.
10. HTTP 400/401/403/404/409/422/429/500/502/503/504 behavior at runtime and Android presentation of each status.
11. Production CORS, reverse proxy, TLS, Nginx, Docker restart, health checks, and secret provisioning.
12. Performance/load, memory, query plans, N+1 behavior, and production log retention.
13. Full visual UI behavior on notch/cutout, landscape, screen sizes, Android versions, and accessibility services.

## 26. Recommended Fix Plan

### Before any release candidate

1. Fix BUG-001 and add an executable Android-serialization-to-backend registration contract test.
2. Implement server-authoritative customer tombstones/reconciliation for BUG-002.
3. Define an explicit freshness/offline policy for station reads and fix BUG-003.
4. Provision PostgreSQL and Android SDK in the audit runner; rerun all blocked suites.
5. Add a device/emulator E2E job against an isolated PostgreSQL-backed API.
6. Execute authenticated Manager A/B isolation tests for every mutating route.
7. Execute concurrent start/settle/payment/reservation tests with database assertions.

### After P0/P1 closure

8. Add contract schemas or generated DTOs to prevent Android/backend drift.
9. Add explicit tombstone/outbox tables or sync cursors for offline customer/order/session reconciliation.
10. Retain CI test reports and the exact tested SHA as release artifacts.

## 27. Regression Test Plan

* **Registration:** valid registration, missing manager/name, duplicate phone, invalid password, response parsing, login after registration.
* **Tenant isolation:** Manager A token plus B IDs across GET/POST/PUT/PATCH/DELETE; expect 404/403 and no data mutation.
* **Customer archival:** archive online, reconnect, local hidden/tombstoned, login rejected, history preserved.
* **Station:** two concurrent starts on one station; same customer in two stations; stale station state; disabled station; console mismatch.
* **Offline:** offline start, reconnect replay, same idempotency key, timeout retry, event ordering, settle after reconciliation.
* **Financial:** duplicate payment/settle, partial payment, guest payer, prepayment, invoice uniqueness, GN/LP idempotency.
* **Reservation:** double booking, approval twice, payment deadline, cancellation boundaries, VIP supersession, refund idempotency.
* **Lifecycle/UI:** rotation, process death, background/foreground, logout/login, token expiry, notch/cutout, offline banner.
* **Release:** debug APK, release APK/AAB with stable signing, install/run on emulator and at least one physical Android device.

## 28. Final Release Readiness Assessment

**Assessment: NOT RELEASE-READY / NOT TRUSTED YET.**

This is not a claim that every backend control is defective. The repository contains several strong static protections and its static audit scripts passed. The decision is based on the confirmed registration integration defect plus the unresolved evidence gaps for PostgreSQL runtime, Android build/test, device E2E, concurrency, offline reconciliation, and deployment consistency.

# WHAT MUST BE FIXED BEFORE I WOULD TRUST THIS BUILD

1. Fix and runtime-test **BUG-001** customer registration contract incompatibility.
2. Fix **BUG-002** server-absent customer reconciliation so archived customers cannot remain active in local state.
3. Resolve or explicitly redesign **BUG-003** stale online station reads.
4. Execute the blocked PostgreSQL backend suite against the exact commit.
5. Execute Android Gradle tests/build with a known SDK and produce release artifacts.
6. Run real Manager/customer/session/reservation/payment/offline/trial E2E tests.
7. Run concurrent and cross-manager isolation tests against live PostgreSQL.
8. Verify GitHub commit, VPS source, running container, and running API are identical.

# WHAT IS ACTUALLY PROVEN

1. The exact GitHub `main` commit tested is `a845c2e35b4678bf11b7ee746bd4d8d941602551`.
2. The repository contains Android, backend, PostgreSQL schema, Docker/Nginx, CI, and test assets listed in the inventory.
3. The repository static contract checker passed its reported 51 Retrofit and 76 raw API path matches.
4. The repository deep static audit passed its reported 53 invariants.
5. JWT/bcrypt authentication code, entitlement gates, tenant-scoped canonical queries, advisory locks, settlement locking, and reservation overlap constraints are present in source.
6. The Android registration request/response mismatch is proven by the exact source comparison in BUG-001.
7. The customer-sync omission and local station early-return behavior are proven by the exact source in BUG-002 and BUG-003.
8. The backend test suite could not connect to PostgreSQL in this environment.
9. Android Gradle tests could not locate an Android SDK in this environment.

# WHAT COULD NOT BE VERIFIED

1. Any claim about production/VPS/container state.
2. Runtime correctness of the database schema and transaction behavior.
3. Android compilation, APK/AAB execution, UI rendering, lifecycle, and device behavior.
4. Runtime API status/error contracts.
5. End-to-end financial, reservation, subscription, trial, offline, and concurrency flows.
6. Performance, load capacity, memory, and production operational behavior.

