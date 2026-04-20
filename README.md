This is a Snowflake‑only target architecture designed for healthcare use cases. Snowflake is the system of record for structured data like HCPs, HCOs, affiliations, and claims, so metrics like TRx and referral behavior are accurate and auditable. At the same time, call notes and market intelligence documents from SharePoint are brought into Snowflake so they’re governed alongside the data rather than sitting in silos.
We prepare structured data using SQL, Dynamic Tables, and Snowpark only where needed, and then layer a semantic model on top that defines canonical healthcare entities and certified business metrics. This clearly separates raw data from business meaning, which is critical for SQL accuracy.
Relationships like HCP‑to‑HCO remain in core data, while more interpretive relationships like brand relevance or referral influence are handled in the semantic layer. The SharePoint documents are processed using a Snowflake‑native RAG approach built from Snowpark, Cortex, and vector search, so we can retrieve and summarize relevant content without moving data out of the platform.
Orchestration happens entirely inside Snowflake, combining structured facts with unstructured context, and governance—access control, lineage, data quality, and PII protection—is enforced end‑to‑end by default. The outcome is a trusted, explainable way for healthcare leaders to understand not just what is happening, but why it’s happening.

FLow cHART EXPLANATION 
What happens here:

HCP, HCO, affiliation, and claims data are stored in core Snowflake tables
SharePoint documents are ingested into Snowflake stages and document tables
Everything immediately falls under:

Security
Access control
Auditability



✅ Key design principle:
Snowflake is the single trusted place where data lives.

3️⃣ Structured Data Preparation Layer
(Cleaning and shaping data)

“Structured data is prepared using Snowflake‑native capabilities—SQL for transformations, Dynamic Tables for incremental processing, and Snowpark only where advanced logic is required.”

This layer:

Cleans data
Handles incremental updates
Prepares analytics‑ready datasets

✅ No external ETL tools
✅ Scales automatically
✅ Keeps transformations close to data

4️⃣ Unstructured Content Preparation Layer (RAG)
(Turning documents into searchable knowledge)

“At the same time, unstructured SharePoint documents go through a preparation process.”

Flow:

Text is extracted from Word/PDF/PPT files
Content is broken into chunks
Each chunk is tagged with context:

HCP
HCO
Brand
Geography
Time period


Snowflake Cortex generates embeddings
Embeddings are stored in Snowflake tables

✅ Documents are now:

Searchable by meaning
Governed
Linked to healthcare entities


5️⃣ Semantic Layer
(The meaning layer — the heart of the architecture)

“On top of prepared data, we introduce a semantic layer that defines business meaning.”

This layer contains:

Canonical entities (HCP, HCO, Brand)
Certified metrics (TRx decline, referral behavior, activity thresholds)
Business logic and relationships

Important distinction:

Raw data stays in tables
Business meaning lives in the semantic layer

✅ This ensures SQL accuracy
✅ Everyone uses the same definitions
✅ AI and analytics speak the same language

6️⃣ Relationships – Where They Live
(Explicit architectural decision)

“We intentionally split where relationships live.”


Authoritative relationships
(HCP → HCO affiliations) → Core tables
Interpretive relationships
(HCP → Brand relevance, referral influence) → Semantic layer

✅ Preserves factual integrity
✅ Allows flexible business interpretation

7️⃣ Orchestration & Routing Layer
(How everything works together)

“When a question is asked, Snowflake orchestrates the flow.”

What happens:

Semantic layer determines intent
Structured SQL runs first for factual accuracy
Vector search retrieves relevant documents
Both results are combined

Snowflake Tasks, Streams, and procedures coordinate this flow.
✅ No manual stitching
✅ Deterministic execution
✅ Fully governed

8️⃣ Answering the Simulation Queries
(This proves the design works)
Example 1:
“Show me oncologists and HCO locations whose TRx declined and summarize why.”
Flow:

TRx trends → Structured data via semantic views
Oncologist filter → Semantic entity
High field activity → Certified business rules
“Why” → Call notes & market intel via RAG

✅ Facts + explanation, together

Example 2:
“Which HCPs drive referral behavior in an HCO network?”
Flow:

HCP–HCO relationships → Core data
Network influence → Semantic logic
Ranking & insights → Snowflake SQL

✅ Clean, explainable answer

9️⃣ Consumption Layer
(How users see results)

“Insights are delivered through Snowflake consumption tools—dashboards, analytics, and AI responses.”

Users receive:

Accurate metrics
Human‑readable explanations
Data they can trust

