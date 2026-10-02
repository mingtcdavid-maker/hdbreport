# hdbfirereport

A small static web app for generating SCDF HDB Fire Risk Report
(pre-plan) documents — no backend required.

Users fill in a form in the browser; client-side JavaScript
(`docxtemplater` + `pizzip` + an open-source image module) fills in the
`{{tag}}` placeholders in
`pptx_template/SCDF_HDB_Report_TEMPLATE.pptx` directly in the browser.
The 4 uploaded photos are cropped (centre-crop to match each grey box's
aspect ratio) on a `<canvas>` and embedded into the slide at the exact
position and size of the original placeholder box. The finished
`.pptx` is downloaded — everything runs entirely on the client, with no
server-side processing of form data. A hand-built live preview panel
mirrors both slides' layout as you type/upload, so you can check the
result before generating the file.

## Run locally

Any static file server works, e.g.:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000 in a browser, fill in the form, upload
the 4 photos, and click "Generate & Download Report".

(Opening `index.html` directly via `file://` will not work — browsers
block `fetch()` of local files under that scheme, so the template can't
be loaded. Serve it over `http://` instead.)

## Project layout

- `index.html` – the data-entry form, the photo cropper, the live
  preview, and the pptx-generation logic.
- `pptx_template/SCDF_HDB_Report_TEMPLATE.pptx` – the source PowerPoint
  template with `{{tag}}` text placeholders and `{{%tag}}` image
  placeholders, used by docxtemplater and its image module.
- `static/vendor/` – vendored copies of `pizzip`, `docxtemplater`, and
  an open-source docxtemplater image module (all MIT-licensed), so the
  page has no runtime dependency on any CDN. `imagemodule.js` carries
  two small local patches (see below) — the licence and upstream
  behaviour are otherwise unchanged.

## Report fields

- **Appliance**, **Date**, **Time**, **HDB Block**, **Address**
- **Rank / Name of Inspecting Officer** (two rows; the second is
  optional and renders as "NIL" if left blank)
- **4 photos**: Overview, Best appliance placement, Nearest riser
  location, Nearest hydrant — all required. Each is centre-cropped to
  the aspect ratio of its grey box in the template (≈1.28:1) before
  being embedded, so no stretching/distortion occurs regardless of the
  uploaded photo's original aspect ratio.

## Notes

- The template was originally authored with spaced tags (`{{ appliance
  }}`); these were flattened to `{{appliance}}` (no inner whitespace)
  because docxtemplater does not trim whitespace inside its
  delimiters — a tag with spaces simply never resolves.
- The 4 grey placeholder boxes on slide 2 were originally empty
  shapes; each had a `{{%photo_xxx}}` image-module tag inserted into
  its (previously empty) text run. The image module replaces the whole
  shape with a `<p:pic>` at the same position/size when rendering, so
  the box "becomes" the photo.
- `static/vendor/imagemodule.js` (the open-source
  `docxtemplater-image-module-free` browser build, v1.1.1) has two
  patches applied directly to the vendored file:
  1. `newTag.namespaceURI = null` is wrapped in a `try/catch`. This
     line assumes a mutable `xmldom`-style DOM; real browsers bundle
     this module against the native `DOMParser`, whose `namespaceURI`
     is a read-only getter, so the unpatched line throws.
  2. Generated image filenames use a `.jpg` extension instead of the
     hardcoded `.png`, matching the JPEG bytes this app actually
     produces from the `<canvas>` crop (avoiding a declared vs. actual
     content-type mismatch in the resulting `.pptx`).
- Both slides' backgrounds (grid lines, pulse-line graphic,
  "RESTRICTED" watermark, checkerboard strip) come from the template's
  slide layouts/master and are untouched — they appear in the
  generated `.pptx` even though the on-page live preview doesn't try to
  recreate them pixel-for-pixel.
