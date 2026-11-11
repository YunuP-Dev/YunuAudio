<div align="center">

# 🔊 YunuAudio V0.1.1 Beta



**Invisible USB Audio Streaming from Windows PC to Android — Powered by ADB**

</div>

---

## 🖼️ Screenshots



---

## 📖 What is YunuAudio?

**YunuAudio** is an ultra-lightweight, portable Windows tool that sends your **PC's sound to an Android phone through the USB cable** — and plays it on the phone's speaker or headphones. No root, no Bluetooth pairing, no Wi-Fi, no virtual audio driver.

It is made of three tiny pieces that work together:

| Piece | What it does | Size |
| --- | --- | :---: |
| 🖥️ **YunuAudio.exe** | One-click AutoIt GUI that controls everything | small |
| 🎙️ **YunuCapture.exe** | Records "what you hear" on the PC via **WASAPI Loopback** and streams it out | ~255 KB |
| 📱 **YunuAudio Receiver (APK)** | A **100% invisible** Android background service that plays the stream | ~12 KB |

The Android side follows the same *invisible app* philosophy as Gnirehtet: **no launcher icon, no screen, no popup**. You can keep gaming, scrolling or watching videos on the phone while the PC's audio flows in the background. The PC controls everything through ADB commands.

Perfect for:
- 🎧 Using your phone as a wired, extra speaker for your PC
- 🔌 Places where Wi-Fi is weak, crowded or unavailable
- 🙅 People who don't want to pair Bluetooth or install virtual cable drivers
- 💻 Old / budget PCs that can't afford heavy audio-routing software

---

## 🔥 What's New in Latest Release 🎉

### 🔊 YunuAudio *(FIRST OFFICIAL RELEASE V0.1.1 Beta 🎉)*

This is the **first official release** of YunuAudio. Here's everything packed into it:



### 🎙️ Built-in Audio Capture — No FFmpeg, No Virtual Cable
- **YunuCapture.exe** records the Windows default output device through **WASAPI Loopback** — a feature built into Windows.
- ❌ No `ffmpeg.exe`
- ❌ No "Stereo Mix" hunting in sound settings
- ❌ No *Screen Capturer Recorder* / virtual-audio-capturer / VB-Cable
- Automatically converts **any** Windows mix format (float32, 16/24/32-bit, mono, stereo, 5.1/7.1, 22.05 kHz – 96 kHz+) to **PCM 16-bit / 44.1 kHz / stereo**. The conversion engine is unit-tested (frequency, amplitude and waveform continuity).
- Waits for the phone to connect *before* it starts recording, so there is no stale audio backlog at start.

### 👻 100% Invisible Android Receiver
- **No icon** in the app drawer, **no activity**, **no popup** — the APK is a single exported foreground service.
- Plays the stream through `AudioTrack` in **low-latency performance mode** with a small read buffer for instant response.
- Foreground service type `mediaPlayback` + partial WakeLock, so audio keeps flowing with the screen off.
- Connects **only to `127.0.0.1`** (the USB bridge). No analytics, no internet access, no ads.
- Controlled entirely by ADB: `am start-foreground-service` to start, `am stopservice` to stop.

### ⚡ Zero-Setup One-Click Pipeline
- **Auto-installs the APK** to the phone the first time you connect, and grants the notification permission on Android 13+.
- Sets up the USB bridge (`adb reverse`) and **verifies** it before continuing.
- Starts the PC capture engine, then starts the phone service — in the correct order, every time.
- **Android 5.0 – 14+** supported (automatic fallback from `start-foreground-service` to `startservice` on old Android versions).

### 🔌 Auto-Connect & Cable Resilience
- **Auto-connect when the USB cable is plugged in** (can be switched off with one checkbox).
- Detects **unplugged cable** instantly and resumes by itself when you plug it back in.
- **Watchdog:** if the stream drops, the capture engine is restarted automatically (up to 5 times, then stops with a clear error instead of looping forever).
- **Service revival:** if the phone's OS kills the background service, YunuAudio starts it again.
- Handles `unauthorized` and `offline` ADB states with clear on-screen instructions.
- A manual **Disconnect** pauses auto-connect until you re-plug the cable, so it never fights you.

### 🛡️ System Hardening
- **Single-Instance Enforcer** (`_Singleton`): two copies can't fight over the same port.
- **Isolated command runner** with strict **timeouts** on every ADB call — no frozen GUI, no hanging pipes.
- Direct process launching (no `cmd /c`), so paths with spaces just work.
- **Clean shutdown:** closing the app stops the phone service, removes the `adb reverse` rule and kills the capture process.
- Log box auto-trims itself to avoid memory growth.

---

## ✨ Key Features