✅ No data movement
✅ No loss of governance

🔟 Governance Layer (Cross‑Cutting)
(Always on)

“Governance is enforced from start to finish.”

Includes:

Role‑based access
PII masking
Lineage
Data quality checks

✅ Applied automatically
✅ No special handling required


This is how unstructured data is processed 
So the first thing we do is bring those SharePoint documents into Snowflake in a controlled way. We don’t query SharePoint directly every time. Instead, we ingest the documents into Snowflake so they are governed, secured, and audited just like our structured data. At that point, Snowflake becomes the place where both data and documents live under the same rules.
Once the documents are in Snowflake, we extract the text from them — for example, from Word or PDF call notes. We then break that text into smaller, meaningful sections so it’s easier to work with. Each of those sections is tagged with business context, like which HCP, HCO, brand, geography, or time period it relates to. That’s what links unstructured documents back to our core healthcare data.
Next, we convert that text into something Snowflake can search intelligently. Using Snowflake’s AI capabilities, we create embeddings, which is just a way of teaching the system the meaning of the text, not just the words. These embeddings stay inside Snowflake — we don’t send them to another system — so governance and security remain intact.
When someone asks a question like ‘Why did TRx decline even though field activity was high?’, the system does two things at the same time. First, it runs accurate SQL on structured data to identify the right HCPs, HCOs, and trends. Second, it searches those processed SharePoint documents to find the call notes or market intelligence that are most relevant to those HCPs and that time period.
Finally, Snowflake combines the facts from the structured data with the context from the documents and summarizes them in plain language. Throughout this entire process, access controls, masking, lineage, and compliance rules are enforced automatically.
So in simple terms, we’ve taken SharePoint documents that used to sit in silos and turned them into governed, searchable business context that explains what’s happening in the data — without losing accuracy, trust, or control.

1️⃣ Source → Data Layer (Document Store)
Diagram layer:
“Data Layer – Snowflake CDM + Document Store”
What you explained:

SharePoint call notes and market intelligence documents are ingested
Documents are brought into Snowflake as governed assets

✅ This matches your diagram’s document storage and staging area
✅ SharePoint is simply the source, Snowflake is the managed store

2️⃣ Text extraction & chunking → Unstructured Content Preparation
Diagram layer:
“Unstructured Content Preparation Layer”
What you explained:

Extract text from Word / PDF / PPT
Break documents into chunks
Attach business metadata (HCP, HCO, brand, geography)

✅ This is exactly what that layer exists for
✅ You didn’t invent anything outside the diagram

3️⃣ Embeddings → Vector Search
Diagram layer:
Same Unstructured Content Preparation layer
What you explained:

Create embeddings using Snowflake Cortex
Store vectors inside Snowflake
Enable semantic search

✅ This is the vector embeddings + context search box in your diagram

4️⃣ Linking documents to HCP/HCO → Semantic Layer
Diagram layer:
“Semantic Layer (Ontology, Entities, Relationships)”
What you explained:

Documents are linked to HCPs, HCOs, brands
Relationships are not hard‑coded in raw tables
Business meaning lives in the semantic layer

✅ This exactly aligns with:

Ontology
Entity models
Business vocabulary


5️⃣ Query execution → Orchestration & Routing
Diagram layer:
“Orchestration / Routing Layer”
What you explained:

SQL runs first for accuracy (TRx, decline, field activity)
Vector search runs in parallel for context
Snowflake orchestrates the flow

✅ That is precisely why Tasks, Streams, and procedures are in your diagram

6️⃣ Explanation generation → Consumption layer
Diagram layer:
“Consumption Layer”
What you explained:

Combine structured facts with unstructured context
Summarize “why” something happened
Present insights to users

✅ This lands exactly where Snowsight / AI responses sit
7️⃣ Governance → Governance layer
Diagram layer:
“Snowflake Native Governance”
What you explained:

Access controls
PII protection
Lineage
Auditability

✅ This is explicitly shown in your diagram as a cross‑cutting layer



