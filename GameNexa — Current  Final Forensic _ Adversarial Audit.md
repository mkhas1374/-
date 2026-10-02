# GameNexa — Current `main` Final Forensic / Adversarial Audit

**Repository:** `mkhas1374/GameNexa-release-2026`  
**Branch under audit:** `main` (checked out detached from `origin/main`)  
**Exact commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`  
**Commit date:** `2026-10-03T00:21:39+03:30`  
**Commit subject:** `Use single-source app versioning and enforce release lint`  
**Audit environment:** Ubuntu 24.04 sandbox; no Android SDK; no PostgreSQL service; no Docker CLI; no emulator/device; no VPS/runtime access  
**Tester:** Manus  
**Scope:** Android → ViewModel → Repository → HTTP/API → Backend → PostgreSQL, plus GitHub Actions → artifacts → Docker/VPS/runtime where access permitted.

> **Evidence rule:** Only source-proven defects are `CONFIRMED`. Runtime claims not executable in this environment are `UNVERIFIED`; they are not represented as passes.

# Version Freeze

| Item | Value / status |
|---|---|
| Repository | `mkhas1374/GameNexa-release-2026` |
| Branch | `main` / `origin/main` |
| Commit SHA | `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c` |
| Android applicationId | `com.MinmKhas.studio.GameNexa.wrtx` (`app/build.gradle.kts:18`) |
| Android namespace | `com.example` (`app/build.gradle.kts:10`) |
| versionCode | `4` (`gradle.properties:31`, consumed by Gradle) |
| versionName | `1.0.2` (`gradle.properties:32`, consumed by Gradle) |
| Backend package version | `1.0.0` (`backend/package.json`) |
| Android Room version | `13` |
| PostgreSQL schema version | No explicit PostgreSQL schema-version/migration marker found |
| Docker API image | `node:18-alpine` |
| Docker database image | `postgres:16` |
| Docker proxy image | `nginx:latest` |
| Docker install | `npm install` |
| Docker ignore file | **Absent** in current `main` |
| CI workflows | Android debug, Android release validation, backend integration, delete-runs |
| VPS/runtime commit | `UNVERIFIED` — no VPS access |
| APK/AAB SHA256 | `UNVERIFIED` — no artifact built locally |
| Release signing | `UNVERIFIED` — CI secrets/workflow execution unavailable |
| Working tree | Clean at checkout before this report artifact was written; no tracked source was modified by the audit |

## Source-of-truth observations

* App version values now have one Gradle property source: `gradle.properties:31-32`, consumed by `app/build.gradle.kts:13-22`. This part is source-proven as fixed.
* Release lint is enabled: `app/build.gradle.kts:31-35` sets `checkReleaseBuilds = true`, `abortOnError = true`, and `checkDependencies = true`.
* Backend has a committed `package-lock.json`, but the Dockerfile does not use it deterministically.
* Backend and Android contain both canonical and legacy-looking route surfaces. Static contract checker reports `51 Retrofit contracts` and `76 raw API paths` with backend matches, but this does not prove every field/status/DB mutation at runtime.

# Test Execution Evidence

| Check | Result | Evidence |
|---|---|---|
| Exact `main` commit freeze | PASS | `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c` |
| Android/backend contract checker | PASS | `51 Retrofit contracts and 76 raw API paths have backend matches` |
| Deep static release audit | PASS | `deep release audit (53 invariants)` |
| Backend JS syntax | PASS | `npm --prefix backend run check`, exit 0 |
| Tenant-isolation static check | PASS | `tenant-isolation static checks passed` |
| Lightweight database/security/concurrency scripts | Exit 0 | Static/lightweight checks only; no live DB proof |
| Dependency installation | PASS | `npm ci --ignore-scripts`, exit 0 |
| Full backend `npm test` | BLOCKED | PostgreSQL connection refused on `::1:5432` and `127.0.0.1:5432` at `backend/test_reservation.js:12` |
| Android `./gradlew test` | BLOCKED | Android SDK location unavailable; no `ANDROID_HOME`/`local.properties` |
| Docker build/Compose | BLOCKED | `docker: command not found` |
| Emulator/device E2E | UNVERIFIED | No emulator/device |
| VPS/runtime/hash comparison | UNVERIFIED | No VPS access |
| GitHub Actions execution | UNVERIFIED | Workflow files inspected; no run result available |
| APK/AAB/signature/checksum | UNVERIFIED | No artifact produced |

The repository test command passed its static prefix (`tenant-isolation static checks`, `financial mock`) and then stopped at the first PostgreSQL-backed test. This is a blocked suite, not a release pass.

# Previous Findings Status

Comparison baseline: previous exact RC commit `184336b66b36e1adf0ee897a669a87b317cf0a1e`.

| Previous finding | Current status | Evidence |
|---|---|---|
| BUG-001 / registration mismatch | **REGRESSED — STILL BROKEN** | `GameNetViewModel.kt:5338-5345` again builds old `UserRegisterRequest`; `GameNetApi.kt:260-265` again sends `username/phone/email/role`, while backend requires `phone_number/manager_id/full_name/password` at `server.js:251-257` |
| BUG-002 / archive sync | **STILL BROKEN; RC remediation reverted** | `Dao.kt` no longer has `deleteCustomersMissingFromServer`; `GameNetRepository.kt:271-287` remains upsert-only |
| BUG-003 / stale online station read | **STILL BROKEN** | `GameNetRepository.kt:316-329`, line 318 returns local state before network read |
| BUG-004 / runtime evidence gap | **STILL UNVERIFIED** | PostgreSQL, Android SDK, Docker, device, VPS unavailable |
| BUG-005 / plaintext Customer password in state | **STILL BROKEN** | `SelfHostedManager.kt:436` and `:501` copy plaintext passwords into Customer objects/state |
| BUG-006 / Docker reproducibility | **REGRESSED** | Current Dockerfile reverted to `node:18-alpine` + `npm install`; Compose reverted to `nginx:latest`; `.dockerignore` deleted |
| Release version single source | **FIXED at source level** | `gradle.properties` values consumed by `app/build.gradle.kts` |
| Release lint disabled | **FIXED at source level** | `checkReleaseBuilds`, `abortOnError`, `checkDependencies` enabled |

# A. CONFIRMED RELEASE BLOCKERS

## REG-001 — Customer registration contract regression

* **Finding ID:** `REG-001` / prior `BUG-001`
* **Title:** Current `main` reverted the reachable Customer registration flow to an incompatible Android/backend contract
* **Status:** `CONFIRMED`
* **Severity:** `HIGH`
* **Priority:** `P1`
* **Component:** Android API contract / customer onboarding
* **Exact Commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`
* **File:** `app/src/main/java/com/example/ui/GameNetViewModel.kt`; `app/src/main/java/com/example/data/network/GameNetApi.kt`; `backend/server.js`
* **Line:** `GameNetViewModel.kt:5333-5345`; `GameNetApi.kt:260-265`; `server.js:251-257`
* **Function:** `registerUser(...)`; `GameNetApi.registerUser(...)`; customer registration route
* **Endpoint:** `POST /api/auth/customer/register`

