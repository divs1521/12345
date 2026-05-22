How data processing looks — end to end
Data starts as raw chaos from many source systems and ends as a clean, trusted, AI-consumable entity. Here is every step in detail.


Step 1 — source systems (raw chaos)
Step 1 - source systems (raw chaos)
Data comes from everywhere, in different formats, with different naming, different quality. No two systems agree on how to describe the same doctor.
1. CRM says "Dr. John A. Smith, Cardiologist, Apollo Hospital"
2. MDM says "J. Smith MD, Cardiology, Apollo Hospitals Ltd"
3. EHR says "SMITH JOHN, CARDIAC SURGERY, APOLLO"
4. External (IQVIA) says "John Smith, Card, Apollo Hospitals Pvt. Ltd."
These are all the same doctor — but no system knows that yet. That's the problem all subsequent layers solve.
 
Step 2 — Bronze (raw landing, no transformation)
Data lands exactly as received. No renaming, no cleaning, no business logic. Everything is timestamped and tagged with source system ID.
1. CRM data drops via SFTP at 2 AM daily
2. Oracle JDBC pull extracts 50 tables nightly
3. REST API calls pull JSON from external vendors every 6 hours
4. S3 file copies from Salesforce land as CSV
Bronze stores: raw column names like spclt_cd, original values like NULL, original encoding issues, all duplicates. Nothing is cleaned here. It is a permanent, append-only archive.
What Bronze adds as metadata only: src_load_timestamp, src_system_name, batch_id
 
 
Step 3 — Silver INT (integration / unification)
The hardest, most important layer. Raw data from all sources is standardised, deduplicated, and resolved into unified entities.
1. Schema standardisation: cust_nm, customer_id, CUST_NUM -> all become hcp_id
2. Data type fixes: dates in 10 different formats -> one ISO standard format
3. Entity resolution: "Dr. John Smith" + "J. Smith MD" + "SMITH JOHN" -> one golden HCP record
4. Reference data decoding: specialty code CD_CARD -> "Cardiology"
5. Null handling: missing territory -> assigned to default based on pincode
6. Cross-source deduplication: same NPI in CRM and MDM -> keep MDM as master, CRM as secondary
 
 
 
Step 4 — Silver DWH (data warehouse / refinement)
Business logic is applied on top of the unified entities from INT. This is where the data becomes analytically useful.
1. Business rules: what counts as an "active HCP"? (visited in last 180 days + valid credentials)
2. Historical tracking (SCD Type 2): doctor changes hospital — store old affiliation with end date, new one as current
3. Fact tables: HCP_Interaction_Fact — one row per visit, with date, rep, product, channel
4. Dimension tables: HCP_Dim — one row per doctor with all attributes
5. PII masking: some fields masked for non-privileged consumers
6. Aggregations: interaction counts per HCP per quarter pre-calculated
 
 
Step 5 — Gold CDM (common data model)
The 5–10 Silver tables per domain are collapsed into one wide, flat, denormalised entity table. This is the "AI-ready" format.
 
1. 5 HCP tables (specialty, affiliation, credential, territory, interaction) -> 1 cdm_hcp table with 65 columns
2. Pre-computed metrics: interaction_count_90d, prescribing_index, last_visit_date
3. Business-friendly column names (not cryptic codes)
4. Domain CDMs: cdm_hcp, cdm_hco, cdm_territory, cdm_product, cdm_sales
 
Step 6 — Gold semantic layer
Business definitions layered on top of CDM. This is what makes AI agent answers consistent and trusted.
1. "high value doctor" → prescribing_index ≥ 7 AND segment = 'A'
2. "engaged" → last_interaction_date ≥ CURRENT_DATE - 180
3. "South region" → state IN ('AP','TG','TN','KA','KL')
4. "coverage gap" → target_flag = 'Y' AND interaction_count_90d = 0
5. Metric: "coverage_rate" → COUNT(DISTINCT hcp_id WHERE visited) / COUNT(DISTINCT hcp_id WHERE target)
 
Step 7 — AI Agent consumes CDM + Semantic
The agent gets a natural language question, maps it through the semantic layer, generates a simple single-table SQL on CDM, and returns a trusted answer.
User asks: "Which high-value cardiologists in Hyderabad haven't we visited this quarter?"
1. Agent resolves "high-value" → prescribing_index ≥ 7
2. Agent resolves "cardiologists" → primary_specialty = 'Cardiology'
3. Agent resolves "Hyderabad" → affiliation_city = 'Hyderabad'
4. Agent resolves "not visited this quarter" → interaction_count_current_quarter = 0
5. Runs one clean SQL on cdm_hcp — no joins — returns correct answer
 
 
 
