# 03 - Events Automation Architecture

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Related Documents**: [01-Root Cause](01-root-cause-analysis.md), [08-Technical Architecture](08-technical-architecture.md), [11-Events Management UI](11-events-management-ui.md)

---

## Executive Summary

Rho's current events management relies on manual CSV exports from Luma and ad-hoc HubSpot imports, creating:
- **Data Quality Issues**: Inconsistent field mapping, missing UTM parameters, manual errors
- **Delayed Follow-Up**: 24-72 hour lag between event registration and CRM visibility
- **Limited Scalability**: Cannot support partner events, sales roundtables, or non-Luma sources
- **Attribution Gaps**: No consistent event taxonomy preventing ROI analysis

**Proposed Solution**: Source-agnostic Events Automation Platform with:
1. **Airtable Event Registry** - Central source of truth for all events (Luma, partners, internal)
2. **Automated Field Mapping** - CSV normalization engine similar to HubSpot's list import
3. **Real-Time Sync** - Webhook-driven updates to HubSpot/Salesforce (<5 min)
4. **Self-Service UI** - Allow Sales/Partnerships to upload event CSVs without MOps involvement

**Timeline**: Week 1-2 MVP (Luma only) → Week 3-4 CSV normalization → Week 5-6 UI build

---

## Problem Statement

### Current State Issues

#### 1. Manual Process Dependencies
```
Current Workflow (2-3 days per event):
Luma Event Created
    ↓ [MANUAL: Christopher exports CSV]
HubSpot Import Tool
    ↓ [MANUAL: Map 15+ fields every time]
Dedupe/Enrichment
    ↓ [MANUAL: Tag with campaign, create list]
Sales Notification
    ↓ [MANUAL: Email or Slack alert]
Follow-Up Workflows
```

**Pain Points**:
- Christopher is single point of failure for all event imports
- Field mapping errors common (wrong date format, missing UTMs)
- No standardized taxonomy (event names, types, tiers)
- Cannot handle non-Luma events (partner CSVs, sales dinners)

#### 2. Data Quality Problems

| Issue | Impact | Frequency |
|-------|--------|-----------|
| Missing UTM parameters | Attribution gaps, cannot measure event ROI | 40% of imports |
| Inconsistent event naming | Reporting fragmentation, duplicate campaigns | 60% of events |
| Wrong date formats | Failed workflows, incorrect segmentation | 25% of imports |
| Duplicate contacts | Inflated event counts, skewed analytics | 15% of registrations |
| Missing company enrichment | Sales cannot prioritize outreach | 30% of contacts |

#### 3. Scalability Constraints

**Current Volume**:
- 8-12 Luma events/month (manageable manually)

**Q1 2026 Projected Volume** (with Demand Gen hire):
- 20-30 Luma events/month
- 10-15 partner events/month (Stripe, Ramp, Mercury)
- 5-10 sales roundtables/month (internal)
- **Total: 35-55 events/month** (~2 events/day)

**Manual Process Will Break**: 2 hours/day just importing CSVs, no time for strategy/analysis.

---

