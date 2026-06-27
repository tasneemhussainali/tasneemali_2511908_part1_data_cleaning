# Data Cleaning Log
**Generated:** 2026-06-26 21:04
**Source file:** raw_orders.xlsx
**Output files:** cleaned_orders.xlsx, data_quality_report.xlsx, pivot_summary.xlsx

---

## 1. Issues Found

- Missing/unparseable order_date: 0 rows.
- Missing/unparseable ship_date: 0 rows.
- Ship date before order date: 22 rows.
- Exact duplicate rows found: 20.
- Duplicate order_id values with conflicting data: 12 unique IDs (24 rows).
- Inconsistent date formats across order_date and ship_date (at least 8 different formats detected: DD Mon YYYY, MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD, DD-MM-YYYY, etc.)
- Case inconsistencies in segment, region, category, sub_category, ship_mode, payment_status, order_status.
- Extra spaces (leading, trailing, double internal spaces) in multiple text fields.
- Negative discounts in 15 records (invalid, treated as bad data).
- Missing discount values in 26 records.
- Missing region in 25 records.
- Missing ship_mode in 21 records.
- 20 exact duplicate rows found.
- 12 order_ids appear multiple times with conflicting data (24 rows total).
- 22 records where ship_date is earlier than order_date.

---

## 2. Cleaning Actions Performed

- Cleaned 'customer_name': trimmed spaces, fixed case, removed special chars on 12 rows.
- Cleaned 'segment': trimmed spaces, fixed case, removed special chars on 18 rows.
- Cleaned 'region': trimmed spaces, fixed case, removed special chars on 17 rows.
- Cleaned 'category': trimmed spaces, fixed case, removed special chars on 18 rows.
- Cleaned 'sub_category': trimmed spaces, fixed case, removed special chars on 12 rows.
- Cleaned 'ship_mode': trimmed spaces, fixed case, removed special chars on 10 rows.
- Cleaned 'payment_status': trimmed spaces, fixed case, removed special chars on 6 rows.
- Cleaned 'order_status': trimmed spaces, fixed case, removed special chars on 13 rows.
- Standardised variant spellings in segment, category, sub_category, ship_mode (e.g. 'Small  Business' → 'Small Business').
- Removed 20 exact duplicate rows (kept first occurrence).
- Flagged 24 rows with conflicting order_ids as 'DUPLICATE_ORDER_ID - Review Required'. NOT removed.
- Quality flags assigned: Clean=541, Warning=313, Invalid=58.
- Parsed all date columns using 8 known format patterns; failed parses stored as blank.
- Standardised all successfully parsed dates to ISO format (YYYY-MM-DD).
- Normalised all text fields: TRIM (leading/trailing/internal spaces), Title Case, removed non-alphanumeric characters except hyphens.
- Collapsed variant spellings (e.g. 'Small  Business' → 'Small Business', 'Office  Supplies' → 'Office Supplies').
- Converted discount column to numeric; flagged non-convertible values.

---

## 3. Business Rules Applied

- Rule BR-01: 25 rows with missing region filled as 'Unknown' and flagged.
- Rule BR-02: 21 rows with missing ship_mode filled as 'Unknown' and flagged.
- Rule BR-03: 26 rows with missing discount set to 0 (all other sales fields valid).
- Rule BR-04: 15 rows with negative discount flagged as invalid.
- Rule BR-05: 0 rows with discount > 1 (>100%) flagged as invalid.
- Rule BR-06: 22 rows where ship_date < order_date flagged as invalid shipping records.
- Rule BR-07: Cancelled orders and Failed payment records excluded from completed sales pivots.
- Rule BR-08: Refunded orders summarised separately in pivot_summary.xlsx (Problem Orders by Region sheet).
- Rule BR-09: Only order_status='Completed' AND payment_status='Paid' records used for sales/profit pivot summaries.

---

## 4. Assumptions Made

- Missing discount treated as 0 only when unit_price, quantity, and sales are all present and non-null.
- Date format ambiguity (e.g. 01/07/2024 could be Jan 7 or Jul 1): Attempted MM/DD/YYYY first, then DD/MM/YYYY. Where year is unambiguous (YYYY in position), format was determined directly.
- Discount range: Valid discount treated as 0–1 (0%–100%). Values > 1 flagged as invalid.
- 'Small Business' treated as a distinct segment (not merged with 'Corporate' or others).
- Records with conflicting order_ids are retained with a flag rather than deleted, as the conflict may require manual resolution.
- For pivot summaries, 'completed sales' means order_status = 'Completed' AND payment_status = 'Paid'.

---

## 5. Records Removed

- **20 exact duplicate rows** removed (first occurrence retained).
- No conflicting duplicate records were deleted — they are flagged only.

---

## 6. Records Flagged

- **24 rows** flagged as DUPLICATE_ORDER_ID in duplicate_flag column.
- **15 rows** with negative discount (cleaned_discount set to NaN).
- **22 rows** with ship_date < order_date (shipping_delay_days is negative).
- **25 rows** region filled as 'Unknown'.
- **21 rows** ship_mode filled as 'Unknown'.

**Quality flag breakdown:**
- Clean: 541
- Warning: 313
- Invalid: 58

---

## 7. Limitations of the Cleaning Process

- **Ambiguous date formats:** Some dates (e.g. 05/08/2024) cannot be unambiguously parsed without additional context. The script attempts MM/DD/YYYY before DD/MM/YYYY, which may misclassify some dates.
- **Conflicting duplicates not auto-resolved:** Records with the same order_id but different data are flagged but not automatically merged, as the correct version cannot be determined without business context.
- **No external validation:** Region/state/city combinations are not validated against a geographic reference. Spelling errors in city or state names are not corrected beyond standard text cleaning.
- **Discount range assumption:** The upper valid discount bound is assumed to be 1.0 (100%). This is a conservative assumption and may need to be adjusted per business policy.
- **Sales formula mismatch:** Calculated sales (qty × unit_price × (1-discount)) may differ from reported sales due to rounding or unstated business rules (e.g. bulk pricing, additional fees). Mismatches are documented in data_quality_report.xlsx.
- **No currency normalisation:** All monetary values are assumed to be in the same currency (INR). No exchange rate conversion is applied.