I'll describe the goal to you. So we have in Novartis, we have structured data saved in Snowflake, right? 90% of data are structured in Snowflake, saved in CDM. Then we have 10% of data, which are different type of documents, Word documents, PDF documents, maybe notepads, whatever is, maybe images. Those... OK, OK, OK, sorry, I'm talking. OK, I'm staying here. I'm staying. You go to slide. OK, OK, you go. So the 10% is unstructured data saved in SharePoint. Now your goal is when a user is asking questions, why sales dropped 15%, Snowflake not gonna answer the question. Snowflake has no why. It's just numbers, right? Yes. So you need to go and find supporting documents that would explain, for example, sales dropped 15% because the hospital changed policy for this drug, and it explains why component, right? Yes. Follow me? Yes. OK, so to do that, to answer those why questions, you have to query unstructured data. That's the only way to find why. So now we have no RAM. We have no AWS graph. We have nothing like this. So we have only Snowflake. OK. You cannot use anything outside of Snowflake. So your goal is stay with Snowflake. So you have 1,000 documents in SharePoint. How to use Snowflake to find answers for why questions in those documents? So that is your goal. Your consumers are the same. It could be Power BI. It could be a person asking questions. You don't need to put notebooks here. It's completely relevant. You are creating architecture. You're not creating pipelines. So then you need to think architecturally how you would be able, using Snowflake, to query unstructured data. OK. And that's...     


Don't make it very difficult. Don't remove all notebooks. It's completely irrelevant here. You're creating an architectural diagram. And think about how to query unstructured data. So that is your goal, to design architecture using only Snowflake tools. You're not allowed to use anything else, no AWS, nothing. But you still have ways to query unstructured data. So that's the goal for your diagram. And it could be multiple ways. So then show both ways, whatever way Snowflake allows. Okay, I'm with you, my love. I'm with you. Go, go, go. Okay, good job. Okay, do you follow what I'm saying? Yes, yes, yes. I'm gonna look up on that, and I'm gonna work and send it back to you. Yeah, so your data layer stays the same. No changes here. This is constant. It's not a part of the creation. This is given to us. We have CDN, we have SharePoint. Take it from the diagram which I created, and this is your data layer. Your consumer are still the same. It could be Power BI, it could be a person, it could be AWS agent which would come and work with Snowflake, right? So, but the rest is yours. What Snowflake does to handle unstructured data. Okay.





I’ll explain this diagram from bottom to top, focusing on how unstructured data is handled architecturally.

1. Data Layer (Given)
At the bottom of the diagram, we start with the data layer, which is already given and unchanged.

SharePoint contains unstructured content such as PDFs, Word documents, reports, and notes.
Snowflake CDM contains structured enterprise data like sales, TRx, field activity, HCP, and HCO data.

From an architecture perspective, SharePoint remains the system of record for documents. We do not query SharePoint directly at runtime.

2. Unstructured Data Entry into Snowflake
Moving upward into Snowflake, the key architectural decision is that unstructured documents are made queryable inside Snowflake.

Documents from SharePoint are ingested or referenced in Snowflake.
The document content is extracted and stored as document text tables, rather than being treated as files.
Each document (or document chunk) is associated with business entities such as HCP, HCO, brand, disease, and time period.

At this point, unstructured data becomes first‑class data inside Snowflake, governed and searchable like structured data.

3. Unstructured Data Processing (WHY Layer)
On the right side of the Snowflake box, the diagram shows Unstructured Document Data (WHY).
Architecturally, this layer provides:

Document text tables that store extracted text
Search or semantic indexes that allow keyword and semantic lookup across documents

This layer is responsible for answering why questions, such as:

Policy changes
Access restrictions
Competitive or market explanations

No external vector database or external AI platform is involved — everything happens inside Snowflake.

4. Linking Structured and Unstructured Data
The most important architectural point in the diagram is the join between structured and unstructured data.

Structured data explains what happened (for example, TRx declined).
Unstructured documents explain why it happened.
Both are linked using shared business keys such as:

HCP
HCO
Product / Brand
Region
Time



This linkage is shown in the diagram as “Join metrics with explanations”.
Architecturally, this allows Snowflake to return results that include both numbers and narrative context.

5. Orchestration (High Level)
When a user asks a question, Snowflake orchestrates the response:

Structured queries run against CDM or semantic tables.
Unstructured queries search document text and indexes.
The results are combined into a single, governed response.

This orchestration is logical, not pipeline‑level — the diagram intentionally stays at architecture level.

6. Consumption Layer
At the top of the diagram, the consumption layer remains unchanged.

Power BI
Business users
Agents

They ask questions like:

Why did sales drop 15%?
Why did TRx decline despite high field activity?

They receive metrics with explanations, without needing to read documents manually or access SharePoint.

7. Governance (Applies Everywhere)
Finally, the governance layer applies across the entire flow:

Role‑based access
Row‑level security
Audit and lineage

This ensures unstructured document content is accessed with the same controls as structured data.
