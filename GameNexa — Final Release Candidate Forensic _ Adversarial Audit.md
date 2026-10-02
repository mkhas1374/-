# GameNexa — Final Release Candidate Forensic / Adversarial Audit

**Repository:** `mkhas1374/GameNexa-release-2026`  
**Expected branch:** `hardening-release-20261002`  
**Audited commit:** `184336b66b36e1adf0ee897a669a87b317cf0a1e`  
**Commit date:** `2026-10-02T20:05:38+00:00`  
**Commit subject:** `Harden backend container reproducibility`  
**Audit environment:** Ubuntu 24.04 sandbox; no Android SDK; no Docker CLI; no PostgreSQL service; no emulator/device; no VPS access  
**Tester:** Manus  
**Audit date:** 2026-10-02

## Version Freeze and Integrity

The requested branch and commit were found on GitHub and fetched before testing:

```text
origin/hardening-release-20261002 -> 184336b66b36e1adf0ee897a669a87b317cf0a1e
```

The worktree was checked out at the exact expected commit. The worktree was detached at the expected SHA; the remote branch reference is `origin/hardening-release-20261002`. No tracked project source was modified during this audit.

| Item | Value / status |
|---|---|
| Android applicationId | `com.MinmKhas.studio.GameNexa.wrtx` |
| Android namespace | `com.example` |
| versionCode | `4` |
| versionName | `1.0.2` |
| Backend package version | `1.0.0` |
| Android Room database version | `13` |
| PostgreSQL schema version | No explicit schema-version/migration marker found |
| API Docker base | `node:22-alpine` |
| PostgreSQL Docker image | `postgres:16` |
| Nginx Docker image | `nginx:1.29-alpine` |
| Compose format | Compose Specification; no top-level version field |
| VPS runtime commit | `UNVERIFIED` — no VPS access |
| Test APK SHA256 | `UNVERIFIED` — no APK produced; Android SDK unavailable |
| Release signing | `UNVERIFIED` — CI secrets/runtime unavailable |

### RC file hashes

```text
643c72511cdc09955664378894f3cb356c69277392aa7443cf50c57118680be9  backend/server.js
08642f338b87ba755233a4f89cf6ec967fed67f1ccf5d44abefc25a7866f524e  backend/canonical_routes.js
c210233c8d487875cd4f00be54297955c96da3cf4b7060f21a15fb90ecb2d322  backend/reservationService.js
31106ea89494bece440888fd277f3d79fd6aa16a6bd836a9dabd2d621061cffc  backend/schema.sql
abd4d5e4750f4f5eb360c44e2e7d801269fa3ce20cc7f6f4b2fa403e5b0fc108  backend/Dockerfile
ad15d77bc323ba5b6935756039f009d9a00c3b36aa471e3add37c0d7be885ece  backend/docker-compose.yml
```

## Architecture Reviewed

```text
Compose UI
  -> GameNetViewModel / LicenseViewModel
  -> GameNetRepository or SelfHostedManager
  -> Retrofit/OkHttp or raw OkHttp
  -> Express middleware/routes
  -> PostgreSQL
  -> JSON response
  -> Room / StateFlow / in-memory auth state
  -> Compose UI

GitHub branch/commit
  -> GitHub Actions
  -> APK/AAB or Docker image
  -> Docker Compose
  -> VPS containers
  -> Nginx/TLS
  -> Running API
```

The inspected implementation includes Android Compose screens, Room entities/DAOs/migrations, Retrofit and raw OkHttp clients, Express routes, JWT/bcrypt authentication, PostgreSQL schema/constraints, Docker/Compose/Nginx, CI workflows, and static/runtime-oriented tests.

# Test Evidence

