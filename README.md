# বাংলা QR — Bangla QR Amount Tool

A single-file, fully offline tool that turns a **static bKash (EMV QRCPS) QR** into ready-to-scan **fixed-amount QRs**.

The whole app is one `index.html` file — no server, no build step, no dependencies to install. Open it in a browser and it works. After the page loads, it makes **no network requests at all**.

**Repository:** <https://github.com/mdmonirhossen6/banglaqr>

---

## এক নজরে (Quick start)

1. `index.html` ডাউনলোড করুন বা ক্লোন করুন।
2. ফাইলটি ব্রাউজারে খুলুন (ডাবল-ক্লিকই যথেষ্ট)।
3. আপনার স্ট্যাটিক bKash QR-এর ছবি আপলোড করুন, অথবা QR টেক্সট পেস্ট করুন।
4. পরিমাণ লিখুন → QR তৈরি হবে → PNG ডাউনলোড করুন।

---

## Features

- Upload or drag-and-drop a QR image (PNG, JPG, WebP, GIF, BMP · max 25 MB), or paste the raw QR payload.
- The last source you provide wins. The last valid QR text is saved in your browser's `localStorage` and restored on reload.
- Prefilled amount, with quick chips: 100, 200, 350, 500, 1,000, 5,000, 10,000, 50,000 BDT.
- Optional dynamic-QR toggle (changes EMV tag `01` from `11` to `12`). Off by default.
- Bulk mode: paste up to 100 amounts (comma or newline separated) and download each PNG or one ZIP.
- Download PNG (1024 px or larger), copy the payload, copy the QR image.
- Every generated QR is decoded again by jsQR before it is shown, including the center logo. A green **Verified ✓** badge means the payload, amount tag, and CRC in the final composed image all match.
- High error correction (ECC **H**) with the Bangla QR logo at 20% width in the center, on a white rounded backing.
- Bangla-first, bilingual UI; light and dark mode; mobile-friendly.
- Works fully offline and from `file://`.

---

## Download

### Option 1 — Download the ZIP (easiest)

1. Open <https://github.com/mdmonirhossen6/banglaqr>.
2. Click the green **Code** button → **Download ZIP**.
3. Extract the ZIP.
4. Double-click **`index.html`** (or right-click → Open with → your browser).

### Option 2 — Clone with Git

```bash
git clone https://github.com/mdmonirhossen6/banglaqr.git
cd banglaqr
```

Then open `index.html` in your browser. No `npm install`, no build command.

### Option 3 — Single file

Download only `index.html` from the repository and save it anywhere. That one file is the entire tool.

> Tip: if your browser restricts `localStorage` on `file://`, run a tiny local server instead:
> `python -m http.server 8000` then open <http://localhost:8000/>.

---

## How to use

### 1. Add your source QR

- **Upload an image** — click the drop area or drag a QR screenshot into it.
- **Paste QR text** — paste the raw EMV payload (starts with `000201`) into the text box.

The panel shows the merchant name, city, any existing amount, and the CRC so you can confirm you loaded the right QR. The source is remembered in your browser; **মুছুন (Clear)** removes it.

### 2. Set an amount

Type the amount in BDT or tap a chip. Rules: greater than zero, at most 13 characters, decimals only if you type them, no commas.

### 3. Download or copy

- **PNG ডাউনলোড (Download)** — saves a print-ready PNG with a `৳amount` caption.
- **Copy payload** — copies the new EMV payload text.
- **Copy image** — copies the PNG to the clipboard (if the browser allows it; otherwise use Download).

### 4. Bulk mode

Paste amounts separated by commas or new lines (for example `100, 200, 350, 500`) and press **QR তৈরি (Generate)**. Each QR is verified, then you can download cards individually or grab everything as a ZIP.

### Dynamic QR toggle

Leave it **off** for a standard amount QR. Turn it on only when you specifically need the payload to be marked as a dynamic QR (tag `01` becomes `12`). The source QR's other fields are never reinterpreted.

---

## How it works

- Parses EMV TLV: `tag(2) + length(2) + value`.
- Removes any existing amount (tag `54`) and CRC (tag `63`), then inserts the new tag `54` in ascending tag order (after `53`, before `58`).
- Preserves every other tag **byte-for-byte**, including nested templates (tags 26–51, 62, 64). Nested values are never parsed or rewritten.
- Appends `6304` and a **CRC-16/CCITT-FALSE** checksum (uppercase, zero-padded to 4 hex digits).
- Renders at ECC level **H**, adds a 4-module white quiet zone, and places the logo only in the center so finder patterns are untouched.
- The self-check runs jsQR on the final composed canvas and compares the decoded string with the expected payload, the amount, and the CRC.

