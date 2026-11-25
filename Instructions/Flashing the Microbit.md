# Flashing code onto the micro:bit

How to get a MicroPython script onto the micro:bit. There are two ways to do it: the **web editor** (no install needed) or **`uflash`** (command line, works with your own editor).


---

## What "flashing" actually means

The micro:bit can't run a `.py` file by itself. It needs a **`.hex` file**, which is a single firmware image made of:

1. the **MicroPython interpreter** (the runtime), and
2. **your script**, stored in the micro:bit's small internal filesystem as `main.py`.

When you plug the micro:bit in over USB it shows up as a USB drive called **`MICROBIT`**. Copying a `.hex` onto that drive makes the board reflash itself. The yellow LED on the back flashes while it writes, then the board resets and runs `main.py`.

Both methods below build that `.hex` and copy it across for you.

> [!tip] Check the cable first
> A lot of micro-USB cables only carry power, not data. If no `MICROBIT` drive appears when you plug the board in, try another cable before anything else.

---

## Method 1: Web editor (python.microbit.org)

Use this when you're on a uni PC, a laptop without Python set up, or you just want something quick.

### Option A: Connect over WebUSB (quickest)

Needs **Chrome or Edge**. Firefox and Safari don't support WebUSB.

1. Go to **https://python.microbit.org**
2. Write or paste your code into the editor.
3. Plug in the micro:bit and click **Send to micro:bit**.
4. The first time, it asks you to **Connect**. Pick the micro:bit (`BBC micro:bit CMSIS-DAP`) in the browser popup.
5. Wait for the progress bar to finish. The board resets and runs your code.

Once it's connected, each later **Send to micro:bit** only reflashes the parts that changed, so it's fast.

The **Show serial** panel gives you a live REPL and shows `print()` output and error messages. That's very useful for debugging sensor values.

### Option B: Download the .hex and drag it across

Use this when WebUSB won't connect (wrong browser, locked-down PC, Linux permissions).

1. In the editor, click the **⋯** menu next to Send, then **Save** to download `<project-name>.hex`.
2. Open your file manager and find the **`MICROBIT`** drive.
3. **Drag the `.hex` onto `MICROBIT`.**
4. The yellow LED flashes, the drive disconnects and reconnects (this is normal), and the code runs.

> [!note] The .hex disappears from the drive afterwards
> This is expected. `MICROBIT` isn't a real filesystem, just an interface for flashing. Your code is now in the chip's flash memory.

### Loading a `.py` from GitHub

**Open…** in the editor accepts a `.py` or a `.hex`. So you can pull a file from the team repo, open it, and send it. You don't need to copy and paste.

---

## Method 2: `uflash` (command line)

Use this when you're writing code in VS Code or Neovim locally, working from the team git repo, or want to flash again every time you save.

### Install

**Linux / macOS**

```bash
pipx install uflash        # recommended: puts it on PATH in its own isolated environment
# or
pip install --user uflash
```

**Windows** (PowerShell or Command Prompt)

```powershell
py -m pip install uflash
```

On Windows, `py` is the Python launcher that comes with the python.org installer. If `py` isn't found, use `python` instead, or install Python from python.org and tick **"Add python.exe to PATH"**.

Check it's installed:

```bash
uflash --version           # this note was written against uflash 2.0.0
```

> [!warning] Windows: `'uflash' is not recognized...`
> pip put `uflash.exe` in a Scripts folder that isn't on your PATH. You don't need to fix PATH. Run it as a module instead:
> ```powershell
> py -m uflash main.py
> ```
> This works the same as `uflash main.py`. Use `py -m uflash` in place of `uflash` in every command below.

`uflash` 2.x supports **both V1 and V2** micro:bits. It builds a "universal hex" that works on either board.

### Flash a script

```bash
uflash main.py
```

That's all you need most of the time. `uflash` will:

1. bundle `main.py` with the built-in MicroPython runtime into a `.hex`
2. **auto-detect** the `MICROBIT` drive
3. copy the hex onto it

The file doesn't have to be called `main.py`. `uflash fall_detect.py` works too, because the script is always saved on the board as `main.py`.

### Useful variations