| Check | Result | Evidence |
|---|---|---|
| Exact expected branch/commit | PASS | `184336b66b36e1adf0ee897a669a87b317cf0a1e` |
| Android/backend contract checker | PASS | `51 Retrofit contracts and 76 raw API paths have backend matches` |
| Deep release audit | PASS | `deep release audit (53 invariants)` |
| Backend JS syntax | PASS | `npm run check`, all four files passed |
| Git diff whitespace check | PASS | `git diff --check`, exit 0 |
| Tenant isolation static check | PASS | `tenant-isolation static checks passed` |
| Python database/security/concurrency lightweight checks | Exit 0 | No live DB/runtime proof; treated as static/lightweight evidence |
| Backend dependency installation | PASS | `npm ci --ignore-scripts`, exit 0 |
| Backend full `npm test` | BLOCKED | PostgreSQL connection refused on `::1:5432` and `127.0.0.1:5432` |
| Android `./gradlew test` | BLOCKED | Android SDK location not found; no `ANDROID_HOME`/`local.properties` |
| Docker build/Compose validation | BLOCKED | `docker` command unavailable |
| Emulator/device E2E | UNVERIFIED | No emulator/device |
| VPS/runtime/hash comparison | UNVERIFIED | No VPS access |
| APK/AAB/signature/SHA256 | UNVERIFIED | No Android build artifact |

The first RC test invocation from `backend/` used incorrect relative paths for the two Python scripts and therefore returned “file not found”; this was not treated as a test result. The same scripts were rerun from the RC repository root and passed as recorded above.

# Previous Findings Re-audit

| Previous finding | Old status | Current status | Evidence |
|---|---|---|---|
| BUG-001 — Android/customer registration contract mismatch | CONFIRMED | **PARTIALLY FIXED** | `GameNetViewModel.registerUser()` now calls `SelfHostedManager.registerCustomer()` with backend-shaped fields; stale Retrofit method remains but is not referenced by the UI |
| BUG-002 — Archived customer remains in Room sync | CONFIRMED | **STILL PRESENT / INCOMPLETE FIX** | `CustomerDao.deleteCustomersMissingFromServer()` was added at `Dao.kt:131-132` but `syncAllWithServer()` at `GameNetRepository.kt:271-287` never calls it |
| BUG-003 — Online station read returns local state without revalidation | CONFIRMED | **STILL PRESENT** | `GameNetRepository.kt:316-329` still returns local row at line 318 before network read |
| BUG-004 — Release runtime evidence gap | CONFIRMED evidence blocker | **STILL PRESENT** | RC backend tests blocked by missing PostgreSQL; Android tests blocked by missing SDK; Docker unavailable; no E2E/VPS |
| BUG-005 — Customer password retained in memory/state | CONFIRMED | **STILL PRESENT** | `SelfHostedManager.kt:436` and `:501` still copy plaintext password into `Customer` objects and current-customer state |
| BUG-006 — Non-reproducible Docker inputs | CONFIRMED | **FIXED at source level** | RC uses `npm ci --omit=dev`, `node:22-alpine`, `nginx:1.29-alpine`, and `.dockerignore`; actual image build/digest remains unverified |

# CONFIRMED BUGS

## BUG-002-RC — Customer sync fix is incomplete and does not execute the added deletion method