**Description:** The current `main` reintroduced the pre-RC registration implementation. The active UI path calls Retrofit `api.registerUser()` and sends a generic operator-style DTO.

**Expected:** Android request fields, backend validation, response shape, and Android parser must be one canonical contract.

**Actual:** Android serializes `username`, `password`, optional `phone`, optional `email`, and `role`. Backend reads `phone_number`/`phoneNumber`, `manager_id`/`managerId`, `full_name`/`fullName`, and `password`, and rejects missing required fields with HTTP 400.

**Reproduction:** Reach either registration UI call at `GameNexaWelcomeScreen.kt:506` or `AuthDialog.kt:371`; `GameNetViewModel.kt:5338-5345` creates the old request and calls `api.registerUser`. The serialized body lacks `manager_id` and `full_name`; backend validation at `server.js:256` rejects it.

**Evidence:** Active call site and request construction at the exact lines above; backend required fields at `server.js:251-257`. The prior RC had switched this path to `SelfHostedManager.registerCustomer`; the current diff reverts that change.

**Root Cause:** The current `main` was advanced without carrying forward the RC customer-registration hardening.

**Business Impact:** New Customer registration fails or is reported incorrectly.

**Security Impact:** Manager binding and customer onboarding semantics are inconsistent; no separate privilege bypass is claimed.

