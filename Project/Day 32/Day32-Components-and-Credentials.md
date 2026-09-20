# Day 32 — Solution Architecture
## Project: Company Knowledge AI Assistant

**Design status:** Proposed capstone architecture. Day 24's working RAG prototype is the starting point; validation, structured routing, escalation, audit logging, and dedicated error handling are planned components and should not be represented as already implemented.

## 1. Main user-question workflow

| Stage | n8n component / node group | Purpose |
|---|---|---|
| 1. Intake | When Chat Message Received (Chat Trigger) | Receive a question and session identifier. |
| 2. Validation | IF + Edit Fields (or Code) | Reject blank questions; normalize question/session ID; avoid logging personal information unnecessarily. |
| 3. Agent / generation | AI Agent + Google Gemini Chat Model | Interpret question, select the retrieval tool, draft an answer strictly grounded in evidence. |
| 4. Retrieval | Supabase Vector Store, **Retrieve Documents as Tool** | Search the company knowledge base, up to four matching documents. |
| 5. Query embeddings | Google Gemini Embeddings 001 | Embed each query in the same embedding space used for ingestion. |
| 6. Context | Simple Memory | Support follow-up questions within the chat session; not an authoritative policy source. |
| 7. Output validation and routing | Structured Output Parser (if supported) + IF/Switch | Differentiate `answered`, `not_found`, and `needs_human`; check answer/source/fallback before routing. A simpler IF after a controlled answer is acceptable initially. |
| 8. Response | Chat response / Respond to Chat (if needed in n8n version) | Return grounded answer with readable source, clarification request, or exact fallback. |
| 9. Escalation (human-approved) | Gmail notification or Google Sheets escalation queue | On unresolved or sensitive requests, prepare an escalation for an approved HR/admin recipient; require project/team-lead approval before sending real company data. |
| 10. Optional audit | Supabase table or Google Sheets append | Store minimal timestamp, outcome, and non-sensitive diagnostics where permitted. |

**Fallback message:** `Information not available in the knowledge base.` For ambiguous questions, ask a clarifying question rather than guessing.

## 2. Knowledge-ingestion workflow

| Stage | n8n component / node group | Purpose |
|---|---|---|
| Source | Google Sheets (read rows) | Import at least 20 illustrative knowledge entries (ID, Topic, Question, Answer, Source). Replace unapproved samples with approved company material before production use. |
| Preparation | Edit Fields / Code (optional) | Clean formatting, preserve source and topic; avoid secrets and confidential data. |
| Loader | Default Data Loader | Convert source rows into documents and retain source information in content/metadata. |
| Embedding | Google Gemini Embeddings 001 | Generate vectors for source documents. |
| Storage | Supabase Vector Store, **Insert Documents** | Save records into `company_documents`. |
| Verification | Supabase Table Editor / SQL count | Confirm at least 20 populated records and the correct embedding dimensions. |

**Database:** Supabase Postgres + pgvector; vector table `company_documents` with `id`, `content`, `metadata`, and `embedding`. Search uses the configured RPC `match_documents` against `company_documents` and a retrieval limit of 4. Reuse the existing working embedding dimension (3072 in the prototype) rather than changing dimensions without verifying the model's output.

## 3. Integrations, models, data stores

| Item | Role |
|---|---|
| n8n | Workflow orchestration, trigger, agent, branching, execution history, and error workflow. |
| Google Sheets | Knowledge-source file; optionally an escalation or error log. |
| Supabase PostgreSQL + pgvector | Vector storage and similarity search; optional minimal operational log. |
| `match_documents` PostgreSQL function | Vector retrieval for `company_documents`. |
| Google Gemini Embeddings 001 | Index-time and question-time embeddings; must match. |
| Google Gemini Chat Model | Agent reasoning and answer generation constrained by retrieved context. |
| Simple Memory | Per-session conversational context; never treated as policy evidence. |
| Gmail (optional, approved) | Human-escalation and failure-alert email. |

**MCP and arbitrary external API tools are not required** for this option. Avoid extra Day 23 tools unless there is a specific approved business need.

## 4. Credentials and access list (names only — never secret values)

| Credential | Used by | Access principle |
|---|---|---|
| Google Gemini API credential | Chat Model and both Embeddings nodes | Store in n8n Credentials; monitor rate limits. |
| Supabase credential | Both Vector Store nodes / logging, if enabled | Minimum necessary role/permissions; do not expose service-role keys in client-facing tools or documentation. |
| Google Sheets OAuth credential | Source ingestion and optional approved logs | Limit access to the designated spreadsheet. |
| Gmail OAuth credential (only if escalation/alerts used) | HR/admin notification and error workflow | Approved recipient and least-privilege permissions. |

No actual API keys, access tokens, passwords, private employee records, or real addresses belong in exported JSON, images, or this document. Review screenshots and n8n exported JSON for embedded secrets before upload.

## 5. Failure handling and reliability

- **Invalid input:** Validate blank/malformed questions before the AI Agent; return a clear clarification message.
- **Unknown answer / weak evidence:** Do not infer policies from nearest-neighbor results. Return the exact fallback; queue for human review only if approved.
- **Gemini 429 / timeouts:** Enable bounded retry and backoff on supported nodes; avoid concurrent bulk ingestion; do not retry indefinitely. Clearly communicate temporary unavailability.
- **Database/RPC failure:** Stop unsupported answers; surface a safe service-error response and log minimal diagnostics.
- **Error workflow:** Link the main workflow's Error Workflow setting to a separate `Error Trigger → Prepare Error → approved log → admin alert` workflow. Check trigger behavior using a full eligible workflow execution, not just a node test.
- **Logging:** Execution ID, time, node and sanitized error; avoid raw questions and full private records unless explicitly approved.
- **Testing:** Successful sourced answer, fallback, follow-up memory, blank question, API failure and admin alert (if implemented).

## 6. Security and governance

- Obtain Team Lead approval for project selection and for treating any company policy records as official. The current 20-entry dataset is illustrative and **not verified as real Petalnex policy**.
- Secure chat access; use authentication/role-based access for a production deployment.
- Keep credentials in n8n's credential store; restrict Supabase and Google integrations to minimum permissions.
- Prevent leaking retrieved confidential information across users; isolate sessions and restrict documents by access level if needed.
- Make citation/source names human-readable; distinguish policy evidence from model-generated prose.
- Require explicit authorization before emailing or escalating sensitive questions; use only approved destinations.
- Define retention for chat logs, execution data and escalations; do not store unnecessary personal data.

## 7. Implementation order (fastest path)

1. Reuse Day 24 ingestion, `company_documents`, Gemini embedding model and working agent.
2. Add input validation and consistent fallback/source formatting.
3. Add a simple structured outcome or reliable routing for unanswered questions.
4. Add approved human escalation and optional minimal logging.
5. Link dedicated error workflow and test an intentional failure.
6. Record real screenshots and demo evidence from actual executions.

**Day 32 deliverables:** architecture diagram + this component and credentials list. Diagram shows intended complete architecture; items marked planned must be implemented/tested in subsequent capstone days.
