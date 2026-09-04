# HR Policy RAG Assistant

## 1. Project Name

**HR Policy RAG Assistant**

## 2. Problem Statement

Employees often need quick answers to common company and HR policy
questions. Manually searching policy documents can be slow, while a
normal AI model may invent information that is not part of the company's
actual policies.

This project implements a Retrieval-Augmented Generation (RAG) assistant
in n8n. The assistant searches an HR policy knowledge base stored as
embeddings in Supabase and uses the retrieved information to answer
employee questions. If the requested information is not available in the
knowledge base, the assistant returns a clear fallback instead of
inventing a policy.

## 3. Objective

The objective is to build an HR Policy Assistant that:

-   Accepts employee questions through a Webhook.
-   Searches an HR policy vector database.
-   Retrieves the most relevant policy information.
-   Uses an AI model to generate a grounded answer.
-   Answers only from the supplied HR knowledge base.
-   Does not invent missing policies.
-   Returns a clear fallback when information is unavailable.
-   Mentions the source/reference whenever available.

## 4. Workflow Architecture

The main Day 21 workflow is:

**Postman / User Question → Webhook → AI Agent → Final Answer**

The AI Agent uses:

-   **Google Gemini Chat Model** as the language model.
-   **Supabase Vector Store** as a retrieval tool.
-   **Google Gemini Embeddings 001** for vector similarity search.

Conceptually:

``` text
Webhook
   |
   v
AI Agent
   |-- Google Gemini Chat Model
   |
   `-- Supabase Vector Store (Tool)
          |
          `-- Google Gemini Embeddings 001
```

The HR policy documents were ingested separately into the `hr_documents`
Supabase vector table before running the RAG assistant.

## 5. Technologies Used

-   n8n
-   Supabase
-   PostgreSQL / pgvector
-   Google Gemini Chat Model
-   Google Gemini Embeddings 001
-   AI Agent
-   Supabase Vector Store
-   Webhook
-   Postman
-   Google Sheets for the original HR policy dataset

## 6. Knowledge Base

The HR knowledge base contains information covering:

-   Leave Policy
-   Working Hours
-   Attendance
-   Internship Guidelines
-   Code of Conduct

Example knowledge entry:

``` text
Topic: Leave
Question: How many casual leaves are allowed?
Answer: Employees are allowed 12 casual leaves per year.
Source: Leave Policy
```

The source field is stored with the knowledge entry so that the
assistant can mention the relevant policy when answering.

## 7. HR Policy Vector Database

A separate Supabase vector table named:

``` text
hr_documents
```

is used for the HR knowledge base.

The table stores information such as:

-   `id`
-   `content`
-   `metadata`
-   `embedding`

The HR documents were converted into embeddings using **Google Gemini
Embeddings 001** and inserted into the vector store.

Keeping HR documents in a separate table prevents them from being mixed
with other knowledge bases such as customer-support FAQs.

## 8. Vector Search Function

A Supabase RPC function named `match_documents` is used by n8n to
perform similarity search against the HR vector table.

The function accepts:

-   `query_embedding`
-   `match_count`
-   `filter`

It searches the `hr_documents` table and returns the most similar
documents.

This function was required so the Supabase Vector Store could retrieve
HR policy documents successfully.

## 9. Webhook Input

The workflow accepts questions through an n8n Webhook.

Example POST request:

``` json
{
  "question": "How many casual leaves are allowed?"
}
```

The Webhook passes the employee question to the AI Agent.

## 10. Supabase Vector Store Configuration

The Supabase Vector Store is configured for retrieval rather than
document insertion.

**Operation Mode:**

``` text
Retrieve Documents (as Tool for AI Agent)
```

**Table:**

``` text
hr_documents
```

The **Google Gemini Embeddings 001** node is connected to the Vector
Store so that the employee question can be converted into an embedding
and compared with the stored HR policy vectors.

## 11. AI Agent Instructions

The AI Agent is instructed to answer strictly from the HR policy
knowledge base.

A prompt/instruction similar to the following is used:

``` text
You are an HR Policy Assistant.

Answer employee questions using ONLY information retrieved from the HR policy knowledge base.

Rules:
1. Always search the HR policy vector store before answering.
2. Do not invent, assume, or guess company policies.
3. Do not use general knowledge to fill missing information.
4. If the requested information cannot be found in the retrieved documents, respond:
   "Information not available in the knowledge base."
5. Keep answers clear and concise.
6. Mention the source policy whenever the source is available.

Employee Question:
{{ $json.body.question }}
```

## 12. Grounded Answer Requirement

The assistant must only answer from retrieved HR documents.

For example:

**Question:**

``` text
How many casual leaves are allowed?
```

**Expected answer:**

``` text
Employees are allowed 12 casual leaves per year.
Source: Leave Policy
```

The assistant should not add additional leave rules unless they exist in
the knowledge base.

## 13. Safe Fallback

A key requirement of the project is preventing hallucinated company
policies.

For a question whose answer does not exist in the knowledge base, such
as:

``` text
What is the company's maternity leave policy?
```

the assistant should return:

``` text
Information not available in the knowledge base.
```

It must not create or assume a maternity leave policy.

## 14. Test Cases