* **Title:** Server-absent/archived customers remain in local Room state
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P1`
* **Commit:** `184336b66b36e1adf0ee897a669a87b317cf0a1e`
* **Component:** Android sync / Offline / Data integrity
* **File:** `app/src/main/java/com/example/data/GameNetRepository.kt`; `app/src/main/java/com/example/data/Dao.kt`
* **Exact Line:** `GameNetRepository.kt:271-287`; `Dao.kt:131-132`
* **Function:** `syncAllWithServer()`; `CustomerDao.deleteCustomersMissingFromServer(...)`
* **Endpoint:** `GET /api/v1/manager/customers`

**Description:** The RC adds a DAO method intended to delete local customers missing from the server, but the synchronization function still only upserts returned customers. The new method is not called.

**Expected:** After receiving an authoritative server customer list, server-backed local rows absent from that list should be hidden/tombstoned or removed, while unsynced local records should be preserved separately in an outbox.

**Actual:** `syncAllWithServer()` gets `remoteCustomers`, loops over returned rows, and inserts/replaces them. It performs no set-difference or tombstone operation. An archived customer remains in Room if it was already local.

**Reproduction:** Keep customer A in Room; archive A on the server; run `syncAllWithServer()`; the server omits A and the loop at `GameNetRepository.kt:275-284` never touches A. The added DAO deletion method is not invoked.

**Evidence:** `Dao.kt:131-132` adds `deleteCustomersMissingFromServer`; `GameNetRepository.kt:271-287` contains no call to it. The backend customer route excludes archived customers.

**Root Cause:** The remediation added an unused DAO operation but did not integrate authoritative set reconciliation into the sync transaction/flow.

**Security Impact:** Backend canonical authorization still appears to reject archived use; a backend IDOR bypass is not proven. Stale local selection increases risk of unintended requests.

**Financial Impact:** No direct charge was proven; stale customer selection can affect later session/order attribution.

**Data Integrity Impact:** Android and server customer sets diverge; stale local data can be re-upserted by local-to-cloud paths.

**User Impact:** Managers may see or select an archived customer after reconnect.

**Recommended Fix:** Call a safe reconciliation method only after a complete authoritative response, use tombstones rather than unconditional deletion where local history/pending creates matter, and handle the empty server list explicitly. Scope reconciliation by manager/account and test archive/reconnect.

**Regression Risk:** Pending local customer creation, manager switching, local history, and offline mode.

**Verification Test:** Archive customer remotely, sync, assert active local query excludes it; assert pending local customer is preserved; assert archived customer cannot start/order/login.

**Confidence:** High; source-proven.

## BUG-003-RC — Online station read remains local-first and stale

* **Title:** Online station lookup returns Room state without server revalidation
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P2`
* **Commit:** `184336b66b36e1adf0ee897a669a87b317cf0a1e`
* **Component:** Android station state / Offline-online behavior
* **File:** `app/src/main/java/com/example/data/GameNetRepository.kt`
* **Exact Line:** `316-329`
* **Function:** `getStationStateById(id: Int)`
* **Endpoint:** Station fetch is skipped whenever a local row exists

**Description:** The RC did not change the early local return.

**Expected:** Online mode should use a freshness policy or server-authoritative fetch; local state should be an explicit offline fallback.

**Actual:** `if (local != null) return local` prevents any online revalidation.

**Reproduction:** Change a station server-side, retain an old Room row, invoke the method in sync mode; it returns the old Room row.

**Evidence:** `GameNetRepository.kt:316-329`, specifically line 318.

**Root Cause:** No online freshness/authority policy exists for this read path.

**Security Impact:** Backend start checks remain authoritative, so no backend authorization bypass is claimed.

**Financial Impact:** A stale station price/configuration could cause a rejected or confusing start, but no incorrect charge was runtime-proven.

**Data Integrity Impact:** Local operational state diverges from server state.

**User Impact:** Manager sees stale station status/configuration.

**Recommended Fix:** Fetch/revalidate in online mode, or expose a clearly stale local state and use local-first only when the network is verified unavailable.

**Regression Risk:** Offline responsiveness and request volume.

**Verification Test:** Server-side station change followed by online/offline reads and UI assertion.

**Confidence:** High; source-proven.

## BUG-005-RC — Customer plaintext password remains in memory/state

