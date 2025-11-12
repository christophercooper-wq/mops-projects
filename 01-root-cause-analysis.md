# ROOT CAUSE ANALYSIS: DATA ARCHITECTURE FAILURE

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)

---

## EXECUTIVE SUMMARY

The Lifecycle Marketing Manager hire failed not due to candidate quality (all 3 finalists scored 7/7 "YES" from interview panels), but due to **systemic data architecture failures** that would prevent any hire from succeeding.

### **The Real Problems**

1. **No Canonical Data Schemas**: Signup Services → Data Warehouse → Salesforce → HubSpot have no agreed schema contracts
2. **40-45 Minute Application Status Lag**: Batch sync prevents real-time lead acceleration
3. **300+ HubSpot Workflows Built on Faulty Assumptions**: Data availability/accuracy issues cause 15% sync failures
4. **No Ownership Model**: Unclear who owns data infrastructure, ETL processes, experimentation
5. **Leadership Misalignment**: Tommy (CRO) wants demand generation, job description asked for lifecycle marketing

**Cost**: $100K+ annual inefficiency, 300% slower campaign deployment, failed hiring process

---

## THE 45-MINUTE GAP: SYMPTOM OF SYSTEMIC FAILURE

### **Current State Data Flow**

```
User starts application (Signup Services - Node.js/TypeScript)
    ↓
    [NO REAL-TIME EVENT EMISSION] ❌
    ↓
    [40-45 minute batch sync]
    ↓
Data Warehouse (Snowflake)
    ↓
    [Scheduled reverse ETL sync]
    ↓
Salesforce
    ↓
    [HubSpot-SFDC integration - 15% error rate]
    ↓
HubSpot
    ↓
    [Workflow triggers - too late]
    ↓
Marketing Automation (user already abandoned)
```

### **Why This Fails: User Journey Example**

**Scenario**: Enterprise prospect starts application at 10:00 AM

| Time | User Action | System Response | Problem |
|------|-------------|-----------------|---------|
| 10:00 AM | Starts application, enters email/company | Signup Services logs event | ❌ No event emitted to marketing systems |
| 10:05 AM | Completes Step 1 (basic info) | Proceeds to Step 2 | ❌ HubSpot has no visibility |
| 10:12 AM | Stuck on Step 3 (document upload - confusing UI) | User abandons, returns to work | ❌ No abandonment trigger |
| 10:45 AM | Warehouse batch sync runs | Status: "Step 3 incomplete" appears in Snowflake | ⏱️ 45 minutes later |
| 10:50 AM | Reverse ETL syncs to Salesforce | `Application_Status_Signup__c` = "Documents Pending" | ⏱️ 50 minutes later |
| 10:55 AM | HubSpot pulls from Salesforce | Contact created/updated in HubSpot | ⏱️ 55 minutes later |
| 11:00 AM | HubSpot workflow triggers | Email: "Need help with document upload?" | ⏱️ 60 minutes TOO LATE |
| Result | User says: "I already moved on" | Email ignored, conversion lost | ❌ Opportunity missed |

### **What "Speed-to-Lead" Actually Measures vs What We Need**

| Metric | Current Measurement | Actual Problem | What We Actually Need |
|--------|-------------------|----------------|----------------------|
| **Speed-to-Lead** | Time from lead in SFDC → sales touch | Looks good (<5 min SLA) | ❌ Measures wrong thing |
| **Lead Acceleration** | Time from interest signal → meaningful interaction | 45+ minutes | ✅ This is what matters |
| **Pipeline Throughput** | SQL → Close velocity | Days/weeks | ❌ Not the bottleneck |

**Christopher Cooper's Diagnosis**:
> "Speed-to-lead (which is actually a Lead interaction/acceleration issue not pipeline throughput)"

---

## TAXONOMY & SCHEMA FAILURE: THE ROOT CAUSE

### **Problem: No Canonical Data Contracts**

Each system represents the same concept (user progress) differently:

| System | Field Name | Data Type | Example Value | Sync Issues |
|--------|-----------|-----------|---------------|-------------|
| **Signup Services** | `app_status` | String | `"step_3_documents"` | ❌ Not synced in real-time |
| **Data Warehouse** | `app_status_signup` | JSON | `{"step": 3, "substep": "upload", "timestamp": "2025-11-12T10:12:00Z"}` | ❌ Parsing errors |
| **Salesforce** | `Application_Status_Signup__c` | Picklist | `"Documents Pending"` | ❌ Value mismatch |
| **HubSpot** | `Application Status Signup` | Single-line text | `"STEP_3"` | ❌ No standardization |

**Result**: Same semantic concept, 4 different implementations, no agreement on:
- Field names
- Data types
- Allowed values
- Update frequency
- Transformation rules
- Error handling

### **ETL Process Failure: No Ownership**

```
Engineering (Signup Services)
    ↓ Emits: Whatever fields they think marketing needs
    ↓ No contract, no versioning, no changelog
    ↓
Data Team (Warehouse)
    ↓ Transforms: Applies business logic (undocumented)
    ↓ JSON → Picklist mapping (breaks when format changes)
    ↓
RevOps (Anthony's team)
    ↓ Expects: Specific Salesforce fields populated
    ↓ Doesn't know warehouse transformations exist
    ↓
Marketing Ops (Christopher)
    ↓ Needs: HubSpot fields for segmentation
    ↓ Relies on fields that are never populated
    ↓
Result: Silent failures, no alerts, data loss
```

**Nobody Owns**:
- ❌ End-to-end schema definition
- ❌ Field naming standards
- ❌ Data type enforcement
- ❌ Transformation documentation
- ❌ Error monitoring/alerting
- ❌ Version control for schema changes

### **Example: Application Status Parsing Error**

**From Existing Documentation** (`Rho _ Marketing Org_ Strategic Audit.md`):

> Error when Parsing Application Status Signup field:
>
> `System.AuraException: Error to parse Application Status Signup. Error: Unexpected character ('N' (code 78)): was expecting comma to separate OBJECT entries at [line:1, column:108]`

**Root Cause**:
1. Signup Services changes JSON format (adds field, changes nesting)
2. Data Warehouse transformation expects old format
3. Parsing fails silently (no alert)
4. Bad data syncs to Salesforce
5. Salesforce validation rule blocks record update
6. HubSpot sync fails (record locked in SFDC)
7. Lead never enters HubSpot
8. **Marketing never knows the lead exists**

**Who Should Have Caught This**: Nobody owned the schema contract

---

## ATTRIBUTION BREAKDOWN: PRIMARY INBOUND CHANNEL CHAOS

### **Problem: 3+ Conflicting Implementations**

| Implementation | System | Status | Workflows Using It | Fill Rate | Notes |
|---------------|--------|--------|-------------------|-----------|-------|
| **Primary Inbound Channel (Legacy)** | HubSpot | ⚠️ Deprecated | 47 workflows | 8% | 24 months old, quarantined but not removed |
| **HubSpot L3 [SFDC Deal Object] / PIC [L3, 0.2: Source]** | HubSpot | ✅ Active (workaround) | 12 workflows | 67% | RevOps workaround, complex logic |
| **Original Traffic Source** | HubSpot (native) | ✅ Active | 3 workflows | 85% | HubSpot auto-set, limited values |
| **primary_inbound_channel_v2** | Salesforce | 📋 Planned | 0 workflows | 0% | Not yet implemented |

**From Existing Documentation**:

> "Primary Inbound Channel" is used as a proxy for best touch for attribution. The workaround fails at times due to associated data being disassociated at the time of submission or creation, or data being appended after submission.
>
> For instance:
> - Contact A UTM information is dropping/failing due to use of Incognito
> - Contact B, stating they found Rho via Perplexity [no UTM capture]

### **Why This Matters**

**Can't Answer**:
- ❓ Which marketing channel drives most revenue?
- ❓ Should we invest more in events vs paid ads?
- ❓ What's the ROI of partner campaigns?
- ❓ Which campaigns lead to fastest conversions?

