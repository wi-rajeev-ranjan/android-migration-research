# WorkIndia Android — Possible Migrations Inventory

> Status: **inventory only** (not prioritized yet).  
> Source baseline: `workindiaandroid` as of scan (target/compile SDK **36**, Kotlin **2.1.21**, AGP **8.6.1**, Compose BOM **2025.09.01**).  
> Next step: decide which migrations to take up for the next release(s).

---

## How to read this

Each item is a **possible** migration — something that *can* be done based on what’s in the codebase today.  
Evidence counts are approximate from static scan and are meant for sizing, not as exact backlog tickets.

Legend for *Driver*:
- **Play / platform** — required or strongly forced by Google Play / Android OS
- **Tech debt** — modernize dual stacks / deprecated APIs
- **Product / DX** — velocity, UI consistency, maintainability
- **Security / privacy** — permissions, networking, PII
- **Build / tooling** — compile times, AGP flags, CI

---

## 1. Platform / Android version

| ID | Migration | Current → Target | Driver | Why it shows up here |
|----|-----------|------------------|--------|----------------------|
| P1 | **Android 17 SDK compile** | compileSdk 36 → **37** | Play / platform | Needed to build against Android 17 APIs / preview behaviors |
| P2 | **Android 17 targetSdk** | targetSdk 36 → **37** | Play / platform | Play expects API **37** for new apps/updates by **Aug 2027**; behavior changes apply when targeted |
| P3 | **Android 17 large-screen adaptability** | portrait/`resizeableActivity` opt-outs → adaptive layouts | Play / platform | On sw≥600dp, orientation/resizability/aspect-ratio restrictions are **ignored** and **opt-out removed** at API 37. Manifest has **~48** `screenOrientation` attrs |
| P4 | **Local network permission** | none → `ACCESS_LOCAL_NETWORK` (if LAN used) | Privacy | Mandatory for target 37 if app talks to LAN devices |
| P5 | **SMS OTP path** | custom SMS reads → SMS Retriever / User Consent | Privacy | Standard SMS OTP delayed ~3h for target 37 |
| P6 | **Background audio / FGS hardening** | current audio + FGS usage → compliant FGS/WIU | Platform | Background audio stricter when targeting 37 |
| P7 | **BAL / activity-start security** | legacy BAL allow → granular `MODE_BACKGROUND_ACTIVITY_START_*` | Security | Activity security hardening on 17 |
| P8 | **Certificate Transparency default-on** | optional → CT by default | Security | Networking / pinning interactions to validate |
| P9 | **Safer native DCL** | `System.load` writable libs → read-only native loads | Security | Native libs must be read-only when loaded |
| P10 | **Contacts Provider PII / SQL strictness** | CP2 data-view usage → RawContacts join / query fixes | Privacy | App declares `READ_CONTACTS`; CP2 behavior changes at 37 |
| P11 | **ContentCapture opt-out** | `setContentCaptureEnabled(false)` → `FLAG_SECURE` if needed | Platform | Old API deprecated / no-op at 37 |
| P12 | **RemoteViews / widget memory ceiling** | unconstrained bitmaps → downsampled widgets | Platform | Strict bitmap memory for widgets at 37 (if widgets used) |
| P13 | **MessageQueue / reflection assumptions** | reflective MessageQueue clients → supported APIs | Platform | Lock-free MessageQueue may break reflection |
| P14 | **minSdk strategy** | minSdk **21** → 23/24/26+ (optional) | Tech debt / size | Shrink API surface, drop old workarounds; product decision |
| P15 | **Edge-to-edge completeness** | partial helpers → all Activities/Fragments | Platform / UX | Helpers exist; many legacy screens may still need insets |

---

