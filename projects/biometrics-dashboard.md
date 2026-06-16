# Biometrics Dashboard

![Private](https://img.shields.io/badge/Private-&nbsp;-lightgrey)
![MVP](https://img.shields.io/badge/MVP-&nbsp;-blue)

A private, single-user dashboard for turning lab PDFs into structured biometric trend data. The project is built around a reviewable workflow: upload a PDF, parse it into normalized lab results, inspect the extraction, import confirmed rows, and compare biomarkers over time.

## Screenshots

<table>
<tr>
<td width="50%">
<img src="../assets/biometrics-dashboard/health-multi.webp" alt="Biometrics Dashboard with multiple selected lab trend charts">
</td>
<td width="50%">
<img src="../assets/biometrics-dashboard/health-single.webp" alt="Biometrics Dashboard focused on a single biomarker trend chart">
</td>
</tr>
</table>

## What It Does

- Authenticates a private dashboard with TOTP-backed session flow.
- Accepts lab PDF uploads and checks for duplicate source documents.
- Runs a Python parser pipeline that extracts PDF text and tables, falls back to OCR for sparse pages, and sends extracted text through a configured LLM provider.
- Presents parsed results for review before import.
- Stores confirmed reports and lab rows in MySQL.
- Charts selected biomarker groups with raw and normalized views.
- Streams the original source PDF only to authenticated sessions.

## Architecture

The public endpoint is plain PHP, JavaScript, and CSS under `html/`. Private parser and database scripts live under `scripts/`, with Apache denied access to that folder. The database schema is kept in `scripts/health_db.sql`; local user/grant SQL and upload workspaces are ignored.

The parser currently has three versions, with `v3` as the active path. It uses `pdfplumber`, `pypdfium2`, `pytesseract`, and Pillow, then routes extracted text through either OpenRouter or Ollama based on runtime configuration.

## Why It Matters

This project is a practical example of building around messy real-world input. The hard part is not only charting values; it is validating uploads, preserving source provenance, handling parser uncertainty, protecting private health data, and making the import step reviewable instead of blindly trusting automation.

## Status

MVP. The core private workflow works end to end, with room to improve parser tests, migration strategy, and fully database-driven chart definitions.