em
Why Each Layer Requried
Each layer solves a specific class of problem that cannot be solved by any other layer. Remove one and something breaks downstream.
 
BRONZE:
Why: audit, reprocessing, compliance
Without Bronze, you have no raw data history. If a business rule changes next year, you must re-extract 3 years of data from source systems (which may have changed). If a regulator asks what the CRM said about an HCP on a specific date, you cannot answer. If a Silver pipeline corrupts data, you cannot recover.
 
What Bronze gives you
1. Permanent audit trail
2. Re-process without re-extraction
3. Root cause debugging path
4. Regulatory data retention (HIPAA)
5. Source system independence
 
Cost of removing Bronze
1. Cannot trace bad data back to source
2. Rule changes = months of re-extraction
3. Failed compliance audits
4. No recovery path if Silver corrupts
5. Cannot answer "what did the source say on day X?"
 
 
SILVER:
Why: entity resolution, standardisation
Without INT, the same doctor appears as 3 different people in downstream data. AI agents built on top will count duplicates, answer with wrong HCP counts, and give sales reps conflicting information about the same doctor.
 
What INT gives you
1. One golden HCP record per doctor
2. Consistent column names and types
3. Cross-source data merged correctly
4. Reference data decoded to business terms
5. Nulls handled predictably
 
Cost of removing INT
1. AI sees 3 versions of the same doctor
2. Duplicate HCPs inflate counts
3. No consistent ID across CRM + MDM + EHR
4. AI writes SQL on raw column names (spclt_cd) — breaks often
5. Business reports disagree with AI agent answers
 
 
SILVER DWH : Why: business rules, history, modelling
Without DWH, there is no consistent definition of business metrics. "Active HCP" means different things in different reports. Historical tracking is lost — you can't see what segment a doctor was in last quarter vs this quarter.
 
What DWH gives you
1. Consistent business rule application
2. Historical tracking (SCD Type 2)
3. Pre-calculated metrics for performance
4. Domain data model (fact + dimension)
5. PII governance and masking
 
Cost of removing DWH
1. Every team calculates metrics differently
2. No historical view of HCP changes
3. AI must compute everything on-the-fly = slower, less accurate
4. No consistent "active HCP" definition
5. PII leaks to downstream consumers
 
 
GOLD:
GOLD CDM : Why: AI accuracy, single entity, no joins
Without CDM, AI agents must write 5–8 table SQL joins to answer basic questions. Error rate is 30–40%. With CDM, it is one table, one query. Error rate drops below 5%. CDM is not optional for AI use cases.
 
What CDM gives you
1. One table per entity domain
2. AI writes simple single-table SQL
3. Pre-computed metrics embedded
4. Consistent HCP definition enterprise-wide
5. Sub-5% AI error rate on queries
 
 
Cost of removing CDM
1. Agent must write 5-8 table SQL joins
2. 30–40% wrong answer rate from AI
3. Different reports define HCP differently
4. Slow query performance (no pre-aggregation)
5. Each agent team builds their own CDM anyway
 
 
 
 
 
 
SEMANTIC : Why: consistent meaning, business terms, governed metrics
Without a semantic layer, two people asking the same question in different ways get different answers because "coverage" and "high value" mean different things to different SQL queries. The semantic layer defines it once for everyone.
 
What Semantic gives you
1. One definition of every business term
2. Agent always uses same metric formula
3. Natural language maps correctly to SQL
4. Governed, auditable query logic
5. No metric inconsistency across reports
 
Cost of removing Semantic
1. Agent interprets "high value" differently each time
2. "Coverage" means different things in different queries
3. Business reviews question AI answers
4. No trust in AI agent outputs
5. Teams build their own definitions — inconsistency
 
 
 
With all layers — full architecture Pros and Cons
Running the complete Bronze → Silver (INT + DWH) → Gold (CDM + Semantic) → Agent stack.
This is the proven, enterprise-grade approach. Every layer earns its place.
 

 
PROS OF FULL ARCHITECURE
Full audit trail
Bronze preserves every raw record forever — regulators, debugging, reprocessing all covered
Clean entity resolution
Silver INT ensures one HCP = one record across all sources. No duplicates reach AI agent 
Consistent business rules
Silver DWH applies business logic once — all consumers (BI, AI, reports) use same definitions
AI accuracy very high
Agent queries CDM — single table, pre-computed metrics — error rate below 5%
Semantic consistency
"Coverage rate" always means the same thing. No metric inconsistency across teams
Multiple consumers
BI tools consume Silver DWH, AI consumes Gold CDM, operations consume INT — each gets right layer
Regulatory compliance
HIPAA, GDPR, UCPMP — Bronze gives data lineage, Silver gives quality evidence, Gold gives masking
Failure recovery​
Any layer fails — downstream can wait or fall back. Bronze always has the truth
 
