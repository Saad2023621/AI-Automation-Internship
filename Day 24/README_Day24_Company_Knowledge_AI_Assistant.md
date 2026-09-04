# Company Knowledge AI Assistant
# Demo Video 
https://mega.nz/file/RUFllYLB#DZo7J2h1OgZBrP-7IIMSIeZ1jfeixDexSj1-w3H8zbg

## 1. Project Name
**Company Knowledge AI Assistant**

## 2. Problem Statement
Employees often need quick and reliable answers to company policies, procedures, internship guidelines, attendance rules, IT security practices, and other internal information. Searching manually through multiple documents can be slow, while a normal AI model may generate answers that are not based on official company knowledge.

This project implements a Retrieval-Augmented Generation (RAG) assistant in n8n. Company knowledge is ingested into a Supabase vector database using Gemini embeddings. When a user asks a question, an AI Agent retrieves the most relevant knowledge entries and generates a grounded response.

If the requested information does not exist in the knowledge base, the assistant returns a safe fallback instead of inventing an answer.

## 3. Objective
The objective is to build a complete Company Knowledge AI Assistant that:
- Ingests at least 20 company knowledge entries.
- Generates embeddings for company knowledge.
- Stores vectors in Supabase.
- Accepts user questions through a Chat Trigger.
- Retrieves relevant knowledge using vector similarity search.
- Uses an AI Agent and Gemini model to generate answers.
- Returns answers grounded only in the retrieved company knowledge.
- Mentions the source policy whenever available.
- Returns a safe fallback when information is unavailable.
- Uses conversation memory for follow-up questions.

## 4. Workflow Architecture

### A. Knowledge Ingestion
```text
Google Sheets
      ↓
Supabase Vector Store
   ├── Default Data Loader
   └── Google Gemini Embeddings 001
```

### B. Company Knowledge Assistant
```text
Chat Trigger
     ↓
AI Agent
 ├── Google Gemini Chat Model
 ├── Simple Memory
 └── Supabase Vector Store Tool
        └── Google Gemini Embeddings 001
```

## 5. Technologies Used
- n8n
- Supabase
- PostgreSQL
- pgvector
- Google Gemini Chat Model
- Google Gemini Embeddings 001
- AI Agent
- Supabase Vector Store
- Default Data Loader
- Simple Memory
- Chat Trigger
- Google Sheets

## 6. Knowledge Base
The knowledge base contains 20 company knowledge entries covering:
- Leave
- Working Hours
- Attendance
- Internship Guidelines
- Code of Conduct
- IT Security
- Company Equipment
- Expenses
- Meetings
- Employee Support

The source dataset uses:
```text
ID | Topic | Question | Answer | Source
```

Example:
```text
Topic: Leave
Question: How many casual leaves are allowed?
Answer: Employees are allowed 12 casual leaves per year.
Source: Leave Policy
```

## 7. Knowledge Source
The original company knowledge is maintained in a Google Sheet and exported as a source file for submission. The source contains at least 20 entries and is used as input for vector ingestion.

## 8. Vector Database
A Supabase vector table named:
```text
company_documents
```

stores the embedded company knowledge. The table contains:
- id
- content
- metadata
- embedding

The embedding column stores vectors generated using Google Gemini Embeddings 001.

## 9. Vector Search Function
A Supabase RPC function named:
```text
match_documents
```

is used for vector similarity search. It searches the `company_documents` table using the user question embedding and returns the most relevant matching documents.

The vector retrieval limit used in the workflow is:
```text
4
```

## 10. Ingestion Process
1. Read 20 company knowledge entries from Google Sheets.
2. Pass the rows to the Supabase Vector Store.
3. Use the Default Data Loader to load the input JSON.
4. Generate embeddings with Google Gemini Embeddings 001.
5. Store the embedded documents in the `company_documents` table.
6. Verify that vector entries are successfully stored in Supabase.