## Proposed Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    EVENT SOURCES                             │
├─────────────────────────────────────────────────────────────┤
│ • Luma API (webhook)                                         │
│ • Partner CSVs (manual upload)                               │
│ • Sales roundtables (Google Forms)                           │
│ • Webinars (Zoom/Goldcast)                                   │
└─────────────────────┬───────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────────┐
│           AIRTABLE EVENT REGISTRY (Source of Truth)          │
├─────────────────────────────────────────────────────────────┤
│ Tables:                                                       │
│ • Events (metadata, taxonomy, UTM generator)                 │
│ • Registrations (normalized contact data)                    │
│ • Field Mappings (CSV → standard schema)                     │
└─────────────────────┬───────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────────┐
│              TRANSFORMATION LAYER (n8n/Zapier)               │
├─────────────────────────────────────────────────────────────┤
│ • Dedupe logic (email + event_id)                            │
│ • Enrichment (Clearbit/ZoomInfo)                             │
│ • Field normalization (dates, phones, UTMs)                  │
│ • HubSpot property mapping                                   │
└─────────────────────┬───────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────────────┐
│                 CRM SYSTEMS (Distribution)                   │
├─────────────────────────────────────────────────────────────┤
│ • HubSpot (marketing workflows, email nurture)               │
│ • Salesforce (qualified accounts only, sales alerts)         │
│ • Amplitude/PostHog (event analytics)                        │
│ • Slack (Sales notifications for high-intent registrations)  │
└─────────────────────────────────────────────────────────────┘
```

---

## Airtable Event Registry Schema

### Table 1: Events

**Purpose**: Central catalog of all Rho events with standardized taxonomy and metadata.

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `event_id` | Auto-number | Unique identifier | `EV-2025-001` |
| `event_name` | Single-line text | Official event name | "CFO Roundtable: Banking Stack Optimization" |
| `event_type` | Single-select | Category | `Luma Event`, `Partner Event`, `Internal Roundtable`, `Webinar` |
| `event_tier` | Single-select | Priority level | `Tier 1 (Flagship)`, `Tier 2 (Strategic)`, `Tier 3 (Tactical)` |
| `event_date` | Date | Event start date/time | `2025-12-15 18:00` |
| `event_location` | Single-line text | Venue or virtual | "Rho Office - 100 Crosby St, NY" |
| `host` | Single-select | Owning team | `Growth`, `Sales`, `Partnerships`, `Customer Success` |
| `luma_url` | URL | Luma event page (if applicable) | `https://lu.ma/rho-cfo-dec-2025` |
| `registration_count` | Rollup | Count from Registrations table | `42` |
| `attendance_count` | Number | Actual attendees (manual entry) | `38` |
| `requires_nurture` | Checkbox | Enable automated follow-up sequence | ☑ |
| `utm_campaign` | Formula | Auto-generated | `event_cfo-roundtable_2025-12-15` |
| `utm_source` | Single-select | Traffic source | `luma`, `partner_stripe`, `direct` |
| `utm_medium` | Single-select | Channel | `event`, `webinar`, `roundtable` |
| `hubspot_campaign_id` | Number | HubSpot campaign ID (after creation) | `123456789` |
| `salesforce_campaign_id` | Single-line text | SFDC campaign ID | `701XX000000XXXX` |
| `status` | Single-select | Event lifecycle | `Planned`, `Registration Open`, `Completed`, `Cancelled` |
| `notes` | Long text | Internal notes | "Co-hosted with Stripe, need custom email template" |

**Formula for `utm_campaign`**:
```
"event_" &
LOWER(SUBSTITUTE({event_name}, " ", "-")) &
"_" &
DATETIME_FORMAT({event_date}, 'YYYY-MM-DD')
```

### Table 2: Registrations

**Purpose**: Normalized contact data for all event registrations across all sources.

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `registration_id` | Auto-number | Unique identifier | `REG-2025-001234` |
| `event_id` | Link to Events | Related event | `EV-2025-001` |
| `email` | Email | Contact email (dedupe key) | `jane.doe@acme.com` |
| `first_name` | Single-line text | Contact first name | `Jane` |
| `last_name` | Single-line text | Contact last name | `Doe` |
| `company` | Single-line text | Company name | `Acme Corp` |
| `job_title` | Single-line text | Job title | `CFO` |
| `phone` | Phone | Contact phone | `+1 (555) 123-4567` |
| `registration_date` | Date/time | When they registered | `2025-12-01 14:32` |
| `registration_source` | Single-select | Where registration came from | `Luma API`, `CSV Upload`, `Webinar Platform` |
| `attendance_status` | Single-select | Did they show up | `Registered`, `Attended`, `No-Show`, `Cancelled` |
| `hubspot_contact_id` | Number | HubSpot contact vid | `98765432` |
| `salesforce_lead_id` | Single-line text | SFDC Lead/Contact ID | `00QXX000000XXXX` |
| `sync_status` | Single-select | Integration status | `Pending`, `Synced to HubSpot`, `Synced to SFDC`, `Error` |
| `sync_error` | Long text | Error message if sync failed | `"HubSpot API rate limit exceeded"` |
| `enrichment_status` | Single-select | Data enrichment status | `Not Enriched`, `Enriched`, `Failed` |
| `enriched_company_size` | Number | Employee count (from Clearbit) | `250` |
| `enriched_revenue` | Currency | Annual revenue (from ZoomInfo) | `$50,000,000` |
| `utm_source` | Lookup | From Events table | `luma` |
| `utm_medium` | Lookup | From Events table | `event` |
| `utm_campaign` | Lookup | From Events table | `event_cfo-roundtable_2025-12-15` |

