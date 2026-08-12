# Business Requirements Document (BRD)
**Project Name:** RecordTwin (Intelligent Land & Legal Records Management System)
**Document Version:** 1.0 (MVP Scope)

---

## 1. Executive Summary
RecordTwin is a specialized digital archiving and management system tailored for the Pakistani agricultural real estate and legal ecosystem. It replaces error-prone traditional OCR with a highly accurate Voice Dictation and Retrieval-Augmented Generation (RAG) pipeline. The system allows users to ingest, organize, interlink, and semantically search their physical property deeds, Patwari records, and court documents. It acts as both a secure digital ledger and a contextual educational tool to help users comprehend complex legal terminology.

## 2. System Objectives
* Provide a 1:1 structured digital hierarchy for physical legal and land records.
* Enable cross-lingual semantic search (dictate in Urdu, search in English).
* Replace automated text extraction (OCR) with user-verified speech-to-text dictation.
* Facilitate on-the-fly learning of complex legal terminology without integrating expensive, native LLM chat modules.
* Map chronological lifecycles of land mutations and legal disputes.

---

## 3. Detailed Functional Requirements

### 3.1 Document Ingestion & UI Handling
* **Capture & Upload:** The system shall allow users to ingest documents via the device's native camera or gallery selection.
* **Resolution Management:** The UI must provide a mechanism to compress images (e.g., 480p, 720p) and preview file size/quality before saving to optimize storage.
* **Manual Cropping:** A manual bounding-box cropping tool must be presented immediately post-capture to remove background noise.
* **Parent-Child Grouping:** 
  * Users can upload a **Primary Document** (e.g., e-Stamp paper, court order).
  * Users can attach multiple **Sub-Documents** (e.g., CNIC copies, bank slips, payment receipts) specifically nested under the Primary Document.
* **Split-Screen Verification Mode:** The document detail UI shall feature a split-screen layout. The top container will hold a zoomable, pannable image viewer, while the bottom container holds an editable text area for dictation verification without breaking user context.

### 3.2 Dictation & Text Pipeline (Bypassing OCR)
* **Voice Input (STT):** The application shall provide an in-app microphone toggle to record the user dictating or summarizing the document in Urdu or Roman Urdu.
* **Transcription & Editing:** The recorded audio must be transcribed into raw text and populated into the bottom-half text editor for manual review and correction prior to database submission.

### 3.3 Cross-Lingual AI Processing & Metadata
* **Translation & Summarization:** Upon submission of the transcribed text (Roman Urdu/Urdu), the backend shall prompt an LLM to generate a concise summary strictly in **English**.
* **Keyword & Entity Extraction:** The LLM shall extract key entities (names, dates, dimensions, revenue terms) in English and return them as a structured array.
* **Vectorization:** The system shall generate and store vector embeddings for *both* the original Roman Urdu transcription and the English summary using a multilingual embedding model.

### 3.4 Hierarchical Organization
* **Recursive Folder Structure:** The database must support self-referencing folders to allow infinite nesting depths (e.g., `Root > Agricultural Land > Village Name > Specific Acre > Civil Case`).
* **Manual Navigation:** The UI shall feature a file-explorer style interface to create, rename, navigate, and manually drop files into specific folders.

### 3.5 AI-Assisted Smart Routing
* **Voice Command Routing:** The user can append a routing instruction to their dictation (e.g., *"Place this in the judge's ongoing case folder"*).
* **Constrained JSON Output:** The backend shall provide the LLM with the user's existing directory tree and force the LLM to output a strict JSON payload containing only the target `folder_id`.
* **Security & Validation:** The backend must independently validate the LLM's JSON output (e.g., via Zod) to ensure the `folder_id` exists and belongs to the authenticated user before executing any database insertion.

### 3.6 Explicit Cross-Document Linking
* **Manual Linking:** The UI shall allow the user to explicitly link any two Primary Documents across the entire system regardless of their folder location (e.g., linking a new *Inteqal* to an old *Registry*).
* **Relationship Annotation:** When creating a link, the user must be able to input a text description defining the relationship (e.g., *"This court order supersedes the 2018 registry"*).

### 3.7 Multilingual Semantic Search
* **Contextual Retrieval:** The search bar must accept queries in English or Roman Urdu, converting the query into a vector to find semantically matching documents across the multilingual embeddings.
* **Granular Entity Filtering:** The search UI must provide toggles/dropdowns to restrict search results to:
  * `Search All`
  * `Search Folders Only` (returns strictly folder objects, ignoring the files inside)
  * `Search Files Only` (returns strictly document objects)

### 3.8 Document Tagging & State Management
* **Custom Status Badges:** Users can create and apply functional colored tags to Primary Documents (e.g., `[Pending Mutation]`, `[Active Dispute]`, `[Resolved]`).
* **Tag Search Integration:** Tags must be injected into the vector database metadata, allowing users to filter semantic search results strictly by specific document states.

### 3.9 Timeline & Deadline (Tareekh) Management
* **Chronological Folder View:** Every folder shall feature a "Timeline Toggle." When activated, the UI sorts and displays all documents chronologically based on a user-provided `Event_Date`.
* **Manual Deadline Tracking:** Users can manually input a `Hearing_Date` and `Hearing_Note` on any document.
* **Upcoming Dashboard:** The application home screen shall feature a dashboard aggregating and displaying all pending deadlines/hearings across all folders that are greater than or equal to the current date.

### 3.10 Multi-Document AI Synthesis & Export
* **Checkbox Selection:** The UI shall allow users to multi-select several Primary Documents within a folder or search result.
* **Prompt Configuration:** Users can define and save default learning/comparison prompts in their settings (e.g., *"Explain the terms in this text"*, or *"Compare the boundaries in these two documents"*).
* **Clipboard Bundling:** A dedicated export button will concatenate the images (if supported by clipboard/API), the transcribed text, the generated summaries, and the pre-configured prompt into a single payload copied to the user's system clipboard for instant pasting into an external LLM.

---

## 4. Architectural & Security Constraints
* **AI Database Isolation:** LLMs must never have direct write access to the database or SQL execution privileges. All AI routing must occur via validated JSON payloads.
* **Relational Integrity:** The system must use a robust SQL database (e.g., PostgreSQL) to handle the complex self-referencing folder hierarchies and parent-child document relationships.
* **Vector Separation:** A dedicated Vector Database (e.g., Pinecone, Qdrant, or pgvector) must be utilized alongside the relational database to handle embedding storage and semantic similarity execution.

---

## 5. Out of Scope (Future Phases)
* Automated edge-detection and image auto-cropping.
* Direct in-app native LLM chat (replaced by Clipboard Bundling in MVP).
* Geospatial/Map View integration for dropping pins on specific acres.
* Automated CRON job push notifications for deadlines.
* Role-Based Access Control (RBAC) and secure read-only URL generation for external sharing.