**Data Integrity Impact:** No valid customer row is reliably created through the active UI path; corrected/retried requests may create operational confusion.

**Recommended Fix:** Restore one canonical registration path and DTO, remove or deprecate the stale Retrofit method, and add a serialized request/response integration test.

**Regression Risk:** Login, manager scoping, device binding, and account recovery.

**Verification:** Run registration against isolated PostgreSQL; assert request fields, HTTP status, response parse, database row, duplicate rejection, and subsequent login.

**Confidence:** High; source-proven and regression-proven by diff against the audited RC.

## REG-002 — Docker reproducibility and build-context hardening regression

* **Finding ID:** `REG-002` / prior `BUG-006`
* **Title:** Current `main` reverted deterministic Docker installation, image pinning, and `.dockerignore`
* **Status:** `CONFIRMED`
* **Severity:** `HIGH`
* **Priority:** `P1`
* **Component:** Docker / deployment / supply-chain / secret handling
* **Exact Commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`
* **File:** `backend/Dockerfile`; `backend/docker-compose.yml`; missing `backend/.dockerignore`
* **Line:** `Dockerfile:1-6`; `docker-compose.yml:22`
* **Function:** Docker build and Compose deployment
* **Endpoint:** Running API deployment

**Description:** The RC hardening was reverted. The Dockerfile uses `node:18-alpine` and `npm install`; Compose uses `nginx:latest`; and no `.dockerignore` exists.

**Expected:** Build inputs should be deterministic and sensitive files excluded from the build context.

**Actual:** The same Git commit does not uniquely determine the npm dependency/image set. More importantly, `COPY . .` can copy any `.env`, logs, or other files present under `backend/` into the image because no `.dockerignore` excludes them.

**Reproduction:** Place a `backend/.env` containing a test secret, run the Dockerfile build, and inspect `/app/.env` in the resulting image. The Dockerfile has `COPY . .` and the current repository has no `backend/.dockerignore`.

**Evidence:** `backend/Dockerfile:1` is `FROM node:18-alpine`; line 4 is `RUN npm install`; line 5 is `COPY . .`. `backend/docker-compose.yml:22` is `image: nginx:latest`. `git diff` from the previous RC shows `.dockerignore` deleted.

**Root Cause:** Deployment hardening changes were not preserved when `main` advanced.

**Business Impact:** A tested deployment may differ from a later build; operations cannot reliably identify the deployed dependency/proxy set.

**Security Impact:** If `.env` or another secret-bearing file exists in build context, it can be embedded into the container image and exposed to anyone with image access.

**Data Integrity Impact:** No direct DB mutation proven; different dependency/runtime versions can alter business behavior.

**Recommended Fix:** Restore `.dockerignore` excluding `.env`, logs, `node_modules`, tests, and repository metadata; use `npm ci --omit=dev`; pin Node/Nginx/Postgres to reviewed immutable versions or digests; label images with the Git SHA.

**Regression Risk:** Native bcrypt compatibility, build cache, TLS/proxy behavior, and runtime Node version.

**Verification:** Build twice from the same SHA; inspect image contents for secrets; compare image digests, lockfile dependency tree, and runtime labels.

**Confidence:** High for configuration defect; actual secret exposure is conditional on files present at build time and was not executed because Docker was unavailable.

# B. CONFIRMED SECURITY ISSUES

## SEC-001 — Plaintext Customer password remains in memory/state

* **Finding ID:** `SEC-001` / prior `BUG-005`
* **Title:** Customer password is copied into the authenticated Customer object and state
* **Status:** `CONFIRMED`
* **Severity:** `MEDIUM`
* **Priority:** `P1`
* **Component:** Android authentication/privacy
* **Exact Commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`
* **File:** `app/src/main/java/com/example/data/network/SelfHostedManager.kt`; `app/src/main/java/com/example/data/Entities.kt`
* **Line:** `SelfHostedManager.kt:436-439`, `:501-509`; `Entities.kt:177-183`
* **Function:** `loginCustomer(...)`; `registerCustomer(...)`
* **Endpoint:** Customer login/register

