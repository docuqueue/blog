---
layout: default
title: "How to Convert HTML to PDF in Python (2026 Guide)"
date: 2026-07-07
categories: ["Engineering", "How-To"]
tags: ["python", "html-to-pdf", "playwright", "weasyprint", "xhtml2pdf", "pdfkit"]
---

Generating a PDF from HTML shows up in almost every backend at some point — an invoice, a signed contract, a report a customer downloads. Python has no shortage of ways to do it, and picking the wrong one usually shows up later, as a slow request, a broken layout, or a dependency nobody wants to touch.

This guide walks through the main options — when to reach for a browser-based renderer, when a pure-Python library is enough, and when it's worth skipping the infrastructure entirely and calling an API instead.

## Why HTML as your PDF source

Most teams end up generating PDFs from HTML rather than a dedicated PDF library because:

- **You already know it.** CSS handles layout, branding, and print styles without learning a separate document format.
- **It's testable in a browser.** You can preview the exact markup before it ever becomes a PDF.
- **It's flexible.** Tables, images, page breaks, and even charts render the same way they would on a webpage.

The tradeoff is that "HTML to PDF" isn't one problem — a marketing site with client-side JavaScript and a static invoice template need very different renderers.

## The Python options

### Playwright — best for real, JavaScript-heavy pages

Playwright drives an actual browser engine, so if your source page depends on client-side rendering, this is usually the right tool. It's heavier to set up (it installs a real browser binary) but the output matches what a user actually sees.

```bash
pip install playwright
playwright install chromium
```

```python
from playwright.sync_api import sync_playwright

def html_to_pdf(html: str, output_path: str):
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.set_content(html, wait_until="networkidle")
        page.pdf(path=output_path, format="A4")
        browser.close()
```

### WeasyPrint — best for structured documents

WeasyPrint doesn't run a browser at all — it's a dedicated HTML/CSS-to-PDF rendering engine, which makes it lighter to deploy and well suited to print-style layouts like invoices and reports, where you want predictable pagination rather than a snapshot of a live page.

```bash
pip install weasyprint
```

```python
from weasyprint import HTML

def html_to_pdf(html: str, output_path: str):
    HTML(string=html).write_pdf(output_path)
```

### xhtml2pdf — best for simple layouts

xhtml2pdf is the lightest dependency of the group, with no browser and no external rendering engine — but it also supports the least CSS, so it's a better fit for simple, table-heavy documents than for anything visually complex.

```bash
pip install xhtml2pdf
```

```python
from xhtml2pdf import pisa

def html_to_pdf(html: str, output_path: str):
    with open(output_path, "wb") as f:
        pisa.CreatePDF(html, dest=f)
```

### python-pdfkit — a maintained wrapper, over an unmaintained engine

pdfkit is a thin Python wrapper around `wkhtmltopdf`. It's still in a lot of production codebases, but worth flagging clearly: the `wkhtmltopdf` project itself is archived and no longer maintained, so this is generally a "keep it running" choice for existing systems rather than a pick for something new.

```bash
pip install pdfkit
```

```python
import pdfkit

def html_to_pdf(html: str, output_path: str):
    pdfkit.from_string(html, output_path)
```

## Comparing the four

| | Playwright | WeasyPrint | xhtml2pdf | pdfkit |
|---|---|---|---|---|
| Renders JavaScript | Yes | No | No | Limited |
| CSS support | Strongest, browser-accurate | Strong for print/page layouts | Basic | Depends on wkhtmltopdf |
| Setup weight | Heavy (browser binary) | Moderate (system libs) | Light | Requires wkhtmltopdf install |
| Best for | Live web pages, dashboards | Invoices, reports, contracts | Simple table-based docs | Legacy systems already using it |
| Maintenance outlook | Actively maintained | Actively maintained | Actively maintained | Underlying engine unmaintained |

Rule of thumb: if the source has to run JavaScript to look right, use Playwright. If you're generating a structured business document from data you already have, WeasyPrint is usually the cleaner fit. Reach for xhtml2pdf only when the layout is genuinely simple, and treat pdfkit as something to migrate away from rather than adopt fresh.

## When a library isn't the right layer

All four options above mean you're now responsible for the renderer: browser binaries or system packages to install, memory usage to watch under load, and — for the browser-based options — the exact kind of infrastructure that tends to be the first thing to break in production (timeouts, zombie processes, container image bloat).

If PDF generation is a small part of what your app does, it's often simpler to hand the rendering off entirely and just send HTML or a URL to an API:

```python
import requests

response = requests.post(
    "https://api.docuqueue.com/v1/convert/html",
    headers={"Authorization": "Bearer YOUR_API_KEY"},
    json={"html": "<h1>Invoice #1042</h1><p>Total: $480.00</p>"},
)

pdf_url = response.json()["download_url"]
```

Or straight from a URL:

```python
response = requests.post(
    "https://api.docuqueue.com/v1/convert/url",
    headers={"Authorization": "Bearer YOUR_API_KEY"},
    json={"url": "https://example.com/invoice/1042"},
)
```

No browser install, no rendering engine to patch — you get a finished PDF back from one request.

### If what you actually need is a document, not a page

There's a related but different problem worth calling out: if you're not converting an arbitrary page, but generating the *same* structured document repeatedly — an invoice, a certificate, a contract — with your own data dropped in, you don't need to hand-build that HTML at all. DocuQueue's template `fill` endpoint takes a template and your data directly:

```python
response = requests.post(
    "https://api.docuqueue.com/v1/templates/invoice/fill",
    headers={"Authorization": "Bearer YOUR_API_KEY"},
    json={
        "customer": {"name": "Acme Corp"},
        "invoice_number": "INV-1042",
        "line_items": [
            {"name": "Consulting", "quantity": 4, "unit_price": 120},
        ],
    },
)
```

And for generating many at once — end of month, end of cohort, end of a batch job — `fill-batch` takes an array of records and returns a PDF per entry in one call, rather than looping requests one at a time.

## Which path to take

- **Prototyping or a one-off page snapshot:** any of the four Python libraries will do; start with WeasyPrint if the source is your own HTML.
- **Rendering JavaScript-heavy pages in production, at scale:** either Playwright with real infrastructure planning around it, or an API that runs the browser for you.
- **Recurring business documents (invoices, certificates, contracts) from structured data:** skip HTML generation entirely and fill a template directly — it's less code to maintain and there's no rendering layer to keep patched.

*DocuQueue converts URLs and HTML to PDF, and fills 6 professional templates (invoice, contract, certificate, letter, report, shipping label) with your own data via API. [Try it free](https://docuqueue.com/register) — 50 credits/month, no credit card required.*
