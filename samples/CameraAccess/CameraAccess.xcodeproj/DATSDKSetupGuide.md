# Meta Ray-Ban Glasses + DAT SDK Setup Guide
## CameraAccess App — Complete Configuration Reference

---

## Section 1: Meta Account & Device Setup

### 1.1 Account Requirements
The most critical requirement is that **one single Meta account** must link everything together:

```
sinhar71@gmail.com (Meta Account)
    ├── Facebook Profile → Rajnish Sinha
    ├── Meta AI Profile  → rajnish.sinha.982
    └── Devices          → Ray-Ban Meta (Wayfarer 0326)
```

### 1.2 Initial Glasses Setup
1. Download the **Meta AI app** from the App Store
2. Log in with your **Facebook/Meta account email**
3. If prompted to create a Meta AI username, choose one (e.g. `rajnish.sinha.982`)
4. Pair your Ray-Ban Meta glasses via Bluetooth following the in-app instructions
5. Confirm glasses appear under **Profile → Devices** in Meta AI

### 1.3 Factory Reset Glasses (if switching accounts)
If your glasses were previously registered to a **different Meta account**, a factory reset is required:
1. Power off the glasses (slide switch to red)
2. Hold the **capture button** while powering on
3. Follow the reset procedure in the Meta AI app
4. Re-pair with the correct account

> ⚠️ **Critical:** Glasses must be registered to the **exact same account** logged into the Meta AI app. A mismatch causes silent "Connecting…" hangs.

### 1.4 Enabling Developer Mode in Meta AI
Developer Mode must be enabled and **re-enabled after every logout**:

1. Open **Meta AI app**
2. Tap your **profile photo** (bottom right)
3. Tap the **gear icon ⚙️** → Settings
4. Navigate to **About** or the app version screen
5. **Tap the build number 5 times rapidly** — Developer Mode unlocks
6. Confirm the **Developer Mode toggle** is now visible and **ON**

> ⚠️ Developer Mode resets every time you log out of Meta AI. Always re-enable it after logging back in.

### 1.5 Verifying Everything is Linked Correctly
In the **Facebook app**:
- Menu → Settings → scroll to **"Accounts Center"**
- Confirm Facebook, Meta AI, and your glasses all appear under one account

---

## Section 2: Registering Your App on Meta Developer Portal