- 🔊 **PC → Phone Audio over USB** — Plug in, click, listen
- 👻 **Invisible Receiver** — No icon, no UI, no interruptions on the phone
- 🎙️ **Built-in WASAPI Capture** — Nothing else to install on the PC
- 🪶 **Tiny & Portable** — The APK is ~12 KB and the capture engine ~255 KB; the rest of the package is the standard Android platform-tools (ADB)
- 🔌 **No Root Required** — Works through standard USB debugging
- 🔁 **Self-Healing** — Auto-reconnect, watchdog, unplug detection
- 🚦 **3-Color Status Box** — Red = idle/error, Orange = working, Green = streaming
- 🖱️ **One-Click GUI** — No manual `adb` commands needed
- 🔒 **Private by Design** — Localhost-only traffic, no internet permission use
- 🖥️ **Windows 10 / 11 (64-bit)**

---

## 🚦 Status Box Guide

| Color | Label shown in the app | Meaning |
| :---: | --- | --- |
| 🔴 | `HP BELUM DICOLOK` | No phone detected — plug in the USB cable |
| 🔴 | `IZINKAN DI LAYAR HP` | Accept the "Allow USB debugging" prompt on the phone |
| 🔴 | `HP OFFLINE` | ADB session is stale — unplug and replug the cable |
| 🔴 | `HP SIAP` | Phone detected, waiting for you to connect (auto-connect off) |
| 🟠 | `MENYAMBUNGKAN...` / `MEMASANG APK...` | Connecting / installing the receiver (first time only) |
| 🟢 | `SUARA TERSAMBUNG KE HP` | Streaming! PC audio is playing on the phone |
| 🔴 | `KABEL DICABUT` | Cable removed — will resume when plugged in again |
| 🔴 | `ENGINE AUDIO ERROR` | The capture engine stopped repeatedly (see Troubleshooting) |

---

## 🧠 How It Works

```
 ┌────────────────────────── PC (Windows) ──────────────────────────┐        ┌──────── Android phone ────────┐
 │                                                                  │        │                               │
 │  Any app playing sound                                           │        │   YunuAudio Receiver (APK)    │
 │          │                                                       │        │   invisible foreground service│
 │          ▼                                                       │        │            ▲                  │
 │  Windows default output                                          │        │            │ AudioTrack       │
 │          │  WASAPI Loopback                                      │  USB   │            │ (low-latency)    │
 │          ▼                                                       │ cable  │   127.0.0.1:8080              │
 │  YunuCapture.exe ──► PCM 16-bit / 44.1 kHz / stereo              │        │            ▲                  │
 │          │                                                       │        │            │                  │
 │   TCP 127.0.0.1:8080 ◄────────────── adb reverse ────────────────┼────────┼────────────┘                  │
 │                                                                  │        │                               │
 │  YunuAudio.exe (GUI) ── controls everything via ADB ─────────────┼───────►│  am start-foreground-service  │
 └──────────────────────────────────────────────────────────────────┘        └───────────────────────────────┘
```

**Audio format (fixed):** PCM signed 16-bit little-endian • 44,100 Hz • 2 channels (stereo)

---

## 🆚 Overview / Approach Comparison

| Feature | 🔊 YunuAudio (USB) | 📶 Wi-Fi Audio Apps | 🎧 Bluetooth |
| --- | :---: | :---: | :---: |
| Needs a Wi-Fi network | ❌ No | ✅ Yes (same network) | ❌ No |
| Needs pairing | ❌ No | ⚠️ Usually server/IP setup | ✅ Yes |
| Virtual audio driver on PC | ❌ No | ⚠️ Varies by app | ❌ No |
| App icon / UI on the phone | ❌ Invisible | ✅ Visible app | — |
| Root Required | ❌ No | ❌ No | ❌ No |
| Works with a weak / no Wi-Fi area | ✅ Yes | ❌ No | ✅ Yes |
| Phone charges while streaming | ✅ Yes (USB) | ⚠️ Only if plugged separately | ⚠️ Only if plugged separately |
| Price | 🆓 Free | ⚠️ Varies | 🆓 Built in |

**Bottom Line:** YunuAudio turns the USB cable you already have into a simple, private audio line — with nothing running on the phone that you can see.

---

## 💻 System Requirements

### PC
- **OS:** Windows 10 or 11 (64-bit)
- **USB:** A working USB port + **data-capable** USB cable
- **Audio:** A working default playback device (speakers / headphones / HDMI audio)

### Android Phone
- **Android 5.0 (API 21) or newer** (built for Android 14 / API 34)
- **USB Debugging** enabled (Settings → Developer Options)
- Some brands also need **"Install via USB"** enabled (Xiaomi / Redmi / POCO, Oppo, Vivo, Realme)

---

## 🚀 Quick Start / Installation Guide

### 🖥️ Windows Installation

