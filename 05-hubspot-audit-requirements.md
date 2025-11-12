# 05 - HubSpot Audit Requirements & Workflow Optimization

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Related Documents**: [01-Root Cause Analysis](01-root-cause-analysis.md), [06-Vendor SOW - Skydog](06-vendor-sow-skydog.md)

---

## Executive Summary

Rho's HubSpot instance has **300+ active workflows** with no governance, documentation, or sunset policy, creating:
- **Operational Risk**: Critical workflows could break with no owner to fix them
- **Performance Issues**: Overlapping workflows create duplicative processing, slow system performance
- **Maintenance Burden**: Christopher spends 10+ hours/month troubleshooting workflow conflicts
- **Compliance Gaps**: No audit trail for why workflows exist or when to deprecate them

**Proposed Solution**: Comprehensive HubSpot Audit + Governance Framework

### Three-Phase Approach

**Phase 1 (Week 1-2)**: Automated Workflow Inventory
- Build audit tool to extract all workflows via HubSpot API
- Generate dependency map (which workflows trigger others)
- Identify orphaned workflows (no activity in 90+ days)
- **Target**: Reduce to <100 active workflows

**Phase 2 (Week 3-4)**: Property Cleanup & Standardization
- Audit custom properties (likely 500+, many duplicates)
- Consolidate "Primary Inbound Channel" implementations (currently 3 conflicting versions)
- Create property naming conventions
- **Target**: Archive 50% of unused properties

**Phase 3 (Week 5-8)**: Governance Framework Implementation
- Workflow approval process (require Christopher sign-off before activation)
- Documentation templates (purpose, owner, success criteria)
- Sunset policy (auto-disable workflows with no activity for 180 days)
- **Target**: Zero undocumented workflows

**Timeline**: 8 weeks (aligned with Skydog Ops engagement)

---

## Current State: HubSpot Chaos

### Problem 1: 300+ Workflows (Target: <100)

**Breakdown by Category** (estimated, requires audit to confirm):

| Category | Count | Status | Issue |
|----------|-------|--------|-------|
| **Lead Lifecycle** | 50+ | Active | Overlapping rules (3 workflows set "MQL" status, conflicting logic) |
| **Event Automation** | 40+ | Mostly active | One workflow per event (should be 1 template workflow) |
| **Email Nurture** | 60+ | Mixed | Many duplicates (copied workflow instead of editing original) |
| **Salesforce Sync** | 30+ | Active | Redundant (HubSpot native sync should handle most) |
| **Data Enrichment** | 20+ | Active | Calling Clearbit on every contact update (expensive, inefficient) |
| **Internal Notifications** | 50+ | Active | Slack/email alerts for every minor action (notification fatigue) |
| **Abandoned/Test** | 50+ | Inactive | Never turned off after testing, cluttering workspace |

**Key Issues**:

1. **No Naming Convention**: Workflows named "Test", "Copy of Workflow 1", "Final Version 2 (Actually Final)"
2. **No Ownership**: 80% of workflows have no documented owner (creator may have left company)
3. **No Documentation**: 95% have blank "Description" field (no one knows why it exists)
4. **No Sunset Policy**: Workflows inactive for 2+ years still sitting in "Active" state
5. **No Dependency Mapping**: Can't safely delete workflow (might break downstream dependencies)

**Example Conflict**:
```
Workflow A: "If form submit = 'Contact Sales', set Lifecycle Stage = 'SQL'"
Workflow B: "If form submit = ANY, set Lifecycle Stage = 'MQL'"
Workflow C: "If Lifecycle Stage = 'MQL', wait 3 days, set = 'SQL'"

Result: Contact submits "Contact Sales" form
- Workflow A fires → sets SQL ✅
- Workflow B fires 5 seconds later → OVERWRITES to MQL ❌
- Workflow C fires 3 days later → sets back to SQL 🤔
- Sales team confused why lead took 3 days to route
```

### Problem 2: Property Chaos (Estimated 500+ Custom Properties)

**Issues Identified**:

