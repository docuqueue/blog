---
layout: default
title: "Batch PDF Generation: How to Generate PDFs from CSV or Excel"
date: 2026-11-19
categories: ["PDF Generation", "API", "Automation"]
tags: ["batch pdf generation", "bulk pdf", "csv to pdf", "excel to pdf", "pdf api", "document automation"]
description: "Generate personalized PDFs in bulk from a CSV or Excel file. Learn how batch PDF generation works in DocuQueue, including upload, field mapping, API workflow, limits, and ZIP download."
permalink: /batch-pdf-generation/
---

**TL;DR:** Batch PDF generation turns rows in a CSV or Excel file into personalized PDFs using one template. In DocuQueue, you upload a file, map columns to template fields, generate up to 50 PDFs per batch request, and download results individually or as a ZIP. Each generated document uses 1 credit.

*Written by Arun, Founder of [DocuQueue](https://docuqueue.com)*

---

If you need to create the same document for many people — invoices, certificates, onboarding letters, contracts, or shipping labels — creating PDFs one by one is slow and error-prone. Batch PDF generation solves that by merging structured data with a template.

![Batch PDF generation flow showing CSV or Excel upload, template mapping, and ZIP download of generated PDFs](https://docuqueue.com/blog/assets/images/batch-pdf-generation-flow.svg)

## What is batch PDF generation?

**Batch PDF generation** is the process of generating multiple PDFs from a single template and a structured data source, such as a CSV or Excel file. Each row in the data file becomes one personalized PDF.

For example, a CSV with 30 rows of invoice data can generate 30 invoice PDFs. The template stays the same; only the mapped fields change for each row.

## When to use batch PDF generation

Batch generation is a good fit when:

- **You have repeated document structure.** Invoices, certificates, letters, reports, and labels usually share the same layout.
- **Data already lives in a spreadsheet.** CSV/Excel exports from CRM, HR, ERP, or operations tools can become document inputs.
- **You need consistency.** Templates reduce formatting drift and manual copy-paste mistakes.
- **You need speed at moderate scale.** A batch request can generate many documents in one workflow instead of manual one-by-one exports.

## How batch PDF generation works in DocuQueue

DocuQueue supports batch generation through the dashboard/API workflow. The core flow is:

1. **Upload your data file.** Upload a CSV or Excel file. DocuQueue parses headers and rows for mapping.
2. **Select a template.** Use a built-in template or your own template with fields.
3. **Map columns to fields.** Match spreadsheet columns to template fields. Review the preview before running the full batch.
4. **Generate PDFs.** Each record creates a PDF generation job.
5. **Track and download.** Check batch status, download individual PDFs, or download all completed PDFs as a ZIP.

**[Try batch PDF generation free →](https://docuqueue.com/register?utm_source=blog&utm_medium=content&utm_campaign=batch-pdf-generation)**

## Verified batch limits and behavior

These details are based on the current DocuQueue API implementation:

| Item | Current behavior |
|------|------------------|
| **Input formats** | CSV and Excel (`.xlsx`) for batch upload |
| **Upload size** | Up to 10 MB |
| **Rows parsed from upload** | Up to 500 rows |
| **Records per batch generate request** | 1 to 50 records |
| **Credit cost** | 1 credit per generated document |
| **Batch status** | Aggregate batch status with per-job status |
| **Download** | Completed batch PDFs can be downloaded as a ZIP |

If a queueing failure happens after credits are deducted for unqueued jobs, DocuQueue refunds the unqueued jobs’ credits. Always check your current plan, rate limits, and product UI for the latest operational limits before large runs.

## Batch PDF generation via API

For developers, DocuQueue exposes batch endpoints that can be used programmatically.

### Typical API workflow

1. **Upload and parse file**

   `POST /api/v1/templates/{template_id}/batch-upload`

   Upload a CSV/Excel file as `multipart/form-data` (`file`). The response returns headers, rows, row count, and a preview.

2. **Generate batch PDFs**

   `POST /api/v1/templates/{template_id}/batch-generate`

   Send JSON with `records` (1–50) and optional `share`. The response includes `batch_id`, `job_ids`, `credits_used`, and `credits_remaining`.

3. **Check batch status**

   `GET /api/v1/batch/{batch_id}/status`

   Returns overall batch status (`pending`, `processing`, `completed`, `failed`) and per-job status/download URLs.

4. **Download ZIP**

   `GET /api/v1/batch/{batch_id}/download-zip`

   Downloads completed PDFs from the batch as a ZIP file.

### Example: generate a batch

```bash
curl -X POST "https://api.docuqueue.com/api/v1/templates/{template_id}/batch-generate" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "records": [
      {"name": "Asha", "course": "Safety Training", "date": "2026-11-19"},
      {"name": "Ravi", "course": "Safety Training", "date": "2026-11-19"}
    ],
    "share": false
  }'
```

The API returns a `batch_id`. Poll the batch status endpoint until jobs complete, then download the ZIP.

**[Read the API docs →](https://docuqueue.com/docs)**

## Practical insights for reliable bulk runs

- **Test with one row first.** Generate a sample PDF before running a full batch to catch mapping or formatting issues.
- **Keep headers clean.** Use clear column names like `customer_name`, `invoice_date`, or `total_amount`.
- **Validate data before upload.** Remove empty rows, fix date formats, and confirm required fields are present.
- **Use batches intentionally.** The generate endpoint accepts up to 50 records per request. For larger datasets, split into multiple batch requests and track each `batch_id`.
- **Plan for rate limits.** API rate limits depend on plan. If you automate large runs, add retry/backoff logic.
- **Keep an audit trail.** Store `batch_id`, `job_id`, input row identifiers, and output filenames in your system.

## Common batch PDF generation use cases

| Use case | Data source | Output |
|----------|-------------|--------|
| Invoices | Billing CSV/ERP export | One invoice PDF per customer |
| Certificates | Course completion spreadsheet | One certificate per attendee |
| HR letters | Employee onboarding sheet | Offer/confirmation letters |
| Contracts | CRM deal export | Personalized agreements |
| Shipping labels | Order export | Labels for fulfillment batches |

## Frequently Asked Questions

<details>
<summary>What is the difference between batch and bulk PDF generation?</summary>
<p>They are usually used interchangeably. Both mean generating multiple PDFs from structured data in one workflow. "Bulk PDF generation" is common in product pages; "batch PDF generation" is common in developer/API contexts.</p>
</details>

<details>
<summary>How many PDFs can I generate in one DocuQueue batch request?</summary>
<p>The current batch generate endpoint accepts 1 to 50 records per request. Each record generates one PDF job and uses 1 credit. For larger datasets, split the data into multiple batch requests.</p>
</details>

<details>
<summary>Can I upload CSV and Excel files?</summary>
<p>Yes. DocuQueue batch upload supports CSV and Excel (.xlsx) files. The upload endpoint parses headers and rows, enforces file/row limits, and returns data for mapping and preview.</p>
</details>

<details>
<summary>Can I download all generated PDFs at once?</summary>
<p>Yes. Once jobs in a batch are completed, you can use the batch ZIP download endpoint to download completed PDFs as a ZIP file.</p>
</details>

<details>
<summary>Does batch generation cost more credits?</summary>
<p>No special multiplier is added per document. Batch generation uses 1 credit per generated document, the same per-document credit model used for standard generation.</p>
</details>

---

## Related DocuQueue resources

- [Bulk PDF Generator](https://docuqueue.com/docs/bulk-pdf-generator) — Product page for CSV/Excel bulk generation
- [PDF Templates](https://docuqueue.com/templates) — Browse built-in templates
- [API Docs](https://docuqueue.com/docs) — REST API reference
- [PDF Form Filling API](https://docuqueue.com/pdf-filling-api) — Fill existing PDF forms programmatically
- [Pricing](https://docuqueue.com/pricing) — Plans, credits, and rate limits

*Need to generate personalized PDFs at scale? [Create a free DocuQueue account](https://docuqueue.com/register?utm_source=blog&utm_medium=content&utm_campaign=batch-pdf-generation) and start with free credits.*
