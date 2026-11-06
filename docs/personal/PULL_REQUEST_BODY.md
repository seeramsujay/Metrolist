# Pull Request: High-Precision Audio Synchronization for Listen Together

## PR Title
```
feat(listentogether): high-precision NTP clock sync, proportional drift correction, and audio latency calibration
```

---

## Description

### Overview
Upgrades the **Listen Together** synchronization engine from a coarse heartbeat model into a **high-precision phase-locked playback system**, enabling synchronized multi-device acoustic playback without echo, flanging, comb-filtering, or audio dropouts.

---

### Key Improvements

#### 1. Minimum-RTT Clock Filter (`ServerClock.kt`)
* Implements Cristian's algorithm (NTP / RFC 5905) using a sliding window of recent RTT samples.
* Because network queueing delay is strictly additive, the packet with the lowest RTT experienced the least queuing delay and represents the most symmetric transit path.
* Clock offset is anchored to the minimum-RTT sample in the window, protecting against asymmetric downstream/upstream delay distortion.
* Detects and adapts to sustained RTT baseline shifts (e.g., cell tower handoffs) after consecutive elevated samples, while rejecting isolated latency spikes.

#### 2. Burst Pings & In-Room Lifecycle (`ListenTogetherClient.kt`)
* Added `triggerSyncBurst()`: Dispatches a 3-ping burst (100 ms apart) on track transitions and buffer completion to refresh clock lock.
* Bursts are tied to the connection lifecycle, coalesce overlapping requests, and terminate on disconnect.
* Uses a 5 s steady-state ping interval while in an active room (25 s when idle) for sub-5 ms clock accuracy with minimal network overhead.

#### 3. Continuous Proportional Drift Controller (`ListenTogetherManager.kt`)
* Tightened `SOFT_SYNC_THRESHOLD_MS` to **4 ms** for acoustic phase alignment.
* Replaced coarse speed jumps with smooth **proportional speed trimming** (up to ±0.5%), closing drift gaps cleanly without pitch artifacts or buffer re-initialization.
* Unconditionally restores nominal speed upon entering the 4 ms deadband.
* Dynamically detects external user speed/pitch adjustments (e.g. from `SpeedDialog`) and preserves the user's preference as the baseline.
* Suspends drift seeks and speed trims while the player is buffering (`Player.STATE_BUFFERING`) to avoid interrupting recovery.
* Halts drift correction and restores base speed if the server clock is lost or upon disconnect.

#### 4. Hardware Audio Latency Calibration (`ListenTogetherSettings.kt`, `PreferenceKeys.kt`)
* Added manual latency offset calibration slider (±150 ms) in Listen Together settings to compensate for device-specific DAC, Bluetooth, or speaker hardware latency differences.
* Passed raw reference positions through track reconciliation so calibration is applied exactly once during recovery.
* Added localized English strings to `metrolist_strings.xml`.

#### 5. Regression & Unit Test Coverage (`ServerClockTest.kt`)
* Added unit tests verifying:
  - Isolation from transient jitter spikes.
  - Immunity to clock shift under asymmetric one-way delays when a lower-RTT sample is present.
  - Adaptation to sustained RTT increases.
* 100% KDoc docstring coverage on all touched functions.

---

### Verification & Quality Assurance
- [x] Follows Material 3 and Metrolist Kotlin style conventions.
- [x] No database migrations or external dependencies introduced.
- [x] All touched functions with cyclomatic complexity ≥ 2 documented with KDoc.
- [x] Unit tests pass for clock synchronization and latency adaptation.
