# 06 - Vendor SOW: Skydog Ops (REFINED - 2-3 Month Engagement)

**Document Status**: 🔄 Updated for Extended Engagement
**Last Updated**: 2025-11-12
**Budget**: $32,000 (Phase 1) + TBD (Phase 2 extension)
**Timeline**: 12 weeks (December 2025 - February 2026)
**Vendor**: Skydog Ops
**Point of Contact**: Christopher Cooper (Rho) ↔ [Skydog Ops Lead - TBD]

---

## Executive Summary

This refined SOW extends the Skydog Ops engagement from 8 weeks to **12 weeks (2-3 months)** to accommodate:
1. **Deeper HubSpot/Salesforce fixes** (not just audits, but implementation)
2. **Cross-functional coordination** with RevOps (Anthony Hwang) and potential Lifecycle Manager hire
3. **Phased rollout** allowing for Q1 2026 Demand Gen hire onboarding and training

### Three-Phase Engagement Structure

**Phase 1: Foundation & Audit** (Weeks 1-4, $12,000)
- Events Automation MVP + Email Component Library
- **HubSpot/Salesforce comprehensive audit** (not just workflow count, but root cause analysis)
- Cross-team alignment workshops (Marketing Ops, RevOps, Growth)

**Phase 2: Implementation & Optimization** (Weeks 5-8, $15,000)
- HubSpot workflow consolidation + Salesforce sync fixes
- CI/CD pipeline deployment
- **Attribution model redesign** (Primary Inbound Channel fix)
- **Action items delineated** across Marketing Ops, RevOps, Lifecycle Manager (if hired)

**Phase 3: Transition & Enablement** (Weeks 9-12, $10,000)
- Lifecycle Manager onboarding (if hired by Week 9)
- Advanced automation (CSV normalization, governance framework)
- 60-day support plan (longer runway for stability)

**Total Budget**: $37,000 (vs. original $32K - additional $5K for extended timeline)

---

## Extended Timeline Rationale

### Why 12 Weeks Instead of 8 Weeks?

**Original 8-Week Plan Constraints**:
1. **Insufficient time for Salesforce sync optimization** (originally out of scope)
2. **No buffer for Lifecycle Manager onboarding** (hire timing uncertain)
3. **Rushed governance implementation** (sunset policies need monitoring period)
4. **Limited cross-functional coordination** (RevOps involvement underscoped)

**Extended 12-Week Benefits**:
1. **Salesforce-HubSpot sync fixes included** (address 15% error rate from 01-Root Cause)
2. **Lifecycle Manager onboarding window** (Weeks 9-12 if hire completes by Feb 2026)
3. **Attribution model redesign** (fix "Primary Inbound Channel" chaos - requires leadership alignment)
4. **Longer monitoring period** (4 weeks post-implementation vs. 2 weeks)

---

## Phase 1: Foundation & Comprehensive Audit (Weeks 1-4) - $12,000

**Duration**: 4 weeks (December 2-29, 2025)
**Skydog Ops Hours**: ~85 hours

### Deliverable 1.1: Events Automation MVP (Unchanged from Original SOW)

**Goal**: Automate Luma event registrations → Airtable → HubSpot flow.

**Scope** (32 hours):
- Airtable base configuration (Events + Registrations tables)
- Luma webhook integration
- Airtable → HubSpot sync
- Testing & QA with 3 live events

**Success Metrics**:
- ✅ Sync latency <5 minutes (95th percentile)
- ✅ Error rate <2%
- ✅ Christopher saves 6 hours/month

**Dependencies**:
- Christopher provides Luma API, HubSpot API credentials (Week 1, Day 1)

**Deliverables**:
- Airtable base (shared with Christopher)
- n8n workflows (exported to GitHub)
- Documentation + 10-min video walkthrough

---

### Deliverable 1.2: Email Component Library (Unchanged from Original SOW)

**Goal**: Build reusable email components with visual preview site.

**Scope** (38 hours):
- 7 React JSX components (Header, Hero, BodyWithCTA, TwoColumnCards, Highlight, PreFooter, Footer)
- Preview site with 50/50 split layout
- 3 full email templates (Event Invite, Product Launch, Newsletter)
- HubSpot deployment + email client testing

**Success Metrics**:
- ✅ Time to build email: 2-3 hours → <30 minutes
- ✅ 3 templates live in HubSpot

**Dependencies**:
- Christopher provides Rho brand guidelines (Week 1)
- HubSpot Design Manager access (Week 2)

**Deliverables**:
- Component library (GitHub repo)
- Preview site (`index.html`)
- 3 HubSpot-ready templates
- Documentation + 1-hour training session

