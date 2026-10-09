# Custom Club: Online Racing 3D — Phase 5.3 Disposable Test Environment Feasibility Assessment

- **Target:** `com.GamesEZ.CustomClubOnline` v5.7.1 (versionCode 50701), Unity 2022.3.62f2, IL2CPP, **arm64-v8a**.
- **Scope of this phase:** environment **planning and feasibility assessment only**. No game launch, no install/uninstall, no device-setting change, no emulator creation, no APK modification, no app-data/network/account change, and no new runtime experiment were performed. No Phase 6 work was started.
- **Inputs read (all on disk, read-only):** `CustomClub_consolidated_status.md`, `CustomClub_currency_persistence.md`, `CustomClub_phase5_practical_validation.md`, `CustomClub_phase5_1_evidence_audit.md`, `CustomClub_phase5_2_pid_attribution.md`, `CustomClub_feasibility.md`, `CustomClub_save_architecture.md`, `CustomClub_runtime_recon.md`; raw evidence under `workspace/dynamic/`; environment files `workspace/tool-config.json`, `C:\Users\Lpmarket\.android\avd\Pixel_Fold_API_35.avd\config.ini`, `C:\Users\Lpmarket\AppData\Local\Android\Sdk\system-images\`.
- **Labeling convention (applied to every material claim):** `CONFIRMED` (directly observed in a local artifact) · `PARTIALLY CONFIRMED` (partly evidenced; documented general behaviour plus local inference) · `INCONCLUSIVE` (cannot be decided without an action this phase forbids) · `BLOCKED` (needs new hardware/software/authorization or a state change) · `ESTIMATE` (reasoned forecast, **not** experimentally verified).
- **Rules of engagement assumed:** normal legitimate gameplay + authorized observation only. No currency manipulation, no forged/intercepted multiplayer traffic, no authentication bypass, no anti-cheat defeat, no APK repackaging.

---

## 1. Executive recommendation

**Recommended environment: Option C — a separate, dedicated, ARM64 Android test device with authorized privileged (root) access, prepared with a verified snapshot/restore workflow.**

**Recommended next step (one; not executed):** acquire/prepare that isolated ARM64 test device, then — under a separate, explicitly authorized phase — install the exact split trio (hash-verified) and establish a whole-app-data snapshot/restore loop before any state-changing observation. This requires **new hardware and a separate authorization** for privileged access; stated explicitly rather than assumed.

Why C, not the alternatives:

- The application is **arm64-v8a only** (`CONFIRMED` — Phase 1 split inventory; on-device `primaryCpuAbi=arm64-v8a`). The only emulator system image installed here is **x86_64** (`CONFIRMED`, §3.1). Running an arm64-only Unity build inside an x86_64 image requires ARM→x86_64 translation whose availability on this specific API-35 Play Store image is **unverified** (`INCONCLUSIVE`), and cannot be checked without creating/launching an emulator — which this phase forbids.
- Even if the emulator ran the app, the installed image is a **Google Play (`google_apis_playstore`) image**, normally **not root-accessible via `adb root`** (`ESTIMATE` — documented Android/AVD behaviour, not re-verified here). Without root, the app-private save file (`…/shared_prefs/…v2.playerprefs.xml`) is **unreadable**, which is exactly what the open claims need.
- The current phone (Option A) is non-rooted, non-debuggable, has no rollback path, and is the **daily driver** — it supports only strictly observational checks (`CONFIRMED` across Phases 2/5/5.1/5.2).
- A separate physical ARM64 device (C) is the only option that simultaneously satisfies: (i) native execution of the same ARM64 build, (ii) readable app-private storage (with authorized root), (iii) a real snapshot/restore rollback, and (iv) real Google Play services / Photon networking. Its cost is the highest (hardware + privileged-access setup + boot-time data wipe).

If hardware is impossible to obtain, the **cheapest technical fallback** is Option B — but only after a bounded pre-check proves the ARM64 build executes under translation **and** a root-capable (non-Play-Store) image is provisioned; both are out of scope this phase and currently `INCONCLUSIVE`.

---

## 2. Available artifacts and integrity checks

### 2.1 Canonical APK / split set (matches Phase 1)

The authoritative CustomClub build is the three-part Play App Bundle split set present on the Desktop. Each file was re-hashed this phase (read-only) and **matches the Phase 1 record exactly**:

| Artifact | Path | Size (bytes) | SHA-256 (recomputed this phase) | Matches Phase 1? |
|---|---|---|---|---|
| Base APK | `C:\Users\Lpmarket\Desktop\CustomClub_base.apk` | 110,422,300 | `2E38BE0CCA5C04538AC53B276B09D0FBBFFEBE1E801EB53642149A76AB1352D7` | `CONFIRMED` |
| ARM64 split | `C:\Users\Lpmarket\Desktop\CustomClub_arm64_v8a.apk` | 32,945,922 | `1C612124C935493CC4B2E7C644F20F96995289EB022880FBCBCDC755BAA771D6` | `CONFIRMED` |
| Unity data asset pack | `C:\Users\Lpmarket\Desktop\CustomClub_UnityDataAssetPack.apk` | 232,006,631 | `CF742A4AAE68F29D74F169E8EEFB21A5AB9AC8956ECEEF375BDC2043D5724413` | `CONFIRMED` |

**Integrity verdict:** the exact split set is present locally and hash-verifiable → a prerequisite for Option C is already satisfied (`CONFIRMED`).

### 2.2 Mislabelled non-target artifact in the workspace (warning)

`workspace/extracted/customclub.apk` = 19,179,561 B, SHA-256 `9780783A1E795DE6B337B756ACA915C83F712CD4CAB1DA9A112992E662DE7E58`. `aapt2 dump badging` identifies it as `package: ir.mservices.market` (versionCode 1037, versionName 10.3.7), label **Myket** — the Iranian app market — and it contains **no** `libil2cpp.so` and **no** `global-metadata.dat`. It is **not** the game and must not be used as the target artifact (`CONFIRMED`). (The device does install the game via the Myket store, which explains the stray file.)

### 2.3 Extracted payload present (used by Phases 1–5)

- `workspace/artifacts/base_small/global-metadata.dat` = 14,465,700 B (matches Phase 1), magic `0xFAB11BAF`, metadata version 31, plaintext.
- `workspace/artifacts/base_small/`: `boot.config`, `builddatas.json`, `RuntimeInitializeOnLoads.json`, `ScriptingAssemblies.json`, `com.android.games.engine.build_fingerprint`, `supplierconfig.json`, `unity_app_guid`.
- `workspace/artifacts/so/lib/arm64-v8a/`: `libil2cpp.so` (81,481,784 B), `libunity.so` (19,984,664 B) + 16 other AArch64 libs.
- `workspace/artifacts/assetpack_small/`: `UnityServicesProjectConfiguration.json`, `google-services-desktop.json`.
- `workspace/static/il2cpp_dump/`: `dump.cs`, `script.json`, `il2cpp.h`, `stringliteral.json`.

### 2.4 Frozen runtime evidence (hashed in Phase 5.2)

`workspace/dynamic/phase5/{exp1_baseline_20261009_021547.txt, exp1_pkgdump_20261009_021547.txt, exp1_game_lines_extracted.txt, exp2_probe_20261009_021805.txt}`, `workspace/dynamic/logcat_startup.txt`, `workspace/dynamic/logcat_foreground.txt` — all present with SHA-256 recorded in `CustomClub_phase5_2_pid_attribution.md`.

### 2.5 Tooling relevant to execution (from `workspace/tool-config.json`)

`tools.adb` = `…\Sdk\platform-tools\adb.exe` (`Test-Path` = True), `tools.emulator` = `…\Sdk\emulator\emulator.exe` (`Test-Path` = True), build-tools 35.0.0 (`aapt2`/`apksigner`/`dexdump`), Java, JADX 1.5.6, Apktool, Frida 17.16.1, Ghidra 12.1.3, IDA Free 9.1.

---

## 3. Emulator feasibility

**Bottom line: the emulator as currently provisioned is NOT a viable target environment for this app (`INCONCLUSIVE` → effectively `BLOCKED`): it is x86_64-only while the app is ARM64-only, and the installed image is a non-rootable Play Store image. Making it viable would require provisioning a different system image, which is out of scope for this phase.**

### 3.1 Provisioned emulator environment (facts)

- Only one AVD exists: **`Pixel_Fold_API_35`** (`…\.android\avd\Pixel_Fold_API_35.avd\config.ini`):
  `abi.type = x86_64`, `hw.cpu.arch = x86_64`, `hw.cpu.ncore = 4`, `hw.ramSize = 2048`, `PlayStore.enabled = true`, `tag.id = google_apis_playstore`, `image.sysdir.1 = system-images\android-35\google_apis_playstore\x86_64\`, `hw.gpu.enabled = yes`, `hw.gpu.mode = auto`, `disk.dataPartition.size = 6442450944` (6 GiB), device = Pixel Fold (`CONFIRMED`).
- Only one system image is installed: `system-images\android-35\google_apis_playstore\x86_64`.
  - `source.properties`: `Pkg.Desc = System Image x86_64 with Google Play`, `AndroidVersion.ApiLevel = 35`, `SystemImage.Abi = x86_64`, `SystemImage.TagId = google_apis_playstore`, `SystemImage.GpuSupport = true`, `Pkg.Revision = 8`.
  - Contains `system.img` (2.15 GB), `vendor.img`, `kernel-ranchu`, `ramdisk.img`, `encryptionkey.img` (`CONFIRMED`).
- No ARM/ARM64 or non-Play (`google_apis`) system image is installed anywhere under `…\Sdk\system-images\` (`CONFIRMED`, recursive listing).

### 3.2 Native-library / ABI compatibility

- **Facts (`CONFIRMED`):** the app ships **arm64-v8a only** (only the `config.arm64_v8a` split; no x86/x86_64 libs); the installed image is **x86_64**.
- **`INCONCLUSIVE`:** executing the ARM64-only `libil2cpp.so`/`libunity.so` inside an x86_64 guest requires ARM→x86_64 translation; whether this specific API-35 image exposes such a translator cannot be checked without creating/launching the emulator (forbidden this phase).
- **`ESTIMATE`:** stock Google emulator images are not documented as providing reliable ARM64 translation for arbitrary apps; the default expectation is that the ARM64-only build will not run natively here.
- **Unverified dependency:** app execution on this image. It is **not** assumed.

### 3.3 Unity / IL2CPP graphics and performance

- The build is URP + Unity Burst (`lib_burst_generated.so`), `OptimizedFramePacing=1` (`CONFIRMED`, Phase 1) — a GPU-hungry 3D renderer. The AVD reports GPU support and `hw.gpu.mode=auto` (`CONFIRMED`).
- The AVD is configured with **2,048 MB RAM / 4 cores** (`CONFIRMED`); the install set is ~375 MB compressed and `libil2cpp.so` alone is ~81 MB, plus a ~218 MB asset bundle.
- **`ESTIMATE`:** 2 GB RAM is likely insufficient or borderline for this title; not verified. Whether the game reaches interactive gameplay on an emulator is **`INCONCLUSIVE`**.

### 3.4 GMS / sign-in / billing dependencies

- The image tag is `google_apis_playstore` with `PlayStore.enabled=true` (`CONFIRMED`), so Google Play services / Play Store are present. The app depends on Firebase (Remote Config, Analytics), Google Sign-In, and Google Play Billing / Xsolla (`CONFIRMED`, Phases 1/5).
- **`INCONCLUSIVE`/`ESTIMATE`:** Play licensing / Play Integrity / Firebase App Check behaviour on an emulator with a synthetic device identity is unverified and could block or degrade Firebase-backed calls. Sign-in would require a Google account on the test image (an account-risk decision, §9).

### 3.5 Photon / network under normal gameplay

- Multiplayer uses Photon PUN2 (`CONFIRMED`, Phase 1). Emulators provide NAT'd internet access, so ordinary connectivity is expected (`ESTIMATE`). Photon's UDP path / DTLS is not inspected here; nothing in this phase tests it.

### 3.6 APK availability and app-data access without touching the production device

- **Availability/integrity: `CONFIRMED`** — exact split trio present and hash-matched (§2.1).
- **App-data access: `BLOCKED` as configured.** Reading the app-private `shared_prefs/…v2.playerprefs.xml` requires privileged access. The installed Play Store image is normally not `adb root`-able (`ESTIMATE`); a root-capable `google_apis` (non-Play) image is **not installed**, and provisioning one is a software download (out of scope). The emulator's isolation from the production device is nonetheless a genuine advantage (`CONFIRMED`).

### 3.7 Emulator feasibility verdict

| Aspect | Verdict |
|---|---|
| Local APK set + integrity | `CONFIRMED` |
| ARM64 app execution on installed x86_64 image | `INCONCLUSIVE` (translation unverified; not assumed) |
| Root / app-private storage access as installed | `BLOCKED` (Play Store image; no root-capable image installed) |
| Unity 3D rendering/perf at 2 GB RAM | `ESTIMATE` (likely inadequate/untested) |
| GMS/Play presence | `CONFIRMED` (Play Store image) |
| Play Integrity / Firebase App Check | `INCONCLUSIVE` |
| Overall as currently provisioned | **Not viable; requires a different, root-capable, ARM64-executable image (out of scope)** |

---

## 4. Separate physical device feasibility

**Target: a second, dedicated ARM64 Android device, distinct from the daily driver, with authorized privileged access.**

### 4.1 Minimum capabilities

| Capability | Requirement | Status in the evidence |
|---|---|---|
| CPU/ABI | **ARM64 (AArch64)** so the same arm64-v8a build runs natively | `CONFIRMED` requirement (app is arm64-only) |
| OS | Android ≥ 8.0 (`minSdk 26`); Android 14/15 fine (`targetSdk 36`) | `CONFIRMED` (manifest) |
| RAM / storage | ~375 MB install + Unity working set (≥ 4 GB RAM, ≥ 8 GB free recommended) | `ESTIMATE` |
| Network | Internet for Google sign-in / Firebase / Photon | `CONFIRMED` requirement |
| **Privileged access to app-private dir** | root (or equivalent authorized mechanism) to read `…/shared_prefs/…v2.playerprefs.xml` | `BLOCKED` without root: `run-as` denied, `/data/data/...` denied, `adb backup` 0-byte (`CONFIRMED`, Phases 2/5) |
| **Rollback / restore** | a whole-app-data/whole-system snapshot **proven to restore** before use | `BLOCKED` on a stock non-rooted device; possible only with privileged access above |
| Runtime instrumentation | ARM64 Frida server on the test device | `PARTIALLY CONFIRMED` (feasible in principle; not set up here) |

**Key point:** the app is not debuggable and APK repackaging is forbidden, so the **only** way to read its private save data is a **rooted/privileged** device (or an emulator providing root). Reading the test install's private save data therefore *entails* privileged access as a hard requirement. Acquiring it is deliberately **not** performed in this phase.

### 4.2 Security and data-loss risks (explicit)

- **Bootloader unlock / rooting typically wipes the device and voids warranty** (`ESTIMATE`/documented Android behaviour). Hence the device must be **separate and disposable**, never the daily driver.
- Rooting increases the security surface and **trips Play Integrity / SafetyNet**, which can break banking/payment apps **on that device**; acceptable only on a dedicated test unit.
- **Bricking risk** exists but is low on well-supported devices with official unlock tooling (`ESTIMATE`).
- **Account risk:** using a real Google/Play account on a rooted/emulated device can expose it; a **throwaway account** is strongly advised. Do not use accounts with real-money history.
- **No rooting, unlocking, or flashing is performed in this phase**, and none should ever be performed on the existing phone.

### 4.3 Physical-device feasibility verdict

A dedicated ARM64 device with authorized root and a verified snapshot/restore loop satisfies all four capabilities (native execution, private-storage read, runtime instrumentation, safe rollback). The cost is hardware plus a privileged-access setup carrying wipe/security caveats. This is the **recommended** option (§1, §8).

---

## 5. Current-device limitations

### 5.1 Confirmed limitations

The current phone (`22120RN86G`, Android 14 / SDK 34, `user` build, non-rooted) (`CONFIRMED`, Phases 2/5/5.1/5.2):

- `ro.secure=1`, `ro.debuggable=0`, shell uid 2000, no `su`/`magisk`; app **not debuggable** → `run-as` denied.
- `/data/data/com.GamesEZ.CustomClubOnline` and `…/shared_prefs` → **Permission denied**.
- `adb backup` yields **0 bytes** (Android 14 deprecation); no demonstrated restore path.
- Frida 17.16.1 is **jailed**; no server-attach channel.
- Always-on VPN (`tun0`) and mobile data; unobserved traffic may be tunnelled.
- The only captured launch occurred with `isKeyguardLocked=true`; the foreground window contained **zero** game lines; the game emits no Unity `Debug.Log` and no currency/save events.
- Installed-package inventory shows a **daily driver**, not a disposable test bed.

### 5.2 Claims testable with strictly observational methods (read-only)

| # | Method (read-only, non-invasive) | What it can show | Value |
|---|---|---|---|
| O1 | PID/UID-scoped `logcat` attribution to uid 10341 during a legitimate, user-driven session | whether the game itself emits any endpoint | supports/undercuts "no game endpoint observed" (partly done in 5.1/5.2) |
| O2 | `dumpsys netstats` / `/proc/net/tcp{,6}` scoped to uid 10341 during play | whether the game opens sockets beyond known providers | same; VPN tunnelling caps certainty |
| O3 | `dumpsys activity` / `dumpsys package` / `ApplicationExitInfo` / crash buffer | launch, display, crash-free execution, install metadata | already `CONFIRMED`; re-running adds nothing |

**Guardrail:** O1/O2 require *gameplay*, an app state change that is **not permitted on the daily driver**. They are listed to define the ceiling of Option A, not as something to run now.

### 5.3 Claims NOT meaningfully testable on the current device

Anything needing (a) the raw persisted key/value (`Money`/`Puzzle` mapping — D2), (b) memory/field observation (`PlayerSaves` liveness, `PlayerSaveData.Currency` — D3/D4), or (c) a definitive server-authority verdict beyond "no endpoint observed" (D5/D6). All are file/memory-blind here (`CONFIRMED`).

**Do not repeat:** Exp 2 (ordinary-setting persistence) is confounded (expected default behaviour), state-changing, and has no rollback — `BLOCKED`, near-zero epistemic value. The Exp 3/4 raw log searches were already corrected in Phases 5.1/5.2 and should not be repeated.

---

## 6. Claim-to-experiment matrix

### 6.1 Consolidated matrix

| # | Claim under test | Falsifiable hypothesis | Minimum observation | Preferred environment | Read-only or state-changing | Safe rollback | Result that would prove | Result that would NOT prove |
|---|---|---|---|---|---|---|---|---|
| **D1** | A legitimate currency update persists under normal gameplay | After a *legitimate* in-game currency delta and a cold restart, the stored value equals the post-delta value | Read the app-private store before/after a legitimate delta + `am force-stop` + relaunch | C (or B if it truly runs the arm64 build and is root-capable) | Observation read-only; enabling action is legitimate gameplay (state-changing) | Restore whole-app-data snapshot taken before the action | The delta is present after restart → local persistence of a legitimate update | Exact key identity; absence of a server ledger |
| **D2** | The persisted representation corresponds to `Money`/`Puzzle` | After a legitimate currency delta, the playerprefs keys that change are `Money` and/or `Puzzle` | Read & diff `…/shared_prefs/…v2.playerprefs.xml` across the delta | C (or B + root) | Read-only observation | Snapshot restore | The predicted keys carry the delta → static key mapping confirmed | Whether those keys are authoritative vs a cache |
| **D3** | `PlayerSaveData.Currency` is inactive in practice | The `PlayerSaves`-managed currency field/store never changes during play | Hook/watch `PlayerSaves` (`SetCurrency`, `o_Instance`, `PlayerSaveData`) over a full session | C (Frida) | Instrumentation; enabling action is gameplay | Snapshot restore | No change across a full session → practically inactive | Absolute impossibility (reflection/indirect not excluded) |
| **D4** | `PlayerSaves` is invoked indirectly at runtime | Some runtime path calls `PlayerSaves` despite zero direct BL callers | Frida hook on `PlayerSaves` entry points; log hits across a full session | C (Frida + `dump.cs` RVAs) | Read-only instrumentation | Snapshot restore | A positive hit → liveness confirmed | A null session is strong but not absolute |
| **D5** | Leaderboard/race results are server-validated | A first-party server validates/reconciles results | Observe (not forge) game-uid endpoints/sockets across a race/leaderboard cycle | C (uid-scoped netstat/logcat) | Read-only observation | None needed; snapshot if progression changes | Presence of a validation endpoint → server validation exists | Absence does not prove client-only (Photon UDP/DTLS, VPN) |
| **D6** | Distinguishing client persistence from server-authoritative state | A server-authoritative value would be reconciled/overwritten; a client-authoritative one would not | (a) confirm local write; (b) watch for server overwrite; (c) see whether traffic carries the value | C | Observation read-only; enabling action is legitimate play | Snapshot restore | Local write persists with no reconciliation → client-authoritative for that value | Absolute absence of a hidden ledger without full traffic visibility |

**Guardrails applying to every row:** no currency manipulation, no result falsification, no MP interception/forgery, no auth bypass, no anti-cheat defeat. Only ordinary legitimate gameplay + authorized observation on an isolated environment with a verified snapshot.

### 6.2 Per-claim observations

- **D1** — read the persisted store across a legitimate delta and a restart; proves/refutes *local persistence* only, not key identity or authority.
- **D2** — diff the playerprefs XML across a legitimate delta; proves/refutes the static `Money`/`Puzzle` key prediction (does not establish authority).
- **D3** — instrument `PlayerSaves` and watch; a negative result supports the static zero-caller finding (`PARTIALLY CONFIRMED`) but does not exclude reflection/indirect dispatch.
- **D4** — hook `PlayerSaves` entry points; a positive hit decisively refutes "dead"; a null session is strong but not absolute.
- **D5** — observe game-uid traffic across a race/leaderboard cycle; presence of a validating endpoint proves server validation, but absence stays `INCONCLUSIVE` (Photon UDP/DTLS, VPN). Interception/forgery is not permitted, capping provability.
- **D6** — combine D1/D2 (local write) with a search for server-driven overwrite; a stable local value with no reconciliation indicates client-authoritative persistence for that value, but does not prove the absence of a hidden server ledger without full traffic visibility.

---

## 7. Weighted comparison table

Scale 1 (worst) to 5 (best); higher is better. Options: **A** = existing non-rooted phone; **B** = disposable Android emulator (as provisioned); **C** = separate ARM64 test device.

| Criterion | Weight | A | B | C |
|---|---|---|---|---|
| Setup effort (low = easy) | 15% | 5 | 3 | 2 |
| Cost (low = cheap) | 15% | 5 | 4 | 2 |
| Risk to existing data (low = safe) | 20% | 2 | 5 | 5 |
| Ability to inspect app-private storage | 20% | 1 | 2 | 5 |
| Ability to obtain runtime evidence | 20% | 2 | 3 | 5 |
| Reproducibility | 10% | 2 | 5 | 4 |
| **Weighted total** | **100%** | **2.70** | **3.55** | **4.00** |

Calculations: A = 0.75+0.75+0.40+0.20+0.40+0.20 = 2.70 · B = 0.45+0.60+1.00+0.40+0.60+0.50 = 3.55 · C = 0.30+0.30+1.00+1.00+1.00+0.40 = 4.00.

**Scoring rationale:**

- **A** wins on setup/cost (nothing to build) but is penalised on risk (daily driver, no rollback) and on private-storage/runtime-instrumentation capability (both denied).
- **B** wins on isolation, reproducibility and cost, but is penalised on private-storage access (the installed Play Store image is normally not `adb root`-able; no root-capable image installed) and carries an **unscored pass/fail gate**: whether it can execute the ARM64 build at all. If that gate passes and a root-capable image is provisioned, B's effective score rises; if it fails, B is unusable regardless of the numeric score.
- **C** wins on the capabilities that actually resolve the open claims (private-storage read, runtime evidence, rollback) and on risk to existing data; it is penalised only on cost/setup.

**Selected option:** **C — a separate, authorized, ARM64 test device** (score 4.00). It requires **new hardware and a separate authorization**; stated explicitly, not assumed.

---

## 8. Selected next step and prerequisites

**Selected next step (exactly one; do NOT execute in this phase):**

> **Prepare Option C — a dedicated, separate, ARM64 Android test device with authorized privileged (root) access, plus a verified whole-app-data snapshot/restore workflow, before running any state-changing observation.**

**Prerequisites (all required before any experiment):**

1. **New hardware** — a dedicated ARM64 Android device (Android ≥ 8; ≥ 4 GB RAM recommended), not the daily driver, holding no sensitive data. `BLOCKED` until acquired.
2. **Separate authorization** for privileged access (root / bootloader unlock) on that device only — acknowledging the mandatory data wipe, Play-Integrity loss and security implications. **No rooting/unlocking happens in this phase or on the existing phone.**
3. **Exact artifact install** — the hash-verified split trio (§2.1), installed as the official split set.
4. **Verified rollback** — a snapshot/restore loop **proven** to restore the pre-experiment state (stock `adb backup` is insufficient; 0 bytes on Android 14).
5. **Instrumentation readiness** — an ARM64 Frida server compatible with the device; `dump.cs`/`script.json` already available for symbol resolution.
6. **Account hygiene** — a throwaway Google/Play account for sign-in if required; no account with real-money history.
7. **Environment isolation** — controlled networking so legitimate traffic can be observed; keep gameplay ordinary and non-competitive.

**Fallback (only if hardware cannot be obtained):** re-evaluate Option B, but first provision a **root-capable, ARM64-executable** system image and verify the app reaches gameplay; both are out of scope this phase and currently `INCONCLUSIVE`.

I am **not** executing any part of the above. This phase ends at the recommendation.

---

## 9. Stop conditions and risks

### 9.1 Stop conditions (restated, binding)

- No interaction with the existing phone beyond read-only observation; **no state-changing experiment on the daily driver**.
- No APK modification, patching, or repackaging.
- No emulator creation and no software installation in this phase.
- No change to app data, network configuration, or account state.
- No new runtime experiment; do not start Phase 6. Finish this report and stop for review.

### 9.2 Risks by option

| Option | Principal risks |
|---|---|
| **A — existing phone** | Irreversible changes to a daily driver with **no rollback**; potential impact on a real Google/Play account and unrelated data; near-zero epistemic yield (file/memory-blind). **Not recommended for any state-changing test.** |
| **B — emulator** | ARM64 execution may fail entirely (wasted effort); Play Store image is not root-accessible so private storage stays unreadable; Play Integrity / Firebase App Check may block services; emulator behaviour may not represent real devices; requires a software download (out of scope). |
| **C — separate ARM64 device** | Cost; bootloader-unlock/root causes a data wipe and trips Play Integrity (breaks payment/banking apps **on that device**); low bricking risk; increased security surface; account exposure. Mitigated by a **dedicated, non-sensitive** unit with a **verified snapshot** before every run and a throwaway account. |

### 9.3 Cross-cutting risks

- **ToS/authorization risk:** observation/instrumentation may violate the game's terms of service; explicit authorization is required for all of §6.
- **Misattribution risk (demonstrated in Phase 5.1):** co-located system/Telegram/Play-Store processes emit the interesting URLs; every observation must be PID/UID-scoped to the game (uid 10341) or it is worthless. Phase 5.2 already corrected this.
- **Absence-of-evidence risk:** a null result on any claim must be reported `INCONCLUSIVE`, never as proof of absence — Photon UDP/DTLS, VPN tunnelling, keyguard locking, and the app's silence (`no Debug.Log`/currency events) all cap what can be concluded.
- **Artifact-integrity risk:** the workspace contains a **mislabelled non-target APK** (`workspace/extracted/customclub.apk` = Myket `ir.mservices.market`, §2.2). Future work must use only the hash-verified Desktop split trio (§2.1).

### 9.4 Verdict

- **Option A:** suitable only for strictly read-only, uid-scoped log/socket observation; unsuitable for the open claims.
- **Option B:** not viable as currently provisioned; contingent on a (this-phase-forbidden) ARM64-execution + root-capability check.
- **Option C:** **recommended**; requires new hardware and a separate privilege authorization.

*Phase 5.3 is complete. No game, device, emulator, APK, app-data, network, or account state was changed. Recommend exactly one next step (§8) and stop for review. Phase 6 is not started.*