---

## What "Verified ✓" means — and does not mean

It means: the image on screen (QR + logo) can be decoded, and the decoded payload exactly matches the generated payload, including the amount tag and a valid CRC.

It does **not** mean a payment will be accepted. This tool never contacts bKash or any bank, and it cannot test whether the receiving app honors a specific payload. Before relying on it, scan the result once with a small real payment and confirm the merchant receives the correct amount.

---

## Use it in your own project or checkout page

The tool is intentionally manual and offline: it does not take order data from your site, call an API, or verify payments. That makes it safe and simple to embed — but it is **not** a payment gateway.

### Option A — Link to it (recommended)

Host the file in your site's static assets and link to it:

```html
<a href="/tools/bangla-qr/index.html" target="_blank" rel="noopener">
  QR দিয়ে পেমেন্ট (Pay by QR)
</a>
```

The user uploads/pastes their static QR once; it is saved in **their** browser and reused next time.

### Option B — Embed with an iframe

```html
<iframe
  src="/tools/bangla-qr/index.html"
  title="Bangla QR amount tool"
  width="100%"
  height="900"
  loading="lazy"
  allow="clipboard-write"
  style="border:0;border-radius:14px">
</iframe>
```

Notes for embedding:

- Keep the tool's footer with the disclaimer and licence notice visible. Do not present the tool as an official bKash product.
- The page's own Content-Security-Policy blocks all network access, so it works inside an iframe without leaking data.
- `allow="clipboard-write"` enables the **Copy image** button; without it, users can still download the PNG.
- If your own page has a CSP, allow the iframe's source in `frame-src`.
- For consistent `localStorage` (saved source QR), serve the tool from **your own origin**. Storage may be unavailable or isolated on `file://` or in some third-party iframe setups.

### Option C — Self-host as part of your app

Copy `index.html` into your project, for example `public/tools/bangla-qr/index.html`, and deploy it with your normal static build. It has no runtime dependency and needs no route/server code. Any static host works, including GitHub Pages (repo → **Settings** → **Pages** → deploy from branch → root).

### Typical checkout flow

1. Your checkout shows the order total.
2. The customer or staff opens the QR tool.
3. They enter that exact total and download the QR.
4. The QR is shown on screen, printed, or sent to the payer.
5. Payment is confirmed in the bKash app as usual.

Because the tool is offline by design, there is no automatic link between an order and a generated QR. Always enter the total manually and double-check it before showing the QR.

### Customizing

- Colors and accents are CSS variables at the top of `index.html` (`:root`).
- The logo is embedded as a base64 SVG data URI. If you replace it, keep it small and centered; the tool uses ECC H and a 20% width logo so the QR remains scannable.
- Do not alter the EMV core logic unless you have tested the resulting payloads and CRC values.

---

## Privacy and security

- No analytics, no remote fonts, no remote scripts, no cookies.
- All processing happens in the browser; the only persistence is `localStorage` for the last valid source QR.
- CSP: `default-src 'none'; script-src 'unsafe-inline'; style-src 'unsafe-inline'; img-src data: blob:; connect-src 'none'; ...` — network connections are blocked after load.
- Uploaded images are decoded locally and never stored or transmitted.

---

## Browser support

Recent versions of Chrome, Edge, Firefox, Safari, and mobile browsers. Features used: Canvas, `TextEncoder`, `ClipboardItem` (optional), `localStorage` (optional), and `CanvasRenderingContext2D.roundRect`. If clipboard access is denied, PNG download still works.

---

## Technical notes

- EMV QR / QRCPS TLV; CRC-16/CCITT-FALSE (poly `0x1021`, init `0xFFFF`).
- Libraries embedded in the file: [jsQR 1.4.0](https://github.com/cozmo/jsQR) (Apache-2.0) and [qrcode-generator 1.4.4](https://github.com/kazuhikoarase/qrcode-generator) (MIT). Full licence texts are in the tool's **About & licenses** section.
- File size: about 400 KB (single HTML file, includes the two libraries and the logo).

---

## Disclaimer

This is an independent tool. It is **not** affiliated with, endorsed by, or supported by bKash or Bangladesh Bank. "bKash" is a trademark of its respective owner.

Use it at your own risk. Verify generated QRs with the receiving app and a small test payment before using them for real transactions. Always confirm the amount on the payer's screen before completing a payment.
