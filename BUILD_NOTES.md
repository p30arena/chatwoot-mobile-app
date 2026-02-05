## Build notes (local + release)

### Environment basics

- `.env` is read by Expo/JS, not Gradle. If you run Gradle directly, export env vars in the command.
- Example (from repo root):
  ```
  SENTRY_DISABLE_AUTO_UPLOAD=true NODE_ENV=production \
  ANDROID_HOME=$HOME/Library/Android/sdk JAVA_HOME=$(/usr/libexec/java_home -v 17) \
  ./android/gradlew -p android assembleRelease
  ```

### Release signing (Android)

- Generate keystore:
  ```
  keytool -genkeypair -v \
    -keystore android/app/chatwoot-release.keystore \
    -alias chatwoot \
    -keyalg RSA -keysize 2048 -validity 10000
  ```

- Keystore: `android/app/chatwoot-release.keystore`
- `android/gradle.properties` contains:
  ```
  MYAPP_UPLOAD_STORE_FILE=chatwoot-release.keystore
  MYAPP_UPLOAD_KEY_ALIAS=chatwoot
  MYAPP_UPLOAD_STORE_PASSWORD=...
  MYAPP_UPLOAD_KEY_PASSWORD=...
  ```
- `android/app/build.gradle` uses `signingConfigs.release`.

### Size optimizations

- Enabled:
  - `android.enableProguardInReleaseBuilds=true`
  - `android.enableShrinkResourcesInReleaseBuilds=true`
- ABI splits enabled in `android/app/build.gradle` (no universal APK).

### Sentry (local releases)

- Disable sourcemap uploads:
  - `android/gradle.properties`: `SENTRY_DISABLE_AUTO_UPLOAD=true`

### Release build command

```
SENTRY_DISABLE_AUTO_UPLOAD=true NODE_ENV=production \
ANDROID_HOME=$HOME/Library/Android/sdk JAVA_HOME=$(/usr/libexec/java_home -v 17) \
./android/gradlew -p android assembleRelease
```

### Android dev build

```
ANDROID_HOME=$HOME/Library/Android/sdk JAVA_HOME=$(/usr/libexec/java_home -v 17) pnpm run:android
```

### Gradle cache/lock recovery

Use when Gradle gets stuck or refuses to stop:

```
rm -f ~/.gradle/wrapper/dists/gradle-8.10.2-all/*/gradle-8.10.2-all.zip.lck
cd android
ANDROID_HOME=$HOME/Library/Android/sdk JAVA_HOME=$(/usr/libexec/java_home -v 17) ./gradlew --stop
ANDROID_HOME=$HOME/Library/Android/sdk JAVA_HOME=$(/usr/libexec/java_home -v 17) ./gradlew app:assembleDebug --refresh-dependencies
```

### Metro bundler

- Axios in RN must resolve to browser build.
- `metro.config.js` forces `axios` → `dist/browser/axios.cjs`.

### FFmpeg (Android)

- Official `ffmpeg-kit` repo is unreachable (NXDOMAIN for `maven.arthenica.com`).
- Android uses community fork:
  - `io.github.jamaismagic.ffmpeg:ffmpeg-kit-main-16kb:6.1.4`
- Patch is applied via `patches/ffmpeg-kit-react-native.patch`.

---

## Rebranding & configuration

### Base URL defaults + lock

- Set defaults in `.env`:
  ```
  EXPO_PUBLIC_CHATWOOT_BASE_URL=https://your.chatwoot.domain
  EXPO_PUBLIC_LOCK_BASE_URL=true
  ```
- When locked:
  - The URL entry screen auto-connects to the env base URL.
  - Users cannot change it at runtime.

### App name / identifier

Edit `app.config.ts`:
- `name` (display name)
- `slug`
- `ios.bundleIdentifier`
- `android.package`

### Icons / splash

Update assets:
- `assets/icon.png`
- `assets/adaptive-icon.png`
- `assets/splash.png`

Config in `app.config.ts`:
- `icon`
- `android.adaptiveIcon`
- `splash`

### App branding strings

Search in `src/i18n` for app-facing strings if you want to replace "Chatwoot".
