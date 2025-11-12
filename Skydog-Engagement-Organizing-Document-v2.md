# Rho ↔ Skydog Ops | HubSpot/Salesforce Audit & Optimization

**Last Updated**: 2025-11-12
**Timeline**: 2-3 Months (December 2025 - February 2026)
**Primary DRI**: Christopher Cooper (Marketing Ops) + Anthony Hwang (RevOps)
**Vendor**: Skydog Ops

---

## Interaction Framework

| Team/Role | Responsibilities |
|-----------|-----------------|
| **Christopher (Marketing Ops)** | HubSpot workflow triggers, lifecycle campaign alignment, email infrastructure |
| **Anthony (RevOps)** | Salesforce/Lead Routing, key property schemas, cross-system sync |
| **Skydog Ops** | Data flow analysis, bottleneck identification, architecture recommendations, audit execution |
| **Engineering/Product** | Signup Services event emission changes (Skydog/Growth Engineering supports PRD) |

---

## Goals → Issues → Solutions

| Goal | Issue | Root Cause | Potential Solution |
|------|-------|------------|-------------------|
| **Lead Acceleration** | Database Events / ETL Process During Application<br>• 40-45 min lag between signup and HubSpot visibility<br>• Sales can't act on high-intent leads in real-time | Batch warehouse syncs instead of real-time event streaming | **Signup Services refactoring** (Engineering-led, Skydog advisory)<br>• Document current state bottlenecks<br>• Design event-driven architecture (Segment/RudderStack)<br>• PRD for real-time event emission |
| **Workflow Efficiency** | Existing Sprawl, User/Data Flow<br>• 300+ active workflows, many conflicting<br>• 15% contact sync error rate<br>• No governance or documentation | No workflow governance, 3+ years of technical debt, multiple creators | **Audit, Document, Consolidate, Archive**<br>• Pull all workflows via API → Airtable<br>• Identify conflicts (3 MQL workflows, 40 event workflows)<br>• Consolidate: 300+ → <100 workflows<br>• Create governance framework |
| **Field Maintenance** | Data Schema, Taxonomy, & Sync Properties<br>• No canonical schemas between systems<br>• 15% HubSpot ↔ Salesforce sync error rate<br>• Same concept = 4 different field names | Each system (Signup Services, Warehouse, SFDC, HubSpot) uses different field names/formats | **Property Rationalization (5-Tier System)**<br>• Tier 1: Core (keep)<br>• Tier 2: Attribution (standardize)<br>• Tier 3: Lifecycle (rationalize)<br>• Tier 4: Integration (audit sync)<br>• Tier 5: Legacy/Unknown (archive ~200 properties) |
| **Lifecycle Campaigns (Demand Gen)** | Poorly Defined Audience Goals<br>• Role confusion: Lifecycle vs. Demand Gen<br>• No MOF/BOF ownership or budget<br>• 3 qualified candidates rejected | Leadership misalignment: CRO wants TOF/pipeline, JD asked for MOF/BOF expertise | **New Build Process, Segmentation Refinement**<br>• Define Growth vs. Lifecycle responsibilities<br>• Establish clear lifecycle stage definitions<br>• Workshop: Attribution model, segment strategy<br>• Clarify org ownership (see Key Questions) |
| **Database Cleanup** | Tiering & Value Per Contact / LTV<br>• 70% dead contacts skewing metrics<br>• Inaccurate fill rates (24% reported, 85% actual for live contacts)<br>• HubSpot cost bloat | No sunset policy, no contact scoring, no archival process | **Contact Tiering + Archival**<br>• Tier 1: Customers (revenue >$0)<br>• Tier 2: Active Leads (engaged <90 days)<br>• Tier 3: Nurture (engaged 90-365 days)<br>• Tier 4: Dead (archive/remove from HubSpot) |

---

## Skydog | Workflow Audit – Scope & Deliverables

### Phase 1: Discovery & Audit (Weeks 1-3)

**Deliverable 1.1: Workflow Inventory**
- Pull all workflows via API → export to Airtable/spreadsheet
- For each workflow, extract:
  - Trigger criteria (which properties)
  - Enrollment criteria (which lists/properties)
  - Actions (which properties set/updated)
  - Last modification date, last enrollment date
  - Total enrolled (all-time and last 90 days)
- **Output**: Dependency matrix showing property usage

**Deliverable 1.2: Property Schema Inventory**
- Pull all properties via API (Contacts, Companies, Deals)
- For each property, document:
  ```
  Property: application_status_signup
  Used in Workflows: [ID 123, ID 456, ID 789]
  Used in Forms: [Contact Form, Event Registration]
  Fill Rate: 12% (of non-dead contacts)
  Last Modified: 18 months ago
  Created By: [Unknown]
  Purpose: [Undocumented]
  Recommendation: [Archive - deprecated by app_status_v2]
  ```
- Categorize into 5 tiers (see above)
- **Output**: ~200 Tier 5 properties for archival