**Key Features**:
- **Dedupe Logic**: Before insert, check if `email` + `event_id` already exists
- **Auto-Enrichment**: Trigger Clearbit/ZoomInfo lookup on new registration
- **Sync Tracking**: Monitor HubSpot/SFDC sync status, retry on errors
- **Attendance Tracking**: Update `attendance_status` post-event (manual or via check-in app)

### Table 3: Field Mappings

**Purpose**: Define how to map CSV columns from different sources to standard Registrations schema.

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `mapping_id` | Auto-number | Unique identifier | `MAP-001` |
| `source_name` | Single-line text | Name of CSV source | `Stripe Partner Events` |
| `source_column` | Single-line text | Column name in source CSV | `attendee_email_address` |
| `target_field` | Single-select | Field in Registrations table | `email` |
| `transformation` | Single-select | Data transformation rule | `None`, `Lowercase`, `Date Format: MM/DD/YYYY → ISO`, `Phone Format: E.164` |
| `default_value` | Single-line text | Fallback if source column empty | `Unknown` |
| `required` | Checkbox | Must be present in CSV | ☑ |
| `notes` | Long text | Mapping notes | "Stripe always sends emails in uppercase, need to lowercase" |

**Example Mappings**:

| Source (Stripe Partner CSV) | Target (Registrations) | Transformation |
|-----------------------------|------------------------|----------------|
| `attendee_email_address` | `email` | Lowercase |
| `attendee_first_name` | `first_name` | None |
| `attendee_last_name` | `last_name` | None |
| `company_name` | `company` | None |
| `title` | `job_title` | None |
| `registration_timestamp` | `registration_date` | Date Format: MM/DD/YYYY HH:mm → ISO 8601 |
| (not in CSV) | `registration_source` | Default: "CSV Upload - Stripe" |

---

## Source-Agnostic CSV Normalization

### Requirements

User quote:
> "Not all CSVs will be from Luma, but the properties/value need to be normalized – agnostic from source/formating – akin to field mapping on HubSpot / standard marketing platform list import"

### Approach: Smart Mapping with Presets

#### Phase 1: Manual Mapping UI (Week 3-4)

When Christopher uploads a CSV from a new source (e.g., Mercury partner event):

1. **Upload CSV**: Drag-and-drop interface shows column preview
2. **Auto-Detect**: System suggests mappings based on column names
   - `email`, `e-mail`, `email_address` → `email`
   - `first`, `fname`, `first_name` → `first_name`
   - `company`, `company_name`, `organization` → `company`
3. **Manual Override**: Christopher confirms/adjusts mappings
4. **Save Preset**: Store mapping as "Mercury Partner Events" for future reuse
5. **Transform & Import**: Apply transformations, validate, insert into Registrations table

#### Phase 2: Automated Mapping (Week 5-6)

Once presets exist:
1. Sales/Partnerships upload CSV via self-service UI
2. System prompts: "Is this from a known source?" → Select `Stripe Partner Events`
3. Apply saved mapping automatically
4. Show preview → Confirm → Import

**Fallback**: If unknown source, require Christopher approval or provide manual mapping wizard.

### Validation Rules

Before inserting into Registrations table:

| Field | Validation | Error Handling |
|-------|------------|----------------|
| `email` | Must match regex: `^[^\s@]+@[^\s@]+\.[^\s@]+$` | Reject row, log error |
| `first_name` | Not empty | Use "Unknown" if missing |
| `last_name` | Not empty | Use "Unknown" if missing |
| `company` | Not empty | Use "Not Provided" if missing |
| `registration_date` | Valid ISO 8601 date | Use upload timestamp if invalid |
| `phone` | Valid E.164 format or empty | Strip formatting, validate, or leave blank |

