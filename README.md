# BMW E90 Android head unit (825X_Pro / AC8257): hidden menus, rear camera, notes

Field notes from getting an aftermarket rear camera to work on a Chinese 10.25" Android head unit
for the BMW E90 (CCC, no factory navigation). Everything here was read from the unit itself over ADB
(read-only) or from its own settings app, and then tested in the car.

> 🇷🇴 Rezumat în română: [README.ro.md](README.ro.md)

**Is this your unit?** Check *Settings → About Device*. Mine shows:

| Field | Value |
|---|---|
| HMI | `BMW.G5.D.Q.F.01.2021.06.07.09.30` |
| MCU | `TsBMW.210804` |
| CAN | `YX-BMW-2 V203` |
| MEDIA | `HC_v2.0.0_210506_1600(v1.84)` |

Over ADB it reports `ro.product.model = 825X_Pro`, `ro.hardware = ac8257` (AutoChips), and the UI is
the `com.ts.MainUI` app. Sold under many names (mine came as "AUTOTOP" on AliExpress; the same board
is sold as Alps / Tmotec / Junsun and others). If your MCU string starts with `TsBMW` and your CAN with
`YX-BMW`, most of this probably applies. Other firmware versions may differ.

---

## TL;DR: rear camera shows stripes, a black screen, or a black-and-white tiled image

On this BMW profile the head unit has **two** camera inputs:

| RCA on the harness | What it really is |
|---|---|
| **CAM IN** | CVBS (analog PAL/NTSC), decoded by the SoC |
| **AHD** (label with Chinese characters) | HD analog input, used by the "360 / AVM" path, **720p only** |