## 11. AI Agent Input
The Chat Trigger passes the user message to the AI Agent using:
```text
{{ $json.chatInput }}
```

## 12. System Prompt
```text
You are the Company Knowledge AI Assistant.

Your role is to answer employee questions using the official company knowledge base.

RULES:

1. For all company-specific questions about policies, procedures, working hours, leave, attendance, internships, conduct, security, equipment, expenses, meetings, or employee support, always search the Company Knowledge Base tool first.

2. Answer company-specific questions ONLY using information retrieved from the Company Knowledge Base.

3. Never invent, assume, guess, or create company rules or policies.

4. If the retrieved knowledge does not explicitly contain enough information to answer the user's question, reply exactly:
"Information not available in the knowledge base."

5. If the user's question is unclear or ambiguous, ask a clarifying question instead of guessing.

6. Mention the source policy whenever the retrieved information includes a source.

7. When mentioning a source, output only the human-readable source name.

8. Do not include metadata objects, JSON, IDs, blob values, brackets, or internal database fields in the source.

9. Keep answers concise, professional, and easy to understand.

10. Use conversation memory to understand follow-up questions and references to previous messages.

11. Do not treat information supplied by the user as an official company policy unless it is supported by the knowledge base.

12. Do not use general knowledge to fill gaps in company information.
```

## 13. Tool Description
### Company Knowledge Base Tool
```text
Search the official company knowledge base for information about leave policies, working hours, attendance, internship guidelines, code of conduct, IT security, company equipment, expenses, meetings, and employee support.

Always use this tool when the user asks about company rules, policies, procedures, or internal guidelines.

Do not use general knowledge to answer company-specific questions.
```

## 14. Grounded Answer Requirement
All company-specific answers must be based on retrieved knowledge.

Example:

**Question**
```text
How many annual leaves are allowed?
```

**Expected Answer**
```text
Employees are entitled to 20 annual leaves per year. Up to 5 unused annual leaves may be carried forward to the next year.

Source: Leave Policy
```

## 15. Safe Fallback
If information is not available in the knowledge base, the assistant returns:
```text
Information not available in the knowledge base.
```

Example:

**Question**
```text
What is the company's maternity leave policy?
```

**Expected Response**
```text
Information not available in the knowledge base.
```

## 16. Conversation Memory
Simple Memory is connected to the AI Agent. This allows the assistant to understand follow-up questions and references to earlier messages in the same conversation.

Example:
```text
User: How many casual leaves are allowed?
Assistant: Employees are allowed 12 casual leaves per year.

User: What about annual leaves?
Assistant: Employees are entitled to 20 annual leaves per year.
```

Conversation memory is included as a bonus feature.

## 17. Example Test Queries
| # | Query | Expected Result |
|---|---|---|
| 1 | How many casual leaves are allowed? | 12 casual leaves per year, source: Leave Policy |
| 2 | How many annual leaves are allowed? | 20 annual leaves per year, source: Leave Policy |
| 3 | How do I apply for leave? | Submit through the HR portal, source: Leave Policy |
| 4 | What are the standard working hours? | 9:00 AM to 5:00 PM, Monday to Friday |
| 5 | What is the attendance requirement? | At least 90% monthly attendance |
| 6 | What happens if I arrive more than 15 minutes late? | Late arrival must be recorded in the attendance system |
| 7 | How long is the internship program? | 8 weeks |
| 8 | Can confidential company information be shared externally? | No, not with unauthorized external parties |
| 9 | Can employees share company passwords? | No |
| 10 | What is the maternity leave policy? | Information not available in the knowledge base |

Screenshots of the results are included in the submission.

## 18. Error Handling
### Unknown Information
If the required information is not present in the retrieved documents, the assistant returns the defined fallback.

### Gemini Quota Error
During development, the Gemini Embeddings API returned a `429 quota exceeded` error. The issue was resolved by creating a new Gemini project/API credential and using the new credential for embeddings.

