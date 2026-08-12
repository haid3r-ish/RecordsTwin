# Business Requirements Document (BRD)
**Project Name:** RecordTwin (Intelligent Land & Legal Records Management System)
**Document Version:** 001 (MVP Scope - Architecturally Hardened)

---

## 1. Executive Summary
RecordTwin is a specialized digital archiving and management system tailored for the Pakistani agricultural real estate and legal ecosystem. It bypasses error-prone OCR with a phonetically normalized Voice Dictation pipeline. The system acts as a secure digital ledger and contextual educational tool. It leverages a strictly consolidated PostgreSQL infrastructure (`pgvector` + `ltree`) to ensure ACID-compliant hybrid search (Semantic + Relational Lineage) without distributed database bloat.

## 2. System Objectives
* Provide a 1:1 structured digital hierarchy for physical legal and land records.
* Enable cross-lingual semantic search (dictate in Urdu, search in English) combined with exact graph-like lineage filtering.
* Replace automated text extraction (OCR) with user-verified, domain-normalized speech-to-text dictation.
* Facilitate on-the-fly learning of complex legal terminology via native OS-level sharing to external LLMs.
* Map chronological lifecycles of land mutations and legal disputes.

---

## 3. Pure Functional Requirements (The "What")

### 3.1 Document Ingestion & UI Handling
* **Capture & Upload:** Ingest documents via the device's native camera or gallery.
* **Resolution Management:** Compress images (e.g., 480p, 720p) and preview file size/quality before saving.
* **Manual Cropping:** A manual bounding-box cropping tool post-capture to remove background noise.
* **Parent-Child Grouping:** Users can upload a **Primary Document** and attach multiple **Sub-Documents** (CNIC copies, receipts) nested beneath it.
* **Split-Screen Verification Mode:** Document UI features a split-screen: top container for a zoomable image viewer, bottom container for a text area for dictation verification.

### 3.2 Dictation & Text Pipeline (Bypassing OCR)
* **Voice Input (STT):** In-app microphone toggle to record the user dictating the document in Urdu or Roman Urdu.
* **Transcription & Editing:** Audio is transcribed into raw text and populated into the bottom-half text editor for manual review and correction.

### 3.3 Cross-Lingual AI Processing & Metadata
* **Translation & Summarization:** The backend prompts an LLM to generate a concise summary of the transcribed text strictly in **English**.
* **Keyword & Entity Extraction:** The LLM extracts key entities (names, dates, dimensions, revenue terms) in English.
* **Vectorization:** The system generates vector embeddings for both the original transcription and the English summary.

### 3.4 Hierarchical Organization
* **Recursive Folder Structure:** Support self-referencing folders for infinite nesting depths.
* **Manual Navigation:** File-explorer style interface to create, rename, navigate, and manually drop files into folders.

### 3.5 AI-Assisted Smart Routing
* **Voice Command Routing:** The user can dictate a routing instruction (e.g., *"Place this in the judge's ongoing case folder"*).
* **System-Assisted Placement:** The system will evaluate the user's command against the existing folder structure and securely place the document in the correct folder, requesting user confirmation if needed.

### 3.6 Explicit Cross-Document Linking
* **Manual Linking:** Explicitly link any two Primary Documents across the system (e.g., linking a new *Inteqal* to an old *Registry*).
* **Relationship Annotation:** Input a text description defining the link relationship (e.g., *"This court order supersedes the 2018 registry"*).

### 3.7 Multilingual Semantic Search
* **Contextual Retrieval:** Accept queries in English or Roman Urdu to find semantically matching documents.
* **Granular Entity Filtering:** Toggles to restrict search results to: `Search All`, `Search Folders Only`, or `Search Files Only`.

### 3.8 Document Tagging & State Management
* **Custom Status Badges:** Apply functional colored tags to Primary Documents (e.g., `[Pending Mutation]`, `[Resolved]`).
* **Tag Search Integration:** Filter semantic search results strictly by specific document states.

### 3.9 Timeline & Deadline (Tareekh) Management
* **Chronological Folder View:** A "Timeline Toggle" sorts all documents in a folder chronologically based on a user-provided `Event_Date`.
* **Manual Deadline Tracking:** Input a `Hearing_Date` and `Hearing_Note` on any document.
* **Upcoming Dashboard:** Home screen dashboard aggregating all pending deadlines across all folders.

### 3.10 Multi-Document AI Synthesis & Export
* **Checkbox Selection:** Multi-select several Primary Documents.
* **Prompt Configuration:** Save default learning/comparison prompts (e.g., *"Explain the terms in this text"*).
* **Native OS Export:** A dedicated export button will bundle the selected images, transcribed text, and prompts, invoking the device's native sharing menu to pass the payload directly to external AI applications installed on the device.

---

## 4. Architectural Directives & Constraints (The "How" - Mandatory)

To prevent technical debt, N+1 queries, and hallucination loops, the execution AI must adhere to the following constraints:

* **Database & Vector Isolation:** Use **PostgreSQL exclusively**. Enable the `pgvector` extension for all vector storage and semantic search. **DO NOT** use Pinecone, ChromaDB, or Qdrant. Embeddings must be stored in the same tables as document metadata to guarantee ACID compliance.
* **Hierarchical State (No N+1):** Use the PostgreSQL `ltree` extension to manage the recursive folder structures. Standard recursive joins or parent_id lookups via TypeORM/Prisma are forbidden for deep folder traversals to prevent N+1 degradation.
* **AI Smart Routing Pre-Filter:** The entire folder tree must **never** be passed to the LLM context window. The backend must execute a vector-search pre-filter to retrieve the Top 5 most semantically relevant folders based on the voice command, and pass *only* those 5 candidates to the LLM to output a strictly validated JSON `folder_id`.
* **Speech-to-Text (STT) Domain Normalization:** The ingestion pipeline must include a dictionary normalizer or targeted LLM prompt step immediately after the STT transcription to correct hallucinated Pakistani revenue terminology (e.g., converting "Khabra" back to "Khasra") before it is embedded.
* **Android Export Mechanism:** For multi-document export, **do not** use the Android `ClipboardManager` for bundling images and text. Implement `Intent.ACTION_SEND_MULTIPLE` using `ShareCompat.IntentBuilder` to pass `content://` URIs and `EXTRA_TEXT` natively to external apps via the OS ShareSheet.

---

## 5. Out of Scope (Future Phases)
* Automated edge-detection and image auto-cropping.
* Direct in-app native LLM chat.
* Geospatial/Map View integration for dropping pins on specific acres.
* Automated CRON job push notifications for deadlines.
* Role-Based Access Control (RBAC) and secure read-only URL generation.