**Because**:
- Different workflows use different PIC versions
- Fill rates vary wildly (8% to 85%)
- Data drops during UTM → HubSpot → SFDC → back flow
- No canonical "source of truth"

---

## HUBSPOT BLOAT: 300+ WORKFLOWS BUILT ON SAND

### **Current State**

| Category | Count | Status | Issue |
|----------|-------|--------|-------|
| **Lead Routing** | 23 workflows | ⚠️ Overlapping | 1-2 would suffice |
| **Attribution Setting** | 15 workflows | ⚠️ Conflicting | Using different PIC versions |
| **Event Nurture** | 47 workflows | ⚠️ Unmaintained | Some events no longer exist |
| **Data Hygiene** | 31 workflows | ⚠️ Redundant | Doing same task different ways |
| **Lifecycle Stage** | 12 workflows | ⚠️ Inconsistent | No agreed stage definitions |
| **Other/Unknown** | 172 workflows | ⚠️ Undocumented | No owner, purpose unclear |
| **TOTAL** | **300+ workflows** | 🔴 **BLOAT** | Maintenance nightmare |

### **Why Workflows Proliferate**

1. **No Governance**: Anyone can create workflows, no approval process
2. **No Sunset Policy**: Old workflows never archived
3. **No Documentation**: Purpose/owner not tracked
4. **Fear of Breaking**: Team afraid to delete unknowns
5. **Lack of Testing**: New workflows created instead of fixing existing
6. **Technical Debt**: 15% quarterly growth in maintenance overhead

### **Example: Lead Routing Chaos**

**23 Workflows Doing Similar Thing**:
- `[ID 001] "Route by company size"` (last used: 6mo ago) → Should be archived
- `[ID 045] "Route by industry - OLD"` (last used: 18mo ago) → Should be archived
- `[ID 203] "Route by territory v2"` (active, 1000+ enrollments) → Keep
- `[ID 298] "Route by engagement score"` (active, 500+ enrollments) → Merge with 203?
- ... (19 more)

**Maintenance Burden**:
- Changing routing rules = update 23 workflows
- Testing impact = test 23 workflows
- Onboarding new team member = explain 23 workflows
- RevOps changes territory = hope you found all 23

**Target State**: 1-2 workflows with branching logic

---

## SALESFORCE-HUBSPOT SYNC FAILURES

### **Current Error Rate: ~15%**

**From Existing Documentation** (`Hubspot Integration Errors` section):

#### **Error Type 1: Validation Rule Conflicts**

**Error**: `Disqual_reason_require_for_disqual_statu` validation rule
- **Cause**: HubSpot updates Lead Status to "Disqualified" without providing Disqualification Reason
- **Impact**: Record update blocked, sync fails
- **Workaround Options**:
  - Bypass validation for integration user (security risk)
  - Add Disqualification Reason field to HubSpot (manual entry required)
  - Auto-populate "Disqualified by HubSpot" (loses granularity)

#### **Error Type 2: JSON Parsing Failures**

**Error**: `System.AuraException: Error to parse Application Status Signup`
- **Cause**: Unexpected JSON format from HubSpot
- **Impact**: Opportunity creation fails, lead stuck
- **Root Cause**: Signup Services changed JSON schema, downstream systems didn't adapt

#### **Error Type 3: Required Field Missing**

**Error**: `event_cant_be_blank` validation rule
- **Cause**: "Please pick the Event this Lead was sourced from" but HubSpot doesn't send Event field
- **Impact**: Lead created but Opportunity blocked
- **Workaround**: Bypass validation or manually backfill event

#### **Error Type 4: Picklist Value Mismatches**

**Error**: Picklist sync failures
- **Cause**: HubSpot sends value not in Salesforce picklist (e.g., "In Progress - Step 3" vs "Documents Pending")
- **Impact**: Field update skipped silently
- **Root Cause**: No standardized value list between systems