**Description:** Successful customer login and registration still execute `copy(password = ...)` and assign the resulting `Customer` to `_currentLoggedInCustomer`.

**Expected:** Passwords exist only in request-local authentication DTOs and never in profile/domain/state objects.

**Actual:** Plaintext passwords remain in the returned/state Customer object. Room write paths often clear passwords, but that does not sanitize the in-memory state.

**Reproduction:** Call `loginCustomer` or `registerCustomer` with a password and inspect the returned `Customer`/`currentLoggedInCustomer`; its `password` field equals the input.

**Evidence:** `SelfHostedManager.kt:436`, `:438`, `:501`, `:509`; `Customer.password` at `Entities.kt:182`.

**Root Cause:** Credential request data and Customer domain/profile data share one model.

**Business Impact:** Increased credential exposure and incident severity for affected Customers.

**Security Impact:** Heap inspection, crash/debug inspection, accidental serialization, or future logging can expose the credential. No production crash-log leak was separately proven.

**Data Integrity Impact:** No direct DB corruption proven.

**Recommended Fix:** Remove `password` from the authenticated Customer/domain model, introduce request-only DTOs, and sanitize profile/state objects.

**Regression Risk:** Manager-created Customer password setup and login UI.

**Verification:** Login/register in a test build; inspect object graph, state, Room, network logs, and crash diagnostics for plaintext.

**Confidence:** High; source-proven.

## SEC-002 — Docker build context can include environment secrets

This is the security aspect of `REG-002`, not a duplicate product bug. The exact risk is `COPY . .` without a `.dockerignore`. It should be tracked and fixed with REG-002.

# C. CONFIRMED BUSINESS/FINANCIAL ISSUES

No new financial arithmetic or duplicate-credit defect was source-proven in this audit. Static financial checks and the financial mock passed where executed. However, PostgreSQL-backed settlement, invoice, wallet, payment, refund, GN/LP, VIP supersession, and concurrency tests were blocked; financial readiness is **UNVERIFIED**, not passed.

# D. CONFIRMED ANDROID ISSUES

1. **REG-001 — HIGH/P1:** Active Customer registration path uses the incompatible old DTO/response flow.
2. **SEC-001 — MEDIUM/P1:** Plaintext Customer password remains in Android state.
3. **BUG-002 — MEDIUM/P1:** Customer archive reconciliation remains upsert-only; current `main` removed the RC DAO deletion method.
4. **BUG-003 — MEDIUM/P2:** Online Station read returns local Room data before revalidation.

## BUG-002 — Customer archive reconciliation remains absent

* **Status:** `CONFIRMED`
* **Severity/Priority:** `MEDIUM/P1`
* **Exact Commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`
* **File/Line:** `app/src/main/java/com/example/data/GameNetRepository.kt:271-287`; `Dao.kt:128-132`
* **Function/Endpoint:** `syncAllWithServer()` / `GET /api/v1/manager/customers`

The current main receives remote customers and only inserts/replaces returned rows. It has no set-difference/tombstone phase. The prior RC’s `deleteCustomersMissingFromServer` DAO method was removed. An archived customer already in Room can remain available locally after sync.

**Verification:** Archive remotely, sync, assert local active query excludes the customer while pending unsynced records remain safe.

## BUG-003 — Online Station state remains stale

* **Status:** `CONFIRMED`
* **Severity/Priority:** `MEDIUM/P2`
* **Exact Commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`
* **File/Line:** `app/src/main/java/com/example/data/GameNetRepository.kt:316-329`
* **Function:** `getStationStateById(id: Int)`

Line 318 returns the local Room row before any online request. A server-side status/pricing change is therefore not visible until another broad sync or local mutation.

**Verification:** Change station state server-side, call online read with an existing Room row, and assert server-authoritative freshness.

# E. CONFIRMED BACKEND/DATABASE ISSUES

No new SQL injection or cross-manager authorization bypass was source-proven. Backend source contains manager-scoped queries, role-specific middleware, transaction boundaries, and idempotency paths. Live PostgreSQL constraints, migrations, locks, and race behavior remain unverified.

