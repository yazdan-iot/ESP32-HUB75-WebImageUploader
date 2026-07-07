# 📸 ESP32-HUB75-WebImageUploader

Upload a JPEG from any phone or laptop browser and watch it appear instantly on an ESP32-driven HUB75 LED matrix panel — no reflashing, no cables, no fuss. Just open a web page, pick a photo, and it shows up on the display in seconds. ✨

## 🧩 What it does

1. 📶 Connects to your WiFi network
2. 💾 Mounts LittleFS (auto-formats on first run)
3. 🌐 Serves a tiny mobile-friendly upload page
4. ⬆️ Streams the uploaded JPEG straight to flash — never buffers the whole file in RAM
5. 🧠 Decodes the JPEG block-by-block (MCU by MCU) and downsamples it on the fly into a framebuffer sized exactly to your panel — a 4000×3000 photo costs the same as a tiny icon
6. 🖼️ Pushes the finished frame to the panel in one shot via DMA, so the whole image appears atomically — no half-drawn frames

## ⚙️ Why a separate FreeRTOS task for decoding?

JPEG decoding is a "pull" loop with no clean way to pause mid-block. So decoding runs in its own task pinned to the second CPU core, keeping the web server fast and responsive even while a big photo is being processed.

## 🖥️ Hardware requirements

- ESP32 (classic WROOM/WROVER) — default config in this repo
- HUB75 LED matrix panel (tested with a 64×32, 1/16 scan single module)
- WiFi network

> ⚠️ **Using an ESP32-S3 instead?** The `[env:esp32-s3-devkitc-1]` environment is included in `platformio.ini`, but the HUB75 pin defines in the sketch (`R1_PIN`, `G1_PIN`, etc.) are the **classic ESP32** defaults and won't map cleanly to an S3's GPIO layout. Pick fresh GPIOs and check them against your specific board's pinout before wiring anything up.

## 🔌 Wiring (classic ESP32 defaults)

| Signal | GPIO |
|--------|------|
| R1     | 25   |
| G1     | 26   |
| B1     | 27   |
| R2     | 14   |
| G2     | 12   |
| B2     | 13   |
| A      | 23   |
| B      | 19   |
| C      | 5    |
| D      | 17   |
| E      | 32   |
| LAT    | 4    |
| OE     | 15   |
| CLK    | 16   |

## 📦 Required libraries

Handled automatically via `platformio.ini`, but if you're on Arduino IDE, install these manually:

- [ESP32-HUB75-MatrixPanel-DMA](https://github.com/mrcodetastic/ESP32-HUB75-MatrixPanel-DMA) (mrcodetastic)
- [Adafruit GFX Library](https://github.com/adafruit/Adafruit-GFX-Library) — required by the HUB75 driver, not pulled in automatically
- [ESP32Async/ESPAsyncWebServer](https://github.com/ESP32Async/ESPAsyncWebServer)
- [ESP32Async/AsyncTCP](https://github.com/ESP32Async/AsyncTCP)
- [JPEGDecoder](https://github.com/Bodmer/JPEGDecoder) (Bodmer)

## 🚀 Getting started

1. Clone the repo:
   ```bash
   git clone https://github.com/<your-username>/ESP32-HUB75-WebImageUploader.git
   ```
2. Open in PlatformIO and edit your WiFi credentials in `main.cpp`:
   ```cpp
   const char* WIFI_SSID     = "SSID";      // <-- EDIT ME
   const char* WIFI_PASSWORD = "PASSWORD";  // <-- EDIT ME
   ```
3. Adjust panel geometry if needed:
   ```cpp
   #define PANEL_RES_X   64
   #define PANEL_RES_Y   32
   #define PANEL_CHAIN   1
   ```
4. Build and upload:
   ```bash
   pio run -e esp32dev -t upload
   ```
5. Open the Serial Monitor to grab the ESP32's IP address, then visit it in your browser. 🎉

## 📱 Usage

- Open the ESP32's IP address in a browser
- Tap **Choose File**, pick a `.jpg`/`.jpeg` image
- Tap **Upload** — the image shows up on the matrix within a second or two
- The last uploaded image persists across reboots (stored on LittleFS)

## 🚧 Limits & safety caps

- Only `.jpg` / `.jpeg` files are accepted
- Uploads are capped at 512 KB to protect the flash partition
- Decoding yields periodically so it never starves WiFi or trips the watchdog

## 🎞️ Roadmap: animated GIF support

Not implemented yet — deliberately out of scope for this first pass, since it needs a different memory model (persistent decoder state + per-frame timing). See the `EXTENDING TO GIF` notes at the bottom of `main.cpp` for a concrete plan using [bitbank2/AnimatedGIF](https://github.com/bitbank2/AnimatedGIF). Contributions welcome! 🙌

## 🤝 Contributing

Issues and PRs are welcome, especially around:
- ESP32-S3 pin mapping presets
- GIF support
- Multi-client upload handling

## 📄 License

MIT — do whatever you'd like with it, just don't blame me if your matrix judges your photo choices. 😄
