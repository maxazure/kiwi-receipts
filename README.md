# Kiwi Receipts 🧾🥝

An [OpenClaw](https://github.com/nichochar/openclaw) skill that turns receipt photos into IRD-ready GST reports for New Zealand businesses.

**Send a photo of your receipt via Telegram/WhatsApp → AI extracts the data → generates XLSX reports aligned with the official GST101A form.**

## Features

- **Vision-powered OCR** — snap a photo, get structured data
- **NZ GST compliant** — 15% rate, 3/23 tax fraction, IRD GST101A box mapping
- **XLSX export** — 4-sheet workbook: Summary, All Receipts, By Category, IRD GST101A
- **Auto-fills Box 11–14** — purchases, GST credit, and totals calculated from your receipts
- **Bilingual** — English and Chinese (中文) commands supported
- **Personal use** — runs on your own OpenClaw instance, no server needed
- **7-year compliant** — JSON storage for the Section 75 record-keeping requirement

## Quick Start

### 1. Install the skill

Copy the `kiwi-receipts` folder into your OpenClaw skills directory:

```bash
cp -r kiwi-receipts ~/.openclaw/skills/kiwi-receipts
# or symlink
ln -s $(pwd) ~/.openclaw/skills/kiwi-receipts
```

### 2. Install Python dependency

The report generator needs `openpyxl`:

```bash
pip install openpyxl
```

### 3. Set up your business info

Send "setup" to your OpenClaw bot:

```
You: setup
Bot: What's your business name?
You: My Construction Ltd
Bot: What's your GST/IRD number?
You: 12-345-678
Bot: ✅ Saved!
```

### 4. Start scanning receipts

Send a receipt photo and the skill will extract and store the data.

## Commands

| Command | Description |
|---------|-------------|
| 📸 *Send photo* | Capture a receipt |
| `setup` / `设置` | Set business name & GST number |
| `summary` / `汇总` | Current GST period overview |
| `report` / `报表` | Download XLSX report |
| `report 2026-03` | Report for specific period |
| `list` / `列表` | Show recent receipts |
| `delete last` / `删除上一条` | Remove last receipt |
| `help` / `帮助` | Show all commands |

## GST101A Box Mapping

The generated XLSX includes an IRD GST101A reference sheet:

| Box | Description | Source |
|-----|-------------|--------|
| 5 | Total sales and income (incl GST) | *You enter from accounting* |
| 6–10 | Sales GST calculations | *Auto-calculated* |
| **11** | **Total purchases (incl GST)** | **Auto-filled from receipts** |
| **12** | **GST credit (Box 11 × 3/23)** | **Auto-calculated** |
| 13 | Credit adjustments | *You enter if needed* |
| **14** | **Total GST credit** | **Auto-calculated** |
| 15 | GST to pay or refund | *Box 10 − Box 14* |

## Legal Compliance

This skill is built with reference to:

- **Goods and Services Tax Act 1985** (NZ) — Sections 2, 8, 15, 16, 19F, 20, 51, 75
- **Taxation (Annual Rates for 2021–22, GST, and Remedial Matters) Act 2022** — TSI framework (effective 1 April 2023)
- **IRD GST101A form** (2023 revision)

See [`references/nz-gst-guide.md`](references/nz-gst-guide.md) for the full compliance reference with section-by-section citations.

> **Disclaimer:** This tool assists with record-keeping and report generation. It is not a substitute for professional tax advice. Always verify your GST return figures with a qualified accountant before filing with IRD.

## File Structure

```
kiwi-receipts/
├── SKILL.md                      # OpenClaw skill definition
├── README.md                     # This file
├── scripts/
│   └── generate_report.py        # XLSX report generator
└── references/
    └── nz-gst-guide.md           # NZ GST compliance reference
```

## Data Storage

All data is stored locally on your machine:

```
~/.openclaw/data/kiwi-receipts/
├── config.json      # Your business name + GST number
└── receipts.json    # All captured receipts
```

## License

MIT
