---
name: kiwi-receipts
description: "Process receipt photos into IRD-ready GST reports for NZ businesses. Use when: (1) user sends a receipt/invoice photo, (2) user asks for GST summary or report, (3) user wants to export receipts for tax filing, (4) user mentions IRD, GST return, or tax receipts. Triggers on images with receipt-like content or text mentioning receipts/invoices/GST."
metadata:
  {
    "openclaw":
      {
        "emoji": "🧾",
        "triggers": ["receipt", "invoice", "gst", "ird", "tax", "小票", "发票"],
      },
  }
---

# Kiwi Receipts 🧾🥝

Process receipt and invoice photos into IRD-compliant GST reports for New Zealand businesses. Send photos via Telegram/WhatsApp, the skill extracts data using vision AI, accumulates receipts, and generates XLSX reports ready for GST filing.

This is a personal-use skill — each user runs their own instance. No multi-tenant, no login.

## Data Paths

```
~/.openclaw/data/kiwi-receipts/
├── config.json      # Business name, GST number
└── receipts.json    # All captured receipts
```

## First-Time Setup

When user sends "setup" or "设置", or on first use when `config.json` doesn't exist:

1. Ask for business name
2. Ask for GST/IRD number
3. Save to `~/.openclaw/data/kiwi-receipts/config.json`:

```json
{
  "business_name": "MaxBuild Construction Ltd",
  "gst_number": "88-123-456"
}
```

## Handling Receipt Photos

When the user sends an image (check for `MediaPaths` in context):

### Step 1: Analyze the image

Use your vision capabilities to analyze the receipt image. Extract:

```json
{
  "merchant": "Bunnings Warehouse",
  "date": "2026-03-15",
  "items": [
    { "description": "Timber 2x4 3.6m", "quantity": 10, "unit_price": 12.50, "amount": 125.00 },
    { "description": "Concrete Mix 40kg", "quantity": 5, "unit_price": 9.80, "amount": 49.00 }
  ],
  "subtotal": 174.00,
  "gst": 22.57,
  "total": 174.00,
  "gst_number": "123-456-789",
  "payment_method": "EFTPOS",
  "category": "materials"
}
```

**Extraction rules:**
- NZ GST is 15%. If only total is visible, calculate GST as `total × 3/23`
- If GST is shown separately on the receipt, use that value
- Detect the GST number if printed on the receipt
- Classify into categories: `materials`, `tools`, `fuel`, `safety`, `subcontractor`, `office`, `vehicle`, `other`
- Dates: parse to ISO format (YYYY-MM-DD), assume current year if not shown
- All amounts in NZD

### Step 2: Confirm with user

Send back a summary:

```
🧾 Receipt captured:
📍 Bunnings Warehouse
📅 2026-03-15
💰 $174.00 (GST: $22.57)
📦 2 items: Timber 2x4 3.6m ×10, Concrete Mix 40kg ×5
🏷️ Category: materials

Reply ✅ to save, or correct any details.
```

### Step 3: Save receipt data

After confirmation, append to `~/.openclaw/data/kiwi-receipts/receipts.json`:

```bash
mkdir -p ~/.openclaw/data/receipt-to-ird
```

Read existing `receipts.json` (or start with `[]`), append the new receipt with a generated UUID as `id`, and write back. Each receipt object:

```json
{
  "id": "uuid-here",
  "merchant": "...",
  "date": "2026-03-15",
  "items": [...],
  "subtotal": 174.00,
  "gst": 22.57,
  "total": 174.00,
  "gst_number": "...",
  "category": "materials",
  "payment_method": "EFTPOS",
  "created_at": "2026-03-15T10:30:00Z"
}
```

## Handling Text Commands

### "setup" or "设置"
Create or update `config.json` with business name and GST number.

### "summary" or "汇总"
Read `receipts.json`, filter to current GST period, show:
```
📊 GST Period: Mar-Apr 2026
Total purchases: $1,527.37
Total GST claimable: $199.13
Receipts: 5

By category:
  Materials: $882.97 (GST: $115.17)
  Tools: $114.00 (GST: $14.87)
  Fuel: $131.40 (GST: $17.14)
  Safety: $399.00 (GST: $51.95)
```

### "report" or "报表" or "export"
Generate and send XLSX report:

```bash
python3 {baseDir}/scripts/generate_report.py \
  --data ~/.openclaw/data/kiwi-receipts/receipts.json \
  --output /tmp/gst-report.xlsx \
  --period current \
  --business-name "from config.json" \
  --gst-number "from config.json"
```

