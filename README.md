# VernacPay-Global (VernacPay-AI v2.0)
### Multilingual Voice-Native Agentic Commerce & Cross-Border Export Operating System Built on PayPal

> **"Speak Your Language. Sell to the World. Protected by Deterministic AI."**
> 
> * Built for the **PayPal AI Hackathon 2026**
> * **Live URL:** [https://vernacpay-global-app.onrender.com](https://vernacpay-global-app.onrender.com)
> * **GitHub Repository:** [https://github.com/ayeshasultana78/VernacPay](https://github.com/ayeshasultana78/VernacPay)

---

## 1. Abstract & Problem Statement

Over **15 million vernacular micro-exporters, artisans, handloom weavers, and independent instructors across India** produce high-value goods and services sought after by buyers worldwide. Yet they remain excluded from direct global commerce due to two compounding barriers:
1. **Language Exclusion:** Exporters speak and negotiate naturally in Indic languages (Telugu, Hindi, and Indian English), while international payment setups require navigating complex English-only dashboards.
2. **Cross-Border Payment Friction:** Converting spoken Indian Rupee (INR) amounts into international currencies (USD), sharing immediate checkout links, and reconciling incoming cross-border settlements with bank-grade safety is error-prone.

**VernacPay-Global** bridges this divide by combining **Sarvam AI Indic Speech** (`saaras:v3` STT & `bulbul:v3` TTS), high-speed **Groq LLM Function Calling** (`openai/gpt-oss-120b`), and the **PayPal Developer Platform** (Orders v2 API with instant checkout QR codes and cryptographic webhook capture). 

An artisan in Hyderabad can speak in Telugu:
> *"న్యూయార్క్ లో ఉన్న సారా కి పోచంపల్లి పట్టు చీర కోసం 12,000 రూపాయల పేపాల్ ఆర్డర్ పంపండి"*  
> *(Send a PayPal order to Sarah in New York for 12,000 rupees for a Pochampally silk saree)*

VernacPay-Global deterministically locks an illustrative USD quote ($142.86 USD @ 1 USD = ₹84.00 INR), validates the request via Zod schemas, and presents a bilingual confirmation card. Only after explicit merchant authorization does the system invoke PayPal Sandbox APIs to create a real **PayPal Sandbox Order & QR Code**, enabling the foreign buyer to pay on PayPal Sandbox — with verified settlements committed to an **AG Grid Community** ledger.

---

## 2. The 5-Stage Deterministic Trust Boundary

VernacPay-Global enforces an unbreakable 5-stage state machine where untrusted natural language models never directly mutate financial records or trigger unapproved payments:

```
+-----------------------------------------------------------------------------------------------------+
|                                 VERNACPAY-GLOBAL: 5-STAGE TRUST BOUNDARY                            |
+-----------------------------------------------------------------------------------------------------+

  [1. AI PROPOSES]  ──>  [2. SERVER VALIDATES]  ──>  [3. MERCHANT AUTHORIZES] ──> [4. PAYPAL PROCESSES] ──> [5. LEDGER VERIFIES]
  (Untrusted Speech)     (Zod + Locked FX Quote)    (Explicit Voice OR 1-Tap)     (Orders v2 + Live QR)     (Return Capture / Webhook)
```

### The 7 Core Architectural Guardrails:
1. **AI Proposes, Server Validates, Merchant Authorizes, PayPal Processes, Ledger Verifies:** The LLM is an untrusted natural language parser; it cannot directly transition transaction states.
2. **Zero Secret Exposure:** `PAYPAL_CLIENT_SECRET`, `SARVAM_API_KEY`, and `GROQ_API_KEY` reside exclusively in the backend runtime and never reach the browser or client bundles.
3. **Deterministic FX Conversion:** The LLM is never permitted to perform currency calculations. Spoken INR values are converted into USD by the server using an explicit reference rate locked with a 10-minute TTL.
4. **Mandatory Merchant Authorization:** Financial mutations (`stage_invoice_with_qr`) create temporary staged confirmation records. PayPal APIs are invoked only upon explicit merchant confirmation (voice or click).
5. **Fail-Closed PayPal Verification:** Payment records transition to `VERIFIED_PAID` only upon receiving a cryptographically verified PayPal capture callback or successful synchronous capture. Fake or simulated capture IDs are strictly rejected in live environments.
6. **Pure AG Grid Community:** All tabular data operations leverage pure `ag-grid-community` and `ag-grid-react` without unlicensed Enterprise trial artifacts or watermarks.
7. **Clean Production Ledger:** Production instances start with a pristine ledger containing zero mock or seed records; only real, authoritatively created PayPal transactions appear.

---

## 3. Tools & Capabilities Architecture

The table below outlines the core tools and services integrated within VernacPay-Global:

| Tool / Service | Purpose & How It Is Used | Source Reference |
| :--- | :--- | :--- |
| **`stage_invoice_with_qr`** | Agent tool that parses customer details, item description, and spoken INR amount; validates parameters via Zod schemas and locks an illustrative USD reference quote. | [`src/server/agent/orchestrator.ts`](file:///src/server/agent/orchestrator.ts) |
| **`filter_ag_grid_ledger`** | Agent tool that translates vernacular queries (e.g., "show pending orders") into AG Grid filter models sent over SSE to the client. | [`src/server/agent/orchestrator.ts`](file:///src/server/agent/orchestrator.ts), [`src/client/components/views/TransactionsTable.tsx`](file:///src/client/components/views/TransactionsTable.tsx) |
| **`get_verified_summary`** | Agent tool that reconciles paid export transactions from the ledger and generates vernacular spoken earnings summaries. | [`src/server/agent/orchestrator.ts`](file:///src/server/agent/orchestrator.ts) |
| **`list_unpaid_invoices`** | Agent tool that queries the ledger for orders awaiting buyer payment and reports them to the merchant. | [`src/server/agent/orchestrator.ts`](file:///src/server/agent/orchestrator.ts) |
| **PayPal Orders v2 Service** | Handles live PayPal Sandbox OAuth authentication, creates Orders with `intent: 'CAPTURE'`, generates HATEOAS approval links, and executes captures. | [`src/server/services/paypalService.ts`](file:///src/server/services/paypalService.ts) |
| **PayPal Webhook Verification** | Validates incoming PayPal webhooks using cryptographic signature verification (`/v1/notifications/verify-webhook-signature`) and processes `PAYMENT.CAPTURE.COMPLETED`. | [`src/server/services/paypalService.ts`](file:///src/server/services/paypalService.ts), [`src/server/index.ts`](file:///src/server/index.ts) |
| **Sarvam Indic Speech Suite** | Provides native Indic speech recognition via `saaras:v3` and neural speech synthesis via `bulbul:v3` (Telugu `kavitha`, Hindi `aditya`, English `simran`). | [`src/server/services/sarvamService.ts`](file:///src/server/services/sarvamService.ts) |
| **Groq LLM Agent** | Fast LLM inference engine executing tool selection and schema compliance using `openai/gpt-oss-120b`. | [`src/server/agent/orchestrator.ts`](file:///src/server/agent/orchestrator.ts) |
| **Deterministic FX Engine** | Computes locked exchange rates (1 USD = ₹84.00 INR reference) with expiration timestamps, preventing LLM arithmetic errors. | [`src/server/services/fxService.ts`](file:///src/server/services/fxService.ts) |
| **PostgreSQL Ledger Database** | Persistent transactional storage powered by Knex with connection pooling and schema migrations. | [`src/server/db/database.ts`](file:///src/server/db/database.ts) |

---

## 4. What Existed Before October 1, 2026 and What Changed Since

In strict compliance with the **PayPal AI Hackathon 2026 Official Rules** regarding prior work:

* **What Existed Before October 1, 2026:**
  * An early exploratory prototype exploring voice interactions for domestic Indian merchants using local mock payment stubs. It possessed no international payment integrations, no PayPal APIs, no cross-border currency conversion, no cryptographic webhook validation, and basic unverified static tables.
* **Significant Updates Built After October 1, 2026 for the PayPal AI Hackathon:**
  1. **Full Pivot to Global Cross-Border Export Commerce:** Architected the entire solution around vernacular Indian micro-exporters selling internationally in USD via the **PayPal Developer Platform**.
  2. **Production PayPal Orders v2 Integration:** Implemented live PayPal Sandbox authentication, dynamic order generation with HATEOAS checkout URLs, instant scannable QR code generation, and synchronous capture handling via `/api/paypal/return`.
  3. **Fail-Closed Webhook Security:** Implemented full cryptographic signature verification using PayPal's `/v1/notifications/verify-webhook-signature` API, rejecting unsigned or tampered requests with zero backdoors in production.
  4. **Sarvam AI Indic Speech Suite (`saaras:v3` & `bulbul:v3`):** Integrated live speech-to-text and text-to-speech for Telugu, Hindi, and Indian English with honest documentation that Urdu audio is unsupported by the Bulbul voice model.
  5. **Groq LLM Function Calling:** Integrated Groq's high-throughput `openai/gpt-oss-120b` endpoint with a strict 5-stage Trust Boundary intercepting prompt injections and enforcing schema safety.
  6. **AG Grid Community Financial Command Center:** Built an interactive, responsive export ledger with real-time SSE stream updates, multi-column filtering, and CSV export using pure `ag-grid-community`.
  7. **Managed PostgreSQL & Production Hardening:** Migrated storage to PostgreSQL on Render with strict environment startup guards that refuse to run in production if database or critical API keys are unconfigured.

---

## 5. Verification Evidence: Real vs. Simulated

| Feature / Rail | Production Status | Verification Evidence |
| :--- | :--- | :--- |
| **PayPal Orders v2 Creation** | **LIVE REAL** | Real HTTP 201 calls to `https://api-m.sandbox.paypal.com/v2/checkout/orders` returning live `token` and HATEOAS approval links. |
| **PayPal Buyer Checkout & QR** | **LIVE REAL** | Dynamic Base64 QR code encoding the live PayPal Sandbox checkout URL; allows real sandbox buyer authentication and payment. |
| **PayPal Payment Capture** | **LIVE REAL** | Authoritative capture executed through `/api/paypal/return` or `PAYMENT.CAPTURE.COMPLETED` webhook; records genuine PayPal capture IDs. |
| **Fail-Closed Capture Guard** | **LIVE REAL** | Unapproved capture attempts return PayPal HTTP 422 `ORDER_NOT_APPROVED`; missing signatures return HTTP 400. |
| **Sarvam Indic Speech-to-Text** | **LIVE REAL** | Real `saaras:v3` speech recognition for Telugu, Hindi, and Indian English with language detection. |
| **Sarvam Indic Text-to-Speech** | **LIVE REAL** | Real `bulbul:v3` audio synthesis for Telugu (`kavitha`), Hindi (`aditya`), and English (`simran`). |
| **Urdu Audio Support** | **UNSUPPORTED** | Transparently labeled text-only intent; Sarvam Bulbul does not support Urdu audio synthesis. |
| **Groq Agentic Function Calling** | **LIVE REAL** | Live `openai/gpt-oss-120b` inference via Groq API producing Zod-validated tool calls. |
| **Prompt Injection Interception** | **LIVE REAL** | Adversarial prompts attempting to bypass merchant authorization or forge captures are halted at Stage 1. |
| **Database Persistence** | **LIVE REAL** | Authoritative PostgreSQL database on Render (`vernacpay-db`); transactions persist across service restarts. |
| **AG Grid Integration** | **LIVE REAL** | Pure AG Grid Community v32; sortable, filterable, CSV exportable with zero unlicensed watermarks. |

---

## 6. Hackathon Judge & Reviewer Testing Guide

To test the end-to-end checkout flow on the live deployment:

### Test Credentials
* **Live Web App:** [https://vernacpay-global-app.onrender.com](https://vernacpay-global-app.onrender.com)
* **PayPal Sandbox Buyer Email:** Configured via `PAYPAL_SANDBOX_BUYER_EMAIL` (default: `sb-pj4347w53813178@personal.example.com`)
* **PayPal Sandbox Buyer Password:** Provided securely in the **Devpost Testing Instructions** (never committed to public source control).

### Step-by-Step Verification Flow
1. **Open the Live Application:** Navigate to [https://vernacpay-global-app.onrender.com](https://vernacpay-global-app.onrender.com).
2. **Trigger an Export Order:**
   * Click the **`🎤 [Telugu] Order for Sarah ₹12,000 (Pochampally Saree)`** quick-action chip (or speak via microphone in Telugu/Hindi).
   * Observe the bilingual subtitle strip displaying the native script and verified English translation.
   * Observe the **Stage 2 Confirmation Card** displaying the locked FX rate ($142.86 USD @ 1 USD = ₹84.00 INR).
3. **Authorize the Order:**
   * Click **`[Approve & Send via PayPal Sandbox]`** (or speak *"Avunu"* / *"Yes"*).
   * The app calls the PayPal Sandbox Orders v2 API and displays a scannable **PayPal Checkout QR Code** alongside a direct checkout link.
4. **Complete Buyer Payment:**
   * Open the checkout link or scan the QR code.
   * Log in using the US sandbox personal buyer account credentials from the Devpost instructions.
   * Click **Complete Purchase** on PayPal Sandbox.
5. **Observe Verified Settlement:**
   * PayPal redirects back to the return handler (`/api/paypal/return`), which synchronously executes the capture.
   * The transaction status updates to **`VERIFIED_PAID`** in the AG Grid Command Center with spoken vernacular confirmation.

---

## 7. Local Development & Setup

### Prerequisites
* Node.js 20+
* npm 10+
* Optional: Local PostgreSQL instance (defaults to persistent in-memory SQLite if `DATABASE_URL` is omitted in development mode)

### 1. Clone the Repository
```bash
git clone https://github.com/ayeshasultana78/VernacPay.git
cd VernacPay
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure Environment Variables
Copy `.env.example` to `.env` and populate your sandbox credentials:
```bash
cp .env.example .env
```

Key environment variables:
* `PAYPAL_CLIENT_ID` / `PAYPAL_CLIENT_SECRET`: PayPal Developer Sandbox app credentials.
* `PAYPAL_WEBHOOK_ID`: Webhook ID registered in PayPal Developer Dashboard.
* `SARVAM_API_KEY`: API key from Sarvam AI.
* `GROQ_API_KEY`: API key from Groq Console.
* `DATABASE_URL`: PostgreSQL connection string (required in production).
* `DEMO_MODE`: Set to `true` only for offline/local simulation; set to `false` for live PayPal Sandbox mode.

### 4. Run Test Suites
```bash
# Run unit and security hardening tests:
npm run test:unit

# Run live PayPal integration tests:
npm run test:live
```

### 5. Start Development Server
```bash
npm run dev
```
Open **`http://localhost:3000`** in your browser.

---

## 8. Deployment on Render

This repository includes a declarative `render.yaml` Blueprint qualifying for the **Best Use of Render ($1,000 Credits)** prize track.

### Blueprint Architecture:
* **Web Service (`vernacpay-global-app`):** Express application serving Vite-built static client assets and API routes on Node.js 20.
* **Managed Database (`vernacpay-db`):** Render PostgreSQL instance with automatic connection string wiring via `DATABASE_URL`.

To deploy:
1. Connect this repository to your [Render Dashboard](https://dashboard.render.com).
2. Select **New Blueprint Instance**.
3. Fill in the required environment variables (`PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, `PAYPAL_WEBHOOK_ID`, `SARVAM_API_KEY`, `GROQ_API_KEY`).
4. Click **Apply**.

---

## 9. License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
