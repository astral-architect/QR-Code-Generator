# QR Generator Pro

A fully-featured, single-file QR code generator. No ads, no backend, no signup — just open the HTML file in any browser.

---

## How it was built

### Stack
- **Pure HTML + CSS + JavaScript** — zero frameworks, zero build step
- **Single `.html` file** — download and open, that's it

### Libraries used (loaded from CDN)
| Library | Purpose |
|---|---|
| `qrcode-generator` | Generates the raw QR matrix (`isDark(row, col)`) |
| `jszip` | Bundles batch QR exports into a ZIP file |
| `jsQR` | Decodes QR codes from webcam frames |
| `jsPDF` | Exports QR as a proper PDF (A4) |

---

## How each feature works

### QR Generation
`qrcode-generator` takes your text and returns a 2D grid (matrix). Each cell is either dark or light. We draw that grid manually on an HTML `<canvas>` — this is what allows full visual customization.

```
User input → qrcode-generator → matrix (isDark grid) → Canvas drawing
```

### Custom Dot Styles
Every dark cell in the matrix is drawn with a custom shape on canvas. Style is passed as a parameter:
- **Square** → `fillRect()`
- **Rounded** → `roundedRect()` with radius
- **Circle** → `arc()`
- **Diamond** → 4-point path
- **Star** → 10-point alternating path
- **Cross** → two overlapping rectangles

### Finder Patterns (Corner squares)
The 3 large corner squares in every QR code are called **finder patterns**. Scanners use these to detect orientation. We skip them during the main dot-drawing loop (detected by position: top-left, top-right, bottom-left 7×7 region), then draw them separately with their own style (square / rounded / blob).

### Gradient Colors
A canvas gradient object is created once before the drawing loop. Since the gradient covers the entire canvas, each dot drawn at its position automatically gets the correct gradient color — no per-dot calculation needed.

### Logo Overlay
After drawing the QR, we draw a white padding box at the center, then draw the logo image on top. Error correction level is set to **H (30%)** by default so the QR remains scannable even with the logo covering the center.

### Loading Animation
Pure CSS animations:
- Corner brackets: `scale()` keyframe with staggered `animation-delay`
- Scanning line: `top` property animates 0 → 138px in a loop
- Dot grid: 49 `<div>` elements with random `animation-delay` and `animation-duration` set via JS
- Progress bar: JS interval increments width, completes on `window load` event

### Image Upload → QR
```
User drops image
  → POST to litterbox.catbox.moe (free, no API key, 72h)
  → If fails → fallback to 0x0.st (permanent)
  → Response = direct URL (plain text)
  → URL goes into text input → QR generated
```

### Batch Mode
1. Parse CSV with JS `split('\n')`
2. Loop through rows with `await setTimeout` (so UI doesn't freeze)
3. Each row: generate QR → render on offscreen canvas → `toDataURL` → add to JSZip
4. After loop: `zip.generateAsync()` → download as `.zip`

### QR Scanner
Uses `getUserMedia` to access webcam, draws each video frame to a hidden canvas every 200ms, passes pixel data to `jsQR` which returns decoded text if a QR is found.

### History & Presets
Both stored in `localStorage` as JSON arrays. History stores a base64 thumbnail + original text. Presets store the full style state object + a preview thumbnail.

### Export
- **PNG** → `canvas.toDataURL('image/png')` — direct from canvas
- **SVG** → manually build SVG string with `<rect>` elements per dark module
- **PDF** → jsPDF creates A4 doc, centers the QR image, saves
- **Copy** → `canvas.toBlob()` → `ClipboardItem` → `navigator.clipboard.write()`
- **Share** → `canvas.toBlob()` → `File` → `navigator.share()` (mobile only)

---

## Content Types & their QR format

| Type | QR data format |
|---|---|
| URL / Text | raw string |
| WiFi | `WIFI:T:WPA;S:SSID;P:password;;` |
| vCard | `BEGIN:VCARD ... END:VCARD` (v3.0) |
| UPI | `upi://pay?pa=id&pn=name&am=amount&cu=INR` |
| Email | `mailto:to?subject=sub&body=body` |
| SMS | `sms:number?body=message` |
| WhatsApp | `https://wa.me/number?text=message` |
| Location | `geo:lat,lng?q=lat,lng(label)` |

---

## File structure

Everything is in one file — `qr-generator-pro.html`:

```
qr-generator-pro.html
├── <style>         CSS (layout, loader, animations, components)
├── #loader         Loading screen (animated)
├── <header>        Top nav with Scan / Batch / Presets buttons
├── .layout         3-column grid
│   ├── .panel-l    Content type selector + forms (8 types)
│   ├── .preview-center   Canvas preview + download buttons
│   └── .panel-r    Customization (Style / Color / Logo / Frame)
├── .history-bar    Recent QR thumbnails
├── Modals          Scan, Batch, Presets
└── <script>        All JS logic (~400 lines)
```

---

## Usage

1. Open `qr-generator-pro.html` in any modern browser
2. Pick a content type (URL, WiFi, UPI, etc.)
3. Fill in the fields
4. Click **Generate QR**
5. Customize style, colors, logo, frame
6. Download as PNG / SVG / PDF or copy to clipboard

No internet required after first open (CDN libraries are loaded once).