The menu *Rear Camera Type* looks like it offers AHD, but on this BMW profile **"AHD" means 720p at 25 fps**.
A 1080p AHD camera on the AHD input gives stripes or a broken tiled image, and **there is no menu option
that makes 1080p stick** (see [why](#why-the-camtype-keeps-going-back-to-4)).

What works:

- **AHD 720p 25 fps camera** → AHD input, *Rear Camera Type* = **Aftermarket Rear Camera (AHD)**. (This is the
  path the firmware was built for; I did not have such a camera to confirm it.)
- **Any camera that can output CVBS** → **CAM IN**, *Rear Camera Type* = **Aftermarket Rear Camera** (no
  AHD/AVM). ✅ Confirmed in the car, color, full frame, survives reboots.

### The camera I used: Excitat NG06064 (AHD 1080p, sold on eMAG.ro)

It has three wire loops on its pigtail. The listing does not say what they do. What I observed:

| Loops cut | On the AHD input (CamType 4) | On CAM IN (CVBS) |
|---|---|---|
| none (default, 1080p) | fine horizontal stripes | — |
| brown | black-and-white image, tiled 3×3 | nothing |
| white | no picture at all | "No signal" |
| **white + brown** | — | ✅ **color image** |

Green left connected (cutting it alone changed nothing visible on the AHD input).
Cut loop ends must be insulated **separately**.

> ⚠️ The camera's own power plug made poor contact on mine: the picture dropped out and the connector got
> **hot** (the camera draws only ~0.25 A, so a good contact stays cold). Cut it off and solder the wires,
> with a 1 A inline fuse on the positive wire.

---

## Hidden factory menu and other codes

Codes are typed into a numeric dialog. In the code, the first row of *Settings → About Device* (HMI) opens it
(`SettingEquipmentActivity` → `SettingNumInuptDlg`). The values below are the `custom_*_num` strings inside
`com.ts.MainUI` on this firmware, so a different firmware may use different numbers.

| Code | What it does | |
|---|---|---|
| **8878** | **Factory Setup** (full menu: CAN, Radio, Display, Backcar Setup, Other, …) | ✅ used |
| **8888** | Factory Setup, short "shop" variant (still has Backcar Setup, CAN, Display, Boot Logo) | verified in code |
| **5678** | Boot logo app (`com.ts.logoset`), only if the option switch is enabled | verified in code |
| 1234 / 4321 | copy / delete the demo file in the UI's demo folder | verified in code |
| 9 / 12 | authorization screen / serial-number dialog | verified in code |
| **9090, 9191** | named `destroy_id` / `destroy_id_root` | ⛔ **do not use** |
| 0, 0128, 4, 5, 180, 1701 | named `mcuupdate`, `can_update`, `powerreset`, `setvcom`, `settouch`, `setall` | ⛔ avoid |
| 1101, 1313, 1414, 1515, 1616, 2134, 6666, 0127, 0129, 160 | named screenshot, radio_reset, reboot, testmode, display, appinfo, install, can_keystudy, can_data, getbrance | not tested |

In **Factory Setup**, the button **Device ID** wipes the system. Don't touch **CAN**, **MCU update**,
car type or gear settings unless you know what you're doing. Take photos of every screen before changing
anything.

### Backcar Setup (8878 → Backcar Setup)

| Item | Meaning (from the code) |
|---|---|
| **CamType** | the real camera format (table below) |
| FastAvm | fast 360 view; turned on automatically when CamType = 3 |
| **Camera detect** | *not* signal detection: whether a camera is installed. **Detect** = switch to the camera in reverse; Not detect = parking sensors only |
| Camera mirror / F-CAM mirror | mirror rear / front camera |
| back option | camera option on/off |
| Reverse mode | CAM+OPS / CAM / OPS … (camera, parking sensors, or both) |
| Park mute | how much the music is lowered in reverse |
| Line Type / Line Pos | parking guide lines |

Press **Save**. The "please reboot" message can be dismissed: the camera settings apply immediately.

---

## CamType values

What each CamType writes to the unit's config partition ("metazone"), from `FsBaseActivity.ReSetBackCarsource`:

| CamType | Label | Source (`65607`) | Format |
|---|---|---|---|
| 0 | CVBS | 1 = CVBS (CAM IN) | |
| 1 | AHD-720P-25 | 2 = HD | AHD mode 3 |
| 2 | AHD-1080P-25 | 2 = HD | AHD mode 5 |
| 3 | 360-720P | 3 = AVM | AVM mode 3 (+FastAVM) |
| **4** | **360-720P-Rear** | **3 = AVM** | **AVM mode 3** |
| 6 | AHD-720P-30 | 2 = HD | AHD mode 4 |
| 7 | AHD-1080P-30 | 2 = HD | AHD mode 6 |
| 8 | TVI-720P-25 | 3 = AVM | AVM mode 9 |
| 9 | TVI-720P-30 | 3 = AVM | AVM mode 10 |
| 10 | 360-720P-F+R | 3 = AVM | AVM mode 3 |
| 11 | 360-720P-30-Rear | 3 = AVM | AVM mode 4 |

Mode numbers line up as: 3 = 720p25, 4 = 720p30, 5 = 1080p25, 6 = 1080p30, 9/10 = TVI 720p25/30.

**Observed on my unit:** the AHD RCA is wired to the **AVM** path. With CamType 2 (HD path), the camera service
started (`sourceType 2 ==> start`) and the screen stayed **completely black**. With CamType 4 the same camera
gave a picture (garbled, but a picture). So CamType 1, 2, 6 and 7 can't show the AHD input on this unit, and
the AVM path has no 1080p option.

### Why the CamType keeps going back to 4

The BMW CAN profile (CAN type 176, `CanBmwZmytWithCDCarInitView`) **rewrites CamType on every cold boot**
(`CanFunc.CarCanInit` → `CanBmwZmytWithCDExdActivity.Init()`) from the simple *Rear Camera Type* menu:

| *Rear Camera Type* | CamType after boot |
|---|---|
| Aftermarket Rear Camera | **0** (CVBS) |
| Original Rear Camera | 5 |
| Aftermarket Rear Camera (AVM) | 3 |
| Aftermarket Rear Camera (AHD) | **4** |

It runs when the UI app starts. Within the same session a CamType set in Backcar Setup stays (I saw it hold
for 10+ minutes), and after the next full boot it's back to the mapped value. That's why it seems to reset
"randomly". (Waking from QuickBoot standby probably doesn't restart the app, but I haven't verified that.)
For a permanent setting, pick it through the simple menu: CVBS → 0, AHD → 4.

---

## Wiring the camera on an E90

- Power it from the **reverse lamp** so it is only on in reverse. The E9x light module checks bulbs with short
  pulses, which can make a camera flicker or trigger bulb-out warnings, so switch the camera with a **relay**:

```
 reverse lamp (+) ──────────► 86 ┐
                                 │ relay coil      (add a 1N4007 across 86/85,
 ground ─────────────────────► 85 ┘                  stripe towards 86, if the
                                                     relay has no built-in diode)
 +12 V ── fuse 2 A ──────────► 30 ┐
                                 │ contact (closes in reverse)
 camera (+) ◄── fuse 1 A ──── 87 ┘

 camera (−) ─────────────────► ground (clean body bolt)
```

- Find the reverse-lamp positive with a multimeter (12 V only with reverse engaged). Don't trust wire colors.
- Solder and heat-shrink every joint. No cheap plugs: see the hot-connector warning above.
- The camera needs 1–2 s to start, so a moment of "No signal" when engaging reverse is normal.

---

## Other things found on this unit

- **It runs Android 9**, not 11: `ro.build.version.release` says 11, but `ro.build.version.sdk = 28` and the build
  id is `PPR1…` (Pie). Build date April 2020, `eng`/`test-keys`.
- **Sleep mode** (*Settings → General*) is the QuickBoot standby switch: **On** = fast start (standby),
  **Off** = full shutdown. The 10/20/30/60-minute strings exist in the app but this version only offers On/Off.
  *ACC Delay* (Factory Setup → Other, 0–10) is a short delay before shutdown, not standby length.
- **Built-in 4G modem with a SIM slot**: LTE baseband, dual-SIM configuration, slot 1 sees the network
  ("emergency only" without a SIM). A data SIM gives the unit its own internet while wireless CarPlay uses its Wi-Fi.
- **Native CarPlay is wired only** (`ro.atc.carplay.wireless = 0`). Wireless CarPlay goes through the *AutoLink*
  app, which takes over the unit's Wi-Fi.
- Camera decoder driver: Techpoint **TP2850** (`/proc/driver/camsensor`), plus TP2854 (4-channel) tuning libs
  in `/vendor/lib`. Reading the TP2850 registers from Android returned only zeros, even in reverse. The reverse
  camera seems to be handled by the secondary core (the logs mention "ARM2"), but I haven't confirmed that.

---

## Reading your own unit over ADB (read-only)

1. *Android Settings → About → tap Build number 7×* → Developer options → **USB debugging** on.
2. Put the unit and a laptop on the same Wi-Fi (a phone hotspot works). Wireless CarPlay/AutoLink must be off,
   otherwise the unit has no Wi-Fi. A USB cable to a laptop didn't work for me on either port, including the
   CarPlay one. They seem to be host-only.
3. `adb connect <unit-ip>:5555`

Useful, harmless reads:

```sh
adb shell getprop ro.product.model
adb shell cat /vendor/etc/atc_camera_config.xml
# active CamType and camera path, logged when you engage reverse (engine running):
adb shell "logcat -d | grep -E 'nSigType|sourceType'"
```

> ⚠️ **Turn ADB off when you're done.** This firmware is signed with the public AOSP test keys
> (platform cert SHA-1 `27:19:6E:38:6B:87:5E:76:AD:F7:00:E7:EA:84:E4:C6:EE:E3:3D:FA`) and SELinux is
> permissive, so anyone on the same Wi-Fi with ADB access has a lot of control over the unit.

---

## Disclaimer

Notes from one unit and one car. Firmware varies between sellers and batches. Changing factory settings can
break CAN integration (steering wheel keys, reverse detection) or worse. Photograph every screen before you
change it. No vendor files are included here, only my own notes.

Text licensed under [CC BY 4.0](LICENSE).