### 2.1 Create a Developer Account
1. Go to **[developers.facebook.com](https://developers.facebook.com)**
2. Log in with your **Meta/Facebook account** (`sinhar71@gmail.com`)
3. Accept the developer terms if prompted

### 2.2 Create Your App
1. Click **"My Apps" → "Create App"**
2. Choose **"Other"** as the use case
3. Select **"Consumer"** as the app type
4. Enter your app name (e.g. `AthleteCoach`)
5. Enter your **contact email**
6. Click **"Create App"**

### 2.3 Add the Wearables/DAT SDK Product
1. In your app dashboard, click **"Add Product"**
2. Find and add **"Wearables API"** or **"DAT SDK"**
3. Follow the setup wizard

### 2.4 Get Your Credentials
Navigate to **Settings → Basic**:

| Credential | Location | Used For |
|---|---|---|
| **App ID** (`META_APP_ID`) | Settings → Basic → App ID | DAT SDK initialization |
| **Client Token** | Settings → Advanced → Client Token | DAT SDK authentication |

> ⚠️ **Never hardcode these values** directly in source files. Store them as Xcode Build Settings variables (see Section 4).

### 2.5 Configure for Development Testing
Before App Review is approved, add yourself as a developer/tester:
1. Go to **Roles → Developers**
2. Add your Meta account (`sinhar71@gmail.com`)
3. This allows real glasses testing without full App Review approval

### 2.6 App Review (for distribution)
To distribute to users beyond your developer account:
1. Go to **App Review → Permissions and Features**
2. Request the **camera access / wearables API** permission
3. Provide usage description and demo video
4. Submit for review

---

## Section 3: Apple Developer Account & Entitlements

### 3.1 Why a Paid Account is Required
A **free Personal Team** account blocks the entitlements needed for glasses Wi-Fi streaming:

| Entitlement | Personal Team | Paid Account |
|---|---|---|
| `HotspotConfiguration` | ❌ Blocked | ✅ Available |
| `Access WiFi Information` | ❌ Blocked | ✅ Available |

Without these, the app hangs on `Stream: waitingForDevice` indefinitely.

### 3.2 Enrolling in the Apple Developer Program
1. Go to **[developer.apple.com/enroll](https://developer.apple.com/enroll)**
2. Sign in with your **Apple ID**
3. Choose **"Enroll as Individual"**
4. Pay the **$99/year** fee
5. Wait for activation (usually instant to 24 hours)
6. Your **Team ID** will be assigned (e.g. `5636BJ6W55`)

### 3.3 Adding Required Entitlements in Xcode
Once enrolled in the paid program:

1. Select your target → **Signing & Capabilities**
2. Click **+ Capability**
3. Search for and add **"Hotspot Configuration"**
4. Search for and add **"Access WiFi Information"**

These automatically add entries to your `.entitlements` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)$(CFBundleIdentifier)</string>
    </array>
    <key>com.apple.developer.networking.HotspotConfiguration</key>
    <true/>
    <key>com.apple.developer.networking.wifi-info</key>
    <true/>
</dict>
</plist>
```

---

## Section 4: Xcode Project Configuration

### 4.1 Signing & Capabilities
| Setting | Value |
|---|---|
| Team | Your paid account (e.g. `Rajnish Sinha (Individual)`) |
| Bundle Identifier | `com.yourname.AppName` (no "personal" in the ID) |
| Provisioning Profile | Xcode Managed Profile |
| Capabilities | Hotspot Configuration, Access WiFi Information |

> ⚠️ If you previously used a Personal Team, the old bundle ID contains `.personal.` — change it to remove that segment to avoid duplicate app installs.

### 4.2 Build Settings — User-Defined Variables
Add these under **Build Settings → + → Add User-Defined Setting**:

| Variable | Value |
|---|---|
| `META_APP_ID` | Your App ID from Meta Developer portal |
| `CLIENT_TOKEN` | Your Client Token from Meta Developer portal |
| `DEVELOPMENT_TEAM` | Your Apple Team ID (e.g. `5636BJ6W55`) |

### 4.3 Info.plist — Required Keys
All of these must be present:

```xml
<!-- DAT SDK Configuration -->
<key>MWDAT</key>
<dict>
    <key>AppLinkURLScheme</key>
    <string>yourapp://</string>
    <key>ClientToken</key>
    <string>$(CLIENT_TOKEN)</string>
    <key>MetaAppID</key>
    <string>$(META_APP_ID)</string>
    <key>TeamID</key>
    <string>$(DEVELOPMENT_TEAM)</string>
</dict>

<!-- URL Scheme for OAuth callback -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>yourapp</string>
        </array>
    </dict>
</array>

<!-- Privacy descriptions -->
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Needed to connect to Meta AI Glasses</string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>Needed to connect to Meta AI Glasses</string>
<key>NSCameraUsageDescription</key>
<string>Needed to stream from your glasses camera</string>
<key>NSMicrophoneUsageDescription</key>
<string>Needed to record audio from your glasses</string>
<key>NSLocalNetworkUsageDescription</key>
<string>Needed to connect to your glasses over Wi-Fi</string>

<!-- Required for glasses WiFi discovery -->
<key>NSBonjourServices</key>
<array>
    <string>_bonjour._tcp</string>
    <string>_rbmeta._tcp</string>
    <string>_rbmeta._udp</string>
</array>

<!-- Background modes -->
<key>UIBackgroundModes</key>
<array>
    <string>bluetooth-central</string>
    <string>bluetooth-peripheral</string>
    <string>audio</string>
    <string>processing</string>
</array>
```

### 4.4 Required Frameworks (General → Frameworks)
| Framework | Status |
|---|---|
| `MWDATCore` | Required — core SDK |
| `MWDATCamera` | Required — camera streaming |
| `MWDATMockDevice` | Required (DEBUG only) — mock testing |

> ⚠️ Duplicate symbol warnings between `MWDATCamera` and `MWDATMockDevice` are **expected and harmless** — both frameworks are needed.

### 4.5 iPhone Permissions to Grant on First Launch
When running the app for the first time on a fresh install, grant all prompts:

| Permission | When prompted | Required for |
|---|---|---|
| **Bluetooth** | On first launch | Glasses connection |
| **Local Network** | On first launch | Glasses Wi-Fi streaming |
| **Camera** | When tapping Preview | Mock device testing |
| **Microphone** | When recording | Audio capture |

---

## Section 5: Troubleshooting Guide

### 5.1 App stuck on "Connecting…" / never registers

| Check | How to verify | Fix |
|---|---|---|
| Meta AI logged in with correct account | Profile → check email shown | Log out and back in with glasses account email |
| Glasses registered to same account | Facebook → Settings → Devices | Factory reset glasses and re-pair |
| Developer Mode enabled | Meta AI → Settings → build number | Tap build number 5x to re-enable |
| Meta AI app installed | Check App Store | Install Meta AI app |

### 5.2 Session starts but Stream stuck on "waitingForDevice"

| Check | How to verify | Fix |
|---|---|---|
| Local Network permission | Settings → Apps → CameraAccess → Local Network | Enable it |
| Hotspot Configuration entitlement | Signing & Capabilities | Add capability (paid account required) |
| Old duplicate app install | Settings → Apps (look for two CameraAccess entries) | Delete old app, fresh install from Xcode |
| Glasses broadcasting WiFi | Settings → WiFi → look for "RB Meta XXXX" | Power cycle glasses |
| Meta AI running in background | App switcher | Keep Meta AI running in background |

### 5.3 "Device is not connected" error after Meta AI redirect

| Check | How to verify | Fix |
|---|---|---|
| App switched to background during WiFi join | Check if Meta AI opened mid-stream | End session, restart session, tap Preview again — permission now cached |
| Glasses went to sleep during app switch | LED/tone check | Keep glasses on face, try again immediately |
| Permission now cached | Logs show "Camera permission status: granted" | Should go straight to streaming without redirect |

### 5.4 Build errors after adding entitlements

| Error | Cause | Fix |
|---|---|---|
| `Provisioning profile doesn't include entitlement` | Personal Team account | Upgrade to paid $99 Apple Developer account |
| `No such module 'MWDATMockDevice'` | Missing xcframework file | Ensure `MWDATMockDevice.xcframework` exists at package root |
| Duplicate class warnings in console | Both MWDATCamera and MWDATMockDevice embedded | Expected — ignore these warnings |

### 5.5 iPhone Settings to verify

Go to **Settings → Apps → CameraAccess** and confirm:
- ✅ **Bluetooth** — ON
- ✅ **Local Network** — ON
- ✅ **Camera** — ON
- ✅ **Microphone** — ON
- ✅ **Background App Refresh** — ON

Go to **Settings → Privacy & Security → Local Network**:
- ✅ **CameraAccess** — ON

### 5.6 Meta AI settings to verify

1. **Profile → Devices** — glasses show as **"Connected"**
2. **Developer Mode** — enabled (tap build number 5x if missing)
3. **App permissions** — CameraAccess shows "Always allow" for camera
4. **Firmware** — no updates pending for glasses

### 5.7 Glasses physical state checklist
Before tapping Preview, always verify:
- 👓 Glasses **on your face**
- 📂 Arms fully **unfolded**
- 🔋 Battery **above 20%**
- 🔊 **Startup chime** heard after powering on
- 📱 **Meta AI app running** in background (not force-quit)

---

## Quick Reference Checklist

### First-time setup
- [ ] One Meta account linked to Facebook + Meta AI + glasses
- [ ] Developer Mode enabled in Meta AI
- [ ] App registered at developers.facebook.com
- [ ] `META_APP_ID` and `CLIENT_TOKEN` obtained
- [ ] Paid Apple Developer account enrolled
- [ ] `HotspotConfiguration` + `Access WiFi Information` capabilities added
- [ ] Bundle ID set (no `.personal.` in the ID)
- [ ] `META_APP_ID`, `CLIENT_TOKEN`, `DEVELOPMENT_TEAM` in Build Settings
- [ ] All `Info.plist` keys present
- [ ] Fresh install on device (no duplicate app entries)
- [ ] All iPhone permissions granted on first launch

### Every time you test
- [ ] Glasses on face, unfolded, charged
- [ ] Meta AI app running in background
- [ ] Developer Mode still enabled in Meta AI
- [ ] Local Network permission ON in iPhone Settings

---

*Document prepared based on hands-on debugging session — October 2026*