---

### Deliverable 1.3: HubSpot/Salesforce Comprehensive Audit (NEW - Expanded Scope)

**Goal**: Not just count workflows, but **diagnose root causes** of sync failures and workflow conflicts.

**Scope** (35 hours):

#### 1.3a: HubSpot Workflow Deep Dive (15 hours)

**Beyond Original Audit**:
- Original: Count workflows, categorize (orphaned, duplicates, etc.)
- **New**: Identify **conflicting workflow logic** causing 15% contact errors
  - Example: 3 workflows set "MQL" status with different criteria
  - Example: Event workflows triggering duplicate emails

**Methodology**:
1. **Automated Audit** (8 hours):
   - Python script: Fetch all workflows via API
   - Extract metadata: name, status, enrollment counts, last executed
   - **Parse workflow JSON** to identify:
     - Property conflicts (multiple workflows updating same property)
     - Trigger overlaps (same trigger, different actions)
     - Dependency chains (Workflow A → B → C)
   - Export to Airtable "Workflows Audit" table + Mermaid dependency diagram

2. **Manual Review Workshop** (4 hours, Christopher + Skydog):
   - Top 50 workflows: Classify as "Keep", "Consolidate", "Deprecate"
   - Identify **consolidation candidates**: 40 event workflows → 1 template
   - Document **critical dependencies**: Which workflows can't be touched

3. **Conflict Resolution Recommendations** (3 hours):
   - Specific fixes for top 10 conflicts
   - Example: "MQL Status Conflict" → Consolidate 3 workflows into 1 master workflow
   - Prioritize by business impact (which conflicts cause most errors?)

**Deliverables**:
- HubSpot Workflow Audit Report (CSV + Airtable table)
- Workflow dependency map (Mermaid diagram)
- **Conflict Resolution Plan** (prioritized list of fixes for Phase 2)

---

#### 1.3b: Salesforce-HubSpot Sync Analysis (12 hours)

**Goal**: Diagnose why **15% of contacts have sync errors** (from 01-Root Cause Analysis).

**Methodology**:
1. **Error Log Analysis** (4 hours):
   - Export HubSpot sync error logs (last 90 days)
   - Categorize errors:
     - Field mapping mismatches (HubSpot property → SFDC field type mismatch)
     - Validation rule failures (SFDC rejects contact due to missing required fields)
     - Duplicate detection issues (HubSpot tries to create duplicate SFDC record)
     - API rate limits (too many sync requests)
   - Quantify: What % of errors fall into each category?

2. **Field Mapping Audit** (5 hours):
   - Compare HubSpot contact properties → Salesforce Lead/Contact fields
   - Identify **orphaned mappings** (HubSpot property no longer exists)
   - Identify **type mismatches** (text field → picklist, causes errors)
   - Document **missing mappings** (critical SFDC fields not populated from HubSpot)

3. **Sync Configuration Review** (3 hours):
   - Review HubSpot-Salesforce connector settings:
     - Which objects sync? (Contacts only? Or Contacts + Companies + Deals?)
     - Sync frequency? (Real-time vs. batch?)
     - Duplicate detection rules?
   - Identify **configuration issues** causing errors

**Deliverables**:
- Salesforce Sync Error Report (breakdown by error type, top 10 errors)
- Field Mapping Audit Spreadsheet (HubSpot ↔ Salesforce)
- **Sync Optimization Recommendations** (specific fixes for Phase 2)

---

#### 1.3c: Attribution Model Analysis (8 hours - NEW)

**Goal**: Fix "Primary Inbound Channel" chaos (3 conflicting implementations, see 01-Root Cause).

**Methodology**:
1. **Property Inventory** (3 hours):
   - Identify all attribution-related properties:
     - `primary_inbound_channel_legacy` (8% fill rate)
     - `primary_inbound_channel_l3` (67% fill rate, contractor workaround)
     - `hs_analytics_source` (85% fill rate, HubSpot native)
   - Document which workflows/reports reference each property
   - Calculate impact: How many contacts have conflicting values?

2. **Workflow Impact Analysis** (3 hours):
   - 47 workflows reference "Primary Inbound Channel" (from 01-Root Cause)
   - Which workflows will break if we consolidate properties?
   - Which reports depend on old properties?