## 2. Build system & tooling

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| B1 | **AGP upgrade** | 8.6.1 → **8.9+** (Android 17 docs require ≥8.9.0-rc01) | Build / platform | `libs.versions.toml` |
| B2 | **Gradle wrapper upgrade** | 8.10.2 → version required by new AGP | Build | `gradle-wrapper.properties` |
| B3 | **Android Studio / JDK toolchain alignment** | local variance → pinned JDK 17 (or 21 if team chooses) | Build | Already JVM target 17 |
| B4 | **Disable Jetifier** | `android.enableJetifier=true` → false (+ remove last support libs) | Build | Still enabled in `gradle.properties` |
| B5 | **Enable R8 fullMode** | `enableR8.fullMode=false` → true (with keep-rule cleanup) | Build / size | Flag currently off |
| B6 | **nonTransitive R class** | `nonTransitiveRClass=false` → true | Build / modularity | Flag currently off |
| B7 | **nonFinal resource IDs** | `nonFinalResIds=false` → true | Build | Flag currently off; may break switch-on-R.id Java |
| B8 | **Finish kapt → KSP** | kapt still applied → KSP-only | Build | App applies both `kapt` and `ksp` plugins |
| B9 | **Secrets out of gradle.properties** | keys in repo props → CI secrets / local `.secret` | Security | API keys/client secrets present in properties |
| B10 | **Version catalog hygiene** | mixed old/new deps → single BOM-driven set | Build | Many pinned legacy versions alongside modern Compose |
| B11 | **Configuration cache / build perf** | partial → configuration cache friendly scripts | Build | Daemon/parallel/cache already on; scripts may need cleanup |
| B12 | **Multidex revisit** | `multiDexEnabled=true` always → conditional / minSdk-driven | Build | Still forced on |

---

## 3. Language & module architecture

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| L1 | **Java → Kotlin (app)** | ~**375** Java / ~**1388** Kotlin | Tech debt | ~22% still Java |
| L2 | **Root Activities Java → Kotlin** | ~**37** root Java files under feature package | Tech debt | Many entry Activities still `.java` |
| L3 | **Models Java → Kotlin + `@Parcelize`/`@Serializable`** | ~**90** Java under `models` | Tech debt | Large model layer still Java |
| L4 | **Adapters / dialogs / listeners → Kotlin** | adapters 26, dialogs 28, listeners 29 Java | Tech debt | UI glue still Java |
| L5 | **Feature modularization** | monolithic `:app` → feature modules | Product / DX | 1 huge app + 4 small libs |
| L6 | **Shared `:core` / `:data` extraction** | networking/db/analytics in app → libraries | DX | Retrofit/Room/Flagsmith mixed in app |
| L7 | **Package cleanup / naming** | `clitylocationflow`, mixed `Frag*`/`Fragment*` | DX | Legacy naming inconsistency |
| L8 | **Duplicate Activity versions collapse** | V2/V3 parallel Activities → single path | Tech debt | e.g. SearchActivityV2/V3, LanguageActivity* |

---

