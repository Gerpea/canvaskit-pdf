# CanvasKit with PDF Support

This is a custom fork of Google's Skia/CanvasKit, specifically configured and compiled with **PDF export support** enabled. 

Standard CanvasKit WASM builds omit the PDF module to keep the binary size small. This fork enables `skia_enable_pdf` and exposes the `MakePDFDocument` API to the JavaScript layer, allowing web applications (like [skpxr](https://github.com/Gerpea/skpxr)) to render and export crisp, vector-based PDFs directly from the browser.

## 📦 Pre-compiled Binaries

If you don't want to build from source, the pre-compiled `canvaskit.wasm` and `canvaskit.js` files are available in the main `skpxr` repository under `vendor/canvaskit-wasm/`.

---

## 🛠️ Building from Source

Building Skia for WebAssembly requires a Unix-like environment (Linux, macOS, or WSL2 on Windows), Python 3, and Git.

### Prerequisites
* **Ubuntu/Debian:** `sudo apt-get install git python3 clang libgl1-mesa-dev`
* **macOS:** Install Xcode Command Line Tools (`xcode-select --install`) and Python 3.
* **Windows:** It is highly recommended to use **WSL2** (Ubuntu). Native Windows builds are prone to path-length and Emscripten compatibility issues.

### Step 1: Clone the Repository
Clone this fork and ensure you are on the `canvaskit-pdf` branch.

```bash
git clone https://github.com/Gerpea/canvaskit-pdf.git skia
cd skia
git checkout canvaskit-pdf
```
### Step 2: Sync Dependencies
Skia relies on a custom Python script to pull in all necessary third-party C++ libraries.
```bash
python3 tools/git-sync-deps
```
### Step 3: Setup Emscripten (WASM Compiler)
Skia requires a very specific version of Emscripten to compile correctly. 
#### Option A: Skia's Built-in Helper (Recommended)
This script automatically downloads and activates the exact Emscripten version pinned by Skia.
```bash
bin/activate-emsdk
```
#### Option B: Manual Emscripten Setup
If the helper script fails or you prefer to manage `emsdk` manually, you can use the bundled external SDK:
```bash
# 1. Navigate to the Emscripten external directory
cd third_party/externals/emsdk

# 2. Install & activate the Emscripten SDK
./emsdk install latest
./emsdk activate latest

# 3. Activate it in your current terminal
source ./emsdk_env.sh

# 4. Go back to the Skia root directory
cd ../../..
```
*(⚠️ Note: Using `latest` might occasionally cause build mismatches if Skia's internal C++ code relies on an older Emscripten ABI. If you hit weird C++ linker errors, fall back to Option A).*

### Step 4: Build CanvasKit with PDF
Navigate to the CanvasKit module and run the custom PDF build target.
```bash
cd modules/canvaskit
make npm_pdf
```
### Step 5: Locate the Output
Once the build completes successfully (grab a coffee, it takes a while), the generated WebAssembly files will be located in the output directory:
```text
npm_build/canvaskit/
├── canvaskit.js
├── canvaskit.wasm
└── ...
```

## ⚠️ Troubleshooting
 * `python3 tools/git-sync-deps` fails: Ensure you have an active internet connection and that your Git credentials are configured correctly. Skia downloads dozens of submodules.
 * **`make npm_pdf` fails with C++ errors:** This is almost always an Emscripten version mismatch. Delete `third_party/externals/emsdk` and use `bin/activate-emsdk` instead of the manual `latest` installation.
 * **Out of Memory during build:** Compiling Skia with Emscripten is memory-intensive. If the build crashes, try limiting the ninja jobs by modifying the make command or running ninja directly: `ninja -C out/canvaskit_pdf -j 4`.
 * **"Command not found" errors:** Ensure you ran source `./emsdk_env.sh` (or `bin/activate-emsdk`) in **the same terminal window** where you are running `make npm_pdf`. The environment variables do not persist across new terminal sessions.