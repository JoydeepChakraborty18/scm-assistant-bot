# Supply Chain Audit Agent (RAG Assessment)

## 1. Project Overview
This project implements a Retrieval-Augmented Generation (RAG) agent designed to audit supply chain governance policies and supplier performance data. The agent leverages Google Gemini for reasoning and Pinecone for vector retrieval.

## 2. Architecture
* **Vector Database:** Pinecone (Index: `supplier-gemini-index`)
* **Orchestrator:** Flowise (Conversational Retrieval QA Chain)
* **LLM Model:** Gemini 3.1 Flash Lite Preview
* **Embedding Model:** Google Gemini Embedding (Task Type: `RETRIEVAL_DOCUMENT`)
* **Record Manager:** PostgreSQL (Neon)

## 3. Configuration & Tuning
To optimize retrieval accuracy for structured CSV data, I experimented with three chunk configurations:

| Configuration | Chunk Size | Chunk Overlap | Total Records Indexed |
| :--- | :--- | :--- | :--- |
| **Config A** | 500 | 50 | 135 |
| **Config B** | 1500 | 300 | 111 |
| **Config C** | 1000 | 500 | 134 |

* **Tuning Note:** I increased the `Top K` parameter to 50 to ensure the agent retrieves enough context rows to perform numerical aggregations (like PO value summing) across the CSV dataset.

## 4. Evaluation Results
The agent was validated against the five provided sample questions.

| Question | Agent Response (Verbatim) |
| :--- | :--- |
| **Q1: Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?** | Based on the provided supplier performance data and the Disruption Response Procedures (Section 9), the following Tier-3 suppliers have active disruption flags: SUP-096 (Lima Polymer SA): Flagged for "Audit overdue; compliance review". As this is a High Risk supplier, a Level 3 — Activate response applies. SUP-095 (Cerromax Mineria): Flagged for "Audit overdue; compliance review". As this is a High Risk supplier, a Level 3 — Activate response applies. SUP-096 (Lima Polymer SA): Flagged for "Port closure risk". As this is a High Risk supplier, a Level 3 — Activate response applies. SUP-090 (Plataforma Metales SA): Flagged for "Export license risk". This category (export control restrictions) requires a mandatory Level 3 — Activate response regardless of risk level. SUP-030 (Bohai Electronics): Flagged for "Regulatory enforcement pending". This category (regulatory enforcement actions) requires a mandatory Level 3 — Activate response regardless of risk level. SUP-031 (Varna Electronics EAD): Flagged for "Port congestion risk". As this is a Medium Risk supplier, a Level 2 — Manage response applies. SUP-036 (Maghreb Castworks): Flagged for "Regulatory enforcement pending". This category (regulatory enforcement actions) requires a mandatory Level 3 — Activate response regardless of risk level. SUP-038 (Vistula Pack Sp): Flagged for "Regulatory enforcement pending". This category (regulatory enforcement actions) requires a mandatory Level 3 — Activate response regardless of risk level. SUP-007 (Longhua Polymer Works): Flagged for "Seasonal flood risk". As this is a Medium Risk supplier, a Level 2 — Manage response applies. |
| **Q2: Which suppliers qualify for the annual Volume Rebate Program and how many are there?** | Only one supplier qualifies for the annual Volume Rebate Program: Crestline Chemical Supply (SUP-102): Contract Tier: Tier-1, OTD Rate: 93.8% (at or above 93.0%), Defect Rate: 0.39% (below 0.5%), Sustainability Score: 92 (at or above 85). There is a total of 1 supplier that qualifies. |
| **Q3: Which region has the highest total PO value, and does it breach the concentration limit?** | To determine the region with the highest total PO value, I have aggregated the PO_Value_USD from all provided records by region: NA: $3,306,979.75; EMEA: $5,017,210.91; APAC: $1,345,432.64; LATAM: $1,942,703.10. Total Spend: $11,612,326.40. EMEA has the highest total PO value at $5,017,210.91. EMEA's share is 43.21%. Since 43.21% is less than the 45% threshold, the concentration limit is not breached. |
| **Q4: Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?** | According to Section 3.4 of the governance policy, a supplier is placed on Supplier Watch List (SWL) status if they have a Compliance Score below 60. Additionally, per Section 7.1, any supplier classified as Audit Overdue is placed on provisional SWL status. Suppliers identified: SUP-090 (Plataforma Metales SA), SUP-096 (Lima Polymer SA), and SUP-095 (Cerromax Mineria). SWL status limits new Purchase Order (PO) issuance to 20% of the prior quarter’s volume. |
| **Q5: Which product category has the highest average defect rate and does it exceed the Tier-2 limit?** | Packaging Materials category has the highest average defect rate at 2.51%. The maximum permissible defect rate for a Tier-2 supplier is 2.50%. The Packaging Materials category average of 2.51% exceeds the Tier-2 limit. |

## 5. Live Chatbot URL
https://cloud.flowiseai.com/chatbot/8d6a655b-7544-4d94-a6a1-6c3028afc6cb

## 6. Reflections & Improvements
* **Challenges:**
    * **Data Ingestion:** Encountered a discrepancy between the total chunks produced by the document loader and the final record count in Pinecone. This was resolved by adjusting the chunking parameters and ensuring batch processing stability.
    * **Rate Limiting:** Managed intermittent `429 Too Many Requests` errors from the Gemini API by implementing appropriate request pacing.
    * **Database Integration:** Successfully implemented a PostgreSQL-based Record Manager using Neon to manage document versioning and prevent duplicate record creation across multiple upsert attempts.
* **Future Improvements:** To improve accuracy on complex numerical tasks (like regional spend aggregation), I would look into implementing a specialized SQL-based agent or a Pandas-querying tool. While text-based RAG is excellent for governance policy retrieval, structured numerical analysis is more reliably handled by agents capable of executing code or formal queries.