* **Title:** Login/register copy the plaintext password into Customer objects
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P1`
* **Commit:** `184336b66b36e1adf0ee897a669a87b317cf0a1e`
* **Component:** Android privacy / authentication state
* **File:** `app/src/main/java/com/example/data/network/SelfHostedManager.kt`
* **Exact Line:** `436-439`, `501-510`
* **Function:** `loginCustomer(...)`; `registerCustomer(...)`
* **Endpoint:** Customer login/register

**Description:** The RC still executes:

```kotlin
val cust = parseCustomerObject(JSONObject(profileBody)).copy(password = cleanPass)
val newCust = parseCustomerObject(cObj).copy(password = passwordText.trim())
```

Those objects are assigned to `_currentLoggedInCustomer` and returned as authenticated customer state.

**Expected:** Password exists only in a request-local authentication DTO and is never placed in domain entities, profile responses, state flows, logs, or persisted models.

**Actual:** The plaintext password remains in the in-memory `Customer` model/state after successful login or registration. Room writes often clear it, but clearing persistence does not clear the state object.

**Reproduction:** Call customer login/register with a password; inspect returned `Customer` and `_currentLoggedInCustomer`; its `password` field equals the input.

**Evidence:** `SelfHostedManager.kt:436`, `:438`, `:501`, `:509`; `Entities.kt:177-199` defines `Customer.password`.

**Root Cause:** Authentication request data and customer profile/domain data share one model.

**Security Impact:** Heap inspection, accidental serialization, crash/debug inspection, or future logging of the Customer object can expose credentials. No production crash-log leak was claimed.

**Financial Impact:** No direct financial mutation.

**Data Integrity Impact:** No direct DB corruption; credential compromise could enable unauthorized operations.

**User Impact:** Customers whose passwords are reused elsewhere.

**Recommended Fix:** Remove password from the domain Customer entity and authenticated profile state. Use dedicated login/register DTOs, set state from a sanitized profile, and add a test that asserts the returned Customer cannot contain a password.

**Regression Risk:** Manager-created customer password setup and login UI.

**Verification Test:** Login/register, inspect state/object graph, network logs, crash diagnostics, and Room; assert no plaintext password outside request-local scope.

**Confidence:** High; source-proven.

## BUG-001-RC — Stale incompatible Retrofit registration method remains in dead code

* **Title:** Unused Retrofit registration endpoint retains an incompatible response contract
* **Status:** `LIKELY` as a maintainability/API-contract defect; not a reachable UI failure in the RC
* **Severity:** `LOW`
* **Priority:** `P2`
* **Commit:** `184336b66b36e1adf0ee897a669a87b317cf0a1e`
* **Component:** Android API contract / dead code
* **File:** `app/src/main/java/com/example/data/network/GameNetApi.kt`
* **Exact Line:** `174-175`, `260-265`
* **Function:** `GameNetApi.registerUser(...)`
* **Endpoint:** `POST /api/auth/customer/register`

**Description:** The RC corrected `UserRegisterRequest` field names to `phone_number`, `manager_id`, `full_name`, and `password`, but the UI no longer calls `api.registerUser`; `GameNetViewModel.registerUser()` now calls `SelfHostedManager.registerCustomer()`.

**Expected:** There should be one reachable registration implementation and one response contract, or the unused Retrofit method should be removed/fully aligned.

**Actual:** The stale method remains as an alternate API surface. It may now serialize the request correctly, but its `UserAuthResponse` contract and behavior are not proven against the customer response, and no current call site invokes it.

**Reproduction:** Search shows only the interface declaration for `api.registerUser`; current ViewModel registration calls `SelfHostedManager.registerCustomer()`.

**Evidence:** `GameNetViewModel.kt:5320` calls `SelfHostedManager.registerCustomer`; `GameNetApi.kt:174-175` retains the unused Retrofit method.

**Root Cause:** The functional fix bypassed rather than removed/consolidated the old Retrofit path.

**Security Impact:** No immediate reachable bypass proven.

**Financial Impact:** None proven.

**Data Integrity Impact:** A future caller could reintroduce contract drift.

**User Impact:** None in the current reachable UI path; future maintenance risk.

**Recommended Fix:** Remove the unused method/DTO or add a single integration test and use the same canonical implementation.

**Regression Risk:** Other hidden/reflection callers and API contract checks.

**Verification Test:** Compile, search call graph, serialize exact request, execute endpoint, parse exact response.

**Confidence:** Medium; dead-code reachability is source-proven, runtime response mismatch is unverified.

# SECURITY FINDINGS

## Confirmed

* **BUG-005-RC:** Plaintext customer password is retained in memory/state after authentication.

## Not confirmed

No confirmed backend IDOR, JWT forgery, SQL injection, trial bypass, cross-token substitution, or production-secret exposure was established from available evidence. Static checks for manager/customer token separation and tenant checks passed, but live adversarial requests were not executed.

# FINANCIAL FINDINGS

No new confirmed financial arithmetic or double-credit defect was found in the source evidence for this RC. The financial mock/static checks passed where executed. However, PostgreSQL-backed settlement, payment approval, wallet, GN/LP, refund, VIP supersession, and concurrent idempotency tests were blocked by the missing database and therefore remain `UNVERIFIED`, not passed.

# DATA INTEGRITY FINDINGS

* **BUG-002-RC:** Customer archive/tombstone reconciliation remains incomplete and can leave Android local data divergent from the server.
* **BUG-003-RC:** Station local state can remain stale in online mode.
* Database load/migration/constraint runtime behavior remains unverified.

# REGRESSION FINDINGS

No source-proven regression from `main` to the RC was found in the reviewed diff.

The RC changes improve customer registration routing and Docker reproducibility. The customer sync remediation is incomplete because it adds a DAO method without integrating it into the sync flow. The password privacy issue was not fixed by this RC.

# DOCKER / CI / ARTIFACT REVIEW

## Docker

**Improved and source-fixed relative to previous audit:**

* `backend/Dockerfile` now uses `FROM node:22-alpine`.
* Dependency installation is `npm ci --omit=dev`.
* Nginx is pinned to `nginx:1.29-alpine` rather than `latest`.
* `backend/.dockerignore` excludes `node_modules`, `.env`, `.env.*`, logs, backend test files, and parent `.git` pattern.

**Unverified:** Docker build, image digest, Compose startup, health checks, restart behavior, and secret/build-context behavior could not be executed because Docker is not installed in the audit environment.

## GitHub Actions

Workflow files define Android debug/release and backend integration jobs, including PostgreSQL service setup and release signing secret checks. Actual workflow execution for this commit was not available, so artifact and exit-code claims cannot be made from the local audit.

## Artifact

No APK/AAB was produced. Package/version declarations are source-proven, but APK SHA256, signing certificate, size, and commit correspondence are `UNVERIFIED`.

## VPS

No VPS/runtime access was available. No mismatch is claimed; consistency is simply `UNVERIFIED`.

# FINAL MATRIX

| Area | Tested | Passed | Failed | Unverified |
|---|---:|---|---|---|
| Android | Static | Contract/deep static checks | No runtime result | Build, UI, lifecycle, E2E |
| Authentication | Static | Token separation assertions | BUG-005 privacy state retention | Expiry/replay/revocation runtime |
| Authorization | Static | Tenant-scope source checks | None proven | Full IDOR matrix |
| Customer | Static/source | Registration reachable path corrected | BUG-002-RC; BUG-005-RC | Live create/archive/login |
| Manager | Static/source | Route/middleware present | None proven | Full CRUD/auth runtime |
| Super Manager | Static/source | Static route checks | None proven | Full unauthorized-operation runtime |
| Station | Static/source | Server authority/locks | BUG-003-RC stale local read | Full start matrix |
| Session | Static/source | Transaction/claim code | None proven | Runtime start/settle/recovery |
| Settlement | Mock/static | Arithmetic mock passed | None proven | PostgreSQL and retry/idempotency |
| Invoice | Static/schema | Guest nullable structure/source | None proven | Runtime totals/retry |
| Reservation | Static/source | Contract/static checks | None proven | PostgreSQL state/concurrency |
| VIP | Static/source | Static refund assertion | None proven | Concurrent supersession |
| Payment | Static/source | Bounds/idempotency code | None proven | Runtime approval/replay |
| Wallet | Static/source | Ledger structures | None proven | Runtime credit/refund |
| GN | Static/source | BUY_GN contract assertion | None proven | Runtime ledger/balance |
| LP | Static/source | Data structures | None proven | Runtime ledger/balance |
| Trial | Static/source | Static checks | None proven | Reinstall/device/clock |
| Device | Static/source | Transactional limit code | None proven | 10+ concurrent registrations |
| Security | Static/source | JWT/bcrypt/header redaction | BUG-005-RC | Live attack matrix |
| Database | Schema inspection | Constraints/source reviewed | None runtime-proven | PostgreSQL load/migrations/locks |
| API | Static | 51 Retrofit / 76 raw paths | Stale dead method risk | Full field/status runtime |
| Offline | Static/source | Offline reconciliation code exists | BUG-002-RC/BUG-003-RC risks | Network-loss replay |
| Concurrency | No live DB | Static locks present | None runtime-proven | 10–50+ request tests |
| Docker | Source | Pinning/`npm ci` fixed | None runtime-proven | Build/images/Compose |
| VPS | No | None | None proven | All hash comparisons |
| GitHub CI | Workflow inspection | Workflow definitions | None runtime-proven | Actual run/artifacts |
| UI | Source inventory | Static cutout assertions | None runtime-proven | All screens/buttons/forms |
| Performance | No | None | None proven | Load/latency/resource measurements |

# RELEASE BLOCKERS

1. **BUG-005-RC — MEDIUM/P1:** Plaintext Customer password remains in authenticated in-memory state. This is a confirmed privacy/security defect and must be removed before release.
2. **BUG-002-RC — MEDIUM/P1:** Customer archive synchronization fix is incomplete; the new DAO method is unused and stale customer records remain locally.
3. **BUG-004 evidence blocker:** PostgreSQL-backed backend tests, Android build/tests, Docker runtime, APK verification, emulator/device E2E, and VPS consistency were not proven for this RC.
4. **BUG-003-RC — MEDIUM/P2:** Online station state can be stale and should be resolved or explicitly designed as an offline-only fallback before operational release.

# CONFIRMED BUGS

* BUG-002-RC — Customer sync deletion/tombstone method added but not called.
* BUG-003-RC — Local-first station lookup remains stale in online mode.
* BUG-005-RC — Customer plaintext password retained in memory/state.

# SECURITY FINDINGS

* Confirmed: BUG-005-RC.
* No other security weakness was proven from the available evidence.
* Runtime IDOR, replay, token-expiry, device-identity, and trial-abuse tests remain unverified.

# FINANCIAL FINDINGS

No confirmed new financial defect. Financial runtime integrity remains unverified because PostgreSQL and concurrency tests did not execute.

# DATA INTEGRITY FINDINGS

* BUG-002-RC can leave local Customer data inconsistent with authoritative server state.
* BUG-003-RC can leave local Station state inconsistent with server state.
* Live database/migration/constraint behavior is unverified.

# REGRESSION FINDINGS

None confirmed. The RC contains source-level fixes for registration routing and Docker reproducibility. The sync fix is partial, not a regression classification.

# UNVERIFIED

VPS/GitHub/container identity; APK/AAB/signing/hash; PostgreSQL runtime; Android build/device/UI; live API status/error behavior; concurrency; financial idempotency; offline replay; trial/device abuse; migrations; performance; crash/recovery; and full screen/button/form coverage.

# FALSE POSITIVES

None established in this RC re-audit.

# FINAL COUNTS

* **Confirmed Critical/High:** `0` confirmed CRITICAL/HIGH product bugs; release evidence blocker remains HIGH operationally.
* **Confirmed Medium/Low:** `3` confirmed product bugs: two MEDIUM data/sync issues and one MEDIUM privacy issue. One additional LOW/P2 likely dead-code contract risk.
* **Unverified:** All runtime/device/VPS/database/performance areas listed above; the exact count is intentionally not reduced to a misleading percentage.

# RELEASE STATUS: NOT READY

The exact requested RC commit was found and audited. It is better than the previous `main` commit in two areas: the reachable customer registration path now uses the backend-shaped flow, and Docker build inputs are more reproducible. It is still **NOT READY** because:

1. A confirmed plaintext Customer password remains in memory/state.
2. Customer archive reconciliation is only partially implemented; the added DAO method is unused.
3. Online Station reads still return stale local state without revalidation.
4. PostgreSQL runtime, Android build/device tests, Docker execution, artifact verification, and VPS integrity remain unverified.

## Required fixes before release

1. Remove `password` from the Customer domain/entity/state model; use request-only authentication DTOs.
2. Integrate safe server-authoritative Customer tombstone reconciliation and test empty/non-empty server responses.
3. Resolve station online freshness semantics and add a stale-state test.
4. Run isolated PostgreSQL integration and all backend tests.
5. Run Android Gradle tests/build with SDK and produce APK/AAB SHA256/signature evidence.
6. Run emulator/device E2E including auth, session, settlement, invoice, reservation, offline replay, and crash recovery.
7. Run 10–50+ concurrent requests for station, settlement, payment, reservation, device, trial, wallet, GN, and VIP flows.
8. Verify GitHub SHA, build artifact SHA, Docker image digests, VPS source, running container files, and API runtime commit.

**Final verdict: NOT READY.**
