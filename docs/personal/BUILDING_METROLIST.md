# Comprehensive Build Guide: Metrolist Android App

This guide explains how to build Metrolist locally on your development machine or via GitHub Actions CI on your fork.

---

## 1. Prerequisites

### Required Software & SDKs
* **Java Development Kit (JDK):** JDK 21 (Eclipse Temurin, OpenJDK, or Android Studio bundled JDK).
* **Android SDK:**
  * `compileSdk`: **37**
  * `targetSdk`: **36**
  * `minSdk`: **26** (Android 8.0 Oreo)
  * **Build-Tools:** `36.0.0` or higher
  * **NDK:** CMake and NDK with ABI filters: `arm64-v8a`, `armeabi-v7a`, `x86`, `x86_64`.
* **Git:** With submodule support.

---

## 2. Environment Configuration

### Step 2.1: Clone with Submodules
Ensure submodules (such as `metroproto`) are initialized:
```bash
git clone https://github.com/<your-username>/Metrolist.git
cd Metrolist
git submodule update --init --recursive
```

### Step 2.2: Configure Android SDK Location
Create a `local.properties` file in the project root directory:

**On Linux / macOS:**
```properties
sdk.dir=/home/<your-user>/Android/Sdk
```
*(Alternatively, export the environment variable in your shell profile: `export ANDROID_HOME=$HOME/Android/Sdk`)*

**On Windows:**
```properties
sdk.dir=C\:\\Users\\<your-user>\\AppData\\Local\\Android\\Sdk
```

### Step 2.3: Generate Debug Keystore
If you don't already have a persistent debug keystore, generate one in the `app` directory:
```bash
keytool -genkeypair -v \
  -keystore app/persistent-debug.keystore \
  -storepass android \
  -keypass android \
  -alias androiddebugkey \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -dname "CN=Android Debug,O=Android,C=US"
```

---

## 3. Building the App

Metrolist supports three product flavors:
| Flavor | Description | Typical Target |
|---|---|---|
| **foss** (Default) | In-app updater enabled, Google Cast disabled. | F-Droid / GitHub Releases |
| **gms** | Google Cast and in-app updater enabled. | Play Store / GMS devices |
| **izzy** | In-app updater disabled, Google Cast disabled. | Official F-Droid compliance |

### 3.1 Local Debug Builds (Recommended for Testing)

To build the standard FOSS debug APK:
```bash
./gradlew :app:assembleFossDebug
```
**Output APK Path:**
```
app/build/outputs/apk/universalFoss/debug/app-universal-foss-debug.apk
```
*(Or `app/build/outputs/apk/foss/debug/app-foss-debug.apk`)*

To build the GMS variant (with Google Cast):
```bash
./gradlew :app:assembleGmsDebug
```

To build without running linter (faster build):
```bash
./gradlew :app:assembleFossDebug -x lint -x lintFossDebug
```

### 3.2 Running Unit Tests
To verify all unit tests (including `ServerClockTest`, `PlaybackSyncTest`, `MessageCodecTest`):
```bash
./gradlew :app:testFossDebugUnitTest
```

### 3.3 Installing on a Connected Phone or Emulator via ADB
```bash
adb devices
adb install -r app/build/outputs/apk/universalFoss/debug/app-universal-foss-debug.apk
```

To view Listen Together logs in real time via ADB:
```bash
adb logcat -s ListenTogether ListenTogetherManager
```

---

## 4. Building Free via GitHub Actions (On Your Fork)

If your local computer does not have the Android SDK setup, you can build release and debug APKs using GitHub Actions on your fork:

1. Push your branch to your GitHub fork:
   ```bash
   git push origin main
   ```
2. In your browser, open your GitHub repository and click on the **Actions** tab.
3. Click on the **Quick Test Build** workflow on the left sidebar.
4. Click **Run workflow** → select branch `main` → click **Run workflow**.
5. Once completed (typically ~3–5 minutes), download the generated `app-universal-test-release` artifact containing the signed APK directly to your phone.

---

## 5. Troubleshooting Common Build Issues

1. **`SDK location not found`:**
   * Cause: Missing `local.properties` or unset `ANDROID_HOME`.
   * Solution: Add `sdk.dir=/path/to/Android/Sdk` in `local.properties`.
2. **`compileSdk = 37` is not installed:**
   * Cause: Missing Android 16/17 (API 37) preview/release platform.
   * Solution: Run `sdkmanager "platforms;android-37" "build-tools;36.0.0"`.
3. **`Gradle distribution download timeout`:**
   * Solution: Download `gradle-9.7.1-bin.zip` directly from GitHub releases and place it under `~/.gradle/wrapper/dists/gradle-9.7.1-bin/<hash>/`.