1. Download `YunuAudioV0.1.1BetaWindows.zip` from **Releases**.
2. Extract it anywhere (WinRAR, 7-Zip or Windows' built-in extractor).
3. Enable **USB Debugging** on your Android phone (Settings → Developer Options).
4. Connect the phone with a USB cable and **accept the "Allow USB debugging" prompt** (tick *Always allow*).
5. Run `YunuAudio.exe`.
6. The first time, YunuAudio **installs the receiver automatically** — just wait a few seconds.
7. Play any sound on your PC. It now plays on the phone. 🎉

💡 **Tip:** Leave **"Otomatis sambungkan saat kabel USB dicolok"** ticked, and from now on you only need to plug in the cable.

### 📁 Folder Layout

```
YunuAudio/
├── YunuAudio.exe              ← the GUI (compiled from YunuAudio.au3)
├── YunuAudio.au3              ← GUI source code (AutoIt script)
├── YunuAudio.ini              ← optional (change the port)
├── LICENSE
├── README.md
└── Engine/
    ├── adb.exe                ← ADB (required)
    ├── AdbWinApi.dll          ← ADB dependency (required)
    ├── AdbWinUsbApi.dll       ← ADB dependency (required)
    ├── libwinpthread-1.dll    ← ADB dependency (required)
    ├── YunuCapture.exe        ← PC audio capture engine (required)
    ├── YunuAudio.apk          ← invisible Android receiver (required)
    ├── Driver/                ← USB driver files
    ├── USBDeview.exe          ← optional USB device viewer (NirSoft)
    ├── NOTICE.txt             ← Android platform-tools license notice (keep it)
    └── fastboot.exe, etc1tool.exe, sqlite3.exe, mke2fs.exe, ...
                               ← other platform-tools files, not used by YunuAudio
```

> Only `adb.exe` + its DLLs, `YunuCapture.exe` and `YunuAudio.apk` are needed inside `Engine/`. Everything else is optional.

### ⚙️ Optional Settings (`YunuAudio.ini`)

Create `YunuAudio.ini` **next to `YunuAudio.exe`** (not inside `Engine/`):

```ini
[Settings]
Port=8080
```

If you change the port, YunuAudio automatically passes it to the phone service. Valid range: `1024 – 65535`.

---

## ⚙️ Features Explained

### 🎙️ WASAPI Loopback Capture
Windows can hand any program a copy of *whatever the default output device is playing*. `YunuCapture.exe` uses exactly that, so there is no need for "Stereo Mix", a virtual cable or FFmpeg. It listens on `127.0.0.1`, waits for the phone, then streams converted PCM until the phone disconnects. If you change your default output device (for example plugging in Bluetooth headphones), the capture engine restarts by itself and follows the new device.

### 👻 Invisible Android Service
The receiver has **no Activity at all**, so Android shows no icon and never opens a window. It is started and stopped purely by ADB, runs as a `mediaPlayback` foreground service (with a low-priority silent notification so Android doesn't kill it), and reconnects to the PC stream every 500 ms until audio is available.

### 🔁 Self-Healing Pipeline
Every 1 second the GUI checks that the capture engine is alive; every 3 seconds it checks that the cable is still there and that the phone service is still running. Anything that breaks is restarted automatically, and repeated failures stop with a readable message instead of looping forever.

### 🧹 Clean Shutdown
When you press **Disconnect** or close the program, YunuAudio stops the phone service, removes the `adb reverse` rule and ends the capture process — nothing is left running in the background.

---

## 🧰 Manual Control (Advanced Users)

You don't need the GUI — the receiver is plain ADB. Open a Command Prompt inside the `Engine` folder, then:

```bat
:: Install once
adb install -r YunuAudio.apk
adb shell pm grant com.yunuaudio.receiver android.permission.POST_NOTIFICATIONS

:: USB bridge + PC capture engine
adb reverse tcp:8080 tcp:8080
YunuCapture.exe 8080

:: Start / stop the phone service
adb shell am start-foreground-service -n com.yunuaudio.receiver/.AudioService
adb shell am stopservice -n com.yunuaudio.receiver/.AudioService

:: Use another port
adb shell am start-foreground-service -n com.yunuaudio.receiver/.AudioService --ei port 9090

:: Uninstall (there is no icon, so use ADB)
adb uninstall com.yunuaudio.receiver
```

---

## 🛠️ Troubleshooting

**Q: My phone isn't detected?**
A: Enable USB Debugging, use a **data** cable (not charge-only), and install your phone brand's USB/ADB driver if Windows doesn't recognize the device.

**Q: The status says `IZINKAN DI LAYAR HP`?**
A: Look at the phone screen and tap **Allow** on the "Allow USB debugging?" prompt. Tick *Always allow from this computer* so it never asks again.

**Q: The APK fails to install (Xiaomi / Oppo / Vivo / Realme)?**
A: Turn on **"Install via USB"** (and **"USB debugging (Security settings)"** on MIUI/HyperOS) in Developer Options, then click connect again.

**Q: Connected (green) but no sound on the phone?**
A: Make sure something is actually playing on the PC's **default output device** and that the phone's **media volume** is up. If it still fails, run `adb reverse --list` — you should see `tcp:8080`.

**Q: Sound plays on both the PC and the phone?**
A: Correct — YunuAudio records what the PC plays, so the PC keeps playing too. Use headphones on the PC, or lower the PC volume if you only want the phone to be loud (how muting affects the captured signal can depend on your audio driver, so test it once).

**Q: Audio stops after a while or when the screen is off?**
A: Some phone brands (Xiaomi, Oppo, Vivo, Samsung, Huawei) aggressively kill background services. Disable **battery optimization** / set **"No restrictions"** for *YunuAudio* in the phone's settings. YunuAudio will also restart the service by itself when it notices it has stopped.

**Q: `ENGINE AUDIO ERROR`?**
A: The capture engine stopped repeatedly. Check that a playback device is enabled in Windows Sound settings and that no other program is using the same port (change `Port` in `YunuAudio.ini`).

**Q: Windows Defender / my antivirus flags `YunuCapture.exe`?**
A: Small unsigned tools are sometimes flagged by heuristics (false positive). The full source code is in this repository — you can read it, or rebuild the exe yourself (see *Build From Source*).

**Q: Can I run it over Wi-Fi ADB?**
A: No. YunuAudio is designed for the **USB** bridge (`adb reverse`).

**Q: I can't find the app on my phone to uninstall it!**
A: That's by design (it's invisible). Use `adb uninstall com.yunuaudio.receiver`, or Settings → Apps → *Show system apps / All apps* → **YunuAudio**.

---

## 🧱 Build From Source

```
YunuAudio/
├── YunuAudio.au3                  ← AutoIt GUI / controller
├── YunuAudio.exe                  ← compiled GUI
├── LICENSE
├── README.md
├── Engine/                        ← adb (platform-tools) + prebuilt YunuCapture.exe & YunuAudio.apk
└── Source/
    ├── android-receiver/
    │   ├── AndroidManifest.xml
    │   ├── AudioService.java
    │   └── build.gradle
    └── capture-engine/
        ├── YunuCapture.cpp        ← WASAPI loopback + TCP server
        ├── pcm_convert.h          ← format conversion & resampler (pure C++)
        └── test_convert.cpp       ← unit test for the converter
```

- **GUI:** open `YunuAudio.au3` (project root) in SciTE and compile with Aut2Exe (AutoIt 3.3.14+).
- **Android receiver** (`Source/android-receiver/`): create an Android Studio project with *No Activity*, `com.yunuaudio.receiver`, and drop in the three files (`minSdk 21`, `targetSdk 34`, no extra libraries). Build → *APK*, then copy it to `Engine/YunuAudio.apk`.
- **Capture engine** (`Source/capture-engine/`): cross-compile from Linux with MinGW-w64 (or use the same flags on Windows), then copy the result to `Engine/YunuCapture.exe`:
  ```bash
  cd Source/capture-engine
  x86_64-w64-mingw32-g++ -O2 -static -static-libgcc -static-libstdc++ \
      YunuCapture.cpp -o YunuCapture.exe -lws2_32 -lole32 -lwinmm -luuid
  ```
- **Run the converter test** (any platform with g++), from `Source/capture-engine/`: `g++ -O2 test_convert.cpp -o test && ./test`

---

## 🔒 Privacy

- The phone app connects **only to `127.0.0.1`** (the USB bridge to your own PC).
- No analytics, no telemetry, no ads, no account.
- Audio never leaves your cable.

---

## 💬 Community & Support

- 🐛 **Report a Bug:** Open an Issue
- 💡 **Request a Feature / Ask Questions:** Use Discussions
- ⭐ **Enjoying YunuAudio?** Consider starring the repo — it helps a lot!

---

## 📝 License

Other

## 👤 Author

**YunuP-Dev**
_Modified & Engineered by: Yunu – Skanesa_

Built for people who just want their PC's sound on their phone — through a cable, with nothing to see.

"Inspired by the invisible-app architecture of Genymobile/gnirehtet • Powered by Android Debug Bridge (ADB)"
---

**Keywords:** USB audio streaming Windows to Android • PC audio to phone via USB • WASAPI loopback streaming • ADB reverse audio • invisible Android service • use phone as PC speaker • no Bluetooth no Wi-Fi audio • lightweight audio streaming • portable AutoIt tool • no root USB audio • Windows 10/11 audio to Android • YunuAudio