At least five question-and-answer tests are included in the submission.

  -----------------------------------------------------------------------
  \#                      Question                Expected Result
  ----------------------- ----------------------- -----------------------
  1                       How many casual leaves  12 casual leaves per
                          are allowed?            year; source: Leave
                                                  Policy

  2                       What are the working    9:00 AM to 5:00 PM,
                          hours?                  Monday to Friday;
                                                  source: Working Hours
                                                  Policy

  3                       What is the attendance  At least 90% attendance
                          requirement?            each month; source:
                                                  Attendance Policy

  4                       How long is the         Standard internship
                          internship?             duration is 8 weeks;
                                                  source: Internship
                                                  Guidelines

  5                       What is the company's   Information not
                          maternity leave policy? available in the
                                                  knowledge base.
  -----------------------------------------------------------------------

The actual AI outputs are demonstrated in the submitted screenshots.

## 15. Example Q&A

### Example 1

**Question:**

``` text
How many casual leaves are allowed?
```

**Expected response:**

``` text
Employees are allowed 12 casual leaves per year.
Source: Leave Policy
```

### Example 2

**Question:**

``` text
What are the working hours?
```

**Expected response:**

``` text
Standard working hours are 9:00 AM to 5:00 PM, Monday to Friday.
Source: Working Hours Policy
```

### Example 3

**Question:**

``` text
What is the attendance requirement?
```

**Expected response:**

``` text
Employees are expected to maintain at least 90% attendance each month.
Source: Attendance Policy
```

### Example 4

**Question:**

``` text
How long is the internship?
```

**Expected response:**

``` text
The standard internship duration is 8 weeks.
Source: Internship Guidelines
```

### Example 5 - Fallback

**Question:**

``` text
What is the company's maternity leave policy?
```

**Expected response:**

``` text
Information not available in the knowledge base.
```

## 16. Error Handling

### Missing Vector Search Function

During development, Supabase returned a `PGRST202` error because the
`match_documents` function required by the vector search was not
available.

The issue was resolved by creating the required `match_documents` RPC
function in Supabase.

### Unknown Policy

If relevant information cannot be retrieved, the AI Agent is instructed
to return the defined fallback rather than inventing an answer.

### AI Provider Failure

If the Gemini service is temporarily unavailable, the workflow may fail
to produce an answer. Production implementations should include retry
and error-handling logic.

### Empty Question

A production version should validate that the Webhook contains a
non-empty `question` field before sending the request to the AI Agent.

## 17. Credentials Required

Credential names only:

-   Google Gemini API credential
-   Supabase credential

No passwords, API keys, access tokens, or other secret values are
included in this README or in screenshots.

## 18. Setup Instructions

1.  Create or open the Supabase project.
2.  Enable the vector/pgvector extension if required.
3.  Create the `hr_documents` vector table.
4.  Ingest the HR policy documents into `hr_documents`.
5.  Generate embeddings using Google Gemini Embeddings 001.
6.  Confirm that HR vectors appear in Supabase.
7.  Create the required `match_documents` vector-search function.
8.  Create a new n8n workflow for the HR Policy RAG Assistant.
9.  Add a POST Webhook for receiving employee questions.
10. Add an AI Agent.
11. Connect the Google Gemini Chat Model to the AI Agent.
12. Add the Supabase Vector Store as an AI Agent tool.
13. Set the Vector Store to retrieve documents as a tool for the AI
    Agent.
14. Select the `hr_documents` table.
15. Connect Google Gemini Embeddings 001 to the Vector Store.
16. Add the grounded-answer and fallback instructions to the AI Agent.
17. Test questions through Postman.
18. Verify that known questions return policy-based answers with
    sources.
19. Verify that an unknown policy question returns the fallback
    response.

## 19. Screenshots Included

The submission should contain at least five Q&A screenshots:

1.  Casual leave question and answer.
2.  Working hours question and answer.
3.  Attendance question and answer.
4.  Internship duration question and answer.
5.  Unknown/maternity leave question showing the fallback.

An additional screenshot of the complete n8n workflow may also be
included for clarity.

## 20. Known Limitations

-   The assistant can only answer questions covered by the stored HR
    policy documents.
-   Retrieval quality depends on the quality and wording of the
    knowledge-base entries.
-   The AI model can still produce unexpected wording, so strict
    instructions are important.
-   The current knowledge base is a small demonstration dataset.
-   Changes to HR policies require the vector knowledge base to be
    updated/re-ingested.
-   The workflow depends on Supabase and Gemini availability.

## 21. Future Improvements

-   Add more complete HR policy documents.
-   Add metadata filtering by policy type or department.
-   Add employee authentication before allowing policy queries.
-   Add conversation memory for follow-up questions.
-   Add automatic re-ingestion when HR documents change.
-   Add logging of unanswered questions to identify missing policies.
-   Add stronger output validation.
-   Add monitoring and retry handling.
-   Add a user-facing chat interface instead of relying only on Postman.
-   Return direct document links or policy IDs alongside source names.

## 22. Security Notes

-   API keys and credentials must never be hard-coded.
-   Credentials should be stored using n8n's credential system.
-   Secret values should not appear in screenshots.
-   HR documents should only contain information appropriate for
    employee access.
-   Production webhooks should use authentication or another
    access-control mechanism.

## 23. Conclusion

The HR Policy RAG Assistant demonstrates Retrieval-Augmented Generation
using n8n, Supabase Vector Store, Gemini embeddings, and a
Gemini-powered AI Agent.

Instead of relying on the language model's general knowledge, the
workflow retrieves relevant information from the company's HR policy
knowledge base. This produces more grounded answers, provides policy
sources where possible, and gives a safe fallback when requested
information is not available.