**Dedupe Check**: Query Registrations table for existing `email` + `event_id` → Skip if exists.

---

## Integration Flows

### Flow 1: Luma → Airtable → HubSpot (Automated)

**Trigger**: Luma webhook fires on new registration

**Steps**:
1. **Luma Webhook** → POST to n8n endpoint
   ```json
   {
     "event_name": "CFO Roundtable: Banking Stack",
     "event_date": "2025-12-15T18:00:00Z",
     "attendee": {
       "email": "jane.doe@acme.com",
       "first_name": "Jane",
       "last_name": "Doe",
       "company": "Acme Corp"
     }
   }
   ```

2. **n8n: Create/Update Event in Airtable**
   - Check if event exists (by `luma_url` or `event_name` + `event_date`)
   - If not exists, create new Event record with auto-generated `utm_campaign`
   - If exists, skip

3. **n8n: Create Registration in Airtable**
   - Check for duplicate (`email` + `event_id`)
   - Insert new Registrations record with:
     - `registration_source` = "Luma API"
     - `sync_status` = "Pending"
     - `utm_source`, `utm_medium`, `utm_campaign` (lookup from Events)

4. **n8n: Enrich Contact**
   - Call Clearbit Enrichment API with email
   - Update `enriched_company_size`, `enriched_revenue` if found
   - Set `enrichment_status` = "Enriched" or "Failed"

5. **n8n: Sync to HubSpot**
   - Search HubSpot for contact by email
   - If exists: Update properties
   - If not exists: Create new contact
   - Set HubSpot properties:
     ```
     lifecyclestage: "lead"
     hs_analytics_source: "EVENTS"
     recent_event_name: {event_name}
     recent_event_date: {event_date}
     utm_source: {utm_source}
     utm_medium: {utm_medium}
     utm_campaign: {utm_campaign}
     ```
   - Add contact to HubSpot list: "Event Registrations - {event_name}"
   - Update Airtable: `sync_status` = "Synced to HubSpot", store `hubspot_contact_id`

6. **n8n: Conditional SFDC Sync**
   - If `enriched_company_size` > 100 OR contact already exists in SFDC:
     - Create/update Salesforce Lead
     - Add to SFDC Campaign
     - Update Airtable: `sync_status` = "Synced to SFDC", store `salesforce_lead_id`

7. **n8n: Sales Notification**
   - If high-intent (e.g., CFO title, company size >500):
     - POST to Slack webhook: #sales-events channel
     - Message: "🎯 High-intent registration: Jane Doe (CFO, Acme Corp - 500 employees) for CFO Roundtable 12/15"

**Timeline**: <5 minutes from Luma registration to HubSpot visibility.

### Flow 2: CSV Upload → Airtable → HubSpot (Semi-Automated)

**Trigger**: Christopher uploads CSV via Airtable form or self-service UI

**Steps**:
1. **Upload CSV**: Drag-and-drop to Airtable form or custom UI
2. **Select Event**: Link to existing Event record or create new
3. **Select Mapping Preset**: Choose from saved Field Mappings (or manual map if new source)
4. **Preview**: Show 5 sample rows with applied transformations
5. **Confirm Import**: Christopher approves
6. **n8n: Bulk Insert**
   - Parse CSV, apply transformations
   - Dedupe check (skip if email + event_id exists)
   - Insert into Registrations table (batch of 100 records at a time)
   - Set `registration_source` = "CSV Upload - {source_name}"
7. **n8n: Trigger Enrichment & Sync Flows**
   - For each new registration, follow steps 4-7 from Flow 1
   - Process in batches to avoid API rate limits

**Timeline**: 10-30 minutes for 100-500 registrations (depending on enrichment API speed).

---

## Event Taxonomy & Naming Conventions

### Event Types