### **Cascading Impact**

```
1 Sync Error
    ↓
Contact record locked in Salesforce
    ↓
HubSpot can't update (403 error)
    ↓
HubSpot workflow can't enroll contact
    ↓
Marketing email never sent
    ↓
Lead never nurtured
    ↓
Conversion lost
```

**Cost per Error**:
- Average lead value: $500 (assumption)
- 15% error rate on 10,000 monthly signups = 1,500 lost leads
- Monthly cost: $750,000 in lost opportunity
- **Annual cost: $9M in lost pipeline** (worst case)

**More Realistic Estimate**:
- 50% of errors auto-resolve (duplicate records, transient)
- 25% are low-value leads (would never convert)
- 25% are actual lost opportunities
- = 375 lost leads/month × $500 = $187,500/month
- **Annual cost: $2.25M in lost pipeline**

---

## ENGINEERING EVENT EMISSION GAP

### **What's Missing**

**Current State**: Signup Services only emits on final form submission

```javascript
// Pseudo-code: Current Implementation
function handleApplicationSubmit(data) {
  // User completes ALL steps
  if (data.status === "complete") {
    // ONLY THEN emit event
    emit("application_submitted", data);
    // Batch synced to warehouse 40-45 min later
  }
  // If user abandons at Step 3? NO EVENT
}
```

**Target State**: Emit events on every step

```javascript
// Pseudo-code: Target Implementation
function handleApplicationStep(step, data) {
  // Emit event on EVERY action
  emit("step_viewed", { step, timestamp, user_id, ...data });

  if (step.completed) {
    emit("step_completed", { step, timestamp, user_id, ...data });
  }

  if (step.error) {
    emit("validation_error", { step, error, timestamp, user_id });
  }

  if (step.upload_started) {
    emit("file_upload_started", { step, file_name, timestamp, user_id });
  }
}

// Abandonment detection (no activity for N minutes)
function checkAbandonment() {
  if (inactivityDuration > THRESHOLD) {
    emit("app_abandoned", { last_step, timestamp, user_id });
  }
}
```

### **Events Needed**

| Event | When | Payload | Use Case |
|-------|------|---------|----------|
| `app_started` | User lands on first step | `email`, `utm_params`, `timestamp` | Capture intent, attribution |
| `step_viewed` | User navigates to step | `step_name`, `timestamp` | Funnel analysis |
| `step_completed` | User completes step | `step_name`, `duration`, `timestamp` | Progress tracking |
| `validation_error` | User hits error | `step_name`, `error_type`, `field` | UX improvement, support trigger |
| `file_upload_started` | User uploads document | `file_name`, `file_size` | Understand complexity |
| `file_upload_completed` | Upload succeeds | `file_name`, `processing_status` | Enable next step |
| `app_abandoned` | No activity for N minutes | `last_step`, `duration`, `timestamp` | Winback trigger |
| `app_submitted` | Full completion | `all_data` | Existing behavior |

### **Delivery Mechanism Options**

| Option | Pros | Cons | Recommendation |
|--------|------|------|----------------|
| **Segment / RudderStack** | Industry standard, many integrations | Cost, vendor dependency | ✅ Recommended |
| **AWS Kinesis + Lambda** | Full control, scalable | Engineering overhead | ⚠️ If Segment too expensive |
| **Zapier Webhooks** | Fast setup, no-code | Rate limits, not real-time | ❌ Not production-ready |
| **Direct HubSpot API** | Simple, direct | Tight coupling, no flexibility | ❌ Avoids Event Bus benefits |

**Recommended Architecture**:

```
Signup Services
    ↓ [Emit events via Segment SDK]
Segment (Event Bus)
    ├→ Amplitude / PostHog (Analytics - instant funnel visibility)
    ├→ Data Warehouse (Async enrichment, historical analysis)
    ├→ Salesforce (via Segment integration - qualified leads only)
    ├→ Customer.io / HubSpot (Marketing automation - <5min triggers)
    └→ Redis / Firebase (Progress state cache for "who's stuck" dashboard)
```

