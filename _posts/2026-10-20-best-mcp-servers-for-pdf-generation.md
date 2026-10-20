---
layout: default
title: "Best MCP Servers for PDF Generation in 2026"
date: 2026-10-20
categories: ["Engineering", "How-To"]
tags: ["mcp", "model-context-protocol", "pdf-generation", "claude", "chatgpt", "ai-agents", "list"]
description: "The 7 best MCP servers for PDF generation in 2026, verified October 20, 2026 — hosted and local options compared on transport, templates, pricing, and honest limitations."
last_modified_at: 2026-10-20
---

**TL;DR:** MCP lets AI assistants call PDF generation tools directly from chat. We verified seven real MCP servers with PDF or document-generation capability as of October 20, 2026 — each with its transport, pricing, and honest limits.

*Written by Arun, Founder of [DocuQueue](https://docuqueue.com)*

*Published October 20, 2026*

---

The Model Context Protocol (MCP) lets an AI assistant like Claude or ChatGPT call external tools directly — so "make me a PDF of this" turns into a real generated document instead of a code snippet you paste into a renderer yourself. The problem: most "best MCP server" roundups are stale or list servers that no longer exist, so we re-verified every entry below against its live GitHub repo or official docs.

**List checked October 20, 2026.** Every non-DocuQueue entry here was verified live on that date. DocuQueue is our product, and we listed it first — with the same disclosure standard we'd want from anyone else: here's the honest fit and the limits.

## The ranked list

### 1. DocuQueue MCP

*Disclosure: DocuQueue is our product — we listed it first; here's the honest fit and limits.*

- **Repo:** [github.com/docuqueue/docuqueue-mcp](https://github.com/docuqueue/docuqueue-mcp) (MIT) · hosted endpoint: `https://docuqueue.com/mcp/v1`
- **What it does:** 10 tools for creating templates, previewing them, filling them with JSON, checking job status, and downloading finished PDFs — plus branding extraction and DOCX template upload.
- **Transport / auth:** Streamable HTTP at `https://docuqueue.com/mcp/v1`, or local install via `npx -y docuqueue-mcp`; works with Claude, Claude Code, ChatGPT, Gemini, Codex, OpenCode, and Cursor.
- **Pricing:** Free tier with 25 credits/month; no credit card required.
- **Best for:** Chat-driven document generation from reusable templates — invoices, contracts, certificates — without running a renderer yourself.
- **Limitation:** Credit-based (1 credit per document), and the cloud connectors require the hosted server rather than a purely local install.

The full tool spec and copy-paste client prompts live at [docuqueue.com/mcp](https://docuqueue.com/mcp).

### 2. Carbone MCP

- **Repo:** [github.com/carboneio/carbone-mcp](https://github.com/carboneio/carbone-mcp) (Apache-2.0) · hosted endpoint: `https://mcp.carbone.io` · npm: `carbone-mcp`
- **What it does:** Official MCP server for Carbone — fills Word, Excel, PowerPoint, HTML, or Markdown templates tagged with `{d.field}` syntax to produce PDF, DOCX, XLSX and 30+ other output formats, and batches up to 100 documents per request via webhook.
- **Transport / auth:** Hosted Streamable HTTP (sign-in flows through your browser with a Carbone account) or self-hosted via npm or Docker.
- **Pricing:** Free plan covers 100 documents/month without watermark; output is watermarked if you're on no plan at all.
- **Best for:** Teams that already design templates in Word or Excel and want production document automation from chat.
- **Limitation:** Templates are authored offline with `{d.field}` tags — there's no in-chat template editor — and batch generation is async, delivered to a webhook.

### 3. Playwright MCP (`browser_pdf_save`)

- **Repo:** [github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) · docs: [playwright.dev/mcp/tools/pdf](https://playwright.dev/mcp/tools/pdf)
- **What it does:** Microsoft's official browser-automation MCP server; with the `pdf` capability enabled, the `browser_pdf_save` tool exports the currently open page as a PDF.
- **Transport / auth:** Local stdio — `npx @playwright/mcp@latest --caps=pdf`; no account or API key.
- **Pricing:** Free and open-source.
- **Best for:** Printing live, JavaScript-heavy web pages — dashboards, receipts, rendered reports — exactly as Chromium sees them, print stylesheets included.
- **Limitation:** It's a general browser automation server, not a document generator: the agent has to navigate to and wait for the page first, and PDF export is Chromium-only.

### 4. docgen-mcp-server (cyanheads)

- **Repo:** [github.com/cyanheads/docgen-mcp-server](https://github.com/cyanheads/docgen-mcp-server) (Apache-2.0) · npm: `@cyanheads/docgen-mcp-server`
- **What it does:** Renders HTML, Markdown, or a `{{key}}` template plus data object to a downloadable PDF; also exports rows to `.xlsx` and fills AcroForm PDF forms.
- **Transport / auth:** Runs as stdio or a local Streamable HTTP server; auth is pluggable (none, JWT, or OAuth) when exposed over HTTP.
- **Pricing:** Free and open-source; no API keys required.
- **Best for:** Private, local document rendering when files shouldn't leave your machine.
- **Limitation:** The rendering engine is deliberately lightweight (headings, paragraphs, lists, tables) — complex CSS, images, or scripts can be dropped, which the tool reports as `degraded: true`.

### 5. pdfnative-mcp (Nizoka)

- **Repo:** [github.com/Nizoka/pdfnative-mcp](https://github.com/Nizoka/pdfnative-mcp) (MIT) · npm: `pdfnative-mcp`
- **What it does:** Local-first PDF generation with 28 tools — block-based documents, tables, charts, barcodes, AcroForms, PAdES digital signing, encryption, merge/split, plus PDF/A archival and PDF/X-4 print exchange output.
- **Transport / auth:** stdio or Streamable HTTP via `npx -y pdfnative-mcp`; no API keys.
- **Pricing:** Free and open-source.
- **Best for:** Compliance-heavy output — archival PDF/A, signed documents, fillable forms — produced entirely on your machine.
- **Limitation:** Documents are assembled from structured blocks rather than arbitrary HTML/CSS, and it requires Node.js 22 or newer.

### 6. FolioPDF MCP

- **Repo:** [github.com/foliopdf/foliopdf-mcp](https://github.com/foliopdf/foliopdf-mcp) (MIT)
- **What it does:** Single Go binary that renders HTML (with Tailwind CSS v3) and Markdown to PDF, plus merge, split, rotate, and text/metadata extraction.
- **Transport / auth:** Local stdio — install via `brew install foliopdf/tap/foliopdf-mcp` or `go install`; no network calls needed at render time.
- **Pricing:** Free and open-source; works offline.
- **Best for:** Developers who want one dependency for HTML/Markdown → PDF inside Claude Desktop or any stdio MCP client.
- **Limitation:** Local binary only — there's no hosted endpoint or template library, and the toolset stops at four tools.

### 7. MintPDF MCP

- **Repo:** [github.com/TrendTweekers/mintpdf](https://github.com/TrendTweekers/mintpdf) (MIT) · hosted endpoint: `https://mintpdf.dev/mcp`
- **What it does:** Takes HTML or Markdown (or a public URL) and returns a PDF download link — two tools, `generate_pdf` and `pdf_from_url`.
- **Transport / auth:** Hosted Streamable HTTP via `mcp-remote`, or local `npx -y mintpdf-mcp`; no account needed to try it.
- **Pricing:** Free to try on the hosted endpoint; MIT-licensed to self-host (the Docker image includes Chromium and fonts).
- **Best for:** Quick one-off conversions mid-conversation when you don't want to set up templates or infrastructure.
- **Limitation:** Deliberately has no template system and no storage — generated links expire after one hour, so it's not a document of record.

One honest note: if what you need is *reading* PDFs into your agent's context — text, tables, searched excerpts — rather than generating them, [jztan/pdf-mcp](https://github.com/jztan/pdf-mcp) (MIT, verified live) is the strongest dedicated option we checked, but it doesn't generate PDFs, so it isn't ranked above. For browsing the wider field yourself, [mcpservers.org's PDF topic page](https://mcpservers.org/topics/pdf-mcp) tracks 130+ PDF-related servers.

## How to choose an MCP server for PDFs

- **Transport.** stdio servers run as a local process — files stay on your machine, no account needed. Streamable HTTP hosted endpoints work with ChatGPT and cloud-hosted Claude, but your document data passes through their infrastructure.
- **Auth.** Local stdio servers typically need no keys. Hosted servers need OAuth or an API key — and it's worth knowing *where* each document is rendered before you connect one to sensitive data.
- **Template support.** Generating the same invoice or contract repeatedly? Pick a server with a real template library and JSON fill (DocuQueue, Carbone). One-off HTML or Markdown renders don't need it, and servers like MintPDF deliberately skip it.
- **Batch.** Need hundreds of documents per run? Look for batch tools with webhook or zip delivery (Carbone batches up to 100 per request) instead of looping one tool call at a time through the model's context.
- **Licensing and pricing.** Most servers here are MIT or Apache-2.0 and free to self-host — the cost lives in the hosted rendering layer: free tiers, credit systems, and watermark rules differ enough that it's worth comparing before wiring one into production. We published our own numbers at [docuqueue.com/mcp-bench](https://docuqueue.com/mcp-bench).

## FAQ

### What is the best MCP server for PDF generation?

For chat-driven generation from reusable templates with a free tier, DocuQueue (this list's #1, which we make). For teams with existing Word or Excel templates, Carbone's official MCP server is the established choice. For printing live web pages, Microsoft's Playwright MCP with `browser_pdf_save` is the most direct tool — all seven above were verified live on October 20, 2026.

### Can Claude generate PDFs via MCP?

Yes. Claude Desktop and Claude Code both support MCP servers — add the server via stdio config (`npx -y docuqueue-mcp` style) or connect a hosted Streamable HTTP endpoint, then ask Claude to generate a document in plain language. Our step-by-step walkthrough at [docuqueue.com/guides/how-to-generate-pdfs-with-claude](https://docuqueue.com/guides/how-to-generate-pdfs-with-claude) covers the full setup.

### How do I connect an MCP PDF server to ChatGPT?

In ChatGPT, add the server's Streamable HTTP URL as a custom connector (Settings → Connectors). Hosted endpoints like `https://docuqueue.com/mcp/v1` or `https://mcp.carbone.io` connect directly; stdio-only servers like FolioPDF or pdfnative-mcp need a bridge such as `mcp-remote`, or they simply stay in your local clients (Claude Code, Cursor, Codex).

### Are MCP PDF servers free?

Most of the servers in this list are open-source (MIT or Apache-2.0) and free to self-host — you supply the machine and the runtime. The hosted layer is where pricing varies: DocuQueue includes 25 free credits/month, Carbone's free plan covers 100 documents/month without watermark, and fully free hosted options like MintPDF expire downloads after an hour. Compare the free-tier terms before committing a workflow to one.

---

Ready to try one? The DocuQueue MCP endpoint, full tool spec, and copy-paste prompts for Claude, Claude Code, ChatGPT, Gemini, and Cursor are at [docuqueue.com/mcp](https://docuqueue.com/mcp).

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is the best MCP server for PDF generation?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "For chat-driven generation from reusable templates with a free tier, DocuQueue (this list's #1, which we make). For teams with existing Word or Excel templates, Carbone's official MCP server is the established choice. For printing live web pages, Microsoft's Playwright MCP with browser_pdf_save is the most direct tool — all seven above were verified live on October 20, 2026."
      }
    },
    {
      "@type": "Question",
      "name": "Can Claude generate PDFs via MCP?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes. Claude Desktop and Claude Code both support MCP servers — add the server via stdio config (npx -y docuqueue-mcp style) or connect a hosted Streamable HTTP endpoint, then ask Claude to generate a document in plain language. Our step-by-step walkthrough at docuqueue.com/guides/how-to-generate-pdfs-with-claude covers the full setup."
      }
    },
    {
      "@type": "Question",
      "name": "How do I connect an MCP PDF server to ChatGPT?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "In ChatGPT, add the server's Streamable HTTP URL as a custom connector (Settings → Connectors). Hosted endpoints like https://docuqueue.com/mcp/v1 or https://mcp.carbone.io connect directly; stdio-only servers like FolioPDF or pdfnative-mcp need a bridge such as mcp-remote, or they simply stay in your local clients (Claude Code, Cursor, Codex)."
      }
    },
    {
      "@type": "Question",
      "name": "Are MCP PDF servers free?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Most of the servers in this list are open-source (MIT or Apache-2.0) and free to self-host — you supply the machine and the runtime. The hosted layer is where pricing varies: DocuQueue includes 25 free credits/month, Carbone's free plan covers 100 documents/month without watermark, and fully free hosted options like MintPDF expire downloads after an hour. Compare the free-tier terms before committing a workflow to one."
      }
    }
  ]
}
</script>