3. **Migration Plan** (2 hours):
   - Recommended approach:
     1. Backfill `hs_analytics_source` with data from `_legacy` and `_l3` (where missing)
     2. Update 47 workflows to reference `hs_analytics_source` only
     3. Archive old properties (don't delete - keep for historical audit)
   - Estimated effort: 10-12 hours (Phase 2 implementation)

**Deliverables**:
- Attribution Property Inventory (spreadsheet)
- Workflow/Report Impact Analysis
- **Attribution Migration Plan** (step-by-step guide for Phase 2)

---

### Week 4 Milestone: Cross-Functional Alignment Workshop

**Participants**: Christopher (Marketing Ops), Anthony (RevOps), Jeremy (Head of Growth), Skydog Ops Lead
**Duration**: 90 minutes
**Location**: Virtual (Zoom)

**Agenda**:
1. **Audit Findings Presentation** (30 min):
   - HubSpot workflow conflicts (top 10)
   - Salesforce sync errors (breakdown by type)
   - Attribution model chaos (3 properties)
   - **Recommendations for Phase 2**

2. **Responsibility Mapping** (30 min):
   - Which fixes are Marketing Ops? (Christopher)
   - Which fixes are RevOps? (Anthony)
   - Which fixes require Engineering? (Signup Services event emission)
   - **Output**: RACI chart for Phase 2 implementation

3. **Lifecycle Manager Role Discussion** (20 min):
   - IF hiring by Q1 2026, what should they own?
   - Which systems will they use? (HubSpot workflows, Airtable events, etc.)
   - What training do they need? (covered in Phase 3)

4. **Phase 2 Prioritization** (10 min):
   - Confirm priorities: HubSpot consolidation → Salesforce sync → Attribution fix
   - Agree on success criteria for Phase 2

**Deliverable**:
- Workshop summary doc (RACI chart, prioritized Phase 2 tasks, decisions log)

---

## Phase 2: Implementation & Optimization (Weeks 5-8) - $15,000

**Duration**: 4 weeks (December 30, 2025 - January 26, 2026)
**Skydog Ops Hours**: ~105 hours

### Deliverable 2.1: HubSpot Workflow Consolidation & Fixes (NEW - Expanded)

**Goal**: Reduce workflows from 300+ → <100 **AND fix top 10 conflicts** identified in Phase 1 audit.

**Scope** (40 hours):

#### 2.1a: Workflow Consolidation (20 hours)

1. **Event Workflow Templating** (10 hours):
   - Consolidate 40 event workflows → 1 master template
   - Use dynamic properties (`event_name`, `event_date`, `event_type`)
   - Test with 3 upcoming events
   - **Success Criteria**: 1 template workflow handles all future events

2. **Duplicate Removal** (5 hours):
   - Deactivate "Copy of..." and "Test" workflows (20-30 workflows)
   - Verify no downstream dependencies before deleting
   - Document which workflows were removed (audit trail)

3. **Orphaned Workflow Cleanup** (5 hours):
   - Deactivate workflows with 0 enrollments in 90 days (30-50 workflows)
   - Monitor for 1 week (confirm no broken automations)
   - Permanently delete if no issues

**Deliverable**: Active workflow count <150 (50% reduction from 300+)

---

#### 2.1b: Workflow Conflict Resolution (20 hours)

**Top 10 Conflicts from Phase 1 Audit**:

**Example Conflict 1: MQL Status Assignment**
- **Issue**: 3 workflows set `lifecyclestage` = "MQL" with conflicting logic
  - Workflow A: Form submit = "Contact Sales" → MQL
  - Workflow B: ANY form submit → MQL (too broad!)
  - Workflow C: Lifecycle = MQL, wait 3 days → SQL (why?)
- **Fix** (4 hours):
  - Consolidate into 1 master workflow with clear MQL criteria:
    - Form submit = "Contact Sales" OR "Request Demo" → MQL
    - Job title contains "CFO" OR "CEO" OR "VP" → MQL
    - Company size >100 employees → MQL
  - Deactivate Workflows B and C
  - Test with 10 sample contacts

**Example Conflict 2: Event Follow-Up Emails**
- **Issue**: 40 event workflows each send "Thank you" email
  - No consistency in email template, timing, or copy
  - Some send immediately, some wait 1 day, some wait 3 days
- **Fix** (4 hours):
  - Create 1 master event follow-up workflow (uses template from Phase 1)
  - Trigger: Contact property `recent_event_name` is known
  - Actions:
    - Wait 1 day
    - Send "Thanks for attending {event_name}" email (use dynamic property)
    - Update `contact_total_events_attended` += 1
  - Deactivate all 40 individual workflows
  - Test with 3 upcoming events

**Remaining 8 Conflicts** (12 hours):
- Similar approach: Identify root cause → Consolidate workflows → Test
- Prioritize by business impact (highest error rate first)

**Deliverable**:
- 10 workflow conflicts resolved
- Updated workflow documentation (PURPOSE, OWNER, LOGIC)
- Test results summary (% of contacts processed successfully)

---

### Deliverable 2.2: Salesforce Sync Optimization (NEW - Previously Out of Scope)

**Goal**: Reduce Salesforce sync error rate from **15% → <5%**.

**Scope** (30 hours):

#### 2.2a: Field Mapping Fixes (12 hours)

**Issue**: Type mismatches cause sync failures (from Phase 1 audit).

**Fixes**:
1. **Text → Picklist Mismatch** (6 hours):
   - Example: HubSpot `industry` (text) → SFDC `Industry` (picklist)
   - Fix: Add HubSpot workflow to validate/map to picklist values before sync
   - Test: Sync 50 contacts, verify 0 errors

2. **Missing Required Fields** (4 hours):
   - Example: SFDC requires `Company` field, but 30% of HubSpot contacts missing
   - Fix: Add HubSpot workflow to set default `Company` = "Unknown" if blank
   - Test: Sync 50 contacts with missing fields, verify auto-fill works

3. **Date Format Issues** (2 hours):
   - Example: HubSpot date format `MM/DD/YYYY`, SFDC expects `YYYY-MM-DD`
   - Fix: Configure HubSpot-Salesforce connector to transform date formats
   - Test: Sync 20 contacts with dates, verify format correct

**Deliverable**: Field mapping fixes implemented, tested, documented

---

#### 2.2b: Duplicate Detection Configuration (8 hours)

**Issue**: HubSpot creates duplicate SFDC records (same email, different SFDC IDs).

**Fix**:
1. **Enable SFDC Duplicate Rules** (3 hours):
   - Configure Salesforce duplicate rules: Block duplicates based on email
   - Test: Try to create duplicate Lead in SFDC via HubSpot sync → Should be blocked

2. **HubSpot Dedupe Logic** (5 hours):
   - Add HubSpot workflow:
     - Before syncing to SFDC, check if SFDC ID exists on contact
     - If SFDC ID exists, update existing record (don't create new)
     - If SFDC ID missing, search SFDC for email match first
   - Test: Sync 30 contacts (10 new, 10 existing, 10 duplicates), verify no duplicates created

**Deliverable**: Duplicate detection enabled, tested with sample data

---

#### 2.2c: Sync Monitoring Dashboard (10 hours)

**Goal**: Real-time visibility into sync health (prevent future 15% error rate).

**Implementation**:
1. **Airtable "Sync Monitoring" Table** (4 hours):
   - Fields: Date, Object (Contact/Company), Sync Status (Success/Error), Error Message, Record ID
   - Populate via HubSpot workflow: Log every sync attempt
   - Daily rollup: Count of successes vs. errors

2. **Slack Alerts** (3 hours):
   - If error rate >10% in any 1-hour window → Alert #mops-alerts channel
   - Daily summary: "Today: 234 contacts synced, 5 errors (2% error rate)"

3. **Weekly Report** (3 hours):
   - Automated Airtable automation: Every Monday, send Christopher email:
     - Last 7 days sync stats (total synced, error rate, top 3 errors)
     - Trend: Is error rate improving or worsening?

**Deliverable**:
- Sync monitoring dashboard (Airtable)
- Slack alerts configured
- Weekly report automation live

---

### Deliverable 2.3: Attribution Model Migration (NEW)

**Goal**: Consolidate 3 conflicting "Primary Inbound Channel" properties → 1 canonical property.

**Scope** (20 hours):

**Implementation**:
1. **Data Backfill** (8 hours):
   - For all contacts where `hs_analytics_source` is empty:
     - Copy from `primary_inbound_channel_l3` (if populated)
     - Else copy from `primary_inbound_channel_legacy` (if populated)
   - Test: Sample 100 contacts, verify backfill logic correct

2. **Workflow Updates** (10 hours):
   - 47 workflows reference old properties (from Phase 1 audit)
   - Update each workflow to reference `hs_analytics_source` only
   - Test: Run 5 high-traffic workflows, verify no errors

3. **Report Updates** (2 hours):
   - 8 marketing reports use old properties
   - Update reports to use `hs_analytics_source`
   - Verify numbers match (old vs. new)

**Deliverable**:
- Attribution consolidated to 1 property
- Old properties archived (hidden in HubSpot UI)
- Documentation: Migration summary, validation results

---

### Deliverable 2.4: CI/CD Pipeline for Email Templates (From Original SOW)

**Goal**: Automate email template deployment from GitHub → HubSpot.

**Scope** (15 hours):
- GitHub repo setup + branch protection
- Build script (JSX → HTML)
- Automated testing (link validator, image validator, accessibility)
- GitHub Actions workflow
- HubSpot deployment script

**Success Metrics**:
- ✅ Deployment time: 30 min → <5 min
- ✅ 100% of templates have automated tests

**Deliverable**:
- CI/CD pipeline live (GitHub Actions)
- Documentation + 15-min video demo

---

## Phase 3: Transition & Enablement (Weeks 9-12) - $10,000

**Duration**: 4 weeks (January 27 - February 23, 2026)
**Skydog Ops Hours**: ~70 hours

### Deliverable 3.1: Lifecycle Manager Onboarding (NEW - Conditional)

**IF** Lifecycle Manager hired by Week 9, include this deliverable. **ELSE** skip and reallocate hours to 3.2/3.3.

**Goal**: Train Lifecycle Manager on all systems built in Phases 1-2.

**Scope** (20 hours):

#### 3.1a: System Overview Training (8 hours)

**Session 1: Events Automation** (2 hours):
- How to create event in Airtable
- How to upload partner event CSV
- How to monitor sync status (Airtable → HubSpot → Salesforce)
- Troubleshooting common errors

**Session 2: Email Component Library** (2 hours):
- How to build email using components
- How to deploy via GitHub (if technical) or request from Christopher
- Brand guidelines and component props reference

**Session 3: HubSpot Workflows** (2 hours):
- How to request new workflow (approval form)
- How consolidated workflows work (event template, MQL workflow)
- Governance policies (naming conventions, documentation requirements, sunset policy)

**Session 4: Reporting & Analytics** (2 hours):
- Key dashboards (event registration trends, email performance, attribution)
- How to export data from Airtable
- How to create custom HubSpot reports

**Deliverable**:
- 4 training sessions (recorded for reference)
- Lifecycle Manager "Quick Start Guide" (PDF)

---

#### 3.1b: Handoff of Responsibilities (12 hours)

**Responsibility Mapping** (in collaboration with Christopher + Leadership):

| Responsibility | Owner (Before) | Owner (After Lifecycle Manager Hire) |
|----------------|----------------|-------------------------------------|
| **Event Creation** | Christopher (100%) | Lifecycle Manager (80%), Christopher (20% oversight) |
| **Event CSV Uploads** | Christopher (100%) | Lifecycle Manager (90%), Christopher (10% QA) |
| **Email Template Creation** | Christopher (100%) | Lifecycle Manager (60%), Christopher (40% - complex emails) |
| **HubSpot Workflow Creation** | Christopher (100%) | Lifecycle Manager (50% - requests), Christopher (50% - approvals/builds) |
| **Reporting & Analytics** | Christopher (100%) | Lifecycle Manager (70%), Christopher (30% - strategic) |
| **System Troubleshooting** | Christopher (100%) | Christopher (100% - Lifecycle Manager escalates) |

**Handoff Process**:
1. **Week 9**: Lifecycle Manager shadows Christopher (observes all tasks)
2. **Week 10**: Lifecycle Manager performs tasks with Christopher's review
3. **Week 11**: Lifecycle Manager performs tasks independently, Christopher spot-checks
4. **Week 12**: Full handoff, Christopher available for questions

**Deliverable**:
- RACI chart (updated with Lifecycle Manager responsibilities)
- Handoff checklist (task-by-task transfer of knowledge)

---

### Deliverable 3.2: Advanced Automation (From Original SOW + Enhancements)

**Goal**: CSV normalization engine + governance framework.

**Scope** (30 hours):

#### 3.2a: CSV Normalization Engine (20 hours)

**Unchanged from Original SOW**:
- Airtable "Field Mappings" table
- 3 preset mappings (Stripe, Mercury, Zoom)
- n8n workflow: CSV upload → Parse → Transform → Insert to Registrations
- Validation, dedupe, error logging

**New Enhancement** (included in 20 hours):
- **Self-Service UI Prototype** (simple Airtable form):
  - Sales/Partnerships can upload CSV via form (no Christopher involvement)
  - Form fields: Select Event, Upload CSV, Select Mapping Preset
  - Automated workflow processes upload, sends confirmation email
  - Caveat: Full web app (from 11-Events Management UI) still Q2 2026 project

**Deliverable**:
- CSV normalization engine functional
- Self-service Airtable form (basic version)
- Documentation + 10-min video walkthrough

---

#### 3.2b: Governance Framework (10 hours)

**Unchanged from Original SOW**:
- Workflow Request Form (Google Form or Airtable)
- Sunset Policy Automation (Airtable: Weekly check for inactive workflows)
- Training session (2 hours with Christopher + Growth team)

**Deliverable**:
- Workflow approval process live
- Sunset policy automated
- Internal wiki page: "HubSpot Governance Guide"

---

### Deliverable 3.3: Comprehensive Documentation & 60-Day Support Plan (Enhanced)

**Goal**: Ensure long-term system stability post-engagement.

**Scope** (20 hours):

#### 3.3a: Documentation (12 hours)

**Unchanged from Original SOW**:
- System documentation (Events Automation, Email Library, CI/CD, HubSpot Governance)
- 5 video tutorials (10-15 min each)
- All docs in Markdown (GitHub) + PDF (Google Drive)

**New Addition**:
- **Runbooks** (step-by-step troubleshooting guides):
  - "What to do if Luma webhook stops syncing"
  - "How to rollback a broken email template deployment"
  - "How to fix HubSpot-Salesforce sync error"
  - "How to investigate duplicate contact creation"

**Deliverable**:
- Comprehensive documentation (GitHub + Drive)
- 5 video tutorials + 4 troubleshooting runbooks

---

#### 3.3b: 60-Day Support Plan (8 hours of Skydog time allocation)

**Extended from 30 days → 60 days** (Weeks 9-16, through end of March 2026):

**Office Hours**:
- 2 hours/week for Weeks 9-12 (during active Phase 3 work)
- 1 hour/week for Weeks 13-16 (monitoring period)

**Slack Support**:
- Response SLA: 24 hours (business days) for Weeks 9-12
- Response SLA: 48 hours (business days) for Weeks 13-16 (tapering)

**Bug Fix SLA** (unchanged from original):
- Critical: 24-hour response, 48-hour fix
- Major: 48-hour response, 5-day fix
- Minor: Best effort, addressed in office hours

**Monthly Health Check** (new):
- End of Week 12, Week 16: 30-min call to review:
  - System health metrics (sync error rates, workflow performance)
  - Issues encountered (and how resolved)
  - Recommendations for ongoing maintenance

**Deliverable**:
- Support plan document (SLAs, escalation process)
- 2 monthly health check reports

---

## Delineation of Action Items Across Teams

### Marketing Ops (Christopher Cooper)

**Phase 1 (Weeks 1-4)**:
- [ ] Provide API credentials (Luma, HubSpot, Airtable) - Week 1, Day 1
- [ ] Provide brand guidelines for email templates - Week 1
- [ ] Review/approve audit findings - Week 4
- [ ] Participate in cross-functional alignment workshop - Week 4

**Phase 2 (Weeks 5-8)**:
- [ ] Test workflow consolidation (sample contacts) - Week 5-6
- [ ] Review/approve workflow conflict fixes - Week 6
- [ ] Test CI/CD pipeline deployment - Week 7
- [ ] Sign off on Phase 2 completion - Week 8

**Phase 3 (Weeks 9-12)**:
- [ ] Train Lifecycle Manager (if hired) - Weeks 9-10
- [ ] Test CSV normalization engine - Week 9
- [ ] Review documentation and runbooks - Week 11
- [ ] Final acceptance sign-off - Week 12

**Time Commitment**: ~30 hours over 12 weeks (avg 2.5 hours/week)

---

### RevOps (Anthony Hwang)

**Phase 1 (Weeks 1-4)**:
- [ ] Provide Salesforce credentials for sync audit - Week 1
- [ ] Review Salesforce sync error analysis - Week 3
- [ ] Participate in cross-functional alignment workshop - Week 4
- [ ] Approve Phase 2 Salesforce optimization priorities - Week 4

**Phase 2 (Weeks 5-8)**:
- [ ] Collaborate on field mapping fixes (HubSpot ↔ Salesforce) - Week 5
- [ ] Configure Salesforce duplicate detection rules - Week 6
- [ ] Test sync optimization (sample leads) - Week 7
- [ ] Review sync monitoring dashboard - Week 8

**Phase 3 (Weeks 9-12)**:
- [ ] Train Lifecycle Manager on Salesforce workflows (if hired) - Week 10
- [ ] Document Salesforce-specific processes - Week 11

**Time Commitment**: ~15 hours over 12 weeks (avg 1.25 hours/week)

---

### Growth / Leadership (Jeremy Liang, Tommy McNulty)

**Phase 1 (Weeks 1-4)**:
- [ ] Attend Phase 1 kickoff meeting - Week 1
- [ ] Participate in cross-functional alignment workshop - Week 4
- [ ] Provide input on Lifecycle Manager role scope (if hiring) - Week 4

**Phase 2 (Weeks 5-8)**:
- [ ] Review workflow consolidation plan - Week 5
- [ ] Approve attribution model migration approach - Week 6
- [ ] Attend Phase 2 demo - Week 8

**Phase 3 (Weeks 9-12)**:
- [ ] Onboard Lifecycle Manager (if hired) - Week 9
- [ ] Define Lifecycle Manager responsibilities (RACI chart) - Week 9
- [ ] Attend Phase 3 demo and final acceptance - Week 12

**Time Commitment**: ~5 hours over 12 weeks (avg 25 min/week)

---

### Lifecycle Manager (If Hired by Week 9)

**Phase 3 (Weeks 9-12)**:
- [ ] Complete 4 training sessions (Events, Email, Workflows, Reporting) - Weeks 9-10
- [ ] Shadow Christopher on all MOps tasks - Week 9
- [ ] Perform tasks with Christopher's review - Week 10
- [ ] Perform tasks independently (Christopher spot-checks) - Week 11
- [ ] Full handoff, independent ownership - Week 12

**Responsibilities Post-Week 12** (from RACI chart):
- **Own**: Event creation (80%), CSV uploads (90%), email creation (60%), reporting (70%)
- **Collaborate**: Workflow requests (50% submit, Christopher approves/builds)
- **Escalate**: System troubleshooting (Christopher handles all technical issues)

**Time Commitment**: ~30 hours over 4 weeks (Weeks 9-12 onboarding)

---

### Skydog Ops

**Phase 1**: 85 hours (audit, events automation, email library)
**Phase 2**: 105 hours (HubSpot fixes, Salesforce optimization, attribution migration, CI/CD)
**Phase 3**: 70 hours (Lifecycle Manager training, CSV normalization, governance, documentation)

**Total**: 260 hours over 12 weeks (avg 21.7 hours/week)

---

## Updated Budget & Payment Schedule

### Budget Breakdown

| Phase | Deliverables | Original Budget | Revised Budget | Delta |
|-------|-------------|-----------------|----------------|-------|
| **Phase 1 (Weeks 1-4)** | Events Automation + Email Library + **Comprehensive Audit** | $12,000 | $12,000 | $0 |
| **Phase 2 (Weeks 5-8)** | HubSpot Consolidation + **Salesforce Optimization** + **Attribution Fix** + CI/CD | $13,000 | $15,000 | **+$2,000** |
| **Phase 3 (Weeks 9-12)** | **Lifecycle Manager Onboarding** + CSV Normalization + Governance + **60-Day Support** | $7,000 | $10,000 | **+$3,000** |
| **Total** | | **$32,000** | **$37,000** | **+$5,000** |

### Payment Schedule

| Milestone | Payment | Due Date |
|-----------|---------|----------|
| **Contract Signing** | $8,000 (21.6%) | Week 0 |
| **Phase 1 Completion** | $10,000 (27%) | End of Week 4 (Dec 29, 2025) |
| **Phase 2 Completion** | $12,000 (32.4%) | End of Week 8 (Jan 26, 2026) |
| **Phase 3 Completion** | $7,000 (19%) | End of Week 12 (Feb 23, 2026) |

**Total**: $37,000

**Payment Terms**: Net 15 (invoices due within 15 days of milestone completion)

---

## Success Criteria (Updated)

### Phase 1 (Week 4) - Acceptance Criteria

- [ ] Events Automation MVP live (3 events synced, <5 min latency, <2% error rate)
- [ ] Email Component Library complete (7 components, 3 templates in HubSpot)
- [ ] **HubSpot Workflow Audit complete** (CSV + Airtable + dependency map)
- [ ] **Salesforce Sync Audit complete** (error breakdown + field mapping audit)
- [ ] **Attribution Analysis complete** (3 properties inventoried, migration plan)
- [ ] **Cross-functional alignment workshop conducted** (RACI chart, Phase 2 priorities)

**Sign-Off**: Christopher + Anthony confirm all criteria met by December 29, 2025.

---

### Phase 2 (Week 8) - Acceptance Criteria

- [ ] HubSpot workflows reduced to <150 (50% reduction from 300+)
- [ ] **Top 10 workflow conflicts resolved** (tested with sample contacts)
- [ ] **Salesforce sync error rate <5%** (down from 15%)
- [ ] **Field mapping fixes implemented** (text→picklist, missing fields, dates)
- [ ] **Duplicate detection enabled** (tested with sample data)
- [ ] **Sync monitoring dashboard live** (Airtable + Slack alerts)
- [ ] **Attribution migrated to 1 canonical property** (47 workflows updated, 8 reports updated)
- [ ] CI/CD pipeline deployed (GitHub Actions, automated tests)

**Sign-Off**: Christopher + Anthony confirm all criteria met by January 26, 2026.

---

### Phase 3 (Week 12) - Acceptance Criteria

- [ ] **Lifecycle Manager trained** (4 sessions, handoff complete) - IF hired
- [ ] CSV normalization engine functional (3 presets, self-service form)
- [ ] Governance framework live (workflow approval form, sunset policy automated)
- [ ] Comprehensive documentation complete (GitHub + Drive + 4 runbooks)
- [ ] 5 video tutorials recorded
- [ ] **60-day support plan active** (office hours scheduled, Slack channel live)

**Sign-Off**: Christopher confirms all criteria met by February 23, 2026.

---

## Key Changes from Original SOW

### What's New?

1. **Salesforce Sync Optimization** (was out of scope):
   - Field mapping fixes, duplicate detection, sync monitoring dashboard
   - **Rationale**: 15% error rate is too high, can't defer to separate project

2. **Attribution Model Migration** (was out of scope):
   - Consolidate 3 conflicting properties → 1 canonical
   - Update 47 workflows, 8 reports
   - **Rationale**: Attribution chaos blocks accurate ROI reporting

3. **Lifecycle Manager Onboarding** (new deliverable):
   - 4 training sessions, RACI handoff, 4-week shadowing period
   - **Rationale**: If hire completes by Feb 2026, need structured onboarding

4. **Extended Support** (30 days → 60 days):
   - Longer monitoring period ensures stability
   - Monthly health checks (Week 12, Week 16)
   - **Rationale**: Complex systems need longer stabilization period

5. **Cross-Functional Alignment Workshop** (new):
   - Week 4 workshop with Marketing Ops, RevOps, Growth leadership
   - Output: RACI chart, Phase 2 priorities, responsibility mapping
   - **Rationale**: Ensures buy-in and clear ownership across teams

### What's Unchanged?

- Events Automation MVP (Luma integration)
- Email Component Library (7 components, preview site)
- CI/CD Pipeline (GitHub Actions)
- CSV Normalization Engine (3 presets)
- Governance Framework (approval form, sunset policy)
- Core documentation and video tutorials

---

## Risks & Contingencies (Updated)

| Risk | Impact | Probability | Mitigation | Contingency |
|------|--------|-------------|------------|-------------|
| **Lifecycle Manager not hired by Week 9** | Low (training skipped, hours reallocated) | Medium | Confirm hiring timeline by Week 4 | Reallocate 20 hours to extended documentation/support |
| **Salesforce sync fixes require SFDC admin changes** | Medium (delays if Anthony unavailable) | Medium | Anthony commits 15 hours, confirm availability Week 1 | Skydog Ops provides step-by-step guide, Anthony implements in Week 9-10 |
| **Attribution migration breaks reports** | High (leadership can't track ROI) | Low | Test on sample data first, validate numbers match old vs. new | Rollback migration, use hybrid approach (both old + new properties live) |
| **HubSpot workflow consolidation breaks automations** | High (leads not routed, emails not sent) | Medium | 30-day soft delete (pause → monitor → delete), comprehensive testing | Reactivate workflow immediately, document dependency for future |
| **Skydog Ops extends timeline due to complexity** | Medium (delays Q1 2026 hire onboarding) | Low | Weekly sync calls catch delays early, re-prioritize if needed | Extend Phase 3 by 1-2 weeks (Week 13-14) at no additional cost |

---

## Next Steps (Immediate Actions)

### Week 0 (Contract Signing - Target: December 2, 2025)

- [ ] **Leadership Approval**: Tommy McNulty signs SOW
- [ ] **Skydog Ops Contract**: Finalize vendor agreement, identify lead consultant
- [ ] **Kickoff Scheduled**: Week 1 kickoff meeting (Christopher, Skydog, Anthony, Jeremy)
- [ ] **API Credentials Prepared**: Christopher generates all necessary tokens

### Week 1 (Foundation Setup)

- [ ] **Kickoff Meeting**: Intros, timeline review, API credential handoff
- [ ] **Audit Begins**: Skydog Ops starts HubSpot/Salesforce data extraction
- [ ] **Events Automation**: Airtable base configuration starts
- [ ] **Email Library**: Component build begins

---

**Document Version**: 2.0 (Refined for 12-Week Engagement)
**Last Updated**: 2025-11-12
**Next Review**: Upon contract signing (Week 0)
**Maintained In**: GitHub (`mops-projects` repo) + Google Drive (MOps Infrastructure Project folder)