| Problem | Example | Impact |
|---------|---------|--------|
| **Duplicate Properties** | `company_size`, `company_employee_count`, `number_of_employees` (all store same data) | Reporting fragmentation, data inconsistency |
| **Conflicting Definitions** | "Primary Inbound Channel" has 3 implementations (Legacy, L3 workaround, Native) with different fill rates | Attribution is meaningless, can't trust data |
| **Orphaned Properties** | 200+ properties with zero population (created for one-off campaign, never used again) | Cluttered UI, slows down HubSpot performance |
| **No Naming Convention** | Mix of `snake_case`, `camelCase`, `Title Case`, `random-dashes` | Hard to find properties, unclear what they represent |
| **No Descriptions** | 70% of custom properties have blank "Description" field | No one knows what property means or when to use it |
| **Type Mismatches** | `event_date` stored as single-line text instead of date field | Can't sort/filter properly, breaks workflows |

**"Primary Inbound Channel" Case Study**:

User quote from 01-root-cause-analysis.md:
> "Primary Inbound Channel" is a property that should track the first marketing touchpoint. But there are THREE different implementations:
> - **Legacy Version**: 8% fill rate (broken, no longer updating)
> - **L3 Workaround**: 67% fill rate (temporary fix by contractor, uses non-standard taxonomy)
> - **Native HubSpot**: 85% fill rate (correct implementation, but some workflows still reference old versions)

**Impact**: Marketing reports on "Primary Inbound Channel" attribution, but data is meaningless because it's pulling from 3 different properties with conflicting values.

### Problem 3: No Governance = Technical Debt Accumulation

