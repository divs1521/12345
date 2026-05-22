Data Processing Architecture: Thought Leadership Documentation
Executive Summary
The core question being challenged: Do we still need Bronze → Silver → Gold (Medallion Architecture) in an AI-first world, or can we simplify the data processing pipeline?
 
This document covers:
1. Why Medallion Architecture exists
2. What each layer does
3. The challenge to it
4. Futuristic data processing thinking
5. CDM (Common Data Model) role with AI Agents
6. Your Thought Leadership POV
 
 
What is Medallion (Bronze/Silver/Gold) Architecture?
 
Origin & Purpose
The Medallion Architecture was popularized by Databricks as a data design pattern used to organize data in a Lakehouse. The name comes from the progressive quality tiers — like medals — where data gets more refined and trusted as it moves through each layer.
It was designed to solve data chaos — when multiple source systems dump raw data into a warehouse and nobody knows what's clean, what's raw, and what's ready for business use.
 
Deep Dive — Each Layer Explained
 
BRONZE Layer (Landing / STG — Staging)
1. The raw landing zone — data arrives exactly as it comes from the source
2. No transformation, no cleaning, no business logic
3. It's a historical archive of everything that ever came in
 
What happens here:
1. SFTP files land here
2. S3-to-S3 copies happen here
3. API pulls dump data here
4. Oracle/JDBC extracts come in raw
5. Website clickstream, CRM exports, MDM feeds — all land here as-is
 
Why it exists:
ReasonExplanationAudit & LineageYou always have the original source of truth. If anything goes wrong downstream, you can re-process from BronzeReprocessingIf a business rule changes, you don't need to re-extract from the source. Just re-process BronzeDebuggingWhen data looks wrong in reporting, you trace back to Bronze to see what came in originallyComplianceRegulatory requirements (HIPAA, GDPR) often require raw data retentionMultiple consumersMultiple downstream pipelines can read from Bronze independently without hitting the source system again
In our architecture (from the deck):
 
STG (Staging) = Bronze
Data lands from all sources: Internal (CRM, MDM, NPS, Interaction, Promotions) + External (Sales, SP, Affiliation, EHR, Market Research, Longitudinal)
Ingestion framework handles: SFTP, S3-S3, SF Share, JDBC, REST API, Website
 
SILVER Layer (INT + DWH — Integration + Data Warehouse)
1. The curated, cleansed, and integrated layer
2. Raw data from Bronze is cleaned, validated, standardized, and joined across sources
3. This is where data starts to make business sense
 
What happens here (two sub-layers in our architecture):
 
INT (Integration / Unification):
Schema standardization — different source systems use different column names (e.g., cust_id vs customer_id vs CUST_NUM) → unified to one standard
Data type normalization — dates in different formats → unified format
Deduplication — same doctor record from CRM and MDM → deduplicated
Data quality checks — null checks, range checks, referential integrity
Entity resolution — is "Dr. John Smith at Apollo" in CRM the same as "J. Smith, Apollo Hospitals" in MDM? → resolved here
 
DWH (Data Warehouse / Refinement):
Business logic applied — calculations, categorizations, aggregations
Domain modeling — HCP (Healthcare Professional) data, HCO (Healthcare Organization) data structured properly
Historical tracking — Slowly Changing Dimensions (SCD) — track how a doctor's specialty changed over time
Fact and Dimension tables — traditional star schema or data vault modeling
 
Why it exists:
ReasonExplanationData TrustBusiness users and analysts need clean data, not raw dumpsCross-source JoinsCRM data + MDM data + EHR data need to be joined — Silver is where this happensPerformancePre-joined, pre-cleaned data is faster to query than raw tablesGovernanceBusiness rules are applied once here, not in 50 different reports
 
GOLD Layer (CDM + Semantic Data — Common Data Model)
1. The business-ready, use-case-optimized layer
2. Highly aggregated, denormalized, and purpose-built for consumption
3. In your architecture: CDM → Semantic Data → AI Agent
 
 
 
 
 
 
 
What happens here:
 
CDM (Common Data Model):
Takes the 5-10 normalized tables from Silver and collapses them into 1 unified entity
Example: Instead of:
 
HCP_Specialty table
HCP_Affiliation table
HCP_Credential table
HCP_Territory table
HCP_Interaction table
 
CDM has one HCP table with columns: hcp_id, name, specialty, affiliation, credential, territory, last_interaction, etc.
 
Semantic Data:
1. Goes beyond CDM — adds business context and meaning
2. Relationships between entities (HCP ↔ HCO, HCP ↔ Territory, HCP ↔ Product Affinity)
3. Use-case oriented views: "Top Prescribers in Cardiology in South Region"
4. Ready for AI Agents to consume without writing SQL
 
Why it exists:
ReasonExplanationAI ReadinessAI agents need flat, rich, contextual data — not normalized relational tablesSelf-serve analyticsBusiness users can query Gold without needing a data engineerSpeedNo joins needed — everything is pre-computedConsistencyOne single definition of "HCP" across the enterprise
 
