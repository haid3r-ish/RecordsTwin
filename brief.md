# 🏛️ Ultimate Master Architecture Document

## RecordTwin — Smart Legal & Revenue Document Vault

**Document Purpose:** This is the **single source of truth** for all architectural, feature, and implementation decisions. Use this document as the primary reference when generating the `brief.md`, domain knowledge, BRDs, epics, stories, and architecture artifacts within the `ascendra-framework`.

**Platform:** Android-first (native Kotlin). iOS is explicitly excluded from MVP scope.
**Date:** 2026-08-16

---

## 0. Final Sanity Check — All Flags Resolved

> [!NOTE]
> **Flag 1: Image Compression — RESOLVED (Option C: Hybrid)**
>
> - **On-Device (Android):** Compress captures to **720p WebP** using native `Bitmap.CompressFormat.WEBP_LOSSY`. Zero external libraries required. Works on all API levels from API 18.
> - **On-Server (NestJS):** Normalize received WebP uploads to **AVIF** for long-term object storage. AVIF achieves 30–50% better compression than WebP, maximizing server storage density.
> - **Client Download:** When the client requests cached images from the server (LRU refill, backup restore), the server serves **WebP thumbnails** to guarantee compatibility across all Android API levels.

> [!NOTE]
> **Flag 2: Headless LLM Scraper — Acknowledged Constraint**
>
> Free-tier throughput ceiling: **~50–80 documents/day** total across all users (multi-provider failover across ChatGPT, Claude.ai, HuggingChat). Survivable for MVP with < 10 beta testers. The dead-letter queue and circuit breaker patterns handle failures gracefully. The UI must communicate processing delays (e.g., "Your document is queued — we'll notify you when it's ready"). The `ITextWashingService` facade ensures a zero-friction swap to real APIs when funding arrives.

**Verdict:** No fatal architectural contradictions. All flags resolved. The architecture is sound.

---

# PART I — PROBLEM DOMAIN

---

## 1. Problem Statement

Managing agricultural land and navigating the legal and revenue systems in Pakistan relies almost entirely on archaic physical paperwork. This creates a highly inefficient, opaque, and error-prone environment characterized by three core pain points:

1. **Complex and Archaic Terminology Barrier:** The property and legal ecosystem heavily utilizes traditional 19th-century revenue terminology (Khasra, Khewat, Inteqal, Fard, etc.). For younger generations, this vocabulary acts as a massive barrier to entry, making it exceedingly difficult to understand their own assets without depending on government clerks (Patwaris). For older and illiterate users, the barrier is compounded by English-only digital interfaces.

2. **Poor Discoverability and Broken Document Lineage:** Critical documents — deeds, mutations, court orders, tax records — are stored in physical safes or flat digital folders with zero relational context. Finding historical relationships between decades-old registries, superseding court orders, and modern digital mutations is nearly impossible. Cross-referencing a legal dispute to the specific land parcel it concerns requires manual memory. Lineage tracking is fundamentally broken.

3. **Digitization Friction — OCR Failure on Nastaliq:** Traditional OCR technology completely fails at reading handwritten Nastaliq/Urdu property records and complex faded stamp papers. This prevents landowners from extracting any valuable metadata, text, or searchable context from their document images, leaving them as inert photo files.

---

## 2. Project Overview

**RecordTwin** is an offline-first Android mobile application designed for tech-literate landowners and inheritors in semi-urban and rural Pakistan. It transforms scattered physical land and legal records into a searchable, interlinked, AI-enhanced digital vault.

**Core Mission:** Provide landowners with immediate, independent visibility into their legal standing through a unified repository that seamlessly translates complex physical records into actionable digital insights — without requiring internet connectivity for core operations.

**Architectural Philosophy:**
- **Offline-First:** All folder hierarchies, document text, search indexes, and metadata are stored locally in an encrypted on-device SQLite database. The app is fully operational with zero connectivity.
- **Android-First:** Native Kotlin with Jetpack Compose. iOS is explicitly excluded from MVP scope.
- **AI-Augmented, Not AI-Dependent:** The backend LLM pipeline enhances text quality and search indexing, but the app remains fully functional (with degraded search quality) when the backend is unreachable.
- **Voice-First Ingestion:** Bypasses OCR entirely by allowing users or experts to dictate document contents using the phone's native speech-to-text engine.
- **Zero-Budget MVP:** Backend LLM processing uses headless browser scraping of free LLM interfaces, strictly isolated behind a swappable facade for future API migration.

---

## 3. Goals

### Business Goals
1. **Eliminate Intermediary Dependency:** Empower landowners to independently interpret and manage their legal/agricultural assets without relying on Patwaris or external legal counsel.
2. **Achieve Total Estate Transparency:** Enable instant tracing of historical lineage, legal associations, and cross-references for any document in the vault.
3. **Ensure Proactive Legal Compliance:** Surface upcoming court dates, tax deadlines, and mutation windows to prevent missed obligations.

### Technical Goals (MVP)
4. **Deliver a Fully Offline-Capable Vault:** All CRUD operations, search, and navigation must function with zero connectivity. Backend sync is a progressive enhancement, not a requirement.
5. **Build a Swappable AI Pipeline:** The headless LLM scraper must be strictly isolated behind an interface so the entire backend AI layer can be swapped to real APIs (OpenAI, Anthropic, Gemini) with zero client-side changes.
6. **Establish a Scalable Search Foundation:** The FTS5 hybrid keyword engine must support dual-script (Native Urdu + Roman Urdu) prefix matching with sub-10ms query latency on mid-range Android devices.