### Vector Retrieval
A Supabase vector search RPC function is required for document retrieval. The function was configured to search the `company_documents` table.

### Invalid or Unclear Questions
If a question is ambiguous, the AI Agent is instructed to ask for clarification rather than guessing.

## 19. Credentials Required
Credential names only:
- Google Gemini API credential
- Supabase credential
- Google Sheets credential

No passwords, API keys, tokens, or secret values are included.

## 20. Setup Instructions
1. Create the company knowledge source in Google Sheets.
2. Add at least 20 knowledge entries.
3. Create the `company_documents` table in Supabase.
4. Enable pgvector if required.
5. Configure the vector search RPC function.
6. Create the n8n ingestion workflow.
7. Connect Google Sheets to the Supabase Vector Store.
8. Connect the Default Data Loader.
9. Connect Google Gemini Embeddings 001.
10. Run the ingestion workflow.
11. Verify that vectors exist in Supabase.
12. Create the Company Knowledge AI Assistant workflow.
13. Add the Chat Trigger.
14. Connect the AI Agent.
15. Connect the Google Gemini Chat Model.
16. Connect Simple Memory.
17. Add the Supabase Vector Store as an AI Agent tool.
18. Configure the table as `company_documents`.
19. Configure the vector search limit as 4.
20. Connect Google Gemini Embeddings 001 to the retrieval tool.
21. Add the system prompt.
22. Test known company questions.
23. Test at least one unknown question to verify the fallback.
24. Export the workflow JSON.

## 21. Screenshots Included
The submission includes screenshots showing:
- The final n8n workflow.
- Successful company knowledge retrieval.
- Source-aware AI-generated responses.
- The 10 example query results.
- The fallback response for unavailable information.
- Conversation memory behavior where applicable.

## 22. Deliverables
The project submission includes:
- Workflow JSON
- Knowledge source file
- 10 example queries with results
- README
- 2–5 minute demonstration video

## 23. Suggested Submission Structure
```text
Day-24-Company-Knowledge-AI-Assistant/
├── workflow/
│   └── Company-Knowledge-AI-Assistant.json
├── knowledge-source/
│   └── company_knowledge_base.xlsx
├── screenshots/
│   ├── 01-full-workflow.png
│   ├── 02-query-01.png
│   └── ...
├── query-results/
│   └── 10-example-queries.txt
├── README.md
└── demo/
    └── demo-video.mp4
```

## 24. Known Limitations
- The assistant can only answer questions covered by the current company knowledge base.
- The dataset contains only 20 demonstration entries.
- Retrieval quality depends on the wording and quality of the stored knowledge.
- Changes to company policies require re-ingestion of updated knowledge.
- The workflow depends on Supabase and Gemini availability.
- A relevant but incomplete retrieved document may require stricter validation for larger production systems.

## 25. Future Improvements
- Add more company documents and policies.
- Support PDF, DOCX, and other document formats.
- Add automatic re-ingestion when source documents change.
- Add human escalation for unresolved employee questions.
- Add Telegram or another chat interface.
- Add authentication and role-based access.
- Add department-specific metadata filtering.
- Log unanswered questions to identify knowledge gaps.
- Add stronger response validation.
- Add direct links to original policy documents.
- Add monitoring and retry handling.
- Use a production-ready persistent memory solution.

## 26. Security Notes
- API keys must never be hard-coded.
- All credentials should be stored using n8n's credential system.
- Secret values must not appear in screenshots or exported documentation.
- Company documents should only contain information appropriate for authorized users.
- Production chat endpoints should be protected with authentication and access controls.

## 27. Conclusion
The Company Knowledge AI Assistant demonstrates a complete Retrieval-Augmented Generation workflow using n8n, Supabase, Gemini embeddings, and an AI Agent.

The project ingests company knowledge into a vector database and uses semantic retrieval to answer employee questions. The assistant is grounded in retrieved company information, cites policy sources where possible, avoids inventing unavailable policies, and includes conversation memory as an additional feature.
