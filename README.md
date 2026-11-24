# ✋ Control PC using Hand Gesture (J.A.R.V.I.S Gesture Interface)

Control your computer's mouse with nothing but your hand and a webcam. No special hardware, no gloves, no markers. Just Python, OpenCV and Google's MediaPipe hand tracking.

Move the cursor, click, right-click, scroll, and drag-and-drop, all with hand gestures, while a sci-fi style HUD shows what the system sees.

---

## 📑 Table of Contents

1. [Features](#-features)
2. [How It Works](#-how-it-works)
3. [Requirements](#-requirements)
4. [Installation](#-installation)
5. [Running the App](#-running-the-app)
6. [Gesture Guide](#-gesture-guide)
7. [On-Screen HUD](#-on-screen-hud)
8. [Safety & Emergency Stop](#-safety--emergency-stop)
9. [Configuration & Tuning](#-configuration--tuning)
10. [Project Structure](#-project-structure)
11. [Architecture](#-architecture)
12. [Legacy V1/V2 Modules](#-legacy-v1v2-modules)
13. [Troubleshooting](#-troubleshooting)
14. [Tips for Best Results](#-tips-for-best-results)
15. [Known Limitations](#-known-limitations)
16. [Roadmap](#-roadmap)
17. [Contributing](#-contributing)
18. [Credits](#-credits)

---

## ✨ Features

- **Air-mouse cursor control**: the cursor moves *relative* to your hand's motion (like lifting a real mouse), so you can't lose track of where your hand "should" be.
- **Left click, right click, scroll, and drag-and-drop** using simple hand poses.
- **One Euro Filter smoothing**: an adaptive filter that removes hand jitter when you move slowly but stays responsive when you move fast.
- **Physics-style cursor**: dead zone (anti-tremor), motion smoothing, and speed-based acceleration for precise slow moves and quick long moves.
- **Cinematic HUD**: finger trail, rotating reticle, and a live status label showing the current gesture.
- **Safety systems**: automatic drop of a held drag when your hand leaves the frame, click cooldowns, low-FPS safety pause, and a quit key.
- **Runs locally**: all processing happens on your machine. No video is uploaded anywhere.
- **Fully configurable** through a single `gesture_v3/config.py` file.

---

## 🧠 How It Works

```
Webcam ─▶ Mirror/Convert ─▶ MediaPipe Hand Landmarker ─▶ One Euro Filter
                                      │                          │
                                      ▼                          ▼
                              Gesture Classifier         Hand movement (Δx, Δy)
                                      │                          │
                                      └────────────┬─────────────┘
                                                   ▼
                                       State machine + Physics cursor
                                                   ▼
                                  PyAutoGUI (move / click / scroll / drag)
                                                   ▼
                                       HUD drawn on the camera preview
```

1. OpenCV grabs frames from your webcam and mirrors them so movement feels natural.
2. MediaPipe's **Hand Landmarker** finds 21 landmarks (knuckles, joints, fingertips) on your hand.
3. The **index-finger knuckle (landmark 5)** is tracked as the movement anchor because it is the steadiest part of the hand. Its position is smoothed with a One Euro Filter.
4. The frame-to-frame change in that position (`Δx`, `Δy`) drives the cursor through the physics layer.
5. The **gesture classifier** checks which fingers are up and how close fingertips are, and outputs a state: `MOVE`, `CLICK_LEFT`, `CLICK_RIGHT`, `SCROLL`, `FIST`, or `IDLE`.
6. A small state machine turns those states into real mouse actions via PyAutoGUI.

---

## 📋 Requirements

| Requirement | Details |
|---|---|
| **Python** | 3.9 or newer. The project was developed on Python 3.13. If `pip` can't find a MediaPipe build for your Python version, use Python 3.10–3.12. |
| **Webcam** | Any built-in or USB webcam. 720p is requested; lower resolutions still work. |
| **OS** | Windows 10/11 (recommended), macOS, or Linux on X11. Linux on Wayland is not supported by PyAutoGUI. |
| **Libraries** | `opencv-python`, `mediapipe`, `numpy`, `pyautogui` (listed in `requirements.txt`) |
| **Model file** | `hand_landmarker.task` (already included in the repo, about 7.8 MB) |
| **Lighting** | A reasonably well-lit room. The hand should be clearly visible to the camera. |

---

## 🛠 Installation

### 1. Get the code

```bash
git clone https://github.com/<your-username>/Control_pc_using_Hand-Gesture.git
cd Control_pc_using_Hand-Gesture
```

Or download the ZIP from GitHub, extract it, and open a terminal inside the extracted folder.

### 2. Create a virtual environment (recommended)

**Windows (PowerShell / CMD):**
```powershell
python -m venv venv
venv\Scripts\activate
```

**macOS / Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Check the model file

Make sure `hand_landmarker.task` sits in the project root, next to `main_v3.py`. It ships with the repo. If it's missing or corrupted, download it again:

**Windows (PowerShell):**
```powershell
Invoke-WebRequest -Uri https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task -OutFile hand_landmarker.task
```

**macOS / Linux:**
```bash
curl -L -o hand_landmarker.task https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task
```

### 5. Platform-specific setup

<details>
<summary><b>🪟 Windows</b></summary>

Nothing extra is needed. If Windows asks for camera permission, allow it
(Settings → Privacy & security → Camera → allow desktop apps).
</details>

<details>
<summary><b>🍎 macOS</b></summary>

macOS blocks camera access and mouse control by default. Grant both to the app you run Python from (Terminal, iTerm, VS Code, PyCharm, etc.):

- **System Settings → Privacy & Security → Camera** → enable your terminal/IDE
- **System Settings → Privacy & Security → Accessibility** → enable your terminal/IDE

Then fully quit and reopen that terminal/IDE.
</details>

<details>
<summary><b>🐧 Linux</b></summary>

PyAutoGUI needs an X11 session and a few system packages:

```bash
sudo apt update
sudo apt install python3-tk python3-dev
```

Log in using an **"Xorg" / "X11"** session instead of Wayland (choose it from the gear icon on the login screen).
</details>

---

## ▶ Running the App

**Always run from the project folder**, because the model file is loaded using a relative path:

```bash
python main_v3.py
```

A window titled **"J.A.R.V.I.S Gesture Interface"** opens showing your mirrored camera feed.
Show your hand to the camera and start gesturing. 🎉

To quit, click the camera window and press **`Q`** (or press `Ctrl + C` in the terminal).

---

## 🖐 Gesture Guide

All gestures are done with **one hand** in front of the camera. The system is mirrored, so moving your hand right moves the cursor right.

| Action | Gesture | What happens |
|---|---|---|
| **Move cursor** | **Open palm** (all five fingers extended, thumb out) | Cursor moves as your hand moves |
| **Left click** | **Pinch thumb + index finger** | Single left click |
| **Right click** | **Pinch thumb + middle finger** | Single right click |
| **Scroll** | **Peace sign** ✌ (index + middle up; ring + pinky down) | Move hand **up** to scroll up, **down** to scroll down |
| **Grab / Drop (drag toggle)** | **Closed fist** ✊ | First fist = mouse button **down** (grab). Next fist = mouse button **up** (drop) |
| **Idle** | Any other / in-between hand shape | Nothing happens |
| **Quit** | Press **`Q`** | Closes the app |

### Gesture details

#### 1. Move cursor (open palm)
Hold your hand up with all fingers spread. The cursor follows your hand's movement. It works like a mouse: if you run out of room, drop your hand out of motion, reposition it, and keep going. Very tiny movements below the dead zone are ignored to cancel hand tremor.

#### 2. Left click (thumb + index pinch)
Bring your thumb tip and index fingertip together. A click fires, followed by a **0.4 s cooldown** so one pinch can't create a double click by accident.

#### 3. Right click (thumb + middle pinch)
Bring your thumb tip and middle fingertip together. Same cooldown as left click.

#### 4. Scroll (peace sign)
Raise index and middle fingers, curl ring and pinky. While holding the pose, move your hand up or down. Larger movements scroll faster. Drop the pose to stop scrolling.

#### 5. Drag and drop (fist toggle)
Drag is a **toggle**, not a hold, so you don't have to keep a tiring fist while moving:

1. Hover over the item with an open palm.
2. Make a **fist** ✊ → the mouse button is pressed down (the HUD shows a green **DRAG_ACTIVE** lock).
3. **Open your palm** and move your hand to carry the item.
4. Make a **fist** again ✊ → the button is released (drop).

> ⏱ A **1 second cooldown** protects against double toggling. Open your hand again within a second of making the fist, or holding the fist longer will toggle again.

While dragging, clicks and scroll are disabled so you can't break the drag by accident.

---

## 🖥 On-Screen HUD

| HUD element | Meaning |
|---|---|
| Fading trail behind your palm | Your recent hand path |
| Rotating arc / reticle | Tracked hand center |
| `STATUS: …` label | Current detected state (`MOVE`, `CLICK_LEFT`, `CLICK_RIGHT`, `SCROLL`, `FIST`, `DRAG_ACTIVE`, `IDLE`) |
| White small reticle | Move mode |
| Red circle | Left click |
| Blue **R** | Right click |
| Magenta up/down arrows | Scroll mode |
| Orange **TOGGLE** | Fist detected (drag toggle) |
| Green circle with filled square | Drag is locked / active |
| Top-left `J.A.R.V.I.S \| FPS: N` | Live frame rate |
| Red **SAFETY PAUSE: LOW FPS** | Frame rate dropped below the safe limit; control is suspended |

---

## 🛡 Safety & Emergency Stop

This program **controls your real mouse**. Know how to stop it.

- **Press `Q`** with the camera window focused → clean shutdown.
- **`Ctrl + C`** in the terminal → stops the program.
- **Lower your hand out of the camera view** → cursor control stops immediately, and if a drag was active, the mouse button is **released automatically** ("Hand lost. Safety Drop.").
- **Low-FPS safety pause** → if the frame rate falls below `FAILSAFE_FPS` (15), control is paused so the cursor can't behave erratically.
- **Click and toggle cooldowns** → prevent accidental double clicks and flickering drag toggles.

> ⚠ **Note:** PyAutoGUI's usual "slam the mouse into a screen corner to abort" fail-safe is **disabled** in this project (`pyautogui.FAILSAFE = False`), because the air-mouse can legitimately reach corners. Use `Q` or `Ctrl + C` instead.

---

## ⚙ Configuration & Tuning

All V3 settings live in **`gesture_v3/config.py`**. Edit a value, save, and restart the app.

### Cursor movement

| Setting | Default | What it does | Tune it if… |
|---|---|---|---|
| `BASE_SENSITIVITY` | `3.0` | Base cursor speed | Cursor feels too slow (↑) or too fast (↓) |
| `ACCELERATION_FACTOR` | `20.0` | Extra speed for fast hand motion | You want long, quick sweeps to cover more screen |
| `MAX_SENSITIVITY` | `12.0` | Upper cap on total speed multiplier | Cursor overshoots on fast moves (↓) |
| `DEAD_ZONE` | `0.002` | Movements smaller than this are ignored | Cursor drifts/trembles when still (↑) or can't make tiny moves (↓) |
| `DELTA_SMOOTHING` | `0.6` | Motion smoothing. Higher = smoother but laggier | Cursor jittery (↑) or laggy (↓). Range 0–1 |

### Hand signal filtering (One Euro Filter)

| Setting | Default | What it does |
|---|---|---|
| `ONE_EURO_MIN_CUTOFF` | `0.5` | Lower = steadier when the hand is slow/still |
| `ONE_EURO_BETA` | `4.0` | Higher = more responsive when moving fast, but can add jitter |
| `ONE_EURO_D_CUTOFF` | `1.0` | Smoothing of the speed estimate |

### Gestures

| Setting | Default | What it does | Tune it if… |
|---|---|---|---|
| `PINCH_THRESHOLD_NORM` | `0.06` | How close fingertips must be to count as a pinch (fraction of the frame) | Clicks don't trigger (↑ e.g. `0.08`) or trigger by accident (↓ e.g. `0.045`) |
| `CLICK_COOLDOWN` | `0.4` s | Minimum time between clicks | Getting double clicks (↑) |
| `SCROLL_SPEED` | `20` | Scroll sensitivity | Scroll too slow (↑) or too fast (↓) |
| `SCROLL_DEADZONE` | `0.05` | Reserved scroll dead zone setting | – |
| `DRAG_TOGGLE_COOLDOWN` | `1.0` s | Minimum time between fist toggles | Drag toggles twice by accident (↑) |

### System

| Setting | Default | What it does |
|---|---|---|
| `TARGET_FPS` | `60` | Frame rate requested from the webcam |
| `FAILSAFE_FPS` | `15` | Below this frame rate the safety pause kicks in. Lower it on slow PCs |

### Other settings you can change in code

| What | Where | Notes |
|---|---|---|
| Camera index | `gesture_v3/core/system.py` → `cv2.VideoCapture(0)` | Change `0` to `1`, `2`… for an external camera |
| Camera resolution | `gesture_v3/core/system.py` → `CAP_PROP_FRAME_WIDTH/HEIGHT` | Lower (e.g. 640×480) for better FPS on slow PCs |
| Hands tracked | `gesture_v3/perception/tracker.py` → `num_hands=2` | The first detected hand drives control |
| Detection confidence | `gesture_v3/perception/tracker.py` | Raise for fewer false detections, lower to detect more easily |
| HUD colors | `gesture_v3/config.py` → `COLOR_*` (BGR format) | |

---

## 📁 Project Structure

```
Control_pc_using_Hand-Gesture/
├── main_v3.py                  # ▶ Entry point. Run this.
├── hand_landmarker.task        # MediaPipe hand-landmark model
├── requirements.txt            # Python dependencies
│
├── gesture_v3/                 # Current (V3) implementation
│   ├── config.py               # All tunable settings
│   ├── core/
│   │   └── system.py           # Main loop + gesture→action state machine
│   ├── perception/
│   │   ├── tracker.py          # MediaPipe Hand Landmarker wrapper
│   │   └── smoothing.py        # One Euro Filter
│   ├── intent/
│   │   └── classifier.py       # Gesture classification (MOVE/CLICK/SCROLL/FIST…)
│   ├── control/
│   │   └── mouse_physics.py    # Relative "air mouse" cursor with accel + dead zone
│   └── ui/
│       └── hud.py              # Cinematic HUD overlay
│
├── config.py                   # Legacy V1/V2 settings
├── hand_tracking.py            # Legacy V1/V2 hand tracker (standalone test included)
├── gesture_recognition.py      # Legacy V1/V2 gesture recognizer
├── mouse_control.py            # Legacy V1/V2 absolute-position mouse controller
├── README.md                   # This file
└── README_V3.md                # Older V3 notes (superseded by this file)
```

---

## 🏗 Architecture

| Layer | File | Responsibility |
|---|---|---|
| **Core** | `core/system.py` | Owns the camera, runs the loop, wires every layer together, runs the action state machine, handles safety checks |
| **Perception** | `perception/tracker.py` | Runs MediaPipe in VIDEO mode (needs strictly increasing timestamps) |
| | `perception/smoothing.py` | One Euro Filter, an adaptive low-pass filter that minimizes jitter *and* lag |
| **Intent** | `intent/classifier.py` | Converts 21 landmarks into a named gesture state. Priority: Left pinch > Right pinch > Peace > Fist > Open palm > Idle |
| **Control** | `control/mouse_physics.py` | Dead zone → delta smoothing → acceleration → sub-pixel accumulation → relative `pyautogui.move` |
| **UI** | `ui/hud.py` | Draws the trail, reticle, and status text |

**Gesture detection logic (summary)**

- *Finger up?* A fingertip is "up" if its tip is higher on screen than its PIP joint (index, middle, ring, pinky). The thumb counts as "out" if its tip is far enough from the index knuckle.
- *Pinch?* Distance between thumb tip and index/middle tip, in normalized image coordinates, is below `PINCH_THRESHOLD_NORM`.
- *Fist?* Index, middle, ring and pinky are all down.
- *Open palm?* All five fingers up.
- *Peace?* Index and middle up, ring and pinky down.

---

## 🕰 Legacy V1/V2 Modules

The top-level files `config.py`, `hand_tracking.py`, `gesture_recognition.py` and `mouse_control.py` are the earlier (V1/V2) building blocks:

- **Absolute mouse mapping**: your fingertip position in the camera frame is mapped directly to a screen position.
- **Gestures**: index finger up = move, thumb + index pinch = left click, thumb + middle pinch = right click, index + middle up = scroll, fist = pause.
- **Dynamic smoothing**: smoothing varies with movement speed.

You can sanity-check the legacy tracker on its own:

```bash
python hand_tracking.py
```

It opens a camera window with the hand skeleton drawn and prints the index fingertip position. Press `Q` to exit.

> ℹ The legacy modules no longer have their own launcher (`main.py`) in this repository. **V3 (`main_v3.py`) is the supported way to run the project.**

---

## 🧯 Troubleshooting

### Installation problems

<details>
<summary><b>❌ <code>pip install mediapipe</code> fails / "No matching distribution found"</b></summary>

MediaPipe publishes builds only for certain Python versions and 64-bit systems.

- Check your version: `python --version`
- Use **Python 3.10, 3.11 or 3.12** (64-bit) and recreate your virtual environment.
- Upgrade pip: `python -m pip install --upgrade pip`
- On Apple Silicon Macs make sure you're using a native arm64 Python, not an x86 one under Rosetta.
</details>

<details>
<summary><b>❌ <code>ModuleNotFoundError: No module named 'cv2' / 'mediapipe' / 'pyautogui'</code></b></summary>

The dependencies are not installed in the Python you're running.

- Activate your virtual environment (`venv\Scripts\activate` or `source venv/bin/activate`).
- Run `pip install -r requirements.txt` again.
- Make sure `pip` and `python` belong to the same environment: use `python -m pip install -r requirements.txt`.
</details>

<details>
<summary><b>❌ NumPy / protobuf version conflict errors on import</b></summary>

Messages like *"A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x"* or protobuf errors mean package versions disagree.

```bash
pip install --upgrade mediapipe opencv-python numpy
```

If it persists, start fresh with a new virtual environment and reinstall the requirements.
</details>

<details>
<summary><b>❌ <code>cv2.error: ... The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Cocoa support</code></b></summary>

You have the *headless* OpenCV build installed, which can't show windows.

```bash
pip uninstall -y opencv-python opencv-python-headless opencv-contrib-python
pip install opencv-python
```
</details>

<details>
<summary><b>❌ Linux: <code>KeyError: 'DISPLAY'</code>, "Xlib" errors, or cursor doesn't move</b></summary>

- PyAutoGUI needs a graphical **X11** session. It does not work over plain SSH or on Wayland.
- Install: `sudo apt install python3-tk python3-dev`
- Log out and choose an **Xorg/X11** session at the login screen.
</details>

### Startup problems

<details>
<summary><b>❌ <code>FileNotFoundError: Model not found at hand_landmarker.task</code></b></summary>

The model file is loaded relative to where you run the command.

- Run from the project folder: `cd Control_pc_using_Hand-Gesture` then `python main_v3.py`.
- Confirm `hand_landmarker.task` is in that folder. If not, re-download it (see [Installation, step 4](#4-check-the-model-file)).
</details>

<details>
<summary><b>❌ Camera window is black, or "nothing happens"</b></summary>

- **Another app is using the webcam** (Zoom, Teams, browser tab, Camera app). Close it.
- **Wrong camera.** Change `cv2.VideoCapture(0)` in `gesture_v3/core/system.py` to `1` or `2`.
- **Permissions.** Windows: Settings → Privacy → Camera. macOS: allow Camera for your terminal/IDE (then restart it).
- **Physical privacy shutter / hardware switch** is closed.
- The loop silently skips frames that fail to read, so a dead camera shows a frozen or empty window rather than an error.
</details>

<details>
<summary><b>❌ <code>ValueError</code> / MediaPipe timestamp error ("timestamp must be monotonically increasing")</b></summary>

MediaPipe VIDEO mode needs each frame's timestamp to be larger than the last. This can happen if the camera stalls and restarts. Quit and relaunch the app. Don't run two instances at once.
</details>

<details>
<summary><b>❌ macOS: app opens but the mouse doesn't move or click</b></summary>

Grant **Accessibility** permission to your terminal/IDE: System Settings → Privacy & Security → Accessibility. Quit and reopen the terminal afterward.
</details>

### Tracking & gesture problems

<details>
<summary><b>🖐 Hand isn't detected / HUD never appears</b></summary>

- Improve lighting; avoid strong backlight (a bright window behind you).
- Keep your whole hand in frame, roughly **40–80 cm** from the camera.
- Use a plain, contrasting background if possible.
- Lower detection confidence values in `gesture_v3/perception/tracker.py` (e.g. `0.5` → `0.4`).
</details>

<details>
<summary><b>🖱 Cursor is jittery / shaky</b></summary>

- Increase `DELTA_SMOOTHING` (e.g. `0.6` → `0.75`).
- Increase `DEAD_ZONE` (e.g. `0.002` → `0.004`).
- Lower `ONE_EURO_BETA` (e.g. `4.0` → `2.0`) and/or `ONE_EURO_MIN_CUTOFF` (e.g. `0.5` → `0.3`).
- Improve lighting and rest your elbow on a desk.
</details>

<details>
<summary><b>🖱 Cursor lags or feels sluggish</b></summary>

- Decrease `DELTA_SMOOTHING`.
- Increase `ONE_EURO_BETA`.
- Check the FPS counter at top-left. If it's low, lower the camera resolution in `system.py` and close heavy apps.
</details>

<details>
<summary><b>🖱 Cursor moves too slowly / too fast / can't reach screen edges</b></summary>

- Raise or lower `BASE_SENSITIVITY` and `MAX_SENSITIVITY`.
- Increase `ACCELERATION_FACTOR` so quick sweeps cover more distance.
- Since it's a relative (mouse-like) system, you can always reposition your hand and continue.
</details>

<details>
<summary><b>👆 Clicks don't register</b></summary>

- Touch the fingertips together clearly; the thumb tip and index tip must meet.
- Increase `PINCH_THRESHOLD_NORM` (e.g. `0.06` → `0.08`).
- Face your palm toward the camera; pinches seen edge-on are harder to measure.
- Remember the **0.4 s click cooldown** after each click.
- Pinches are measured in screen fraction, so if your hand is far from the camera, fingers look closer together; if very close, they look farther apart. Stay at a consistent distance.
</details>

<details>
<summary><b>👆 Clicks fire by accident / double clicks</b></summary>

- Decrease `PINCH_THRESHOLD_NORM` (e.g. `0.06` → `0.045`).
- Increase `CLICK_COOLDOWN` (e.g. `0.4` → `0.7`).
- Don't rest your thumb against your index or middle finger while moving.
</details>

<details>
<summary><b>✊ Fist shows as a click (or the other way around)</b></summary>

In a tight fist the thumb can touch the index or middle finger, which counts as a pinch (and pinch has higher priority than fist). Keep your thumb **outside / away** from the curled fingers when making a fist.
</details>

<details>
<summary><b>✊ Drag toggles on and off by itself, or drops immediately</b></summary>

The fist toggles drag every time it's detected after the cooldown. If you **hold** a fist longer than `DRAG_TOGGLE_COOLDOWN` (1 s) it toggles again. Make the fist briefly, then open your palm to move. Make another fist to drop. Increase `DRAG_TOGGLE_COOLDOWN` if you need more time.
</details>

<details>
<summary><b>✊ Mouse button seems "stuck" down after a crash</b></summary>

If the program was force-closed during a drag, the OS may think the button is still held. Click the physical mouse button once to release it.
</details>

<details>
<summary><b>✌ Scroll is too sensitive / not working</b></summary>

- Raise or lower `SCROLL_SPEED`.
- Make sure the pose is a clear peace sign: index and middle **up**, ring and pinky **fully curled**.
- Move your hand deliberately up/down while holding the pose.
- Reverse feel? Swap the sign in `scroll_amount = int(-dy * ...)` in `system.py`.
</details>

<details>
<summary><b>🐢 Low FPS / "SAFETY PAUSE: LOW FPS" keeps appearing</b></summary>

- Lower the capture resolution in `gesture_v3/core/system.py` (for example 640×480).
- Close other CPU-heavy programs and any app using the camera.
- Make sure you have good lighting (webcams drop frame rate in dim rooms).
- Lower `FAILSAFE_FPS` in `config.py` if your PC genuinely can't hit 15 FPS (control quality will suffer).
- Set `TARGET_FPS` to a value your camera supports (30 is typical).
</details>

<details>
<summary><b>🖼 Cursor movement direction is mirrored / inverted</b></summary>

The camera image is flipped on purpose (`cv2.flip(img, 1)`) so it behaves like a mirror. If you want the opposite behavior, remove or change that line in `system.py`.
</details>

<details>
<summary><b>🖥 Multi-monitor or display-scaling oddities</b></summary>

The cursor moves by *relative* amounts scaled to your primary screen size, so it generally works on any setup, but speed may feel different on very large or high-DPI displays. Adjust `BASE_SENSITIVITY` to taste.
</details>

---

## 💡 Tips for Best Results

1. **Light your hand from the front**, not from behind.
2. **Keep a steady distance** (about an arm's length from the camera).
3. **Use deliberate, clear poses.** Fully extend or fully curl fingers.
4. **Rest your elbow** on the desk to reduce fatigue and shake.
5. **Reposition freely.** Lower your hand out of the frame, raise it again somewhere comfortable, and keep going, just like lifting a mouse.
6. **Start with the defaults**, then tune one setting at a time.
7. Take breaks: air gestures are tiring over long sessions.

---

## ⚠ Known Limitations

- Controls **one hand** at a time (the first detected hand).
- Works best in good lighting; very dark or very bright backlit scenes reduce accuracy.
- Gesture checks are based on image coordinates, so a hand tilted heavily sideways or upside-down can confuse finger up/down detection. Keep your hand upright.
- Pinch distance isn't normalized by hand size, so its feel depends on how far you are from the camera.
- Not a replacement for a keyboard or a precision mouse for fine work (design, gaming).
- No keyboard input, system shortcuts, or voice control yet.
- Linux on Wayland is not supported (PyAutoGUI limitation).

---

## 🗺 Roadmap

- Two-hand gestures (zoom, window management)
- Keyboard / "air typing" input
- Voice commands ("Computer, open browser")
- Custom user-trained gestures
- Hand-size-normalized pinch detection
- A settings UI for live tuning
- Adjustable active zone and left-handed profile

---

## 🤝 Contributing

Contributions, bug reports and ideas are welcome!

1. Fork the repository
2. Create a branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m "Add my feature"`
4. Push: `git push origin feature/my-feature`
5. Open a Pull Request

When reporting a bug, please include: your OS, Python version, webcam type, what gesture you tried, and the full error message.

---

## 🙌 Credits

- **[MediaPipe](https://developers.google.com/mediapipe)** by Google for the Hand Landmarker model
- **[OpenCV](https://opencv.org/)** for video capture and drawing
- **[PyAutoGUI](https://pyautogui.readthedocs.io/)** for mouse control
- **[One Euro Filter](https://gery.casiez.net/1euro/)** by Géry Casiez, Nicolas Roussel and Daniel Vogel for jitter-free smoothing

Built by **Ali Khan**.

---

## 📄 License

Add a `LICENSE` file to specify how others may use this project (MIT is a common choice for open-source projects).
