<p align="center">
  <img src="assets/icon.png" width="112" alt="Bambu Live View icon" />
</p>

<h1 align="center">Bambu Live View for Raycast</h1>

<p align="center">
  A tiny, borderless, always-on-top window showing your Bambu Lab printer's camera.<br />
  One hotkey to open, the same hotkey to close. Straight from the printer over your local network — no cloud, no Bambu Studio.
</p>

<p align="center">
  <a href="https://github.com/raycast/extensions/pull/31015"><img src="https://img.shields.io/badge/Raycast%20Store-in%20review-FF6363?logo=raycast&logoColor=white" alt="Raycast Store: in review" /></a>
  <img src="https://img.shields.io/badge/platform-macOS-lightgrey?logo=apple" alt="macOS" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="MIT license" /></a>
</p>

---

## Screenshots

| Guided setup | Checks your Mac for you |
| --- | --- |
| ![Printer Setup: find the IP address and access code](metadata/bambu-live-view-1.png) | ![Printer Setup: mpv is installed](metadata/bambu-live-view-2.png) |
| **Printer details, stored encrypted** | **One-press connection test** |
| ![Printer Details form](metadata/bambu-live-view-3.png) | ![Printer Setup: test the connection](metadata/bambu-live-view-4.png) |

## Commands

| Command | What it does |
| --- | --- |
| **Printer Setup** | A step-by-step checklist: enable LAN liveview on the printer, install mpv, save the IP address and access code, and test the connection. Each step shows a live ✓ / ✗. |
| **Toggle Live View** | Opens the camera window, or closes it if it's already open. Give it a hotkey for one-key access. |

## Supported printers

| ✅ Works | ❌ Doesn't work |
| --- | --- |
| X1 / X1 Carbon / X1E, P2S, H2D and other newer models that stream **RTSPS on port 322** | P1P, P1S, A1, A1 mini — these use a proprietary JPEG stream on port 6000 that mpv can't play |

## Install

**From the Raycast Store** — [submitted and in review](https://github.com/raycast/extensions/pull/31015). Once it's approved, search for *Bambu Live View* in the Store.

**From source (now):**

```sh
git clone https://github.com/Fahim8371/raycast-bambu-live-view.git
cd raycast-bambu-live-view
npm install && npm run dev
```

Once the commands appear in Raycast you can stop `npm run dev` (Ctrl+C) — the extension stays installed.

You'll also need [mpv](https://mpv.io) (`brew install mpv`); **Printer Setup** detects it and can open Terminal with the install command for you.

## Setup

Run **Printer Setup** in Raycast and follow the checklist, or do it by hand:

1. **On the printer:** Settings → **LAN Only** → turn on **LAN Only Liveview**.
   The separate **LAN Only** mode switch can stay **off**, so Bambu Cloud and Bambu Handy keep working.
2. On the same screen, note the printer's **IP address** (e.g. `192.168.1.50`) and 8-character **Access Code** (e.g. `12345678`).
3. *(Recommended)* In your router, create a **DHCP reservation** for the printer so its IP address never changes.
4. Enter the IP address and access code in **Printer Setup → Enter Printer Details** and choose **Save and Test**.
5. Run **Toggle Live View**.

The IP address and access code are kept in Raycast's encrypted local storage for this extension. The access code is never shown or logged.

## Using the window

- The window has no title bar: **⌘-drag** to move it.
- Press **q** in the window to close it, or run **Toggle Live View** again.
- Size (small 320×180, medium 480×270, large 640×360), screen corner and window title are in the extension's preferences (Raycast Settings → Extensions → Bambu Live View, or **Window Size & Position…** in the setup command's action panel).

## How it works

Under the hood it runs roughly:

```sh
echo "rtsps://bblp:ACCESS_CODE@PRINTER_IP:322/streaming/live/1" | \
  mpv --ontop --no-border --no-audio --rtsp-transport=tcp --profile=low-latency \
      --autofit=480x270 --geometry=99%:2% \
      --input-ipc-server="$TMPDIR/raycast-bambu-live-view.sock" --playlist=-
```

- **Access code stays private.** The stream URL is piped in on stdin, so it never appears in the process list.
- **No false "opened".** The extension waits for mpv to report the stream actually loaded. A wrong access code shows **Access code rejected** instead.
- **Only closes its own window.** It identifies its mpv by a private IPC socket and asks it to quit over that socket — any other mpv you have open is left alone.
- **No duplicate windows.** Starting twice at once (e.g. a double-pressed hotkey) opens one window, not two.

## Troubleshooting

- **"Printer not responding"** — the printer is off or asleep, its IP address changed, or your Mac is on a different network (guest Wi-Fi, VPN). Check from Terminal:
  ```sh
  nc -zv 192.168.1.50 322
  ```
  If `nc` succeeds but Raycast can't connect, allow Raycast under System Settings → Privacy & Security → **Local Network**.
- **"Port 322 is closed"** — LAN Only Liveview is off, or the printer is a P1/A1-series model (unsupported, see above).
- **"Access code rejected"** — the code was mistyped or regenerated on the printer. Re-enter it in Printer Setup.
- **Why not VLC?** The printer uses a self-signed TLS certificate, which VLC on macOS refuses. mpv plays it fine.

## Development

```sh
npm install
npm run dev      # ray develop — loads the extension into Raycast in development mode
npm run build    # ray build -e dist -o dist
npm run lint
```

Store screenshots live in [`metadata/`](metadata) — the Raycast Store requires 2000×1250 PNGs with roughly 12.5% padding around the window.

## License

[MIT](LICENSE)
