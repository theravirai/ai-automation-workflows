# Sample Invoice Data: INV-2026-0042

This document represents the synthetic test invoice used for end-to-end verification of the **AI Invoice Processing Pipeline**.

---

## 1. Raw Text Content (Simulated PDF OCR Extraction)

```text
================================================================================
                         RheinTech Solutions GmbH
              Software Engineering & Automation Consulting
                    Friedrichstraße 142, 10117 Berlin
                      Tax / VAT ID: DE999999999
                     Email: billing@rheintech.example
================================================================================

INVOICE

Invoice Number : INV-2026-0042
Invoice Date   : 25.02.2026
Due Date       : 11.03.2026
Payment Terms  : Net 14 days
Currency       : EUR

Billed To:
NordWest Digital Services GmbH
Kaiser-Wilhelm-Ring 22, 50672 Köln
Tax / VAT ID: DE888888888

--------------------------------------------------------------------------------
Pos  Description                            Qty   Unit Price    Total Amount
--------------------------------------------------------------------------------
01   AI Document Processing Setup             1     850.00 EUR     850.00 EUR
02   Workflow Automation Development          8      95.00 EUR     760.00 EUR
03   Integration and Testing                  1     320.00 EUR     320.00 EUR
--------------------------------------------------------------------------------

Subtotal (Net)                     : 1,930.00 EUR
VAT Rate                           : 19.00%
VAT Amount                         :   366.70 EUR
--------------------------------------------------------------------------------
Total Amount Payable (Gross)       : 2,296.70 EUR
================================================================================

Payment Details:
Bank Name : Deutsche Bank Berlin
IBAN      : DE12 3456 7890 1234 5678 90
BIC / SWIFT: GENODEM1XXX
Reference : INV-2026-0042

Thank you for your business.
================================================================================
```

---

## 2. Expected Structured Extraction Contract

```json
{
  "invoice_number": "INV-2026-0042",
  "invoice_date": "25.02.2026",
  "due_date": "11.03.2026",
  "vendor_name": "RheinTech Solutions GmbH",
  "vendor_vat_id": "DE999999999",
  "customer_name": "NordWest Digital Services GmbH",
  "customer_vat_id": "DE888888888",
  "subtotal": 1930.00,
  "vat_rate": 19,
  "vat_amount": 366.70,
  "total_amount": 2296.70,
  "currency": "EUR",
  "iban": "DE12 3456 7890 1234 5678 90",
  "payment_terms": "Net 14 days"
}
```

---

## 3. Formatted Google Sheets Row Entry

| Field | Spreadsheet Value |
| :--- | :--- |
| `invoice_number` | `INV-2026-0042` |
| `invoice_date` | `25.02.2026` |
| `due_date` | `11.03.2026` |
| `vendor_name` | `RheinTech Solutions GmbH` |
| `vendor_vat_id` | `DE999999999` |
| `customer_name` | `NordWest Digital Services GmbH` |
| `customer_vat_id` | `DE888888888` |
| `line_items` | `AI Document Processing Setup (1x 850.00) = 850.00; Workflow Automation Development (8x 95.00) = 760.00; Integration and Testing (1x 320.00) = 320.00` |
| `subtotal` | `1930` |
| `vat_rate` | `19` |
| `vat_amount` | `366.7` |
| `total_amount` | `2296.7` |
| `currency` | `EUR` |
| `iban` | `DE12 3456 7890 1234 5678 90` |
| `payment_terms` | `Net 14 days` |