CONS OF FULL ARCHITECTURE
More layers to maintain​
4–5 pipeline layers = more code, more monitoring, more failure points for engineering team
Higher latency
Bronze → Silver → Gold pipeline can take 6–12 hours end-to-end. Real-time use cases suffer
Storage cost (3x data)
Same data stored in Bronze (raw), Silver (clean), Gold (flat) — roughly 2.5–3x storage usage
Slower to onboard new domains
New data source takes longer to reach Gold — every layer must be built and tested
Organisational complexity
Different teams may own different layers — handoffs, SLAs, blame games between teams
 
When full architecture is the right choice
1. Large enterprise with regulated data (pharma, healthcare, finance)
2. Multiple data consumers: BI + AI + Operations + Compliance all need different data
3. Complex entity resolution required (HCP data with multiple source systems)
4. Regulatory audit requirements (HIPAA, GDPR, FDA)
5. AI Agents making business decisions that affect sales reps or patients
 
Without layers — what breaks and when
Scenario A: Skip Bronze — go Source → Silver directly
Never do this. The cost is nearly zero; the risk is enormous.
 
What you save
1. One less pipeline layer
2. Slightly simpler architecture to explain
3. Small storage reduction
 
What breaks
1. No raw data history — can never re-process if rules change
2. Regulators ask for original data — you cannot produce it
3. Silver pipeline corrupts batch — no way to recover
4. Source system changes format — you lose all historical mapping
5. Root cause analysis impossible — Silver data is all you have
 
Scenario B: Skip Silver INT — go Bronze → DWH directly
Dangerous. Entity duplication reaches every downstream system including AI agents.
 
What AI agent sees without INT
Three records for the same doctor:
1. HCP_ID: CRM_8842 — "Dr. John Smith, Cardio"
2. HCP_ID: MDM_1234 — "J. Smith MD, Cardiology"
3. HCP_ID: EHR_9901 — "SMITH JOHN, Cardiac"
Agent counts: 3 cardiologists. Reality: 1 doctor. Every HCP count is wrong. Every "top prescribers" list has duplicates. Sales reps get incorrect call lists.
 
What AI agent sees with INT
One golden record:
1. HCP_ID: HCP_00123 — "Dr. John Smith"
2. Specialty: Cardiology (from MDM, authoritative)
3. Affiliation: Apollo (from CRM, most recent)
4. Credential: DM Cardiology (from EHR, verified)
Agent counts: 1 cardiologist. Correct. All downstream logic builds on clean entity.
 
Scenario C: Skip Silver DWH — go INT → CDM directly
Manageable risk, but CDM becomes harder to build correctly and business consistency breaks.
 

 
Scenario D: Skip Gold CDM — go Silver DWH → Agent directly
The most dangerous skip for AI agents. This is what produces 30–40% wrong answer rates.
 
Agent on Silver (5 tables)
Question: "Top cardiologists in South not visited this quarter"
Agent must write:
1. JOIN HCP_Specialty ON hcp_id
2. JOIN HCP_Territory ON hcp_id
3. JOIN HCP_Affiliation ON hcp_id
4. LEFT JOIN HCP_Interaction ON hcp_id AND quarter filter
5. WHERE specialty = 'Cardiology' AND region = 'South' AND interaction count = 0
 
Wrong join condition on one table = completely wrong HCPs returned. The agent doesn't know it's wrong. Confidently returns an incorrect answer. Sales reps call wrong doctors.
 
Agent on CDM (1 table)
Question: same question
Agent writes:
● SELECT hcp_name, hospital, prescribing_index
● FROM cdm_hcp
● WHERE specialty = 'Cardiology'
● AND region = 'South'
● AND interaction_count_current_quarter = 0
● ORDER BY prescribing_index DESC
No joins. Single table. Pre-computed metric. Correct answer every time. Sales reps trust the output.
 
 
 
 
 
 
 
 
 
 
 
Scenario E: Compressed architecture (INT + DWH → one Curation layer)
This is the valid simplification. Compress Silver's two sub-layers into one — keep all other layers.
 

 
