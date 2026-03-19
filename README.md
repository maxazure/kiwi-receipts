# Kiwi Receipts 🧾🥝

An [OpenClaw](https://github.com/nichochar/openclaw) skill that turns receipt photos into IRD-ready GST reports for New Zealand businesses.

**Send a photo of your receipt via Telegram/WhatsApp → AI extracts the data → generates XLSX reports aligned with the official GST101A form.**

## Features

- **Vision-powered OCR** — snap a photo, get structured data
- **NZ GST compliant** — 15% rate, 3/23 tax fraction, IRD GST101A box mapping
- **XLSX export** — 4-sheet workbook: Summary, All Receipts, By Category, IRD GST101A
- **Auto-fills Box 11–14** — purchases, GST credit, and totals calculated from your receipts
- **Bilingual** — English and Chinese commands supported
- **Personal use** — runs on your own OpenClaw instance, no server needed
- **7-year compliant** — JSON storage for the Section 75 record-keeping requirement

## Installation

### Option A: ClawHub (recommended)

```bash
# Install the ClawHub CLI if you haven't already
npm i -g clawhub

# Install the skill
clawhub install kiwi-receipts
```

### Option B: Git clone

```bash
git clone https://github.com/maxazure/kiwi-receipts.git ~/.openclaw/skills/kiwi-receipts
```

### Option C: Manual download

Download the repository and copy it into any of the supported skill directories:

```bash
# Workspace skills (highest priority)
cp -r kiwi-receipts ~/.openclaw/workspace/skills/kiwi-receipts

# Or managed skills directory
cp -r kiwi-receipts ~/.openclaw/skills/kiwi-receipts
```

You can also register an extra skill directory in `~/.openclaw/openclaw.json`:

```json
{
  "skills": {
    "load": {
      "extraDirs": ["/path/to/your/skills/folder"]
    }
  }
}
```

### Python dependency

The XLSX report generator requires `openpyxl`. It will auto-install on first run, or you can install it manually:

```bash
pip install openpyxl
```

## How It Works

### Daily: Snap and send

Bought materials, filled up the van, picked up safety gear? Just take a photo and send it to your OpenClaw bot (Telegram or WhatsApp):

```
You: [send a photo of your Bunnings receipt]
Bot: 🧾 Receipt captured:
     📍 Bunnings Warehouse
     📅 2026-03-19
     💰 $174.00 (GST: $22.70)
     📦 Timber 2x4 x10, Concrete Mix x5
     🏷️ Category: materials

     Reply ✅ to save, or correct any details.
You: ✅
Bot: Saved! (Period total: $1,527.37, 5 receipts)
```

That's it for the day. All data is stored locally on your machine.

### Every two months: Generate your GST report

New Zealand GST is filed **every two months**. When the period ends, just send `report`:

```
You: report
Bot: 📥 GST Report: Mar-Apr 2026
     Receipts: 23
     Total purchases: $8,420.50
     GST credit: $1,098.33
     [Download XLSX]
```

### Then: File on myIR

Open the XLSX — the **IRD GST101A** sheet has your numbers ready:

| Box | Description | Value |
|-----|-------------|-------|
| **11** | Total purchases (incl GST) | **$8,420.50** (auto-filled) |
| **12** | GST credit (Box 11 × 3/23) | **$1,098.33** (auto-calculated) |
| **14** | Total GST credit | **$1,098.33** (auto-calculated) |
| 5 | Total sales and income | *You fill this from your invoicing system* |

Log in to [myIR](https://www.ird.govt.nz/), copy the numbers into your GST return, and submit. Done.

### NZ GST filing periods and due dates

| Period | Due Date |
|--------|----------|
| Jan – Feb | 28 March |
| Mar – Apr | 28 May |
| May – Jun | 28 July |
| Jul – Aug | 28 September |
| Sep – Oct | 28 November |
| Nov – Dec | 15 January* |

*Special dates: periods ending 31 March are due **7 May**, periods ending 30 November are due **15 January**.*

### In short

**Daily** — snap receipt, send photo, confirm, done.

**Every 2 months** — send `report`, get XLSX, copy numbers to myIR, file.

No more shoeboxes full of faded receipts at tax time.

## First-Time Setup

Send `setup` to your OpenClaw bot:

```
You: setup
Bot: What's your business name?
You: My Construction Ltd
Bot: What's your GST/IRD number?
You: 12-345-678
Bot: Saved!
```

## Commands

| Command | Description |
|---------|-------------|
| 📸 *Send photo* | Capture a receipt |
| `setup` | Set business name & GST number |
| `summary` | Current GST period overview |
| `report` | Download XLSX report |
| `report 2026-03` | Report for a specific period |
| `list` | Show recent receipts |
| `delete last` | Remove last receipt |
| `help` | Show all commands |

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
