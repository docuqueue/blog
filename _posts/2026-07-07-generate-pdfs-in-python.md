---
layout: default
title: "Generate PDFs in Python: Complete Guide 2026"
date: 2026-07-07
categories: ["Engineering", "How-To"]
tags: ["python", "pdf-generation", "html-to-pdf", "playwright", "weasyprint", "xhtml2pdf", "pdfkit"]
description: "How to generate a PDF in Python: working code for HTML, URLs, and forms, plus the best library for each job. Complete guide for 2026."
last_modified_at: 2026-10-21
---

**TL;DR:** To generate a PDF in Python, install WeasyPrint and call `HTML(string=html).write_pdf("output.pdf")` for HTML you control — use Playwright when the page needs JavaScript or lives at a URL, and an API when you'd rather not run a browser in production.

*Written by Arun, Founder of [DocuQueue](https://docuqueue.com)*

*Published July 7, 2026 · Updated October 21, 2026*

---

Generating a PDF shows up in almost every backend at some point — an invoice, a signed contract, a report a customer downloads. Python PDF generation has no shortage of ways to do it, and picking the wrong one usually shows up later: a slow request, a broken layout, or a dependency nobody wants to touch. This guide walks the full path — a first PDF in 30 seconds, the method for each real-world need (HTML, URL, PDF forms), honest detail on each library, and when handing the work to an API is the better choice.

## How to generate a PDF in Python: the quick answer

The simplest complete pipeline — no browser, no external service:

```bash
pip install weasyprint
```

```python
from weasyprint import HTML

HTML(string="<h1>Invoice #1042</h1><p>Total: $480.00</p>").write_pdf("invoice.pdf")
```

## Why HTML as your PDF source

Most teams end up generating PDFs from HTML rather than a dedicated PDF library - **You already know it.** CSS handles layout, branding, and print styles without learning a separate document format.
- **It's testable in a browser.** You can preview the exact markup before it ever becomes a PDF.
- **It's flexible.** Tables, images, page breaks, and even charts render the same way they would on a webpage.

The tradeoff is that "HTML to PDF" isn't one problem — a marketing site with client-side JavaScript and a static invoice template need very different renderers.

## Generate a PDF from HTML in Python

For HTML you already have — a string in your codebase, a template file, a report you build from data — you can either use a library in-process (detailed below) or post the string to an API (covered under [When to use an API](#when-to-use-an-api-instead-of-a-python-library)). If the HTML contains JavaScript, no pure-Python library will execute it — you need a browser (Playwright) or a service that runs one for you.

## Convert a URL to PDF in Python

No pure-Python library converts a URL on its own — a URL means JavaScript, network requests, and client-side rendering, which means a real browser. Playwright drives Chromium directly: `page.goto()` loads the URL, `page.pdf()` prints it.

```bash
pip install playwright
playwright install chromium
```

```python
from playwright.sync_api import sync_playwright

def url_to_pdf(url: str, output_path: str):
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto(url, wait_until="networkidle")
        page.pdf(path=output_path, format="A4")
        browser.close()
```

`wait_until="networkidle"` waits for the page's requests to settle before printing, which is usually what you want for client-rendered pages. The cost: Playwright downloads a real browser binary and keeps it patched, and under concurrent load each browser consumes meaningful memory. That's a fine trade on one server; it's the main reason teams eventually ask whether they need to own the browser at all.

## Python PDF form API: filling PDF forms with Python

Filling a fillable PDF is a different task from generating one — you have an existing PDF with form fields, and you need to write values into them.

**In-process:** `pypdf` writes field values directly (`PdfWriter` plus `update_page_form_field_values`), and `PyPDFForm` offers a higher-level template-and-fields interface. Both work well when you control the PDF and the volume is modest — you just need the field names.

**Via API:** if you'd rather not add a PDF-parsing dependency, DocuQueue exposes a form-fill endpoint at `POST https://docuqueue.com/api/v1/forms/upload` — send the fillable PDF and its field data, get a filled PDF back. See the [PDF filling API](https://docuqueue.com/pdf-filling-api) for details.

## Best Python library to convert HTML to PDF

Four libraries cover almost every case. Code first, honest limitations second.

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

**Honest limitation:** heaviest dependency of the four (a browser binary to download and patch), and memory-hungry if you launch a browser per request. For URLs, it's the only library option here that works at all.

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

**Honest limitation:** no JavaScript (it never runs any), and it needs system libraries (Pango, GDK-Pixbuf) — usually one package-manager install away, but worth knowing before you hit a slim container image.

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

**Honest limitation:** partial CSS support — modern flexbox, advanced selectors, and web fonts will silently misrender. Test your exact template, not a hello-world.

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

**Honest limitation:** the Python wrapper is maintained, but the engine underneath is archived — you'd be building new work on a frozen renderer, plus a system binary to install.

## Comparing the four

| | Playwright | WeasyPrint | xhtml2pdf | pdfkit |
|---|---|---|---|---|
| Renders JavaScript | Yes | No | No | Limited |
| CSS support | Strongest, browser-accurate | Strong for print/page layouts | Basic | Depends on wkhtmltopdf |
| Setup weight | Heavy (browser binary) | Moderate (system libs) | Light | Requires wkhtmltopdf install |
| Converts a URL | Yes | No | No | No |
| Best for | Live web pages, dashboards | Invoices, reports, contracts | Simple table-based docs | Legacy systems already using it |
| Maintenance outlook | Actively maintained | Actively maintained | Actively maintained | Underlying engine unmaintained |

Rule of thumb: if the source has to run JavaScript to look right — or is simply a URL — use Playwright. If you're generating a structured business document from data you already have, WeasyPrint is usually the cleaner fit. Reach for xhtml2pdf only when the layout is genuinely simple, and treat pdfkit as something to migrate away from rather than adopt fresh.

## When to use an API instead of a Python library

All four options above mean you're now responsible for the renderer: browser binaries or system packages to install, memory usage to watch under load, and — for the browser-based options — the exact kind of infrastructure that tends to be the first thing to break in production (timeouts, zombie processes, container image bloat).

If PDF generation is a small part of what your app does, it's often simpler to hand the rendering off and send HTML or a URL to an API. Convert an HTML string synchronously:

```python
import requests

response = requests.post(
    "https://docuqueue.com/api/v1/convert-html",
    headers={"api-key": "YOUR_API_KEY"},
    json={"html": "<h1>Invoice #1042</h1><p>Total: $480.00</p>"},
)
print(response.json()["job_id"])
```

Or submit a URL — conversion is asynchronous, so you get a `job_id` back immediately, then poll status and download the finished PDF:

```python
import requests

headers = {"api-key": "YOUR_API_KEY"}

job = requests.post(
    "https://docuqueue.com/api/v1/convert",
    headers=headers,
    json={"urls": ["https://example.com"]},
)
job_id = job.json()["job_id"]
status = requests.get(
    f"https://docuqueue.com/api/v1/status/{job_id}",
    headers=headers,
)
pdf = requests.get(
    f"https://docuqueue.com/api/v1/download/{job_id}",
    headers=headers,
)
with open("example.pdf", "wb") as f:
    f.write(pdf.content)
```

No browser install, no rendering engine to patch — one request in, a finished PDF out.

### If what you actually need is a document, not a page

If you're not converting an arbitrary page but generating the *same* structured document repeatedly — an invoice, a certificate, a contract — with your own data dropped in, you don't need to hand-build that HTML at all. The template fill endpoint takes a template and your data:

```python
import requests

response = requests.post(
    "https://docuqueue.com/api/v1/templates/YOUR_TEMPLATE_ID/fill",
    headers={"api-key": "YOUR_API_KEY"},
    json={"data": {"customer": "Acme Corp", "invoice_number": "INV-1042", "total": "$480.00"}},
)
```

Full disclosure: DocuQueue is my product — I built it after maintaining exactly the browser-renderer infrastructure this article describes. If you'd rather skip that infrastructure, [try it free](https://docuqueue.com/register): 25 credits per month, no credit card required. More on the [HTML-to-PDF API](https://docuqueue.com/html-to-pdf).

## Which path to take: an honest decision guide

- **Prototyping or a one-off HTML snapshot:** any of the four libraries will do; start with WeasyPrint if the source is your own HTML.
- **A URL, or a page that needs JavaScript:** Playwright — it's the only library here that runs a browser. If owning that browser infrastructure isn't something you want, use an API instead.
- **Recurring business documents (invoices, certificates, contracts) from structured data:** fill a template directly rather than hand-building HTML — less code to maintain, no rendering layer to patch.
- **Fillable PDF forms:** `pypdf` or `PyPDFForm` in-process, or the [form-fill API](https://docuqueue.com/pdf-filling-api) if you want to avoid the dependency.
- **Rendering at scale across many servers:** an API removes the browser-binary-and-memory problem from every deployment.

## Frequently Asked Questions

### How do I generate a PDF in Python?

Install WeasyPrint with pip install weasyprint, then call HTML(string=html).write_pdf("output.pdf") to turn an HTML string into a PDF in one line. If the content lives at a URL or needs JavaScript, use Playwright instead: page.goto(url) followed by page.pdf(path="output.pdf") prints the page from a real browser.

### What is the best Python library to generate PDFs in Python?

It depends on the source. WeasyPrint is the best Python library for HTML you control — invoices, reports, contracts — because it's light and paginates predictably. Playwright is the best choice for JavaScript-heavy pages and URLs, since it runs a real browser engine. xhtml2pdf suits simple table-based documents, and pdfkit wraps the archived wkhtmltopdf engine, so it's a legacy pick rather than a new one.

### How do I generate a PDF from HTML in Python?

Pass your HTML string to a renderer: WeasyPrint's HTML(string=html).write_pdf("output.pdf"), xhtml2pdf's pisa.CreatePDF(html, dest=f), or Playwright's page.set_content(html) followed by page.pdf(path="output.pdf"). If you don't want a rendering dependency in your app, post the HTML string to the convert-html endpoint of a PDF API instead.

### How do I convert a URL to PDF in Python?

Use Playwright: launch Chromium, call page.goto(url, wait_until="networkidle"), then page.pdf(path="output.pdf", format="A4"). No Python library can convert a URL without a browser, because the URL may need JavaScript to render — alternatively submit the URL to a PDF API's convert endpoint, which runs the browser for you.

### How do I fill a PDF form with Python?

Load the fillable PDF with pypdf's PdfWriter and write values into its fields with update_page_form_field_values, or use PyPDFForm for a higher-level template interface. If you'd rather not parse the PDF yourself, upload it to a form-fill API endpoint such as DocuQueue's POST /api/v1/forms/upload and get the filled PDF back.

### When should I use an API instead of a Python library?

Use an API when you'd rather not manage browser binaries, system libraries, and rendering memory across your own servers — especially when generating PDFs from JavaScript-heavy URLs or at scale. Libraries are the better fit when PDF generation is a small, stable part of one service and you want no external dependency.

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How do I generate a PDF in Python?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Install WeasyPrint with pip install weasyprint, then call HTML(string=html).write_pdf(\"output.pdf\") to turn an HTML string into a PDF in one line. If the content lives at a URL or needs JavaScript, use Playwright instead: page.goto(url) followed by page.pdf(path=\"output.pdf\") prints the page from a real browser."
      }
    },
    {
      "@type": "Question",
      "name": "What is the best Python library to generate PDFs in Python?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "It depends on the source. WeasyPrint is the best Python library for HTML you control — invoices, reports, contracts — because it's light and paginates predictably. Playwright is the best choice for JavaScript-heavy pages and URLs, since it runs a real browser engine. xhtml2pdf suits simple table-based documents, and pdfkit wraps the archived wkhtmltopdf engine, so it's a legacy pick rather than a new one."
      }
    },
    {
      "@type": "Question",
      "name": "How do I generate a PDF from HTML in Python?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Pass your HTML string to a renderer: WeasyPrint's HTML(string=html).write_pdf(\"output.pdf\"), xhtml2pdf's pisa.CreatePDF(html, dest=f), or Playwright's page.set_content(html) followed by page.pdf(path=\"output.pdf\"). If you don't want a rendering dependency in your app, post the HTML string to the convert-html endpoint of a PDF API instead."
      }
    },
    {
      "@type": "Question",
      "name": "How do I convert a URL to PDF in Python?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Use Playwright: launch Chromium, call page.goto(url, wait_until=\"networkidle\"), then page.pdf(path=\"output.pdf\", format=\"A4\"). No Python library can convert a URL without a browser, because the URL may need JavaScript to render — alternatively submit the URL to a PDF API's convert endpoint, which runs the browser for you."
      }
    },
    {
      "@type": "Question",
      "name": "How do I fill a PDF form with Python?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Load the fillable PDF with pypdf's PdfWriter and write values into its fields with update_page_form_field_values, or use PyPDFForm for a higher-level template interface. If you'd rather not parse the PDF yourself, upload it to a form-fill API endpoint such as DocuQueue's POST /api/v1/forms/upload and get the filled PDF back."
      }
    },
    {
      "@type": "Question",
      "name": "When should I use an API instead of a Python library?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Use an API when you'd rather not manage browser binaries, system libraries, and rendering memory across your own servers — especially when generating PDFs from JavaScript-heavy URLs or at scale. Libraries are the better fit when PDF generation is a small, stable part of one service and you want no external dependency."
      }
    }
  ]
}
</script>