The following deployment/configuration defect is confirmed and listed under REG-002:

* `Dockerfile` uses non-deterministic `npm install`, mutable `node:18-alpine` tag, and unfiltered `COPY . .`.
* Compose uses mutable `nginx:latest`.

# F. CONFIRMED CI/CD ISSUES

## CI-001 — Release artifact and runtime consistency are not evidenced

* **Finding ID:** `CI-001`
* **Title:** The current audit cannot prove GitHub Actions success, artifact identity, release signature, or VPS correspondence
* **Status:** `CONFIRMED` as a release-evidence blocker; not a claim that the workflow always fails
* **Severity:** `HIGH`
* **Priority:** `P1`
* **Component:** GitHub Actions / release integrity / VPS
* **Exact Commit:** `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`
* **File:** `.github/workflows/android-release-validation.yml`; `.github/workflows/backend-integration.yml`
* **Line:** Release workflow `:63-115`; backend workflow `:62-80`
* **Function:** Android release validation and backend integration jobs
* **Endpoint:** CI/build/deployment pipeline

**Description:** Workflows define meaningful checks, but no execution result, APK/AAB checksum, signing certificate, Docker image digest, or VPS/runtime hash was available in this environment.

**Expected:** A release candidate must have retained workflow run evidence and artifacts tied to the exact Git SHA.

**Actual:** Only workflow source was inspected. Local backend runtime stopped at PostgreSQL connection refusal; local Android build stopped at missing SDK; Docker was unavailable.

**Reproduction:** Run `npm test` after `npm ci`: static prefix passes, then `test_reservation.js` fails at PostgreSQL connection. Run `./gradlew test`: Gradle reports Android SDK location not found.

**Evidence:** Recorded command outputs and workflow definitions.

**Root Cause:** Missing runtime services/artifacts in this audit environment and no externally supplied CI/VPS evidence.

**Business Impact:** Cannot certify that the shipped artifact corresponds to this commit or that DB-backed behavior passes.

**Security Impact:** Cannot certify release signing, image contents, runtime secret handling, or live authorization behavior.

**Data Integrity Impact:** Database migration/concurrency/recovery remain unproven.

**Recommended Fix:** Retain CI run URLs/logs, APK/AAB SHA256/signature, Docker digest, schema migration evidence, and VPS/runtime source hashes for this exact SHA.

**Regression Risk:** Release signing, Gradle lint, Docker runtime, and migration differences.

**Verification:** Execute all workflows and compare artifact/runtime identifiers to `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`.

**Confidence:** High as an evidence blocker.

# G. UNVERIFIED ITEMS

1. VPS commit, container commit, runtime source hashes, and running API correspondence.
2. Android compile/test, APK/AAB build, package metadata, signing certificate, and SHA256.
3. PostgreSQL schema creation, old/new migrations, constraints, indexes, locks, restart recovery, and data-loss behavior.
4. Manager A/B and Customer A/B live IDOR tests across all GET/POST/PUT/PATCH/DELETE routes.
5. Expired/revoked/replayed JWT, privilege escalation, brute-force, enumeration, and rate-limit runtime behavior.
6. 20-request station concurrency and 20–100-request financial/device/trial/VIP races.
7. Settlement retry/double-tap/timeout/network-loss/app-kill behavior.
8. Guest invoice, prepayment, discount, wallet, payment, refund, GN/LP end-to-end reconciliation.
9. Trial reinstall/device-ID/fingerprint/clock manipulation behavior.
10. Offline/online error classification and reconciliation after server-received/client-failed requests.
11. 150-log runtime inspection and client/server diagnostic counter consistency.
12. Tehran timezone, phone-clock manipulation, Persian calendar, digit, and boundary behavior.
13. Full Compose UI screen/button/dialog/form, rotation, process death, keyboard, RTL/LTR, cutout, and small/large-screen tests.
14. Docker build, image content inspection, Compose health/restart, volume recovery, and Nginx/TLS runtime.
15. Dependency vulnerability scanning and immutable image/artifact checksum verification.