**Deliverable 1.3: Salesforce Sync Analysis**
- Export HubSpot-SFDC sync error logs (last 90 days)
- Categorize errors: Field mapping, validation failures, duplicates, API limits
- Current SFDC APEX automation review ([Wexpert internal analysis](link))
- **Output**: Error breakdown, field mapping audit, optimization recommendations

**Deliverable 1.4: Cross-Functional Alignment Workshop**
- Present audit findings (workflow conflicts, property sprawl, sync errors)
- Define Phase 2 priorities, establish RACI chart
- **Output**: Workshop summary, Phase 2 roadmap

---

### Phase 2: Remediation & Implementation (Weeks 4-7)

**Deliverable 2.1: Workflow Consolidation**
- Consolidate 40 event workflows → 1 template (dynamic properties)
- Merge 3 MQL workflows → 1 master workflow
- Deactivate orphaned workflows (~50 workflows: 0 enrollments in 90 days)
- Deactivate duplicates (~20 workflows: "Copy of...", "Test...")
- **Target**: 300+ workflows → <100 workflows

**Deliverable 2.2: Salesforce Sync Optimization**
- Field mapping fixes (text → picklist, missing required fields, date formats)
- Duplicate detection configuration (SFDC rules + HubSpot dedupe logic)
- Sync monitoring dashboard (Airtable + Slack alerts)
- **Target**: 15% error rate → <5% error rate

**Deliverable 2.3: Attribution Property Migration**
- Consolidate 3 "Primary Inbound Channel" properties → 1 canonical (`hs_analytics_source`)
- Backfill missing data from legacy/L3 properties
- Update 47 workflows + 8 reports to use canonical property
- Archive old properties (hide in UI, keep for historical audit)

---

### Phase 3: Advisory & Strategic Roadmap (Weeks 8-10)

**Deliverable 3.1: Real-Time Event Architecture PRD**
- Document current 40-minute lag analysis
- Design target state: Signup Services → Event Bus (Segment/RudderStack) → HubSpot webhook
- Event schema: `app_started`, `step_viewed`, `step_completed`, `app_submitted`, `app_approved`
- **Output**: PRD for Engineering (10-15 pages, event specs, API endpoints)

**Deliverable 3.2: Progress State Service Design**
- Design real-time dashboard for Sales ("who's stuck at step 5")
- Redis/Firebase cache architecture
- API spec: `GET /api/application-progress/:user_id`
- **Output**: Design doc for Engineering (5-10 pages)

---

## Additional Discovery (Out of Scope - Document Requirements Only)

**CAC Analysis**:
- Deal User Acquisition Costs (by cohort: monthly, quarterly)
- CAC based on Event Cost (venue, catering, attribution to deals)
- CAC based on Paid Channel / UTM Parameters (LinkedIn Ads, Google Ads)

**User/Data Flow Mapping**:
- Map complete journey: Signup Services → Warehouse → SFDC → HubSpot
- Identify data transformation points and enrichment steps

---

## Tech Stack Context

**Current Stack**:
- HubSpot: Email, workflows, contact database, forms
- Salesforce: CRM, opportunity tracking, sales workflows
- Snowflake: Data warehouse, ETL from Signup Services
- Signup Services (internal): Self-serve application funnel
- Luma: Event registration platform

**Tech Stack Needs & Requirements**:

| Priority | Need | Current Solution | Gap |
|----------|------|------------------|-----|
| **[P0]** | Ability to send email | ✅ HubSpot | Satisfied |
| **[P2]** | Simple, elegant branded emails with dynamic content | 🟡 HubSpot (limited) | Need component library + CI/CD |
| **[P1]** | Scale/implement other channels (SMS, Push, In-App) | ❌ None | Future: Customer.io or Braze |
| **[P1]** | Automated journeys based on Segments + Behavior | 🟡 HubSpot (workflow sprawl) | Optimize in Phase 2 |
| **[P1]** | Tracking user behavior on Rho.co | ❌ None | Need: Segment/Amplitude (out of scope) |
| **[P0]** | "Seamless" integration with Salesforce | 🟡 HubSpot native (15% error rate) | Optimize in Phase 2 |

---

## Organizational Structure & Key Questions

### Growth vs. Lifecycle: Role Clarity

| Function | Focus | Responsibilities | Metrics |
|----------|-------|-----------------|---------|
| **Growth** | Rapid customer acquisition and activation | • Short-term experimentation and testing<br>• Top and mid-funnel optimization<br>• Paid acquisition, SEO, conversion rate optimization | CAC, MQL/SQL volume, conversion rates, acquisition velocity |
| **Lifecycle** | Long-term customer retention and engagement | • Customer journey orchestration across all stages<br>• Post-purchase activation, retention, expansion, winback<br>• Email, SMS, in-app messaging, customer communication programs | LTV, churn rate, retention, engagement, expansion revenue |

**Current Gap**: Growth team (Jeremy) has zero direct reports. Marketing Ops (Christopher) reports to RevOps (Anthony), not Growth.

### Key Questions Requiring Leadership Alignment