| Type | Description | Example | UTM Medium |
|------|-------------|---------|------------|
| **Luma Event** | Public events hosted on Luma | "CFO Roundtable: Banking Stack" | `event` |
| **Partner Event** | Co-hosted with partners (Stripe, Ramp) | "Rho x Stripe: Fintech CFO Panel" | `partner_event` |
| **Internal Roundtable** | Sales-hosted, invite-only dinners | "Enterprise CFO Dinner - Boston" | `roundtable` |
| **Webinar** | Virtual presentations (Zoom, Goldcast) | "Webinar: Cash Management 101" | `webinar` |

### Event Tiers

| Tier | Definition | Registration Goal | Sales Involvement | Example |
|------|------------|-------------------|-------------------|---------|
| **Tier 1 (Flagship)** | Quarterly tentpole events, high production value | 100+ registrations | AE + SDR attendance required | "Rho Summit 2025" |
| **Tier 2 (Strategic)** | Monthly events targeting key personas | 40-60 registrations | AE attendance encouraged | "CFO Roundtable: Series" |
| **Tier 3 (Tactical)** | Weekly touchpoints, content-driven | 15-30 registrations | SDR follow-up only | "Webinar: Feature Deep Dive" |

### Event Naming Convention

**Format**: `{Audience} {Event Type}: {Topic/Theme}`

**Examples**:
- ✅ "CFO Roundtable: Banking Stack Optimization" (clear audience, type, topic)
- ✅ "Startup Founders Webinar: Fundraising 101" (clear audience, type, topic)
- ❌ "December Event" (no context)
- ❌ "Rho x Stripe" (missing topic)

**UTM Campaign Auto-Generation**:
- Lowercase
- Replace spaces with hyphens
- Append event date
- Example: `event_cfo-roundtable-banking-stack_2025-12-15`

---

## Self-Service UI Requirements (Phase 3, Week 5-6)

### User Personas

1. **Christopher (Marketing Ops)**: Full admin access, can create events, upload CSVs, configure mappings
2. **Sales Team**: Can upload internal roundtable CSVs, view event analytics
3. **Partnerships Team**: Can upload partner event CSVs, view co-hosted event ROI

### Key Features

#### 1. Event Creation Form
- Fields: Event name, type, tier, date, location, host, Luma URL (optional)
- Auto-generates UTM parameters on save
- Creates HubSpot campaign automatically (via API)
- Returns event_id for CSV upload reference

#### 2. CSV Upload Wizard
- **Step 1**: Select event (dropdown of active events)
- **Step 2**: Upload CSV (drag-and-drop, max 10MB)
- **Step 3**: Choose mapping preset or manual map
- **Step 4**: Preview (show 5 sample rows)
- **Step 5**: Confirm import
- **Step 6**: Progress bar (e.g., "Importing 234 registrations... 45% complete")
- **Step 7**: Success summary (e.g., "234 registrations imported, 12 duplicates skipped, 2 errors")

#### 3. Event Dashboard
- List view: All events with registration count, attendance count, status
- Filters: Date range, type, tier, host
- Click event → Detail view:
  - Registration list (name, email, company, registration date, sync status)
  - Export CSV button
  - Sync status chart (Synced to HubSpot: 98%, Errors: 2%)
  - Analytics: Registrations over time, attendance rate, top companies

#### 4. Field Mapping Manager (Admin Only)
- List of saved presets (e.g., "Stripe Partner Events", "Mercury Co-hosted")
- Create/edit/delete mappings
- Test mapping: Upload sample CSV, preview results

### Technical Stack (Recommendation)

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| **Frontend** | Next.js + Tailwind CSS | Fast, SEO-friendly, matches existing Rho tech stack |
| **Backend** | Next.js API routes or Express.js | Serverless functions for CSV parsing, Airtable API calls |
| **Authentication** | Auth0 or Clerk | SSO integration with Google Workspace |
| **CSV Parsing** | Papa Parse (JS library) | Robust, handles large files, error handling |
| **File Upload** | AWS S3 or Vercel Blob | Temporary storage before processing |
| **Job Queue** | Inngest or Trigger.dev | Handle long-running CSV imports asynchronously |
| **Hosting** | Vercel | Easy deployment, Edge Functions, matches Next.js |