## 4. UI framework — XML / Views / Compose

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| U1 | **Compose adoption strategy (product-wide)** | Compose only in onboarding + SDUI DS → staged screen migration | Product / DX | **61** Compose files in app — **all** in `onboarding_revamp`; DS + SDUI modules already Compose |
| U2 | **Expand `:designsystem` as single UI source** | XML Material + Compose tokens coexist → token-driven DS | Product | `:designsystem` exists; XML still on MaterialComponents/Holo mix |
| U3 | **SDUI expansion** | SDUI for forms → more server-driven surfaces | Product | `:sdui-core` / `:sdui-components` already Compose |
| U4 | **DataBinding → Compose / ViewBinding** | ~**443** `<layout>` DataBinding layouts → reduce/remove | Tech debt | Heavy DataBinding surface |
| U5 | **findViewById cleanup** | ~**179** files still use findViewById | Tech debt | Mixed with binding |
| U6 | **Theme stack modernization** | Holo + AppCompat + MaterialComponents Bridge → Material 3 (XML) + Compose M3 | Product | `styles.xml` still has `Theme.Holo.Light` parents |
| U7 | **Material 3 for Views** | Material 1.8 Components → M3 where Views remain | Product | `materialDesign = 1.8.0` |
| U8 | **ViewPager → ViewPager2 completion** | mixed V1 (~22) / V2 (~18) → ViewPager2 only | Tech debt | Dual pager APIs |
| U9 | **Navigation Component / Compose Navigation** | mostly manual Activity/Fragment nav → Nav graphs / Compose Nav | DX | Almost no Navigation Component; Compose Nav only in onboarding |
| U10 | **Fragment → Compose screen (per flow)** | ~**198** Fragment files → Compose destinations | Product | Large Fragment estate (21 Java / 70+ Kotlin Fragment* files) |
| U11 | **Activity consolidation** | ~**92** Activity classes / **59** manifest activities → fewer hosts + Compose | Product | Many single-purpose Activities |
| U12 | **Compose interoperability patterns** | ad-hoc `ComposeView` → standard Fragment/Activity interop guidelines | DX | Onboarding hosts Compose inside XML |
| U13 | **Preview / screenshot testing for Compose** | limited → Paparazzi/Roborazzi + UI tests | DX | Compose test deps present; coverage unknown |
| U14 | **Large-screen / foldable UI** | phone-first → WindowSizeClass / adaptive panes | Platform | Required consequence of P3 |

---

## 5. Architecture patterns (app layer)

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| A1 | **MVVM consistency** | mixed Activity logic / MVVM packages → uniform MVVM/MVI | DX | `mvvm/` exists alongside legacy Activities |
| A2 | **LiveData → StateFlow / Flow** | LiveData ~**204** files → Flow ~**23** today | Tech debt | Dual reactive UI state |
| A3 | **RxJava → Coroutines/Flow** | Rx still on classpath (~**16** files) → remove Rx | Tech debt | Retrofit Rx adapter + RxBinding still present |
| A4 | **EventBus → explicit callbacks / SharedFlow** | EventBus ~**12** files → remove | Tech debt | `eventbus` dependency |
| A5 | **AsyncTask → Coroutines / WorkManager** | AsyncTask ~**24** refs + `async_task` package → remove | Tech debt | Deprecated API |
| A6 | **Handler/Looper → Coroutines** | Handler usage ~**51** files → structured concurrency | Tech debt | Timing/threading glue |
| A7 | **Hilt coverage completion** | ~**144** `@AndroidEntryPoint` → all entry points + workers | DX | Hilt already primary DI |
| A8 | **Repository / data-layer standardization** | ad-hoc retrofit calls in UI → repository pattern | DX | Mixed layers in feature packages |
| A9 | **Unidirectional state for Compose screens** | mixed VM patterns → UDF/MVI for new UI | DX | Onboarding VMs already closer to this |

---

## 6. Jetpack libraries & AndroidX upgrades

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| J1 | **Lifecycle / Activity / Fragment BOM alignment** | pinned 2.8.6 / activity-compose 1.9.0 → latest stable set | Tech debt | Version catalog pins |
| J2 | **Paging upgrade** | Paging **3.1.1** (+ Rx paging) → latest Paging 3.x / drop Rx artifact | Tech debt | Old paging + Rx bridge |
| J3 | **WorkManager upgrade** | 2.9.1 → latest | Tech debt | Catalog pin |
| J4 | **Room upgrade path** | 2.7.2 → keep current or latest; review KSP | Tech debt | Already on KSP Room |
| J5 | **DataStore adoption** | SharedPreferences ~**84** / PreferenceManager ~**5** → DataStore (~**9** today) | Tech debt | Partial adoption |
| J6 | **Browser / AndroidX minor upgrades** | browser 1.5.0 etc. → current | Tech debt | Stale minors |
| J7 | **Remove `legacy-support-v4` / CardView direct deps** | still declared → Material/Compose replacements | Build | Jetifier fuel |
| J8 | **Media → Media3** | ExoPlayer **2.18.4** → **Media3** | Tech debt | No Media3 usage found |
| J9 | **Activity Result API completion** | `startActivityForResult` ~**37** / `onActivityResult` ~**31** → `registerForActivityResult` (~**27** already) | Tech debt | Dual permission/result stacks |
| J10 | **Permissions API modernization** | EasyPermissions ~**10** + `onRequestPermissionsResult` ~**14** → Activity Result + Accompanist/Permissions or custom | Tech debt | EasyPermissions dependency |
| J11 | **LocalBroadcastManager removal** | ~**8** files → Flow/callbacks | Tech debt | Deprecated |
| J12 | **Loader/CursorLoader removal** | ~**4** files → coroutines/Room Flow | Tech debt | Legacy loaders |
| J13 | **SyncAdapter / Authenticator revisit** | sync package + account perms → modern auth/sync or delete | Tech debt | Account/sync permissions still declared |
| J14 | **Navigation Compose expansion** | onboarding-only → other Compose features | Product | Nav Compose already depended |