### **Engineering Ask: PRD Requirements**

**Deliverable**: 1-2 page PRD with:
1. **Event Schema**: JSON format for each event
2. **Emission Points**: Where in codebase to emit
3. **Delivery Mechanism**: Segment vs custom
4. **Error Handling**: What happens if Segment is down
5. **Privacy**: PII handling, data retention
6. **Timeline**: Q1 2026 or Q2 2026 implementation

**Owner**: Christopher Cooper (draft) → Engineering (approve & implement)

---

## ORGANIZATIONAL OWNERSHIP GAPS

### **"Who Owns What" Is Unclear**

**From Christopher Cooper's Slack to Anthony** (October 27, 2025):

> **KEY QUESTIONS:**
> - What team owns data infrastructure?
> - What team owns tool acquisition / owns experimentation (differentiation of experiments)?
> - How does the org support MOF/BOF?
>   - Tommy seems focused on TOF/Sales Pipeline
>   - Lifecycle is typically focused on the Middle to Bottom

### **No RACI Chart Exists**

| Responsibility | Responsible | Accountable | Consulted | Informed |
|---------------|-------------|-------------|-----------|----------|
| **Data Infrastructure** (Schemas, ETL) | ❓ Data Team? Eng? | ❓ | ❓ RevOps? | ❓ MOps? |
| **Tool Acquisition** (HubSpot, Segment, etc.) | ❓ Anthony? Tommy? | ❓ | ❓ Finance? | ❓ |
| **Experimentation** (A/B tests, campaigns) | ❓ Growth? MOps? | ❓ | ❓ | ❓ |
| **MOF/BOF Support** (Nurture, retention) | ❓ Growth? Marketing? | ❓ | ❓ | ❓ |
| **Lead Acceleration** | ❓ RevOps? MOps? Eng? | ❓ | ❓ | ❓ |

**Result**: When something breaks, nobody knows whose job it is to fix it

### **Example: The 45-Minute Gap**

**Who Should Fix This?**
- Engineering? (They emit events)
- Data Team? (They run warehouse syncs)
- RevOps? (They own Salesforce)
- Marketing Ops? (They need fast data)
- Product? (They decide feature priorities)

**Current Answer**: Nobody is assigned, so nobody fixes it

---

## COST OF INACTION

### **Financial Impact**

| Cost Category | Annual Cost | Calculation |
|--------------|-------------|-------------|
| **Lost Pipeline** | $2.25M | 15% sync errors × 10K monthly leads × $500 avg value × 50% recovery rate |
| **Inefficiency Overhead** | $100K+ | Campaign deployment delays, manual work, rework |
| **Technical Debt** | 15% growth/quarter | Compounding maintenance burden |
| **Failed Hiring** | $150K+ | 3 months recruiting, 6 interviews, 1 false start |
| **TOTAL** | **$2.5M+/year** | Does not include opportunity cost of delayed launches |

### **Velocity Impact**

| Metric | Current | Industry Standard | Gap |
|--------|---------|-------------------|-----|
| **Campaign Deployment** | 2-4 days | 4-6 hours | 300% slower |
| **Lead Acceleration** | 45 minutes | <5 minutes | 900% slower |
| **Sync Error Rate** | 15% | <2% | 750% worse |
| **Workflow Maintenance** | Hours/week | Minutes/week | 10x harder |

### **Strategic Impact**

**Can't Execute**:
- ✅ Real-time lead acceleration (45-min lag)
- ✅ Accurate attribution (PIC chaos)
- ✅ A/B testing campaigns (velocity too slow)
- ✅ Lifecycle marketing (no BOF strategy)
- ✅ Demand generation (infrastructure blocks it)

**Can't Hire**:
- ✅ Lifecycle Manager (infrastructure + role misalignment)
- ✅ Demand Gen Manager (infrastructure not ready)

**Can't Grow**:
- ✅ Scale marketing campaigns (workflow bloat blocks changes)
- ✅ Support sales velocity (data arrives too late)
- ✅ Measure ROI (attribution broken)

