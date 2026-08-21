# 📁 Evidence — Extracted OLE Objects & Media

> This folder contains all embedded OLE objects and media files that were previously embedded inside `Testcases.xlsx`.
> They were extracted to reduce the Excel file size from **~46 MB → ~0.08 MB**, enabling GitHub Preview compatibility (< 25 MB limit).

---

## Contents

| File | Type | Approx. Size | Description |
|---|---|---|---|
| `oleObject1.bin` – `oleObject18.bin` | OLE Binary | 0.5 MB – 42 MB | Embedded evidence objects (screenshots/videos) from sheet `02. Test Case` |
| `image1.emf` – `image18.emf` | EMF Image | ~7 KB each | Icon/thumbnail images paired with each OLE object |
| `image19.png` | PNG Image | ~32 KB | Additional screenshot reference |

---

## OLE Object Mapping

All OLE objects originated from sheet `02. Test Case` of `Testcases.xlsx`.
They were referenced via `xl/worksheets/_rels/sheet3.xml.rels` as `rId4` through `rId39`.

The **backup file** with all OLE objects still embedded is:
`Testcases_backup_20260821_195803.xlsx` (46.25 MB)

---

## How to Access

To view the original embedded objects:
1. Open `Testcases_backup_20260821_195803.xlsx` in Microsoft Excel
2. The OLE objects will be visible as embedded items in the `02. Test Case` sheet

Alternatively, the `.bin` files can be opened directly if you know their format (most are `.mp4` video or `.png` image files embedded via Excel OLE).

---

> *Evidence extracted as part of the Medusa V2 QA Portfolio project — Nguyen Le*
