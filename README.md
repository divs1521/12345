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