**Symptoms**:
- **Workflow Conflicts**: 15% of contacts have errors due to overlapping workflows (see 01-root-cause-analysis.md)
- **Performance Degradation**: HubSpot slow to load (300+ workflows running on every contact update)
- **Broken Automations**: Email nurture sequences stop mid-campaign (workflow was deactivated by someone who didn't know it was critical)
- **Compliance Risk**: Email workflows may not honor unsubscribe preferences (no audit trail)

**No Clear Ownership**: RACI chart from 02-organizational-assessment.md shows:
- **Responsible** for HubSpot governance: UNKNOWN
- **Accountable**: UNKNOWN
- **Consulted**: UNKNOWN
- **Informed**: UNKNOWN

Christopher is de facto owner, but has no authority to enforce governance (Growth team can create workflows without approval).

---

## Proposed Solution: HubSpot Audit + Governance Framework

### Phase 1: Automated Workflow Inventory (Week 1-2)

**Goal**: Generate complete audit of all workflows with dependency mapping.

#### Audit Tool Requirements

**Data to Extract** (via HubSpot API):

| Field | Purpose | API Endpoint |
|-------|---------|--------------|
| **Workflow ID** | Unique identifier | `/automation/v4/workflows` |
| **Workflow Name** | Display name | Same |
| **Status** | Active, Paused, Deleted | Same |
| **Type** | Standard, Sequence, Bot | Same |
| **Created Date** | When was it created | Same |
| **Created By** | User who created it (may have left company) | `/settings/v3/users/{userId}` |
| **Last Modified Date** | When last edited | `/automation/v4/workflows/{workflowId}` |
| **Last Executed Date** | When it last ran | Same |
| **Enrollment Count (All-Time)** | Total contacts ever enrolled | Same |
| **Enrollment Count (Last 90 Days)** | Recent activity level | Custom calculation |
| **Current Enrollments** | Active contacts in workflow | Same |
| **Actions** | List of actions (email send, property update, webhook, etc.) | Same (nested in workflow definition) |
| **Triggers** | What causes enrollment (form submit, property change, etc.) | Same |
| **Dependencies** | Which workflows this one triggers (or is triggered by) | Custom analysis (parse action types) |
| **Properties Referenced** | Which custom properties are used | Custom parsing |

**Output Format**: CSV + Airtable Base

**Airtable Schema** (Workflows Audit Table):

| Field | Type | Description |
|-------|------|-------------|
| `workflow_id` | Number | HubSpot workflow ID |
| `workflow_name` | Single-line text | Workflow name |
| `status` | Single-select | Active, Paused, Deleted |
| `category` | Single-select | Lead Lifecycle, Event Automation, Email Nurture, etc. (manual categorization) |
| `owner` | Single-select | Christopher, Growth Team, Unknown (needs assignment) |
| `created_date` | Date | When created |
| `created_by` | Single-line text | User name |
| `last_executed` | Date | When last ran |
| `enrollments_90d` | Number | Enrollments in last 90 days |
| `current_enrollments` | Number | Active enrollments |
| `priority` | Single-select | Critical (do not touch), Standard, Deprecated (can delete) |
| `documentation_status` | Single-select | Documented, Needs Documentation, Unknown Purpose |
| `action_items` | Long text | Notes (e.g., "Merge with Workflow 456", "Delete after migration") |
| `dependencies` | Link to other Workflow records | Which workflows trigger this one |

#### Workflow Categorization Logic

**Automated Rules** (apply via script):

1. **Orphaned Workflows** (High priority for deletion):
   - Status = "Active" BUT `enrollments_90d` = 0
   - Action: Tag as "Deprecated - No Activity"

2. **Abandoned Workflows**:
   - Status = "Paused" for >180 days
   - Action: Tag as "Deprecated - Abandoned"

3. **High-Impact Workflows** (Requires careful review before changes):
   - `current_enrollments` > 100 OR
   - Contains action type "Send to Salesforce" OR
   - Contains action type "Send email" with >1000 all-time sends
   - Action: Tag as "Critical - Requires Sign-Off"

4. **Event Workflows** (Candidate for templating):
   - Name contains "Event" OR "Luma" OR "Roundtable"
   - Action: Tag as "Event Automation - Consolidate"

5. **Duplicate Workflows**:
   - Name contains "Copy of" OR "Test" OR "v2"
   - Action: Tag as "Potential Duplicate - Review"

#### Dependency Mapping

**Logic**:
- Parse workflow actions for "Enroll in workflow" action type
- Create link: Workflow A → Workflow B
- Generate visual map (Mermaid diagram or Lucidchart)

**Example Output**:
```
Contact submits "Request Demo" form
    ↓
Workflow A: "Request Demo - Immediate Response"
    → Sends email confirmation
    → Sets Lifecycle Stage = "SQL"
    → Enrolls in Workflow B
        ↓
        Workflow B: "SQL Nurture Sequence"
            → Wait 1 day, send email 2
            → Wait 3 days, send email 3
            → If no response, enrolls in Workflow C
                ↓
                Workflow C: "SQL Re-Engagement"
                    → Wait 7 days, send Slack alert to sales
```

**Risk**: If Workflow A is deleted, Workflows B and C never trigger (broken automation).

#### Tool Implementation

**Option A: Python Script (Recommended for Week 1)**

**Tech Stack**:
- Python 3.9+
- HubSpot API client (`hubspot-api-client` library)
- Airtable API (`pyairtable` library)
- Pandas (data manipulation)

**Script Workflow**:
```python
import hubspot
from pyairtable import Api
import pandas as pd

# 1. Authenticate with HubSpot
client = hubspot.Client.create(access_token="YOUR_HUBSPOT_API_KEY")

# 2. Fetch all workflows
workflows = client.automation.workflows.get_all()

# 3. For each workflow, extract metadata
workflow_data = []
for wf in workflows:
    data = {
        "workflow_id": wf.id,
        "workflow_name": wf.name,
        "status": wf.enabled,
        "created_date": wf.created_at,
        "created_by": wf.created_by,
        "last_executed": wf.last_executed_at,
        "enrollments_total": wf.total_enrolled,
        "enrollments_current": wf.currently_enrolled,
        # Parse actions, triggers, dependencies
    }
    workflow_data.append(data)

# 4. Convert to DataFrame
df = pd.DataFrame(workflow_data)

# 5. Apply categorization logic
df['category'] = df.apply(categorize_workflow, axis=1)
df['priority'] = df.apply(assign_priority, axis=1)

# 6. Export to CSV
df.to_csv('hubspot_workflows_audit.csv', index=False)

# 7. Upload to Airtable
airtable = Api('YOUR_AIRTABLE_API_KEY')
table = airtable.table('YOUR_BASE_ID', 'Workflows Audit')
for record in df.to_dict('records'):
    table.create(record)
```

**Timeline**: 8 hours to build + 2 hours to run/debug = 10 hours total

**Option B: No-Code Tool (Zapier/Make)**

**Pros**: Faster to set up (4 hours), no coding required
**Cons**: HubSpot API pagination limits, may hit rate limits, harder to do complex dependency parsing

**Recommendation**: Use Python script (Option A) for comprehensive audit.

---

### Phase 2: Property Cleanup & Standardization (Week 3-4)

**Goal**: Audit custom properties, consolidate duplicates, establish naming conventions.

#### Property Audit Process

**Step 1: Extract All Properties** (via HubSpot API)

**Data to Capture**:

| Field | API Endpoint | Purpose |
|-------|-------------|---------|
| **Property Name** | `/properties/v1/contacts/properties` | Internal name (e.g., `primary_inbound_channel`) |
| **Label** | Same | Display name (e.g., "Primary Inbound Channel") |
| **Type** | Same | Text, Number, Date, Dropdown, etc. |
| **Description** | Same | Property documentation (often blank) |
| **Field Type** | Same | Standard vs. Custom |
| **Created Date** | Same | When property was created |
| **Created By** | Same | User who created it |
| **Usage Count** | Custom query | How many contacts have this property populated |
| **Last Updated** | Custom query | When property last changed for any contact |
| **Referenced In Workflows** | Cross-reference with workflows audit | Which workflows use this property (for impact analysis) |

**Output**: `hubspot_properties_audit.csv` + Airtable table

**Step 2: Identify Duplicates**

**Automated Detection**:
1. **Exact Match**: Properties with similar names (Levenshtein distance <3)
   - Example: `company_size` vs `companysize` vs `company_Size`
2. **Semantic Match**: Properties with similar labels
   - Example: "Employee Count" vs "Number of Employees" vs "Company Size"
3. **Low Usage**: Properties with <1% population rate AND no workflow dependencies
   - Action: Tag as "Candidate for Archival"

**Manual Review Required**:
- Christopher reviews flagged duplicates
- Decides which property is "canonical" (keep), which to archive
- Create migration plan (copy data from deprecated property to canonical before archiving)

**Step 3: "Primary Inbound Channel" Consolidation**

**Current State** (from 01-root-cause-analysis.md):
- `primary_inbound_channel_legacy` (8% fill rate) ❌
- `primary_inbound_channel_l3` (67% fill rate) ⚠️
- `hs_analytics_source` (85% fill rate, HubSpot native) ✅

**Migration Plan**:
1. **Audit**: Identify all workflows/reports using old properties
2. **Backfill**: For contacts with data in `_legacy` or `_l3`, copy to `hs_analytics_source` (if empty)
3. **Update Workflows**: Change 15 workflows to reference `hs_analytics_source` instead
4. **Update Reports**: Fix 8 marketing reports/dashboards
5. **Archive Old Properties**: Hide `_legacy` and `_l3` from UI (don't delete, for historical audit)
6. **Document**: Update property description to explain history

**Timeline**: 6 hours (audit) + 8 hours (backfill + updates) = 14 hours

#### Property Naming Conventions (New Standard)

**Format**: `{object}_{descriptor}_{type}`

**Examples**:
- ✅ `contact_lifecycle_stage` (object = contact, descriptor = lifecycle, type = stage)
- ✅ `company_employee_count` (object = company, descriptor = employee, type = count)
- ✅ `event_registration_date` (object = event, descriptor = registration, type = date)
- ❌ `EmployeeCount` (no object prefix, wrong casing)
- ❌ `how-many-employees` (wrong format, not machine-readable)

**Casing**: `snake_case` (consistent with HubSpot API conventions)

**Required Fields for All Custom Properties**:
1. **Description**: 1-2 sentences explaining what property tracks, when to use it
2. **Owner**: Team responsible (Marketing Ops, Sales Ops, Growth)
3. **Created Date**: (auto-populated by HubSpot)
4. **Deprecation Date**: If property is sunset, note when and why

**Enforcement**:
- Christopher approves all new custom properties before creation
- Use HubSpot API to auto-reject properties that don't match naming convention (future enhancement)

---

### Phase 3: Governance Framework Implementation (Week 5-8)

**Goal**: Prevent future chaos with approval processes, documentation templates, sunset policies.

#### Governance Component 1: Workflow Approval Process

**New Rule**: No workflows can be set to "Active" without Christopher's sign-off.

**Workflow** (pun intended):
```
Growth Team: "We need a workflow to nurture event attendees"
    ↓ [Fill out Workflow Request Form]
Growth Team: Submits form with:
    - Purpose: What business problem does this solve?
    - Trigger: What causes enrollment?
    - Actions: What does workflow do?
    - Success Criteria: How do we know it's working?
    - Owner: Who maintains this?
    ↓
Christopher: Reviews request (within 24 hours)
    ↓ [Checks for conflicts with existing workflows]
Christopher: Approves OR Requests Changes
    ↓ [If approved]
Growth Team: Builds workflow in HubSpot
    ↓ [Sets to "Paused"]
Growth Team: Notifies Christopher for final review
    ↓
Christopher: Tests workflow with sample contact
    ↓ [Verifies no conflicts]
Christopher: Sets workflow to "Active"
    ↓
Christopher: Logs workflow in Airtable audit table
```

**Workflow Request Form** (Google Form or Airtable Form):

**Fields**:
1. **Workflow Name**: Clear, descriptive name (must follow naming convention: `{Category} - {Purpose}`)
2. **Category**: Lead Lifecycle, Event Automation, Email Nurture, etc.
3. **Purpose**: 2-3 sentences explaining why this workflow is needed
4. **Trigger**: What causes a contact to enroll? (form submit, property change, etc.)
5. **Actions**: List of actions (send email, update property, enroll in another workflow, etc.)
6. **Success Criteria**: How will we measure if this workflow is successful? (e.g., "30% open rate on emails")
7. **Owner**: Who is responsible for maintaining this workflow?
8. **Dependencies**: Does this workflow depend on (or trigger) other workflows?
9. **Sunset Date**: When should we review if this workflow is still needed? (default: 6 months)

**Christopher's Review Checklist**:
- [ ] Does this workflow conflict with existing workflows? (check Airtable audit for overlaps)
- [ ] Is there an existing workflow that could be modified instead? (avoid duplication)
- [ ] Are all properties referenced valid and correctly named?
- [ ] Does this workflow honor unsubscribe preferences? (compliance check)
- [ ] Is the trigger specific enough? (avoid enrolling too many contacts)
- [ ] Are there clear success criteria? (can we report on this?)

**Timeline**: 30 min per workflow review (avg), 10 workflows/month = 5 hours/month ongoing

#### Governance Component 2: Documentation Templates

**For Every Workflow** (Required Fields):

**HubSpot Workflow Description Field**:
```
PURPOSE: {1-sentence summary}
OWNER: {Name, Team}
CREATED: {YYYY-MM-DD}
LAST REVIEWED: {YYYY-MM-DD}
SUCCESS CRITERIA: {KPI, target}
DEPENDENCIES: {List of workflow IDs this triggers/is triggered by}
NOTES: {Any special considerations, known issues}
```

**Example**:
```
PURPOSE: Nurture event attendees with 3-email sequence over 2 weeks
OWNER: Christopher Cooper, Marketing Ops
CREATED: 2025-12-15
LAST REVIEWED: 2025-12-15
SUCCESS CRITERIA: 25% click-through rate on CTA emails
DEPENDENCIES: Triggered by Workflow 12345 (Event Registration Confirmation)
NOTES: Only enrolls contacts who attended (attendance_status = "Attended"). Excludes customers (lifecycle stage != "Customer").
```

**For Every Custom Property** (Required Fields):

**HubSpot Property Description Field**:
```
DEFINITION: {What does this property track?}
USAGE: {When should this property be updated?}
OWNER: {Name, Team}
CREATED: {YYYY-MM-DD}
SOURCE: {Where does data come from? Manual entry, workflow, API integration?}
EXAMPLES: {2-3 example values}
```

**Example**:
```
DEFINITION: Primary marketing channel that first brought contact to Rho
USAGE: Auto-populated on first form submission, never overwrite
OWNER: Christopher Cooper, Marketing Ops
CREATED: 2024-03-15
SOURCE: HubSpot native tracking (hs_analytics_source), enhanced with UTM parameter logic
EXAMPLES: "Organic Search", "Paid Social - LinkedIn", "Event - Luma"
```

#### Governance Component 3: Sunset Policy

**Rule**: Workflows with zero activity for 180 days are automatically reviewed for deprecation.

**Automated Process**:
1. **Weekly Audit Script**: Run Python script to check all workflows
2. **Flag Inactive**: If `enrollments_last_180_days` = 0, add to "Review Queue" in Airtable
3. **Notify Owner**: Send email to workflow owner: "Your workflow '{name}' has had no activity for 180 days. Please confirm if still needed."
4. **Owner Response**:
   - **Still Needed**: Owner updates "Last Reviewed" date, workflow stays active
   - **No Longer Needed**: Owner deactivates workflow, tags as "Deprecated"
   - **No Response in 14 Days**: Christopher deactivates workflow, notifies team

**Airtable Automation**:
- Every Monday, query Workflows Audit table for workflows where `last_executed` > 180 days ago AND `status` = "Active"
- Send email notification to `owner` field
- Set `documentation_status` = "Pending Sunset Review"

**Benefits**:
- Prevents workflow bloat (auto-prunes dead workflows)
- Forces regular review (owners must actively confirm workflows are still needed)
- Reduces risk (deprecated workflows can't break if they're turned off)

---

## Implementation Roadmap

### Week 1: Workflow Audit Tool Build

**Tasks**:
1. **HubSpot API Setup** (2 hours)
   - Generate private app access token with workflows/properties scopes
   - Test API access (fetch 10 workflows)

2. **Python Script Build** (10 hours)
   - Fetch all workflows, parse metadata
   - Calculate `enrollments_90d` (requires date filtering)
   - Parse workflow definition JSON to extract actions, triggers, dependencies
   - Apply categorization logic

3. **Airtable Base Setup** (3 hours)
   - Create "Workflows Audit" table with schema
   - Create "Properties Audit" table with schema
   - Set up views (e.g., "Orphaned Workflows", "High Priority", "Needs Documentation")

4. **Run Audit + Initial Review** (5 hours)
   - Execute script, export to Airtable
   - Christopher manually reviews first 50 workflows
   - Categorize (Lead Lifecycle, Event, Email, etc.)
   - Assign priority (Critical, Standard, Deprecated)

**Deliverables**:
- ✅ Python audit script
- ✅ Airtable base with full workflow inventory
- ✅ CSV export for stakeholder review

**Owner**: Skydog Ops (build script), Christopher (review/categorization)

### Week 2: Workflow Consolidation Plan

**Tasks**:
1. **Identify Quick Wins** (4 hours)
   - Orphaned workflows (0 enrollments in 90 days): Deactivate immediately (target: 30-50 workflows)
   - Duplicate workflows: Merge or delete (target: 20 workflows)
   - Abandoned workflows (paused >180 days): Delete (target: 10 workflows)

2. **Event Workflow Consolidation** (6 hours)
   - Analyze 40+ event workflows
   - Design 1 template workflow (uses dynamic properties like `event_name`, `event_date`)
   - Test template with 3 upcoming events
   - Deprecate old single-use workflows

3. **Dependency Mapping** (4 hours)
   - Generate Mermaid diagram of critical workflow chains
   - Document high-risk workflows (deactivating would break multiple downstream workflows)

4. **Stakeholder Review** (2 hours)
   - Present findings to Tommy (CRO) and Jeremy (Head of Growth)
   - Get sign-off on deletion plan

**Deliverables**:
- ✅ 60-80 workflows deactivated (orphaned, duplicates, abandoned)
- ✅ Event workflow template designed
- ✅ Dependency map for critical workflows

**Owner**: Christopher (lead), Skydog Ops (technical execution)

### Week 3: Property Audit

**Tasks**:
1. **Property Extraction** (3 hours)
   - Fetch all custom properties via HubSpot API
   - Calculate usage count (% of contacts with property populated)
   - Cross-reference with workflows audit (which workflows use each property)

2. **Duplicate Detection** (4 hours)
   - Run automated similarity matching
   - Christopher manually reviews flagged duplicates
   - Create consolidation plan (which properties to keep, which to archive)

3. **"Primary Inbound Channel" Migration** (8 hours)
   - Backfill data (copy from old properties to `hs_analytics_source`)
   - Update 15 workflows to reference canonical property
   - Update 8 marketing reports/dashboards
   - Archive old properties (hide from UI)

**Deliverables**:
- ✅ Property audit spreadsheet (all custom properties documented)
- ✅ "Primary Inbound Channel" consolidated (1 source of truth)
- ✅ 50-100 properties archived

**Owner**: Christopher (lead), Skydog Ops (backfill script if needed)

### Week 4: Property Naming Conventions

**Tasks**:
1. **Rename Non-Compliant Properties** (6 hours)
   - Identify 20-30 high-usage properties with bad names
   - Rename to follow `{object}_{descriptor}_{type}` convention
   - Update workflows/reports that reference old names (HubSpot auto-updates most, but verify)

2. **Add Descriptions** (6 hours)
   - Fill in "Description" field for all custom properties (500+ properties)
   - Use template: DEFINITION, USAGE, OWNER, CREATED, SOURCE, EXAMPLES
   - Prioritize high-usage properties first (top 100)

3. **Document Property Governance** (2 hours)
   - Write internal wiki page: "How to Create Custom Properties"
   - Include naming conventions, description template, approval process

**Deliverables**:
- ✅ Top 100 properties renamed + documented
- ✅ Property governance documentation published

**Owner**: Christopher (primary), Skydog Ops (bulk description updates via API if feasible)

### Week 5-6: Governance Framework Implementation

**Tasks**:
1. **Workflow Approval Process** (4 hours)
   - Create "Workflow Request Form" (Google Form or Airtable Form)
   - Set up Slack/email notifications when form submitted
   - Document approval workflow in internal wiki

2. **Sunset Policy Automation** (6 hours)
   - Build Airtable automation: Weekly check for inactive workflows
   - Send email notifications to workflow owners
   - Auto-update `documentation_status` field

3. **Documentation Backfill** (10 hours)
   - For top 50 critical workflows, fill in Description field with template
   - Assign owners (Christopher interviews stakeholders to determine)
   - Set "Last Reviewed" date

4. **Training** (4 hours)
   - Train Growth team on new workflow approval process
   - Train Demand Gen hire on documentation standards
   - Q&A session

**Deliverables**:
- ✅ Workflow approval process live
- ✅ Sunset policy automated
- ✅ Top 50 workflows documented
- ✅ Team trained

**Owner**: Christopher (lead), Skydog Ops (automation build)

### Week 7-8: Monitoring & Iteration

**Tasks**:
1. **Weekly Audit Reports** (2 hours/week)
   - Re-run Python audit script
   - Generate metrics:
     - Total active workflows (target: <100)
     - % workflows documented (target: 100% of active)
     - % properties with descriptions (target: 100% of top 100)
   - Share report with Tommy and Jeremy

2. **Fix Workflow Conflicts** (8 hours)
   - Address top 10 workflow conflicts identified in audit
   - Example: Merge 3 "MQL" workflows into 1 with consolidated logic
   - Test thoroughly to avoid breaking existing automations

3. **Ongoing Governance** (ongoing)
   - Christopher reviews 5-10 new workflow requests/month
   - Sunset policy runs weekly (auto-prune inactive workflows)

**Deliverables**:
- ✅ Workflow count reduced to <100 (from 300+)
- ✅ 100% of active workflows documented
- ✅ Governance framework operational

**Owner**: Christopher (ongoing maintenance)

---

## Success Metrics

### Week 2 Targets (Audit Complete)

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| **Active Workflows** | 300+ | 250 (25% reduction) | HubSpot audit script |
| **Orphaned Workflows Identified** | Unknown | 50+ | Airtable "Orphaned" view |
| **Documented Workflows** | ~5% | 10% (30 critical workflows) | Count of workflows with filled Description field |

### Week 4 Targets (Property Cleanup Complete)

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| **Custom Properties** | ~500 | 400 (20% reduction via archival) | HubSpot properties API |
| **Properties with Descriptions** | ~30% | 100% for top 100 properties | Property audit spreadsheet |
| **"Primary Inbound Channel" Consolidation** | 3 conflicting versions | 1 canonical property | HubSpot UI (verify old properties hidden) |

### Week 8 Targets (Governance Framework Live)

| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|
| **Active Workflows** | 300+ | <100 (67% reduction) | HubSpot audit script |
| **Documented Workflows** | ~5% | 100% of active workflows | Airtable audit |
| **Workflow Approval Requests** | 0 (no process) | 10+ requests submitted via form | Airtable form submissions |
| **Sunset Policy Compliance** | 0% | 100% (all inactive workflows reviewed) | Airtable automation log |
| **Time Saved** | 10 hours/month troubleshooting | <3 hours/month (70% reduction) | Christopher time tracking |

---

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **Deleting critical workflow by mistake** | High (breaks automation, angry stakeholders) | Low | Dependency mapping before deletion, 30-day "soft delete" (pause workflow, monitor for issues before permanent delete) |
| **HubSpot API rate limits during audit** | Medium (audit script fails mid-run) | Medium | Implement exponential backoff, batch API calls, run script during off-hours |
| **Growth team resists approval process** | Medium (governance not adopted, chaos returns) | Medium | Emphasize benefits (fewer conflicts, faster troubleshooting), keep approval process fast (<24 hour turnaround) |
| **Property migration breaks reports** | Medium (marketing dashboards show wrong data) | Medium | Test all reports after property renaming, use HubSpot "Associated objects" feature to auto-update references |
| **Sunset policy deactivates needed workflow** | Medium (critical workflow stops, no one notices for weeks) | Low | 14-day grace period + email notifications to owner, Christopher manually reviews "Pending Sunset" queue weekly |

---

## Open Questions & Decisions

### Week 1 (Blocking)

1. **Audit Tool Approach**: Python script or no-code tool?
   - **Recommendation**: Python (more comprehensive, better for dependency mapping)
   - **Decision Maker**: Skydog Ops (technical feasibility) + Christopher (requirements)

2. **Workflow Deletion Authority**: Can Christopher delete workflows without stakeholder approval?
   - Option A: Christopher can delete orphaned workflows (0 activity, no dependencies) unilaterally
   - Option B: All deletions require Growth team sign-off
   - **Decision Maker**: Tommy (CRO) + Jeremy (Head of Growth)

### Week 3 (Important, Not Blocking)

3. **Property Renaming**: Should we rename high-usage properties or leave for backwards compatibility?
   - Risk: Renaming breaks external integrations (Zapier, API connections)
   - **Recommendation**: Only rename if <10 workflows/reports reference it
   - **Decision Maker**: Christopher (assess impact case-by-case)

4. **Sunset Policy**: 180 days or 90 days threshold for inactive workflow review?
   - Shorter = more aggressive pruning, higher risk of false positives
   - **Recommendation**: Start with 180 days, reduce to 90 days in Q2 2026 if working well
   - **Decision Maker**: Christopher

---

## Tools & Resources

### HubSpot API Documentation
- Workflows API: https://developers.hubspot.com/docs/api/automation/workflows
- Properties API: https://developers.hubspot.com/docs/api/crm/properties

### Python Libraries
- `hubspot-api-client`: https://github.com/HubSpot/hubspot-api-python
- `pyairtable`: https://pyairtable.readthedocs.io/

### Workflow Audit Script (Starter Template)
```python
# See Week 1 implementation roadmap for full script
import hubspot
from pyairtable import Api

client = hubspot.Client.create(access_token="YOUR_KEY")
workflows = client.automation.workflows.get_all()

# Extract metadata, apply categorization logic
# Export to Airtable + CSV
```

---

## Related Documentation

- **[01-Root Cause Analysis](01-root-cause-analysis.md)**: Detailed examples of workflow conflicts, property chaos
- **[06-Vendor SOW - Skydog](06-vendor-sow-skydog.md)**: Skydog Ops deliverables include HubSpot audit execution
- **[09-Project Roadmap](09-project-roadmap.md)**: Week-by-week timeline for audit tasks

---

**Next Steps**:
1. Week 1: Skydog Ops builds Python audit script
2. Week 1: Christopher generates HubSpot API token with workflows/properties scopes
3. Week 1: Run initial audit, export to Airtable
4. Week 2: Christopher reviews audit, categorizes workflows, presents findings to leadership

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