| Question | Impact | Current State | Blocker |
|----------|--------|---------------|---------|
| **What team owns the data infrastructure?** | Who is accountable for event emission changes? ETL contracts? Schema governance? | Unclear: Engineering? Data? RevOps? Marketing Ops? | Blocks Phase 3 implementation (real-time events) |
| **What team owns tool acquisition / experimentation?** | Who approves new tools (Segment, Amplitude, Customer.io)? Who owns A/B testing platform? | Unclear: Growth? Marketing? RevOps? | Blocks scaling to multi-channel (SMS, Push) |
| **How does the org support MOF/BOF?** | Is there budget for lifecycle campaigns? Who owns retention/expansion? | Tommy (CRO) focused on TOF/Sales Pipeline. Lifecycle typically focused on MOF/BOF. | Blocks hiring Lifecycle Manager, unclear ownership |
| **Who defines lifecycle stages?** | What are the entry/exit criteria for MQL/SQL/Opportunity? | Conflicting definitions between HubSpot and Salesforce | Blocks consistent funnel reporting |

**Skydog Deliverable** (Phase 1, Week 4): Cross-functional alignment workshop with RACI chart recommendations

---

## Missing Aspects & Cross-Team Function Analysis

### Gaps Identified:

1. **Email Infrastructure Ownership**:
   - **Missing**: Component library CI/CD pipeline (Phase 2, Weeks 5-7)
   - **Need**: React JSX email components, GitHub Actions deployment to HubSpot
   - **Impact**: Currently 2-3 hours to build email, 30 min to deploy → Target: <30 min build, <5 min deploy
   - **DRI**: Christopher (Marketing Ops) + Skydog Ops

2. **Events Automation**:
   - **Missing**: Luma webhook integration, Airtable event registry
   - **Need**: Automated event → HubSpot sync (no manual CSV uploads)
   - **Impact**: Currently manual, error-prone → Target: 100% automated
   - **DRI**: Christopher (Marketing Ops) - parallel workstream to Skydog engagement

3. **Data Team Involvement**:
   - **Missing**: Data team not in interaction framework
   - **Need**: ETL contract documentation, warehouse schema governance
   - **Impact**: Blocks canonical data schemas (Goal: Field Maintenance)
   - **Add to RACI**: Data team as Supporting DRI for property schema rationalization

4. **Sales Team Involvement**:
   - **Missing**: Sales team not in interaction framework (needs real-time lead alerts)
   - **Need**: Sales input on lead acceleration requirements, "stuck application" dashboard
   - **Impact**: Blocks Phase 3 Progress State Service design
   - **Add to RACI**: Sales VP as Approval Required for Phase 3 deliverables

5. **Brand/Creative Team**:
   - **Missing**: Brand team not in interaction framework
   - **Need**: Brand guidelines for email component library (colors, fonts, imagery)
   - **Impact**: Delays email infrastructure build (Phase 2, Deliverable 2.4)
   - **Add to RACI**: Brand team as Supporting DRI for email component library

6. **Monitoring & Alerting**:
   - **Missing**: No proactive monitoring for sync health, workflow failures
   - **Need**: Slack alerts for error rates >10%, weekly automated reports
   - **Impact**: Current state is reactive (discover issues days later)
   - **Skydog Deliverable**: Sync monitoring dashboard (Phase 2, Week 6)

7. **Governance Framework**:
   - **Missing**: No workflow approval process, no property creation standards
   - **Need**: Governance policy (who can create workflows/properties, review process)
   - **Impact**: Without governance, cleanup will be undone in 6-12 months
   - **Skydog Deliverable**: Governance recommendations (Phase 2, end of engagement)

---

## Timeline & Milestones

| Milestone | Target Date | Deliverables | Success Criteria |
|-----------|-------------|--------------|------------------|
| **Phase 1 Complete** | Week 3 (Dec 22, 2025) | Workflow audit, Property audit, SFDC sync analysis, Cross-functional workshop | Audit reports delivered, RACI agreed, Phase 2 priorities confirmed |
| **Phase 2 Midpoint** | Week 5 (Jan 12, 2026) | 50% workflow consolidation, SFDC sync fixes started, Email library started | Workflows <150, Sync error improving (15% → 10%) |
| **Phase 2 Complete** | Week 7 (Jan 26, 2026) | Workflows <100, Sync error <5%, Attribution migrated, Email CI/CD live | All Phase 2 acceptance criteria met, Christopher sign-off |
| **Phase 3 Complete** | Week 10 (Feb 16, 2026) | Real-time event PRD, Progress state design, Governance recommendations | Engineering has specs, Leadership has decision framework |

---

## Communication Channels

- **Weekly Sync**: Every Monday, 30 min (Christopher + Anthony + Skydog Lead)
- **Slack**: `#skydog-engagement` (async questions, daily updates)
- **Phase Demos**: End of Phase 1, 2, 3 (60 min, all stakeholders + Skydog)

---

**Document Maintained In**:
- **GitHub**: `/Users/christopher.cooper/Documents/GitHub/mops-projects/` (version controlled)
- **Google Drive**: `/My Drive/MOps Infrastructure Project/` (collaborative editing)

**Next Review**: Upon Skydog contract signing (target: November 25, 2025)