---

## Implementation Roadmap

### Week 1-2: MVP (Luma Integration Only)

**Goal**: Automate Luma → Airtable → HubSpot flow for current events.

**Tasks**:
1. **Airtable Setup** (4 hours)
   - Create Events table with schema
   - Create Registrations table with schema
   - Set up Luma API key and test webhook

2. **n8n Workflow: Luma to Airtable** (6 hours)
   - Webhook receiver node
   - Airtable search/create event logic
   - Airtable insert registration with dedupe check
   - Error handling and logging

3. **n8n Workflow: Airtable to HubSpot** (8 hours)
   - HubSpot contact search/create/update
   - Property mapping (lifecycle stage, UTM params, event fields)
   - List membership management
   - Error handling and retry logic

4. **Testing** (4 hours)
   - Test with 3 upcoming Luma events
   - Verify dedupe logic works
   - Confirm HubSpot sync accuracy
   - Monitor for 1 week, fix issues

**Deliverables**:
- ✅ Airtable base configured
- ✅ Luma webhook connected to n8n
- ✅ Automated sync to HubSpot (<5 min latency)
- ✅ Documentation: Setup guide, troubleshooting

**Owner**: Skydog Ops (with Christopher review/approval)

### Week 3-4: CSV Normalization Engine

**Goal**: Support manual CSV uploads with field mapping presets.

**Tasks**:
1. **Airtable: Field Mappings Table** (2 hours)
   - Create schema
   - Add 3 preset mappings (Stripe, Mercury, Zoom)

2. **n8n Workflow: CSV Import** (10 hours)
   - Airtable form to receive CSV upload + event_id + mapping_preset_id
   - Fetch mapping rules from Field Mappings table
   - Parse CSV, apply transformations
   - Validate data, log errors
   - Bulk insert into Registrations table (with dedupe)
   - Trigger enrichment & HubSpot sync for each row

3. **Testing** (4 hours)
   - Upload 3 CSVs from different sources
   - Verify transformations applied correctly
   - Confirm no duplicates created
   - Check error logs for invalid rows

**Deliverables**:
- ✅ CSV import workflow functional
- ✅ 3 mapping presets configured and tested
- ✅ Documentation: How to create new presets

**Owner**: Skydog Ops (with Christopher testing/feedback)

### Week 5-6: Self-Service UI

**Goal**: Allow Sales/Partnerships to upload CSVs without MOps involvement.

**Tasks**:
1. **UI Design** (4 hours)
   - Wireframes for Event Creation, CSV Upload Wizard, Dashboard
   - Design system (use Rho brand colors/fonts)

2. **Frontend Build** (16 hours)
   - Next.js app with Auth0 authentication
   - Event creation form (POST to Airtable API)
   - CSV upload wizard (Papa Parse + progress bar)
   - Event dashboard (fetch from Airtable, display table)
   - Field mapping manager (admin only)

3. **Backend Build** (12 hours)
   - Next.js API routes:
     - POST /api/events → Create Airtable Event record
     - POST /api/upload-csv → Parse CSV, validate, trigger n8n import workflow
     - GET /api/events → Fetch events list
     - GET /api/events/:id → Fetch event details + registrations
     - POST /api/mappings → Create/update field mapping preset
   - Error handling and validation

4. **Testing** (6 hours)
   - UAT with Sales team: Upload roundtable CSV
   - UAT with Partnerships: Upload partner event CSV
   - Test error scenarios (invalid CSV, duplicate event)

5. **Deployment** (2 hours)
   - Deploy to Vercel
   - Configure custom domain: events.rho.co (or similar)
   - Set up monitoring (Sentry for errors)

**Deliverables**:
- ✅ Self-service UI live at events.rho.co
- ✅ Sales/Partnerships trained on CSV upload process
- ✅ Christopher has admin dashboard for monitoring

**Owner**: Skydog Ops (frontend/backend), Christopher (UAT/training)

---