# H. PREVIOUS FINDINGS STATUS

* **BUG-001:** `REGRESSED — STILL BROKEN`. The RC fix was reverted; current active UI calls old Retrofit registration.
* **BUG-002:** `STILL BROKEN`. Current main removed even the RC’s unused deletion DAO method; sync remains upsert-only.
* **BUG-003:** `STILL BROKEN`. Local-first Station read unchanged.
* **BUG-004:** `UNVERIFIED / STILL PRESENT`. Runtime and artifact evidence unavailable.
* **BUG-005:** `STILL BROKEN`. Plaintext password copies unchanged.
* **BUG-006:** `REGRESSED`. Docker hardening from RC was reverted; `.dockerignore` deleted, `npm install` and `nginx:latest` restored.
* **Version single source:** `FIXED`, evidenced by `gradle.properties` consumed in `app/build.gradle.kts`.
* **Release lint:** `FIXED at source level`, evidenced by `checkReleaseBuilds=true`, `abortOnError=true`, `checkDependencies=true`.

# I. TOP 10 NEXT ATTACK VECTORS

1. Submit active Android registration with valid UI data and capture exact request/response; confirm HTTP 400 and no customer row.
2. Rebuild Docker with a temporary `.env` and inspect whether `/app/.env` is embedded in the image.
3. Execute Manager A/B and Customer A/B IDOR tests against every canonical and legacy route.
4. Send 20 concurrent starts for one station and one shared customer; verify one session and one financial effect.
5. Replay settlement, payment approval, wallet credit, GN credit, reservation, and VIP requests with identical idempotency keys.
6. Archive a customer server-side, run Android sync, and attempt selection/start from stale Room data.
7. Change station status/pricing server-side and exercise online `getStationStateById` with a populated Room row.
8. Inspect Customer state/heap/crash diagnostics after login/register for plaintext password retention.
9. Test manual phone clock and trial/device identity changes against server-side entitlement/trial rules.
10. Compare GitHub SHA, CI artifact SHA/signature, Docker digest, VPS source hash, and running container files.

# J. FINAL RELEASE STATUS

## Confirmed counts

* **Confirmed Critical/High:** `3` release-blocking findings: `REG-001` HIGH/P1, `REG-002` HIGH/P1, and `CI-001` HIGH/P1 evidence blocker. No CRITICAL product defect was proven.
* **Confirmed Medium/Low:** `4` product/configuration findings: `SEC-001` MEDIUM/P1, `BUG-002` MEDIUM/P1, `BUG-003` MEDIUM/P2, and `BUG-001` as the underlying regression already counted under REG-001. The count of distinct current findings is four if REG-001/REG-002/CI-001 are counted separately as release blockers; no percentage score is used.
* **Unverified:** The runtime/device/VPS/database/performance areas listed in Section G.

## RELEASE STATUS: NOT READY

The current `main` commit is **NOT READY** for release.

The commit does improve two source-level areas: single-source Android versioning and release lint enforcement. However, it regresses the prior RC in the active Customer registration contract and Docker hardening, while the password-in-memory, stale Customer sync, and stale online Station issues remain. PostgreSQL runtime, Android build/device behavior, Docker image contents, artifacts, CI execution, and VPS consistency are also not evidenced.

## Required release actions

1. Restore the canonical Customer registration flow and add an Android serialization/response integration test.
2. Remove plaintext password from Customer entities and authenticated state.
3. Implement and invoke authoritative Customer archive/tombstone reconciliation.
4. Define server-fresh online Station reads and test offline fallback separately.
5. Restore `.dockerignore`, `npm ci --omit=dev`, pinned Node/Nginx images or digests, and inspect image contents for secrets.
6. Execute PostgreSQL-backed backend tests and concurrency/idempotency suites.
7. Execute Android Gradle/lint/build tests with the required SDK and produce signed artifact hashes.
8. Run emulator/device UI, process-death, offline-reconciliation, financial, security, and lifecycle tests.
9. Compare GitHub, artifact, Docker, VPS, and runtime identities against `eab3ebe36e65eecb09ef03cb2614e86a49c9b35c`.

**Final result: NOT READY.**
