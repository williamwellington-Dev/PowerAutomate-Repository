# Attachment Extractor — Automated Equipment Log Archiving Flow

**Power Automate | Office 365 Outlook | SharePoint**

A workflow automation that captures equipment status logs arriving by email as HTML attachments and archives them into a designated SharePoint document library — turning an unsearchable inbox into an organized, team-owned operational record.

> **Note:** This is a sanitized rebuild of a production flow I designed, deployed, and maintain in a live broadcast engineering environment. All addresses, URLs, and identifiers shown here are from my personal demo environment.

---

## 1. Background & Business Problem

Remote transmission equipment automatically emails status logs to the engineering team as **HTML attachments**. Before this flow existed:

- Logs lived only in individual inboxes — no shared, central record
- Finding a specific day's log meant manually searching email history
- No consistent retention: logs disappeared when emails were deleted or mailboxes changed hands
- Troubleshooting reviews required reconstructing equipment history from scattered messages

**Goal:** every incoming log automatically archived to a shared SharePoint location, with zero manual effort from engineers.

## 2. Requirements

| # | Requirement | Type |
|---|-------------|------|
| R1 | Detect incoming equipment log emails automatically, without engineer action | Functional |
| R2 | Process only emails from the equipment's automated sender | Functional |
| R3 | Extract every attachment from each qualifying email | Functional |
| R4 | Store the original files, unmodified, in a designated SharePoint library | Functional |
| R5 | Handle attachments of varying sizes reliably | Non-functional |
| R6 | Run unattended using standard (non-premium) connectors — no added licensing cost | Non-functional |

## 3. Solution Design

The flow is deliberately minimal — four building blocks, each doing one job:

**Trigger — *When a new email arrives (V3)*** (Office 365 Outlook)
- Filtered by **sender address**, so only the equipment's automated emails start a run (R2)
- **`fetchOnlyWithAttachment: true`** — emails without attachments never trigger the flow at all, avoiding wasted runs
- Scoped to a **dedicated mail folder** rather than the whole inbox — a mailbox rule routes equipment mail into this folder first, keeping the trigger's footprint narrow
- **`splitOn`** the trigger output — each email is processed as its own independent flow run

**Loop — *Apply to each*** over the trigger's attachment collection (R3)

**Action — *Get Attachment (V2)*** (Office 365 Outlook)
- Fetches the full attachment content by `messageId` + `attachmentId`
- Used even though the trigger includes attachment metadata, because the explicit V2 fetch reliably returns complete `contentBytes` for larger files — the trigger's inline content is not guaranteed for all sizes

**Action — *Create file*** (SharePoint)
- Writes each attachment to the archive library using its original filename and raw `contentBytes` (R4)
- **Chunked transfer mode** enabled, so large log files upload reliably rather than failing on size (R5)

**Architecture diagram:** see [`/diagrams/flow-architecture.png`](./diagrams/flow-architecture.png)

**Design decisions worth noting:**

- **Filtering at the trigger, not in the loop.** Because the sender filter guarantees every triggering email comes from the equipment, all attachments are archival-worthy by definition — so no per-file type condition is needed. Gatekeeping upstream keeps the flow body simple.
- **Standard connectors only** (Outlook + SharePoint) — the entire flow runs without premium licensing (R6).
- **Originals preserved untouched** — the archive stores exactly what the equipment sent, which matters for troubleshooting integrity.

## 4. Outcome

- 100% of incoming logs archived automatically since deployment — zero manual filing
- Log retrieval went from "search your inbox and hope" to a direct SharePoint link
- Team-owned record survives mailbox turnover — equipment history no longer lives in personal inboxes
- The trigger/extract/archive skeleton is reusable for any recurring email-borne report

## 5. Roadmap — v2 (in development)

The next iteration, built in this repository's demo environment, extends the pipeline with **automated PDF conversion**: each archived HTML log will also be saved as a PDF copy (via a OneDrive for Business temporary file and the standard *Convert file* action), so non-technical stakeholders can open, print, and share logs without handling raw HTML. Design goals: both formats stored side by side, date-based naming convention (`TX-Log_YYYY-MM-DD`), and continued use of standard connectors only.

## 6. Repository Contents

| Path | Description |
|------|-------------|
| `/solution/AttachmentExtractor.zip` | Exported flow package (demo environment) — importable via Power Automate |
| `/diagrams/` | Before/after process map and flow architecture diagram |
| `/screenshots/` | Step-by-step flow designer screenshots |
| `/definition/flow-definition.json` | Sanitized flow logic definition (peek-code export) |

## 7. Importing This Flow

1. Power Automate → **My flows → Import → Import Package (Legacy)**
2. Upload `AttachmentExtractor.zip`
3. Remap the two connections (Office 365 Outlook, SharePoint) to your own
4. Update the SharePoint site/library path and the trigger's sender filter and mail folder
5. Send yourself a test email with an attachment from the filtered address and confirm the file lands in the library

---

*Built and maintained by William Wellington. Part of a portfolio of workflow-automation case studies — see the [profile README](../README.md) for more.*