---

## ROOT CAUSE SUMMARY: IT'S NOT HUBSPOT

### **Anthony's View** (Correct Tactically)

> "Vendor will focus on HubSpot improvements"

**Why This Is Right**:
- HubSpot workflows ARE bloated (300+)
- HubSpot properties ARE a mess (30%+ unused)
- HubSpot-SFDC sync DOES have 15% error rate

### **Christopher's View** (Correct Strategically)

> "Issues stem from a lack of taxonomy and data schemas that flow between systems paired with poor infrastructure between sign-up services (eng/data warehouse) and ETL process."

**Why This Is More Right**:
- HubSpot bloat is a **symptom**
- Root cause = **no data contracts upstream**
- Fixing HubSpot doesn't fix event emission gap
- Fixing HubSpot doesn't fix attribution
- Fixing HubSpot doesn't fix 45-minute lag

### **The Real Problem (Both Are Right)**

**HubSpot cleanup is necessary** → Phase 2 (Weeks 5-6)
**Data architecture documentation is critical** → Phase 1 (Weeks 1-4)
**Event emission is transformational** → Phase 3 Advisory (Weeks 7-8, implementation Q1-Q2 2026)

**All three must happen** or nothing fundamentally changes

---

## RECOMMENDED SOLUTIONS (PREVIEW)

**Detailed in Other Docs**:

1. **Leadership Alignment** ([02-organizational-assessment.md](./02-organizational-assessment.md))
   - Force Tommy/Jeremy/Anthony alignment meeting Week 1
   - Decide: Demand Gen vs Lifecycle role
   - Create RACI chart for ownership

2. **Skydog 3-Phase Engagement** ([06-vendor-sow-skydog.md](./06-vendor-sow-skydog.md))
   - Phase 1: Audit (Weeks 1-4) - Document data architecture gaps
   - Phase 2: Remediation (Weeks 5-6) - Consolidate workflows, fix syncs
   - Phase 3: Advisory (Weeks 7-8) - Event emission architecture

3. **Event Emission PRD** ([08-technical-architecture.md](./08-technical-architecture.md))
   - Collaborate with Engineering Week 2-3
   - Design event schema + delivery mechanism
   - Target Q1 2026 implementation (or Q2 if capacity limited)

4. **Events Automation MVP** ([03-events-automation.md](./03-events-automation.md))
   - Airtable event registry Week 1-2
   - Luma → Airtable → HubSpot flows Week 2-3
   - Source-agnostic CSV normalization Weeks 5-6

5. **Component Library CI/CD** ([10-component-library-cicd.md](./10-component-library-cicd.md))
   - Custom doc site Weeks 5-6
   - Auto-deploy to HubSpot API Week 7
   - Target: <4hr deployment time

---

## CONCLUSION

**The Lifecycle Manager hire didn't fail because of candidate quality.**

It failed because:
1. No real-time data (45-minute lag)
2. No canonical schemas (sync errors, attribution chaos)
3. No ownership model (nobody accountable for infrastructure)
4. Leadership misalignment (demand gen vs lifecycle confusion)

**Fixing HubSpot workflows alone won't solve this.**

We need:
1. ✅ Data architecture documentation (Skydog Phase 1)
2. ✅ HubSpot cleanup (Skydog Phase 2)
3. ✅ Event emission design (Skydog Phase 3 + Eng)
4. ✅ Leadership decisions (Week 1 meeting)
5. ✅ RACI chart (ownership clarity)

**Only then can we hire successfully** (Demand Gen Manager, Q1 2026)

---

**Document Maintained By**: Christopher Cooper
**Last Updated**: 2025-11-12
**Related Docs**: [02-organizational-assessment.md](./02-organizational-assessment.md), [08-technical-architecture.md](./08-technical-architecture.md)
**Status**: ✅ Analysis complete, recommendations in [09-project-roadmap.md](./09-project-roadmap.md)