Then send the file back to user via message tool with `sendAttachment` action.

### "report YYYY-MM" or "报表 2026-01"
Generate report for a specific GST period (the 2-month period containing that month).

**XLSX report contains 4 sheets:**

1. **GST Summary** — Business info, period, total purchases/GST
2. **All Receipts** — Date, merchant, category, items, amounts
3. **By Category** — materials/tools/fuel/safety/etc. subtotals
4. **IRD GST101A** — Pre-filled with official box numbers:
   - Box 5: Total sales (user fills from accounting records)
   - Box 6: Zero-rated supplies included in Box 5
   - Box 7: Box 5 - Box 6
   - Box 8: Box 7 × 3/23 (GST on sales)
   - Box 9: Adjustments
   - Box 10: Total GST collected (Box 8 + Box 9)
   - Box 11: Total purchases incl GST (**auto-filled from receipts**)
   - Box 12: Box 11 × 3/23 (GST credit, **auto-calculated**)
   - Box 13: Credit adjustments
   - Box 14: Total GST credit (Box 12 + Box 13, **auto-calculated**)
   - Box 15: Box 10 - Box 14 (pay or refund)

### "delete last" or "删除上一条"
Remove the most recently added receipt from `receipts.json`.

### "list" or "列表"
Show recent receipts (last 10) with date, merchant, total.

### "help" or "帮助"
```
🧾 Receipt to IRD - Commands:
📸 Send a receipt photo to capture it
⚙️ "setup" - Set business name & GST number
📊 "summary" - Current period overview
📥 "report" - Download GST report (XLSX)
📋 "list" - Show recent receipts
🗑️ "delete last" - Remove last receipt
❓ "help" - This message
```

## GST Period Logic

NZ GST periods (2-monthly, most common for small business):
- Period 1: Jan-Feb → due 28 Mar
- Period 2: Mar-Apr → due 28 May
- Period 3: May-Jun → due 28 Jul
- Period 4: Jul-Aug → due 28 Sep
- Period 5: Sep-Oct → due 28 Nov
- Period 6: Nov-Dec → due 28 Jan

## Compliance Rules

Per the Goods and Services Tax Act 1985 (NZ) and IRD guidelines:

### GST Calculation (Section 8, Section 2 "tax fraction")
1. NZ GST standard rate is **15%** — no exceptions for construction
2. Tax fraction is **3/23** — used to extract GST from inclusive prices
3. Box 12 on GST101A = Box 11 × 3/23 (do NOT sum per-receipt GST)
4. Round to **2 decimal places** (Section 24(6): ≤0.5¢ drop, >0.5¢ round up)

### Taxable Supply Information (Section 19F, effective 1 April 2023)
5. **Under $50**: no TSI required (still best practice to keep receipt)
6. **$50–$200**: need supplier name, date, description, total amount
7. **$200–$1,000**: also need supplier's GST/IRD number + GST shown or "incl GST" statement
8. **Over $1,000**: also need buyer's name + address/phone/email
9. **Don't guess GST numbers** — only record if clearly visible on receipt
10. Supplier must provide TSI within **28 days** of request

### Input Tax Deduction (Section 20(3))
11. Only claim GST on **business** expenses — mixed use must be apportioned
12. Only claim GST from **registered** suppliers (check for GST number)
13. Do NOT claim GST on exempt supplies (residential rent, financial services)
14. **Fuel**: if vehicle has mixed personal/business use, apportion accordingly
15. **Second-hand goods** from unregistered seller: can claim input tax as lesser of purchase price × 3/23 or market value × 3/23 (Section 3A(1)(c))

### Record Keeping (Section 75)
16. Retain all records for **minimum 7 years** (Commissioner may extend to 10)
17. Records must be in **English** and kept **in New Zealand**
18. Electronic records (photos, JSON, XLSX) are acceptable if complete and legible

### Filing (Section 15, 16)
19. GST returns filed electronically via **myIR** (ird.govt.nz)
20. Due by **28th of the month** following period end, except:
    - March period → due **7 May**
    - November period → due **15 January**
21. Late filing penalty: **$50–$250** per return
22. Late payment: **1% immediate** + **4% after 7 days** + interest at ~10.91% p.a.

### General
23. **Always confirm** before saving — OCR can make mistakes
24. **Category matters** — ask user if unclear
25. **Keep it conversational** — concise, friendly, chat-native
26. **Foreign currency**: not directly claimable — convert to NZD at supply date rate