## Success Metrics

### Week 4 Targets (MVP Deployed)

| Metric | Target | How Measured |
|--------|--------|--------------|
| **Luma Events Automated** | 100% of new events | Count of events synced via webhook vs manual imports |
| **Sync Latency** | <5 minutes | Timestamp difference: Luma registration → HubSpot contact update |
| **Data Quality** | <5% error rate | Count of sync errors in Airtable / total registrations |
| **Time Saved** | 8 hours/month | Christopher tracks time no longer spent on manual imports |

### Week 8 Targets (Full Deployment)

| Metric | Target | How Measured |
|--------|--------|--------------|
| **CSV Sources Supported** | 5+ presets | Count of Field Mappings in Airtable |
| **Self-Service Adoption** | 80% of non-Luma events uploaded by Sales/Partnerships | Count of CSV uploads by user role |
| **Total Time Saved** | 20 hours/month | Time previously spent on manual imports + field mapping |
| **Event Visibility** | 100% of events in HubSpot within 24 hours | Audit: Compare Airtable Events vs HubSpot Campaigns |

### Q1 2026 Targets (Post-Handoff)

| Metric | Target | How Measured |
|--------|--------|--------------|
| **Event Volume** | 35-55 events/month | Count of Events in Airtable |
| **System Uptime** | 99.5% | Monitor n8n workflow execution success rate |
| **User Satisfaction** | 4.5/5 rating from Sales/Partnerships | Quarterly survey |
| **Attribution Accuracy** | 95% of registrations have correct UTM params | Audit: Sample 100 registrations, check UTM fields |

---

## Open Questions & Decisions Needed

### Week 1 (Blocking)

1. **Enrichment Provider**: Which tool for company data?
   - Option A: Clearbit (comprehensive, $$$)
   - Option B: ZoomInfo (sales-focused, $$)
   - Option C: Hybrid (Clearbit for email → company, ZoomInfo for revenue/size)
   - **Decision Maker**: Christopher + Anthony (budget approval)

2. **Salesforce Sync Criteria**: When to create SFDC leads?
   - Option A: All event registrations (high volume, sales may ignore)
   - Option B: Only high-intent (company size >100, target titles)
   - Option C: Only registrations for Tier 1/2 events
   - **Decision Maker**: Tommy (CRO) + Sales Leadership

3. **Event Ownership**: Who maintains Airtable Event records?
   - Option A: Christopher creates all events
   - Option B: Growth team creates events, Christopher audits
   - Option C: Self-service (any team can create, auto-approval)
   - **Decision Maker**: Christopher + Jeremy (Head of Growth)

### Week 3-4 (Important, Not Blocking)

4. **CSV Upload Permissions**: Who can upload CSVs?
   - Option A: Christopher only (current state, bottleneck)
   - Option B: Christopher + Sales Ops + Partnerships (3-5 people)
   - Option C: Any Sales/Partnerships team member (15+ people, needs training)
   - **Decision Maker**: Christopher + Tommy

5. **Error Handling**: What happens if HubSpot sync fails?
   - Option A: Retry 3 times, then alert Christopher via email
   - Option B: Retry 3 times, then alert Christopher via Slack
   - Option C: Retry 3 times, then pause workflow and require manual fix
   - **Decision Maker**: Christopher (operational preference)

---

## Technical Dependencies

### External APIs Required

| Service | Purpose | Credentials Needed | Setup Owner |
|---------|---------|-------------------|-------------|
| **Luma API** | Webhook for new registrations | API key, webhook secret | Christopher |
| **Airtable API** | Read/write Events & Registrations | Personal access token or OAuth | Christopher |
| **HubSpot API** | Contact/company/list management | Private app access token (contacts, lists, campaigns scopes) | Christopher |
| **Salesforce API** | Lead/campaign management (optional Week 1) | OAuth credentials or username/password/security token | Anthony (RevOps) |
| **Clearbit/ZoomInfo API** | Company enrichment | API key | Christopher (budget approval from Anthony) |

### Internal Dependencies

