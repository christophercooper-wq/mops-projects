# 06 - Vendor Statement of Work: Skydog Ops

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Budget**: $32,000 (approved)
**Timeline**: 8 weeks (December 2025 - January 2026)
**Vendor**: Skydog Ops
**Point of Contact**: Christopher Cooper (Rho) ↔ [Skydog Ops Lead - TBD]

---

## Executive Summary

Rho has approved a **$32,000 engagement with Skydog Ops** to execute the Marketing Operations Infrastructure Overhaul project over **8 weeks**. This SOW defines deliverables, success criteria, and responsibilities for a **3-phase engagement**:

1. **Phase 1 (Weeks 1-3)**: Foundation Systems - Events Automation MVP + Component Library
2. **Phase 2 (Weeks 4-6)**: Automation & Optimization - CI/CD Pipeline + HubSpot Audit
3. **Phase 3 (Weeks 7-8)**: Documentation & Handoff - Knowledge Transfer + Ongoing Support Plan

**Key Success Criteria**:
- Infrastructure ready for Q1 2026 Demand Generation hire
- Reduce Christopher's operational burden by 70% (from 30 hours/week manual work → 10 hours/week strategic work)
- Establish governance frameworks to prevent future technical debt accumulation

**Budget Allocation**:
- Phase 1: $12,000 (37.5%)
- Phase 2: $13,000 (40.6%)
- Phase 3: $7,000 (21.9%)

---

## Engagement Overview

### Problem Statement

Rho's Marketing Operations infrastructure has accumulated significant technical debt:
- **45-minute data lag** preventing real-time lead acceleration (see [01-Root Cause Analysis](01-root-cause-analysis.md))
- **300+ HubSpot workflows** with no governance, causing conflicts and errors
- **Manual event management** requiring 2-3 days per event for CSV imports
- **No email template system**, forcing Christopher to rebuild emails from scratch (2-3 hours each)
- **$100K+ annual inefficiency** from broken sync processes

**Core Issue**: Christopher Cooper (Marketing Ops, team of 1) is bottleneck for all operations, preventing strategic initiatives.

### Desired End State (Week 8)

By January 2026, Rho will have:

1. **Events Automation Platform**:
   - Luma registrations auto-sync to Airtable → HubSpot/Salesforce within 5 minutes
   - CSV normalization engine for partner events (Stripe, Mercury, etc.)
   - Self-service UI for Sales/Partnerships to upload events (80% reduction in Christopher's involvement)

2. **Email Component Library**:
   - 7+ reusable React JSX components (Header, CTA, Footer, etc.)
   - Preview site for visual QA and code extraction
   - CI/CD pipeline: GitHub commit → auto-deploy to HubSpot Design Manager

3. **HubSpot Governance**:
   - Workflow count reduced from 300+ → <100 (67% reduction)
   - 100% of active workflows documented with owner, purpose, success criteria
   - Automated sunset policy to prevent future bloat

4. **Knowledge Transfer**:
   - Comprehensive documentation for all systems
   - Skydog Ops trains Christopher + Demand Gen hire on new tools
   - Ongoing support plan (office hours, Slack access) for 30 days post-engagement

**ROI**: $32K investment → $50K+/year savings (time saved + reduced HubSpot contact bloat + faster lead routing)

---

## Phase 1: Foundation Systems (Weeks 1-3) - $12,000

**Duration**: 3 weeks (December 2-22, 2025)
**Budget**: $12,000
**Skydog Ops Hours**: ~85-90 hours

### Deliverable 1.1: Events Automation MVP (Luma Integration)

**Goal**: Automate Luma event registrations → Airtable → HubSpot flow.

**Scope**:

1. **Airtable Base Configuration** (6 hours)
   - Create "Events" table (event metadata, UTM generator, taxonomy)
   - Create "Registrations" table (normalized contact data)
   - Set up relationships (Events ← Registrations)
   - Configure views (Upcoming Events, High-Intent Registrations, Sync Errors)
   - **Acceptance Criteria**: Christopher can manually add event, see auto-generated `utm_campaign` field

2. **Luma Webhook Integration** (8 hours)
   - Set up Luma API webhook to trigger on new registration
   - Build n8n workflow: Luma webhook → Airtable insert
   - Implement dedupe logic (check email + event_id before insert)
   - Error handling (log failed inserts, Slack alert to Christopher)
   - **Acceptance Criteria**: New Luma registration appears in Airtable within 2 minutes

3. **Airtable → HubSpot Sync** (12 hours)
   - n8n workflow: New Airtable registration → HubSpot contact create/update
   - Property mapping:
     - `lifecyclestage` = "lead"
     - `hs_analytics_source` = "EVENTS"
     - `recent_event_name` = {event_name}
     - `recent_event_date` = {event_date}
     - UTM parameters (source, medium, campaign)
   - Add contact to HubSpot list: "Event Registrations - {event_name}"
   - Update Airtable: Set `sync_status` = "Synced to HubSpot", store `hubspot_contact_id`
   - **Acceptance Criteria**: Luma registration appears in HubSpot within 5 minutes, all fields populated correctly

4. **Testing & QA** (6 hours)
   - Test with 3 live Luma events
   - Verify dedupe logic (register same email twice, should skip second)
   - Verify error handling (break webhook, confirm Slack alert fires)
   - Load test (simulate 100 registrations in 1 hour, all sync correctly)
   - **Acceptance Criteria**: 100% of registrations sync successfully, no manual intervention needed

**Success Metrics**:
- ✅ 3 Luma events synced automatically (0 manual CSV imports)
- ✅ Sync latency <5 minutes (95th percentile)
- ✅ Error rate <2%
- ✅ Christopher saves 2 hours/event (6 hours/month)

**Dependencies**:
- Christopher provides Luma API credentials (Week 1, Day 1)
- Christopher provides HubSpot API token with contacts/lists scopes (Week 1, Day 1)

**Deliverables**:
- ✅ Airtable base (shared with Christopher, full edit access)
- ✅ n8n workflows (exported as JSON, stored in GitHub)
- ✅ Documentation: Setup guide, troubleshooting runbook
- ✅ Video walkthrough (10 min): How to create event in Airtable, monitor sync status

### Deliverable 1.2: Email Component Library

**Goal**: Build reusable email components with visual preview site.

**Scope**:

1. **Component Build** (16 hours)
   - Create 7 React JSX components:
     - Header.jsx (logo, date/kicker)
     - Hero.jsx (heading, subheading, background image)
     - BodyWithCTA.jsx (paragraph + CTA button with Outlook VML fallback)
     - TwoColumnCards.jsx (side-by-side content blocks)
     - Highlight.jsx (callout box)
     - PreFooter.jsx (testimonials/quotes)
     - Footer.jsx (company info, social links, legal disclaimer)
   - All components use table-based layouts, inline styles, email-safe HTML
   - **Acceptance Criteria**: Each component renders correctly in browser, extracts to valid HTML

2. **Preview Site Build** (10 hours)
   - Create `index.html` with 50/50 split layout:
     - Left: Complete email preview (all components combined, sticky scroll)
     - Right: Individual component previews with props editing
   - Implement "Copy HTML" buttons (clipboard API)
   - Add React/Babel CDN imports for in-browser JSX rendering
   - Tailwind CSS for preview UI styling
   - **Acceptance Criteria**: Christopher can adjust props (change CTA text, URL), see real-time preview, copy HTML for HubSpot

3. **Template Assembly** (6 hours)
   - Build 3 full email templates:
     - Event Invite (Header + Hero + BodyWithCTA + Footer)
     - Product Launch (Header + BodyWithCTA + TwoColumnCards + Footer)
     - Newsletter (Header + Highlight + BodyWithCTA + PreFooter + Footer)
   - Save rendered HTML to `templates/` directory (HubSpot-ready)
   - **Acceptance Criteria**: 3 templates ready for manual HubSpot upload

4. **HubSpot Deployment & Testing** (6 hours)
   - Upload 3 templates to HubSpot Design Manager
   - Send test emails to 5+ email clients (Gmail, Outlook Desktop, Outlook Web, Apple Mail, iOS Mail)
   - Fix rendering issues (Outlook button display, mobile responsiveness)
   - **Acceptance Criteria**: Templates render correctly in all major email clients

**Success Metrics**:
- ✅ 7 components built and functional
- ✅ Preview site accessible to Christopher (bookmark-able URL or localhost instructions)
- ✅ 3 email templates live in HubSpot
- ✅ Time to build email reduced from 2-3 hours → <30 minutes

**Dependencies**:
- Christopher provides Rho brand guidelines (colors, fonts, logo URL) - Week 1
- Christopher provides HubSpot Design Manager access (Week 2)

**Deliverables**:
- ✅ Component library (all `.jsx` files in `rho-hubspot-deployment/components/`)
- ✅ Preview site (`index.html`)
- ✅ 3 full email templates (HTML files + HubSpot templates)
- ✅ Documentation: Usage guide, brand guidelines reference
- ✅ Training session (1 hour): Walkthrough with Christopher + Demand Gen hire (if onboarded by Week 3)

---

## Phase 2: Automation & Optimization (Weeks 4-6) - $13,000

**Duration**: 3 weeks (December 23, 2025 - January 12, 2026)
**Budget**: $13,000
**Skydog Ops Hours**: ~90-95 hours

### Deliverable 2.1: CI/CD Pipeline for Email Templates

**Goal**: Automate email template deployment from GitHub to HubSpot.

**Scope**:

1. **GitHub Repository Setup** (3 hours)
   - Move `rho-hubspot-deployment/` to dedicated repo (or create subfolder in existing monorepo)
   - Configure branch protection (require PR reviews for main branch)
   - Add README with setup instructions
   - **Acceptance Criteria**: Repo accessible to Christopher, branch protection enabled

2. **Build Script** (8 hours)
   - Create `npm run build` script:
     - Transform JSX components → email-safe HTML
     - Use `jsx-email` library or custom Babel transform
     - Output to `dist/templates/` directory
   - Add validation step (check for broken `<img>` tags, malformed HTML)
   - **Acceptance Criteria**: Running `npm run build` generates 3 HTML templates in `dist/`

3. **Automated Testing** (10 hours)
   - Link validator: Check all `href` attributes are valid URLs (not broken links)
   - Image validator: Verify all `src` URLs return 200 status code
   - Accessibility checks: Ensure all images have alt text, sufficient color contrast (WCAG AA)
   - Add to `npm test` script
   - **Acceptance Criteria**: Test suite catches broken links, missing alt text before deployment

4. **HubSpot Deployment Script** (10 hours)
   - Create `deploy.js`:
     - Call HubSpot Design Manager API (`/content/api/v2/templates`)
     - Upload each template with version tag (e.g., `v1.0.0` based on Git commit SHA)
     - Handle errors (rate limits, authentication failures)
     - Retry logic (exponential backoff for 429 errors)
   - **Acceptance Criteria**: Script successfully uploads templates to HubSpot, logs success/errors

5. **GitHub Actions Configuration** (8 hours)
   - Create `.github/workflows/deploy-email-templates.yml`:
     - Trigger: Push to `main` branch (any changes to `components/` or `templates/`)
     - Steps: Checkout → Install deps → Build → Test → Deploy → Notify
   - Add HubSpot API key as GitHub secret
   - Add Slack webhook URL for deployment notifications
   - **Acceptance Criteria**: Commit to main → GitHub Actions runs → Templates deployed to HubSpot → Slack notification sent

6. **Testing & Iteration** (8 hours)
   - End-to-end test: Make change to component, commit to main, verify auto-deploy
   - Error scenario testing (simulate HubSpot API failure, verify retry logic)
   - Performance optimization (reduce build time, parallel processing)
   - **Acceptance Criteria**: CI/CD pipeline completes in <5 minutes, 95%+ success rate

**Success Metrics**:
- ✅ Deployment time reduced from 30 min → <5 min (commit to HubSpot)
- ✅ 100% of templates have automated tests (link validation, accessibility)
- ✅ Zero manual HubSpot uploads needed (all via GitHub Actions)

**Dependencies**:
- Christopher provides GitHub repo access for Skydog Ops (Week 4)
- Christopher configures GitHub secrets (HubSpot API key, Slack webhook) (Week 4)

**Deliverables**:
- ✅ GitHub repo with full CI/CD pipeline
- ✅ Automated test suite
- ✅ GitHub Actions workflow (live and tested)
- ✅ Documentation: How to make changes, deploy, rollback
- ✅ Video walkthrough (15 min): Dev workflow demo

### Deliverable 2.2: HubSpot Audit & Cleanup

**Goal**: Reduce HubSpot workflows from 300+ → <100, document all active workflows.

**Scope**:

1. **Audit Tool Build** (12 hours)
   - Python script to fetch all workflows via HubSpot API
   - Extract metadata: name, status, created date, owner, enrollment counts
   - Calculate `enrollments_last_90_days` (identify orphaned workflows)
   - Parse workflow definitions for dependencies (which workflows trigger others)
   - Export to CSV + Airtable "Workflows Audit" table
   - **Acceptance Criteria**: Script runs successfully, outputs CSV with 300+ workflow records

2. **Automated Categorization** (6 hours)
   - Apply categorization logic:
     - Orphaned (0 enrollments in 90 days) → Tag "Deprecated"
     - Abandoned (paused >180 days) → Tag "Deprecated"
     - High-impact (>100 current enrollments) → Tag "Critical"
     - Event workflows (name contains "Event"/"Luma") → Tag "Event Automation"
     - Duplicates (name contains "Copy of"/"Test") → Tag "Potential Duplicate"
   - Update Airtable records with tags
   - **Acceptance Criteria**: 80%+ of workflows auto-categorized

3. **Christopher Review & Approval** (10 hours, Christopher's time)
   - Christopher manually reviews top 100 workflows
   - Assigns owners (Marketing Ops, Growth, Sales)
   - Confirms which workflows to deactivate (target: 60-80 workflows)
   - **Acceptance Criteria**: Airtable "Workflows Audit" table 100% reviewed, deletion plan approved

4. **Workflow Deactivation** (8 hours)
   - Deactivate orphaned workflows (30-50 workflows)
   - Deactivate duplicate workflows (20 workflows)
   - Consolidate event workflows (40 individual workflows → 1 template workflow)
   - Monitor for 1 week (confirm no broken dependencies)
   - **Acceptance Criteria**: Active workflow count reduced to <150 (50% reduction)

5. **Documentation Backfill** (10 hours)
   - For top 50 critical workflows, fill in HubSpot "Description" field:
     - PURPOSE, OWNER, CREATED, SUCCESS CRITERIA, DEPENDENCIES, NOTES
   - Assign "Last Reviewed" date
   - **Acceptance Criteria**: 50 workflows fully documented

**Success Metrics**:
- ✅ Workflow count reduced from 300+ → <150 by Week 6 (ultimate goal: <100 by Week 8)
- ✅ 50 critical workflows documented
- ✅ Dependency map for critical workflow chains
- ✅ Zero workflow-related incidents during deactivation period

**Dependencies**:
- Christopher provides HubSpot API token with workflows/automation scopes (Week 4)
- Christopher commits 10 hours for manual review (spread across Weeks 4-5)

**Deliverables**:
- ✅ Python audit script (GitHub repo)
- ✅ Airtable "Workflows Audit" table (full inventory)
- ✅ Workflow dependency map (Mermaid diagram or Lucidchart)
- ✅ 60-80 workflows deactivated
- ✅ 50 critical workflows documented
- ✅ Documentation: HubSpot governance guide (workflow approval process, naming conventions, sunset policy)

---

## Phase 3: Documentation & Handoff (Weeks 7-8) - $7,000

**Duration**: 2 weeks (January 13-26, 2026)
**Budget**: $7,000
**Skydog Ops Hours**: ~50 hours

### Deliverable 3.1: CSV Normalization Engine (Events)

**Goal**: Enable Christopher to upload partner event CSVs with field mapping presets.

**Scope**:

1. **Field Mappings Table** (4 hours)
   - Create Airtable "Field Mappings" table
   - Add 3 preset mappings:
     - Stripe Partner Events (10 fields)
     - Mercury Co-hosted Events (10 fields)
     - Zoom Webinars (10 fields)
   - Include transformation rules (lowercase email, date format conversion, phone number formatting)
   - **Acceptance Criteria**: Christopher can view/edit presets in Airtable

2. **CSV Import Workflow** (12 hours)
   - n8n workflow: Airtable form (CSV upload) → Parse CSV → Apply mapping → Insert to Registrations
   - Implement validation (email format, required fields)
   - Dedupe logic (skip if email + event_id already exists)
   - Error logging (invalid rows saved to "Import Errors" table)
   - Trigger HubSpot sync for each new registration
   - **Acceptance Criteria**: Christopher uploads Stripe CSV, 100 registrations imported with correct field mapping

3. **Testing** (4 hours)
   - Test with 3 real partner CSVs (Stripe, Mercury, Zoom)
   - Verify transformations applied correctly (dates, emails, phones)
   - Verify dedupe logic (upload same CSV twice, no duplicates created)
   - **Acceptance Criteria**: 95%+ of rows imported successfully, <5% validation errors

**Success Metrics**:
- ✅ 3 CSV mapping presets configured
- ✅ Christopher can import 100-row CSV in <10 minutes (down from 2 hours manual)
- ✅ 95%+ import success rate

**Dependencies**:
- Christopher provides sample CSVs from 3 partners (Week 7)

**Deliverables**:
- ✅ CSV import workflow (n8n, exported to GitHub)
- ✅ 3 field mapping presets in Airtable
- ✅ Documentation: How to create new presets, troubleshoot import errors
- ✅ Video walkthrough (10 min): Upload CSV, select preset, monitor import

### Deliverable 3.2: Governance Framework Implementation

**Goal**: Establish approval processes and sunset policies to prevent future bloat.

**Scope**:

1. **Workflow Approval Process** (6 hours)
   - Create "Workflow Request Form" (Google Form or Airtable Form)
   - Fields: Name, Category, Purpose, Trigger, Actions, Owner, Success Criteria, Sunset Date
   - Set up email/Slack notifications when form submitted
   - Document approval workflow in internal wiki
   - **Acceptance Criteria**: Growth team can submit workflow request, Christopher receives notification

2. **Sunset Policy Automation** (8 hours)
   - Airtable automation:
     - Weekly: Query workflows with `last_executed` >180 days ago + `status` = "Active"
     - Send email to workflow owner: "Review needed, respond within 14 days or workflow will be deactivated"
     - If no response in 14 days, set `status` = "Pending Deactivation", notify Christopher
   - **Acceptance Criteria**: Automation runs weekly, sends notifications to correct owners

3. **Training & Handoff** (6 hours)
   - Train Christopher on governance processes (approval workflow, sunset policy)
   - Train Demand Gen hire (if onboarded) on how to request workflows
   - Q&A session
   - **Acceptance Criteria**: Team understands new processes, can submit first workflow request

**Success Metrics**:
- ✅ Workflow approval process live (5+ requests submitted via form by Week 8)
- ✅ Sunset policy automated (runs weekly without manual intervention)
- ✅ 100% of new workflows follow approval process (Week 8+)

**Dependencies**:
- Christopher provides workflow approval criteria (Week 7)

**Deliverables**:
- ✅ Workflow request form (live)
- ✅ Sunset policy automation (Airtable)
- ✅ Internal wiki page: "HubSpot Governance Guide"
- ✅ Training session (2 hours): Christopher + Growth team

### Deliverable 3.3: Comprehensive Documentation

**Goal**: Ensure Christopher can maintain all systems independently post-engagement.

**Scope**:

1. **System Documentation** (12 hours)
   - **Events Automation**: Architecture diagram, Airtable schema reference, n8n workflow logic, troubleshooting runbook
   - **Email Component Library**: Component props reference, brand guidelines, how to add new components
   - **CI/CD Pipeline**: How to deploy, rollback, debug GitHub Actions
   - **HubSpot Governance**: Workflow approval process, sunset policy, naming conventions
   - All docs written in Markdown, stored in GitHub repo + Google Drive
   - **Acceptance Criteria**: Christopher can follow docs to troubleshoot issues without Skydog Ops help

2. **Video Tutorials** (6 hours)
   - Record 5 video walkthroughs:
     1. Events Automation: Create event, upload CSV (10 min)
     2. Email Templates: Build email from components, deploy (15 min)
     3. CI/CD: Make change, commit, deploy (10 min)
     4. HubSpot Audit: Run script, review results (10 min)
     5. Governance: Submit workflow request, respond to sunset notification (10 min)
   - Upload to shared Google Drive or Loom
   - **Acceptance Criteria**: Videos clear, cover all key workflows

3. **Ongoing Support Plan** (4 hours)
   - Define 30-day post-engagement support:
     - Office hours: 2 hours/week (Skydog Ops available for Q&A via Zoom)
     - Slack channel: Skydog Ops responds to questions within 24 hours (business days)
     - Bug fixes: Critical issues fixed within 48 hours (included in $32K budget)
     - Enhancements: New features require separate SOW (out of scope)
   - Document escalation process (if Skydog Ops unavailable, who to contact)
   - **Acceptance Criteria**: Support plan documented, Christopher understands scope

**Success Metrics**:
- ✅ All systems documented (GitHub + Google Drive)
- ✅ 5 video tutorials recorded and accessible
- ✅ 30-day support plan active (office hours scheduled)

**Deliverables**:
- ✅ Comprehensive documentation (Markdown files in GitHub, PDFs in Google Drive)
- ✅ 5 video tutorials (shared via Drive/Loom)
- ✅ Support plan document (office hours schedule, Slack channel invite)

---

## Roles & Responsibilities

### Rho (Christopher Cooper)

**Responsibilities**:
- Provide API credentials (Luma, HubSpot, Airtable) within 24 hours of request
- Review/approve audit findings and deletion plans (10 hours commitment in Weeks 4-5)
- Test deliverables and provide feedback within 48 hours
- Participate in training sessions (5 hours total across 8 weeks)
- Sign off on phase completions (confirm acceptance criteria met)

**Time Commitment**: ~20 hours over 8 weeks (average 2.5 hours/week)

### Skydog Ops

**Responsibilities**:
- Deliver all scope items per agreed timeline
- Provide weekly status updates (30-min sync call every Monday)
- Respond to Christopher's questions within 24 hours (business days)
- Escalate blockers immediately (same-day Slack/email)
- Maintain code quality (documented, tested, version controlled)

**Time Commitment**: 225-235 hours over 8 weeks (average 28-30 hours/week)

### Leadership (Tommy McNulty, Jeremy Liang)

**Responsibilities**:
- Attend Week 1 kickoff meeting (Leadership Alignment, see [09-Project Roadmap](09-project-roadmap.md))
- Review Phase 1 and Phase 2 demos (30 min each)
- Approve workflow deletion plan (Week 5)
- Provide Demand Gen hire onboarding timeline (impacts training scheduling)

**Time Commitment**: ~3 hours over 8 weeks

---

## Payment Terms

### Budget Breakdown

| Phase | Deliverables | Budget | % of Total |
|-------|-------------|--------|------------|
| **Phase 1** | Events Automation MVP + Email Component Library | $12,000 | 37.5% |
| **Phase 2** | CI/CD Pipeline + HubSpot Audit | $13,000 | 40.6% |
| **Phase 3** | CSV Normalization + Governance + Documentation | $7,000 | 21.9% |
| **Total** | | **$32,000** | 100% |

### Payment Schedule

| Milestone | Payment | Due Date |
|-----------|---------|----------|
| **Contract Signing** | $8,000 (25%) | Week 0 (contract execution) |
| **Phase 1 Completion** | $10,000 (31.25%) | End of Week 3 (December 22, 2025) |
| **Phase 2 Completion** | $10,000 (31.25%) | End of Week 6 (January 12, 2026) |
| **Phase 3 Completion + Final Acceptance** | $4,000 (12.5%) | End of Week 8 (January 26, 2026) |

**Payment Terms**: Net 15 (invoices due within 15 days of milestone completion)

**Acceptance Criteria for Payments**:
- Phase payments require Christopher's sign-off that all deliverables are complete and meet acceptance criteria
- Final payment requires 30-day support plan active and all documentation delivered

---

## Success Criteria & Acceptance

### Phase 1 Acceptance Criteria

- [ ] Airtable base configured (Events + Registrations tables)
- [ ] Luma webhook connected to Airtable (tested with 3 events)
- [ ] Airtable → HubSpot sync functional (<5 min latency, <2% error rate)
- [ ] 7 email components built and functional
- [ ] Preview site accessible to Christopher
- [ ] 3 email templates deployed to HubSpot (tested in 5+ email clients)
- [ ] Documentation and training completed

**Sign-Off**: Christopher confirms all checkboxes above are complete by December 22, 2025.

### Phase 2 Acceptance Criteria

- [ ] GitHub repo with CI/CD pipeline configured
- [ ] Automated tests running (link validation, accessibility)
- [ ] GitHub Actions deploys templates to HubSpot on commit to main
- [ ] Python workflow audit script generates complete inventory (CSV + Airtable)
- [ ] 60-80 workflows deactivated (active count <150)
- [ ] 50 critical workflows documented
- [ ] Workflow dependency map completed

**Sign-Off**: Christopher confirms all checkboxes above are complete by January 12, 2026.

### Phase 3 Acceptance Criteria

- [ ] CSV import workflow functional (tested with 3 partner CSVs)
- [ ] 3 field mapping presets configured in Airtable
- [ ] Workflow approval process live (form + notifications)
- [ ] Sunset policy automated (Airtable automation running weekly)
- [ ] All documentation complete (GitHub + Google Drive)
- [ ] 5 video tutorials recorded and accessible
- [ ] 30-day support plan active (office hours scheduled, Slack channel invite sent)

**Sign-Off**: Christopher confirms all checkboxes above are complete by January 26, 2026.

---

## Risks & Contingencies

| Risk | Impact | Mitigation | Contingency Plan |
|------|--------|------------|------------------|
| **API credentials delayed** | High (blocks Week 1 start) | Christopher commits to 24-hour turnaround | Skydog Ops starts with mock data, swaps to production Week 2 |
| **HubSpot API changes mid-engagement** | Medium (breaks integrations) | Monitor HubSpot changelog, use versioned API endpoints | Allocate 4 hours buffer for API migration if needed |
| **Demand Gen hire not onboarded by Week 3** | Low (training delayed) | Schedule training for Week 8 instead | Record training session, provide async to new hire |
| **Workflow deletion breaks critical automation** | High (business disruption) | 30-day "soft delete" (pause workflow, monitor, then permanently delete) | Reactivate workflow immediately, document dependency for future |
| **Skydog Ops team member unavailable** | Medium (delays) | Skydog Ops assigns backup team member | Extend timeline by 1 week (Week 9 for Phase 3) at no additional cost |

---

## Out of Scope

The following are **explicitly excluded** from this $32K engagement:

1. **Storyblok CMS Integration** (Phase 3 from [04-Email Template Architecture](04-email-template-architecture.md))
   - Reason: Complex, requires 3-4 weeks additional dev time
   - Recommendation: Separate Q1 2026 project ($15-20K budget)

2. **Customer.io Integration** (alternative to HubSpot for event emails)
   - Reason: Requires Customer.io account setup, CDP configuration
   - Recommendation: Q1 2026 after Events Automation proven with HubSpot

3. **Self-Service Events Management UI** (full web app for Sales/Partnerships)
   - Reason: 6-8 weeks dev time (frontend + backend + auth)
   - Recommendation: Q1 2026 project, see [11-Events Management UI](11-events-management-ui.md)

4. **Salesforce Sync Optimization** (beyond HubSpot workflow cleanup)
   - Reason: Requires Anthony Hwang (RevOps) involvement, cross-team project
   - Recommendation: Separate engagement with RevOps lead

5. **Attribution Model Redesign** (fixing "Primary Inbound Channel" chaos)
   - Reason: Requires leadership alignment on attribution methodology (outside MOps scope)
   - Recommendation: Leadership workshop in Q1 2026, then technical implementation

6. **Engineering Event Emission** (40-45 minute gap fix)
   - Reason: Requires Engineering team buy-in and sprint allocation
   - Recommendation: Christopher drives separate initiative with Engineering (see [01-Root Cause](01-root-cause-analysis.md))

**Change Request Process**:
- If Christopher requests scope additions mid-engagement, Skydog Ops provides estimate (hours + cost)
- Requires written approval before proceeding
- Payment terms: 50% upfront, 50% on completion of change request

---

## Communication Plan

### Weekly Sync Calls

**Frequency**: Every Monday, 30 minutes
**Attendees**: Christopher Cooper + Skydog Ops Lead
**Agenda**:
- Review previous week's deliverables
- Demo any new functionality
- Discuss blockers/risks
- Confirm next week's priorities

**Format**: Zoom (recorded for reference)

### Async Updates

**Slack Channel**: `#skydog-mops-engagement` (private, Christopher + Skydog Ops team)
**Response SLA**: Within 24 hours (business days)
**Daily Standups**: Skydog Ops posts brief update each morning (2-3 sentences on progress)

### Phase Demos

**Timing**: End of Phase 1 (Week 3), End of Phase 2 (Week 6), End of Phase 3 (Week 8)
**Attendees**: Christopher (required), Tommy McNulty (Phase 1 & 3), Jeremy Liang (Phase 2 & 3), Demand Gen hire (Phase 3 if onboarded)
**Duration**: 60 minutes
**Format**:
- Skydog Ops live demo of deliverables (30 min)
- Q&A (20 min)
- Acceptance criteria review + sign-off (10 min)

---

## Post-Engagement Support (Weeks 9-12)

### 30-Day Support Plan

**Office Hours**:
- 2 hours/week (Tuesdays 2-4pm ET or by appointment)
- Christopher can book 30-min slots for troubleshooting, questions, guidance

**Slack Support**:
- Skydog Ops monitors `#skydog-mops-engagement` Slack channel
- Response time: 24 hours (business days)
- Scope: Bug fixes, clarifications, minor adjustments (<30 min fixes)

**Bug Fix SLA**:
- **Critical bugs** (system down, data loss): 24-hour response, 48-hour fix
- **Major bugs** (feature broken, workaround available): 48-hour response, 5-day fix
- **Minor bugs** (cosmetic, low impact): Best effort, addressed in office hours

**Enhancements**:
- New features beyond original scope require separate SOW
- Skydog Ops provides estimate (hours + cost) within 48 hours of request

**Knowledge Base**:
- Skydog Ops maintains FAQ doc (updated weekly based on Christopher's questions)
- Shared in Google Drive

---

## Appendix A: Key Documents

1. **[01-Root Cause Analysis](01-root-cause-analysis.md)**: Detailed problem statement, 45-min gap analysis
2. **[02-Organizational Assessment](02-organizational-assessment.md)**: Rho org structure, role misalignment context
3. **[03-Events Automation](03-events-automation.md)**: Full Events Automation architecture specs
4. **[04-Email Template Architecture](04-email-template-architecture.md)**: Component library + Storyblok integration roadmap
5. **[05-HubSpot Audit Requirements](05-hubspot-audit-requirements.md)**: Workflow audit methodology, governance framework
6. **[09-Project Roadmap](09-project-roadmap.md)**: 8-week timeline, task dependencies, GitHub Projects structure

---

## Appendix B: Contact Information

**Rho**:
- **Primary Contact**: Christopher Cooper, Marketing Operations
  - Email: christopher.cooper@rho.co
  - Slack: @christopher.cooper
  - Availability: Mon-Fri 9am-6pm ET

- **Leadership Stakeholders**:
  - Tommy McNulty (CRO): tommy.mcnulty@rho.co
  - Jeremy Liang (Head of Growth): jeremy.liang@rho.co
  - Anthony Hwang (RevOps Manager): anthony.hwang@rho.co

**Skydog Ops**:
- **Primary Contact**: [Skydog Ops Lead Name] (TBD at contract signing)
  - Email: TBD
  - Slack: TBD (will join Rho Slack workspace)
  - Availability: TBD

---

## Signatures

**Rho Technologies**:
- Signature: _________________________
- Name: Tommy McNulty
- Title: CRO / Head of GTM
- Date: _________________________

**Skydog Ops**:
- Signature: _________________________
- Name: _________________________
- Title: _________________________
- Date: _________________________

---

**Document Version**: 1.0
**Last Updated**: 2025-11-12
**Next Review**: Upon contract signing (Week 0)