---

## 7. Networking, serialization, storage

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| N1 | **OkHttp upgrade** | 4.11.0 → 4.12+/OkHttp 5 (team choice) | Security / tech debt | Catalog |
| N2 | **Retrofit upgrade / consolidation** | 2.10.0 + Jackson converter 2.3.0 → single stack | Tech debt | Dual converters (Gson + Jackson) |
| N3 | **Jackson 2.9.8 upgrade or remove** | **2.9.8** (very old) → modern Jackson or delete | Security | High risk stale JSON lib |
| N4 | **Gson → kotlinx.serialization (gradual)** | Gson ~**291** refs vs serialization ~**29** | Tech debt | Dual JSON stacks |
| N5 | **socket.io client modernization** | `com.github.nkzawa:socket.io-client:0.6.0` → maintained client | Security / reliability | Ancient fork |
| N6 | **Certificate / network security config for ECH** | default → explicit NSC if needed | Privacy | Android 17 ECH default behavior |
| N7 | **Chucker / debug tooling isolation** | already variant-aware → keep; audit analytics variant | DX | analytics build type enables Chucker |
| N8 | **Room schema/export & migration discipline** | existing Room → strict migration testing | Reliability | Room present (~34 annotated files) |

---

## 8. Third-party SDKs

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| T1 | **Facebook SDK** | 11.1.1 → current Meta SDK | Security / Play | Very old |
| T2 | **Singular / attribution** | 12.8.0 → latest required by partner | Product | Catalog |
| T3 | **UXCam / Clarity** | current → latest + Compose-safe APIs | Product | `clarity-compose` already used |
| T4 | **Sentry** | 8.0.0 / gradle 5.7.0 → latest | Reliability | Catalog |
| T5 | **Flagsmith client** | internal 1.0.0 → keep/align with backend | Product | Custom fork |
| T6 | **Lottie / Glide Compose** | present → standardize image/animation on Compose path | DX | Glide compose beta in catalog |
| T7 | **RapidWebView** | 2.1.1 → latest WorkIndia web bridge | Product | WebView-heavy app (~27 WebView refs) |
| T8 | **Play Services BOM / individual pins** | mixed pins → Play services BOM where possible | Build | Location/Maps/Auth/Time pinned separately |
| T9 | **Firebase BOM refresh** | 33.5.1 → latest | Platform | App Check already integrated |
| T10 | **Image cropper libs consolidation** | `edmodo:cropper` + CanHub cropper → one lib | Tech debt | Two croppers declared |
| T11 | **RangeBar / OTP / LoadingButton audit** | unmaintained UI libs → Compose/DS replacements | Tech debt | Legacy UI widgets |

---