| Dependency | Description | Owner | Deadline |
|------------|-------------|-------|----------|
| **Leadership Alignment on SFDC Sync** | Confirm criteria for creating Salesforce leads | Tommy (CRO) | Week 1 |
| **HubSpot Campaign Template** | Standard campaign object properties for events | Christopher | Week 1 |
| **Events Taxonomy Approval** | Confirm event types, tiers, naming conventions | Jeremy (Growth) | Week 2 |
| **Sales Training** | How to use CSV upload UI, when to alert MOps | Sales Ops | Week 6 |

---

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **Luma API changes/deprecation** | High (breaks automation) | Low | Monitor Luma changelog, build fallback CSV import |
| **HubSpot API rate limits exceeded** | Medium (sync delays) | Medium | Implement exponential backoff, batch API calls, upgrade API tier if needed |
| **Enrichment API costs exceed budget** | Medium (need to disable feature) | Medium | Set monthly spend cap, alert at 80% usage, fallback to manual enrichment |
| **Users upload incorrectly formatted CSVs** | Low (errors, manual fixes needed) | High | Robust validation in UI, preview step, clear error messages, training docs |
| **n8n workflow fails silently** | High (data loss, angry sales team) | Low | Set up monitoring alerts (Slack webhook), daily audit report (Airtable → HubSpot sync count) |
| **Dedupe logic fails, creates duplicate contacts** | Medium (data quality issues) | Low | Comprehensive testing with edge cases, fallback to HubSpot native dedupe |

---

## Future Enhancements (Post-Week 8)

### Phase 2 (Q1 2026)
- **Attendance Tracking**: Integrate with check-in apps (Bizzabo, Splash) to auto-update attendance_status
- **Event ROI Dashboard**: Calculate pipeline generated, revenue attributed per event
- **Automated Nurture**: Trigger Customer.io email sequences based on event type/tier

### Phase 3 (Q2 2026)
- **Predictive Registration**: ML model to predict registration → attendance conversion
- **Slack Bot**: `/rho-event create` command to generate event record from Slack
- **Waitlist Management**: Auto-notify waitlisted contacts when spots open

---

## Appendix: Example CSV Formats

### Luma CSV Export (Current State)

```csv
email,name,created_at
jane.doe@acme.com,Jane Doe,2025-12-01 14:32
john.smith@startup.io,John Smith,2025-12-01 15:45
```

**Issues**:
- No first_name/last_name split
- No company field
- No phone field
- Date format inconsistent (sometimes ISO, sometimes MM/DD/YYYY)

### Stripe Partner Event CSV (Example)

```csv
attendee_email_address,attendee_first_name,attendee_last_name,company_name,title,registration_timestamp
JANE.DOE@ACME.COM,Jane,Doe,Acme Corp,Chief Financial Officer,12/01/2025 02:32 PM
JOHN.SMITH@STARTUP.IO,John,Smith,Startup Inc,CEO,12/01/2025 03:45 PM
```

**Issues**:
- Email in uppercase (need lowercase transformation)
- Date format non-standard (need MM/DD/YYYY HH:mm → ISO 8601)
- Column names don't match standard schema

### Target Standard Schema (Registrations Table)

```csv
event_id,email,first_name,last_name,company,job_title,phone,registration_date,registration_source
EV-2025-001,jane.doe@acme.com,Jane,Doe,Acme Corp,Chief Financial Officer,,2025-12-01T14:32:00Z,Luma API
EV-2025-001,john.smith@startup.io,John,Smith,Startup Inc,CEO,,2025-12-01T15:45:00Z,CSV Upload - Stripe
```

**Benefits**:
- Consistent field names
- Lowercase emails
- ISO 8601 dates
- Clear source attribution

---

**Next Steps**:
1. Week 1: Schedule 30-min kickoff with Skydog Ops to review this spec
2. Week 1: Christopher configures Airtable base and Luma API webhook
3. Week 1-2: Skydog Ops builds MVP (Luma → Airtable → HubSpot automation)
4. Week 2: Test MVP with 3 live events, iterate on errors
5. Week 3: Begin CSV normalization engine build

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