| Command | What it does |
|---|---|
| `uflash main.py` | Flash the script to the auto-detected micro:bit |
| `uflash main.py /media/$USER/MICROBIT` | **Linux:** flash to a specific mount point (if auto-detect fails) |
| `uflash main.py E:\` | **Windows:** flash to a specific drive letter (check File Explorer for yours) |
| `uflash main.py /media/$USER/MICROBIT /media/$USER/MICROBIT1` | Flash **several boards at once** (handy for group testing). On Windows: `uflash main.py E:\ F:\` |
| `uflash -w main.py` | **Watch mode**: flash again automatically every time you save the file |
| `uflash` | No script: flash plain MicroPython (gives you a clean REPL, wipes the old program) |

`-e` (extract), `-m` (minify) and `-r` (custom runtime) are **deprecated** in 2.x. You'll still see them in older tutorials, but don't rely on them.

### Where's the drive on Linux?

```bash
ls /media/$USER/           # usually shows MICROBIT
lsblk                      # if it isn't mounted, find it here (sdX, ~8MB)
```

If the file manager hasn't auto-mounted it, open the drive once in the file manager or run `udisksctl mount -b /dev/sdX`.

### Where's the drive on Windows?

Open **File Explorer → This PC**. The micro:bit appears as a removable drive called **`MICROBIT`** with a letter such as `D:` or `E:`. The letter depends on what else is plugged in. `uflash` finds it by name, so you only need the letter when auto-detect fails.

You can also find it from PowerShell:

```powershell
Get-Volume -FileSystemLabel MICROBIT     # the DriveLetter column is the one you want
```

### Seeing output: the serial REPL

`uflash` only flashes, it doesn't show output. To see `print()` and error tracebacks, connect to the board's serial port.

#### Linux

```bash
screen /dev/ttyACM0 115200      # quit with Ctrl-A then K
# or
python3 -m serial.tools.miniterm /dev/ttyACM0 115200   # needs: pip install pyserial
```

You get a `Permission denied` on `/dev/ttyACM0`:

```bash
sudo usermod -aG dialout $USER   # then log out and back in
```

#### Windows

Windows calls the serial port a **COM port** (e.g. `COM3`). To find which one is the micro:bit:

1. Open **Device Manager** (right-click Start, then Device Manager).
2. Expand **Ports (COM & LPT)**.
3. Look for **"USB Serial Device (COMx)"** or **"mbed Serial Port (COMx)"**. Unplug the micro:bit and watch which entry disappears if you can't tell.

Then connect with either of these:

```powershell
py -m pip install pyserial
py -m serial.tools.miniterm COM3 115200     # quit with Ctrl-]
```

Or use **PuTTY**: Connection type **Serial**, Serial line `COM3`, Speed `115200`, then **Open**.

Windows 10 and 11 include the serial driver. Windows 7 and 8 needed a separate "mbed serial driver", but uni PCs won't be running those.

> [!note] Only one program can use the COM port at a time
> If the web editor's serial panel, PuTTY or miniterm is still connected, flashing over WebUSB or opening another terminal will fail with an "Access denied" or "port busy" error. Close the other one first.

Inside the REPL, **Ctrl-C** stops the running program and **Ctrl-D** soft-resets it so `main.py` runs again.

---

## Which one should I use?

| | Web editor | uflash |
|---|---|---|
| Setup | None | `pipx install uflash` |
| Works on uni PCs | ✅ | Only if Python is available |
| Use your own editor / git repo | ❌ (copy-paste or Open…) | ✅ |
| Serial / REPL built in | ✅ | Use `screen` / `miniterm` |
| Fast edit-flash loop | ✅ (WebUSB partial flash) | ✅ (`-w` watch mode) |
| Flash many boards at once | ❌ | ✅ |
| Error checking before flashing | ✅ (highlights syntax errors) | ❌ (you only find out on the board) |

**Rule of thumb:** use the web editor for quick tests and demos. Use `uflash` once the code lives in the team repo.

---

## Quick test script

A minimal script to check the whole flash path works before you flash real project code. It also checks the accelerometer, which the fall-detection idea depends on.

```python
from microbit import *

display.show(Image.HAPPY)
sleep(1000)

while True:
    x, y, z = accelerometer.get_values()
    print(x, y, z)                         # watch these in the serial panel / screen
    if accelerometer.was_gesture("freefall"):
        display.show(Image.SURPRISED)
        sleep(1000)
    display.clear()
    sleep(100)
```

Expected result: a happy face, then accelerometer readings in the serial output. Dropping the board gently onto something soft should show a surprised face.

---

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| No `MICROBIT` drive appears | Charge-only cable, so swap it. Or try a different USB port. |
| Drive is called **`MAINTENANCE`** | The board booted into maintenance mode (reset was held while plugging in). Unplug and plug back in **without** holding reset. |
| Scrolling text with a number, or a sad face | Your **Python raised an error**. The text is the error message. Open the serial REPL to read the full traceback. |
| `uflash` says it can't find the micro:bit | Drive isn't mounted. Mount it, then pass the path explicitly: `uflash main.py /media/$USER/MICROBIT`. |
| WebUSB "Connect" finds nothing | Use Chrome or Edge. On Linux you may need a udev rule for the micro:bit, so fall back to **Save + drag**. |
| `MemoryError` or the script is "too large" | MicroPython has very little RAM (especially on V1). Delete unused imports and comments, and split big lists out. |
| Flashing works but the old program still runs | The copy didn't finish. Wait for the yellow LED to stop, and don't unplug mid-flash. |
| **Windows:** "Do you want to scan and fix MICROBIT?" popup | Harmless. The drive disconnects itself after flashing and Windows thinks it was removed unsafely. Click **Continue without scanning** or close it. Don't run the repair. |
| **Windows:** `'uflash' is not recognized as an internal or external command` | Scripts folder isn't on PATH. Use `py -m uflash main.py` instead. |
| **Windows:** `uflash` can't find the drive | Pass the drive letter yourself: `py -m uflash main.py E:\`. |
| **Windows:** No COM port in Device Manager | Data cable? Also check for **Other devices** entries with a ⚠️ icon. A firmware update usually fixes that. |
| Board behaves strangely after lots of flashing | Update the board firmware: https://microbit.org/get-started/user-guide/firmware/ |

---

## Links

- Python editor: https://python.microbit.org
- MicroPython micro:bit docs: https://microbit-micropython.readthedocs.io
- uflash docs: https://uflash.readthedocs.io
- Firmware update guide: https://microbit.org/get-started/user-guide/firmware/