## 9. Permissions, privacy, Play policy

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| R1 | **Permission minimization audit** | broad sensitive set → justified minimum | Play / privacy | CALL_PHONE, READ_CALL_LOG, READ_CONTACTS, RECORD_AUDIO, LOCATION, CALENDAR, etc. |
| R2 | **Photo/video picker / media permissions** | audio media perm present; storage maxSdk 32 → Photo Picker where possible | Privacy | Partial modern storage already |
| R3 | **Foreground service types completeness** | `FOREGROUND_SERVICE` + `DATA_SYNC` → declare exact types per service | Play | Manifest services need type audit |
| R4 | **Notification permission UX** | POST_NOTIFICATIONS present → review flows | Platform | Already has notification permission activity |
| R5 | **Privacy sandbox / AD_ID declaration** | AD_ID permission present → Play form + SDK alignment | Play | Declared |
| R6 | **Screen capture detection policy** | `DETECT_SCREEN_CAPTURE` declared → intentional UX | Privacy | Present in manifest |
| R7 | **QUERY_ALL / package visibility** | large UPI package query list → keep justified | Play | Large `<queries>` block |

---

## 10. Testing, quality, observability

| ID | Migration | Current → Target | Driver | Evidence |
|----|-----------|------------------|--------|----------|
| Q1 | **Unit test expansion around migrations** | MockK/Turbine present → coverage gates per migrated module | DX | Deps exist |
| Q2 | **Compose UI tests** | deps present → screen-level tests for onboarding/SDUI | DX | Compose test artifacts in catalog |
| Q3 | **Screenshot / visual regression** | none obvious → Paparazzi/Roborazzi | DX | Useful for DS + Compose rollout |
| Q4 | **Android Test Orchestrator / emulator CI matrix** | unknown → API 36 + 37 + tablet/fold | Platform | Needed for P3 |
| Q5 | **StrictMode / lint as errors** | `abortOnError = false` → ratchet lint | DX | Lint currently non-blocking |
| Q6 | **Crash-free migration dashboards** | Sentry/Crashlytics → per-release migration monitors | Reliability | Both present |

---

## 11. Process / delivery migrations (optional but real)

| ID | Migration | Notes |
|----|-----------|-------|
| D1 | **Trunk feature flags for UI rewrites** | Flagsmith already available — use for Compose rollouts |
| D2 | **Migration ADR template** | One ADR per accepted migration ID |
| D3 | **Play target deadline calendar** | API 36 enforcement Aug 2026 (already met); API 37 Aug 2027 |
| D4 | **Release train split** | “must” platform train vs “should” modernization train |

---

## Suggested grouping (for later prioritization — not decided)

These are **buckets**, not a plan commitment:

1. **Must-do platform train** — P1–P3, B1–B2, related R3/R1 audits  
2. **Compose / UI train** — U1–U3, U6, U14, expand from onboarding/SDUI  
3. **Legacy API removal train** — A3–A6, J8–J12, L1–L4  
4. **Dependency security train** — N3, N5, T1, B4, B9  
5. **Architecture / modularization train** — L5–L6, A1–A2, U9–U11  

---

## Baseline snapshot (quick reference)

| Area | Snapshot |
|------|----------|
| Modules | `:app`, `:designsystem`, `:sdui-core`, `:sdui-components`, `:exoverify` |
| SDK | compile/target **36**, min **21** |
| Languages | Kotlin ~1388, Java ~375 |
| UI | XML layouts ~669; DataBinding ~443; Compose files ~99 (app Compose = onboarding only) |
| Components | Activities ~92 / Fragments ~198 / ViewModels ~95 |
| Manifest | activities 59, services 5, receivers 13, providers 6; screenOrientation ~48 |
| DI | Hilt primary |
| Async | Coroutines dominant; Rx + AsyncTask + EventBus still present |

---

## Next decision checkpoint

For each ID above, mark one of:
- **Do now** (next release)
- **Do next** (following release train)
- **Park** (explicitly not now)
- **Reject** (won’t do / not applicable)

Then we turn the “Do now / Do next” set into a sequenced project plan with owners, dependencies, and risk.
