# 08 - Technical Architecture & System Specifications

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Related Documents**: [01-Root Cause Analysis](01-root-cause-analysis.md), [03-Events Automation](03-events-automation.md)

---

## Executive Summary

This document defines the technical architecture for Rho's Marketing Operations infrastructure, including:
1. **Events Automation Platform** - Real-time event registration sync across systems
2. **Email Component Library** - CI/CD pipeline for automated template deployment
3. **HubSpot Integration Layer** - API contracts and error handling
4. **Data Schema Specifications** - Canonical field definitions and transformation rules

**Key Architectural Principles**:
- **Event-Driven**: Webhooks and real-time triggers (not batch ETL)
- **API-First**: All integrations via documented REST APIs
- **Idempotent**: Operations can be safely retried without side effects
- **Observable**: Comprehensive logging, error tracking, and monitoring

---

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                              │
├─────────────────────────────────────────────────────────────────┤
│  Luma API    │  CSV Uploads  │  Partner Events  │  Webinars     │
└───────┬──────────────┬───────────────┬──────────────┬───────────┘
        │              │               │              │
        └──────────────┴───────────────┴──────────────┘
                       ↓
┌─────────────────────────────────────────────────────────────────┐
│                 AIRTABLE (Source of Truth)                       │
├─────────────────────────────────────────────────────────────────┤
│  • Events Table (taxonomy, UTM generation)                       │
│  • Registrations Table (normalized contact data)                │
│  • Field Mappings Table (CSV transformation rules)              │
└───────────────────────────┬─────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│           TRANSFORMATION LAYER (n8n Workflows)                   │
├─────────────────────────────────────────────────────────────────┤
│  • Dedupe Logic                                                  │
│  • Data Validation                                               │
│  • Enrichment (Clearbit/ZoomInfo)                               │
│  • Field Mapping & Normalization                                │
└───────────────────────────┬─────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                   DISTRIBUTION LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  HubSpot        │  Salesforce    │  Amplitude   │  Slack        │
│  (Marketing)    │  (Sales CRM)   │  (Analytics) │  (Alerts)     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component 1: Events Automation Platform

### Airtable Schema Specifications

#### Events Table

**Purpose**: Central registry of all Rho events with standardized taxonomy.

**Fields**:

| Field Name | Data Type | Constraints | Formula/Default | Description |
|------------|-----------|-------------|-----------------|-------------|
| `event_id` | Auto-number | Required, Unique | Auto-increment | Primary key: `EV-2025-001` |
| `event_name` | Single-line text | Required, Max 200 chars | | Event display name |
| `event_type` | Single-select | Required | Options: `Luma Event`, `Partner Event`, `Internal Roundtable`, `Webinar` | Event category |
| `event_tier` | Single-select | Required | Options: `Tier 1 (Flagship)`, `Tier 2 (Strategic)`, `Tier 3 (Tactical)` | Priority level |
| `event_date` | Date/time | Required | | Start date/time (ISO 8601) |
| `event_location` | Single-line text | Optional | | Venue address or "Virtual" |
| `host` | Single-select | Required | Options: `Growth`, `Sales`, `Partnerships`, `Customer Success` | Owning team |
| `luma_url` | URL | Optional | | Luma event page (if applicable) |
| `registration_count` | Rollup | Auto-calculated | `COUNT({Registrations})` | Total registrations |
| `attendance_count` | Number | Optional | | Actual attendees (manual entry post-event) |
| `requires_nurture` | Checkbox | | Default: `true` | Enable automated follow-up |
| `utm_campaign` | Formula | Auto-generated | `"event_" & LOWER(SUBSTITUTE({event_name}, " ", "-")) & "_" & DATETIME_FORMAT({event_date}, 'YYYY-MM-DD')` | UTM campaign parameter |
| `utm_source` | Single-select | Required | Options: `luma`, `partner_stripe`, `partner_mercury`, `direct` | Traffic source |
| `utm_medium` | Single-select | Required | Options: `event`, `webinar`, `roundtable`, `partner_event` | Marketing channel |
| `hubspot_campaign_id` | Number | Optional | | HubSpot campaign ID (populated after creation) |
| `salesforce_campaign_id` | Single-line text | Optional | | SFDC campaign ID (18-char format) |
| `status` | Single-select | Required | Default: `Planned` | Options: `Planned`, `Registration Open`, `Completed`, `Cancelled` |
| `created_at` | Created time | Auto | | Record creation timestamp |
| `updated_at` | Last modified time | Auto | | Last update timestamp |

#### Registrations Table

**Purpose**: Normalized contact data for all event registrations.

**Fields**:

| Field Name | Data Type | Constraints | Formula/Default | Description |
|------------|-----------|-------------|-----------------|-------------|
| `registration_id` | Auto-number | Required, Unique | Auto-increment | Primary key: `REG-2025-001234` |
| `event_id` | Link to Events | Required | | Foreign key to Events table |
| `email` | Email | Required, Indexed | | Contact email (dedupe key with event_id) |
| `first_name` | Single-line text | Required | | Max 100 chars |
| `last_name` | Single-line text | Required | | Max 100 chars |
| `company` | Single-line text | Optional | | Max 200 chars |
| `job_title` | Single-line text | Optional | | Max 150 chars |
| `phone` | Phone | Optional | | E.164 format preferred |
| `registration_date` | Date/time | Required | Default: `NOW()` | When they registered (ISO 8601) |
| `registration_source` | Single-select | Required | Options: `Luma API`, `CSV Upload`, `Webinar Platform`, `Manual Entry` | Data origin |
| `attendance_status` | Single-select | | Default: `Registered` | Options: `Registered`, `Attended`, `No-Show`, `Cancelled` |
| `hubspot_contact_id` | Number | Optional | | HubSpot contact vid |
| `salesforce_lead_id` | Single-line text | Optional | | SFDC 18-char ID |
| `sync_status` | Single-select | | Default: `Pending` | Options: `Pending`, `Synced to HubSpot`, `Synced to SFDC`, `Error` |
| `sync_error` | Long text | Optional | | Error message if sync failed |
| `enrichment_status` | Single-select | | Default: `Not Enriched` | Options: `Not Enriched`, `Enriched`, `Failed` |
| `enriched_company_size` | Number | Optional | | Employee count (from Clearbit/ZoomInfo) |
| `enriched_revenue` | Currency | Optional | | Annual revenue estimate |
| `utm_source` | Lookup | Auto | From Events table | |
| `utm_medium` | Lookup | Auto | From Events table | |
| `utm_campaign` | Lookup | Auto | From Events table | |
| `created_at` | Created time | Auto | | Record creation timestamp |

**Indexes**:
- Composite: `(email, event_id)` - For dedupe lookups

### API Integration Specifications

#### Luma Webhook (Inbound)

**Endpoint**: `https://n8n.rho.co/webhook/luma-registration` (example)
**Method**: POST
**Authentication**: Webhook secret in header (`X-Luma-Signature`)

**Payload Example**:
```json
{
  "event": {
    "api_id": "evt_abc123",
    "name": "CFO Roundtable: Banking Stack Optimization",
    "start_at": "2025-12-15T18:00:00Z",
    "url": "https://lu.ma/rho-cfo-dec-2025"
  },
  "user": {
    "email": "jane.doe@acme.com",
    "name": "Jane Doe",
    "company": "Acme Corp",
    "title": "CFO"
  },
  "registration": {
    "api_id": "reg_xyz789",
    "registered_at": "2025-12-01T14:32:00Z"
  }
}
```

**Processing Logic** (n8n workflow):
1. Verify webhook signature
2. Extract event data → Search/create Events record in Airtable
3. Extract user data → Create Registrations record (with dedupe check)
4. Trigger enrichment workflow (async)
5. Trigger HubSpot sync workflow (async)
6. Return `200 OK` (acknowledge receipt within 5 seconds)

**Error Handling**:
- Invalid signature → `401 Unauthorized`
- Duplicate registration → `200 OK` (idempotent, no error)
- Airtable API error → Retry 3x with exponential backoff, then log to Slack

#### HubSpot Contacts API (Outbound)

**Endpoint**: `https://api.hubapi.com/crm/v3/objects/contacts`
**Method**: POST (create) / PATCH (update)
**Authentication**: Bearer token (Private App)

**Create Contact Payload**:
```json
{
  "properties": {
    "email": "jane.doe@acme.com",
    "firstname": "Jane",
    "lastname": "Doe",
    "company": "Acme Corp",
    "jobtitle": "CFO",
    "phone": "+15551234567",
    "lifecyclestage": "lead",
    "hs_analytics_source": "EVENTS",
    "recent_event_name": "CFO Roundtable: Banking Stack Optimization",
    "recent_event_date": "2025-12-15",
    "utm_source": "luma",
    "utm_medium": "event",
    "utm_campaign": "event_cfo-roundtable-banking-stack_2025-12-15"
  }
}
```

**Update Logic**:
1. Search for contact by email: `GET /crm/v3/objects/contacts/search`
2. If exists: PATCH with updated properties (append to `recent_event_name` if multiple events)
3. If not exists: POST to create new contact
4. Store returned `id` (vid) in Airtable `hubspot_contact_id` field

**Rate Limits**:
- Free/Starter: 100 requests per 10 seconds
- Professional: 150 requests per 10 seconds
- **Mitigation**: Batch contact creates (up to 100 per request via batch API)

**Error Handling**:
- 429 Too Many Requests → Exponential backoff (wait 2^n seconds, max 5 retries)
- 400 Bad Request (invalid property) → Log error to Airtable `sync_error`, alert Christopher
- 401 Unauthorized → Invalidate cached token, re-authenticate, retry once
- 500 Internal Server Error → Retry 3x, then mark `sync_status` = "Error"

#### HubSpot Lists API (Outbound)

**Endpoint**: `https://api.hubapi.com/contacts/v1/lists/{list_id}/add`
**Method**: POST
**Purpose**: Add registrant to event-specific list for segmentation

**Payload**:
```json
{
  "vids": [98765432],
  "emails": ["jane.doe@acme.com"]
}
```

**List Naming Convention**: `Event Registrations - {event_name} - {YYYY-MM-DD}`
- Example: `Event Registrations - CFO Roundtable - 2025-12-15`

**List Creation** (if not exists):
- Endpoint: `POST /contacts/v1/lists`
- Dynamic list criteria: `utm_campaign = event_cfo-roundtable-banking-stack_2025-12-15`

---

## Component 2: Email Component Library

### GitHub Repository Structure

```
rho-hubspot-deployment/
├── .github/
│   └── workflows/
│       └── deploy-email-templates.yml      # GitHub Actions CI/CD
├── components/
│   ├── Header.jsx
│   ├── Hero.jsx
│   ├── BodyWithCTA.jsx
│   ├── TwoColumnCards.jsx
│   ├── Highlight.jsx
│   ├── PreFooter.jsx
│   └── Footer.jsx
├── templates/
│   ├── event-invite.jsx                    # Full email compositions
│   ├── product-launch.jsx
│   └── newsletter.jsx
├── dist/
│   └── templates/                          # Build output (HTML)
│       ├── event-invite.html
│       ├── product-launch.html
│       └── newsletter.html
├── tests/
│   ├── link-validator.test.js
│   ├── image-validator.test.js
│   └── accessibility.test.js
├── scripts/
│   ├── build.js                            # JSX → HTML transformation
│   └── deploy.js                           # HubSpot API upload
├── index.html                              # Preview site
├── package.json
├── .env.example                            # API keys template
└── README.md
```

### Build Pipeline (npm run build)

**Input**: `components/*.jsx` + `templates/*.jsx`
**Output**: `dist/templates/*.html`

**Transformation Steps**:

1. **JSX to HTML** (using `jsx-email` or custom Babel)
   ```javascript
   import { renderToStaticMarkup } from 'react-dom/server';
   import EventInvite from './templates/event-invite.jsx';

   const html = renderToStaticMarkup(<EventInvite />);
   fs.writeFileSync('dist/templates/event-invite.html', html);
   ```

2. **Inline CSS** (all styles must be inline for email compatibility)
   - Components already use inline styles, but verify no `<style>` tags in output

3. **Validate HTML**
   - Check for unclosed tags
   - Verify all `<img>` tags have `src` and `alt` attributes
   - Ensure all `<a>` tags have `href`

4. **Add Email Client Resets** (prepend to `<head>`)
   ```html
   <style>
     body { margin: 0; padding: 0; }
     img { border: 0; }
     table { border-collapse: collapse; }
     /* Additional resets */
   </style>
   ```

### Deployment Pipeline (npm run deploy)

**HubSpot Design Manager API**:

**Endpoint**: `POST https://api.hubapi.com/content/api/v2/templates`

**Payload**:
```json
{
  "name": "Event Invite - v1.2.3",
  "source": "<html>...</html>",
  "folder": "Email Templates / Automated",
  "template_type": "email",
  "author": "christopher.cooper@rho.co"
}
```

**Versioning Strategy**:
- Use Git commit SHA as version tag: `Event Invite - {SHA:0:7}`
- Store `template_id` returned by HubSpot in Git tag annotation

**Rollback Process**:
1. Identify Git commit with working version: `git log --oneline`
2. Checkout that commit: `git checkout abc1234`
3. Rebuild and redeploy: `npm run build && npm run deploy`

### Automated Testing (npm test)

**Test Suites**:

1. **Link Validator**
   ```javascript
   // Verify all href attributes are valid URLs
   const links = html.match(/href="([^"]+)"/g);
   links.forEach(link => {
     const url = link.match(/href="([^"]+)"/)[1];
     if (!url.startsWith('http')) {
       throw new Error(`Invalid link: ${url}`);
     }
   });
   ```

2. **Image Validator**
   ```javascript
   // Check all image URLs return 200
   const images = html.match(/src="([^"]+\.(png|jpg|gif))"/g);
   for (const img of images) {
     const url = img.match(/src="([^"]+)"/)[1];
     const response = await fetch(url);
     if (response.status !== 200) {
       throw new Error(`Broken image: ${url}`);
     }
   }
   ```

3. **Accessibility**
   ```javascript
   // Ensure all images have alt text
   const imgsWithoutAlt = html.match(/<img(?![^>]*alt=)[^>]*>/g);
   if (imgsWithoutAlt.length > 0) {
     throw new Error('Images missing alt text');
   }

   // Check color contrast (WCAG AA: 4.5:1 for text)
   // Use library like 'color-contrast-checker'
   ```

---

## Component 3: HubSpot Integration Layer

### Custom Property Definitions

**Naming Convention**: `{object}_{descriptor}_{type}`

**New Properties to Create**:

| Property Name | Object | Type | Description | Example Values |
|---------------|--------|------|-------------|----------------|
| `contact_recent_event_name` | Contact | Single-line text | Most recent event attended | "CFO Roundtable: Banking Stack" |
| `contact_recent_event_date` | Contact | Date | Date of most recent event | `2025-12-15` |
| `contact_total_events_attended` | Contact | Number | Lifetime event count | `3` |
| `contact_first_event_date` | Contact | Date | First event attended (never overwrite) | `2024-06-10` |
| `contact_lifecycle_event_source` | Contact | Single-line text | Which event converted lead to MQL/SQL | "CFO Roundtable" |
| `company_total_event_attendees` | Company | Number | Rollup: Count of contacts from this company at events | `5` |

**Property Groups**:
- Create "Event Tracking" property group for organization

### Workflow Templates

**Template 1: Event Registration Workflow**

**Trigger**: Contact property `contact_recent_event_name` is known
**Actions**:
1. Set `lifecyclestage` = "lead" (if currently "subscriber")
2. Add to list: "Event Registrations - All"
3. Add to list: "Event Registrations - {event_name}" (dynamic)
4. If `job_title` contains "CFO" OR "CEO" OR "Founder":
   - Set `contact_lifecycle_event_source` = {event_name}
   - Send internal notification: Slack alert to #sales-events
5. Enroll in nurture sequence (if `requires_nurture` = true)

**Template 2: Event Attendance Follow-Up**

**Trigger**: Contact property `attendance_status` = "Attended"
**Actions**:
1. Wait 1 day
2. Send email: "Thanks for attending {event_name}"
3. Wait 3 days
4. If no reply: Send email: "Join our next event"
5. Update `contact_total_events_attended` += 1

---

## Data Schema & Transformation Rules

### Canonical Contact Schema

**Standard Fields** (all systems must map to these):

| Canonical Field | Type | Constraints | Transformation Rules |
|-----------------|------|-------------|---------------------|
| `email` | String | Required, lowercase, valid email format | `LOWER(TRIM(source.email))` |
| `first_name` | String | Required, titlecase | `TITLECASE(TRIM(source.first_name))` |
| `last_name` | String | Required, titlecase | `TITLECASE(TRIM(source.last_name))` |
| `company` | String | Optional, trimmed | `TRIM(source.company)` |
| `job_title` | String | Optional, trimmed | `TRIM(source.job_title)` |
| `phone` | String | Optional, E.164 format | `PHONE_FORMAT(source.phone, 'E164')` |
| `registration_date` | ISO 8601 DateTime | Required | `ISO8601(source.registration_timestamp)` |

**Phone Formatting Function**:
```javascript
function formatPhoneE164(phone, countryCode = 'US') {
  // Remove all non-numeric characters
  const cleaned = phone.replace(/\D/g, '');

  // If no country code, add US +1
  if (cleaned.length === 10 && countryCode === 'US') {
    return `+1${cleaned}`;
  }

  // If already has country code
  if (cleaned.startsWith('1') && cleaned.length === 11) {
    return `+${cleaned}`;
  }

  // Otherwise return as-is (may be invalid)
  return phone;
}
```

**Date Parsing** (handle multiple formats):
```javascript
function parseDate(dateString) {
  // Try ISO 8601 first
  if (/\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/.test(dateString)) {
    return new Date(dateString).toISOString();
  }

  // Try MM/DD/YYYY
  if (/\d{2}\/\d{2}\/\d{4}/.test(dateString)) {
    const [month, day, year] = dateString.split('/');
    return new Date(`${year}-${month}-${day}`).toISOString();
  }

  // Try MM-DD-YYYY
  if (/\d{2}-\d{2}-\d{4}/.test(dateString)) {
    const [month, day, year] = dateString.split('-');
    return new Date(`${year}-${month}-${day}`).toISOString();
  }

  // Fallback: Current timestamp
  return new Date().toISOString();
}
```

---

## Monitoring & Observability

### Key Metrics to Track

**System Health**:
| Metric | Target | Alert Threshold | Measurement |
|--------|--------|-----------------|-------------|
| **Event Sync Latency** | <5 min (p95) | >10 min | Timestamp diff: Luma registration → HubSpot contact update |
| **Sync Success Rate** | >98% | <95% | (Successful syncs / Total registrations) × 100 |
| **API Error Rate** | <2% | >5% | Count of 4xx/5xx responses / Total API calls |
| **Webhook Delivery** | 100% | <99% | Luma webhook receipts vs. expected events |
| **Build Pipeline Success** | >95% | <90% | GitHub Actions workflow pass rate |

**Business Metrics**:
| Metric | Target | Measurement |
|--------|--------|-------------|
| **Event Registration Volume** | 100+ registrations/week | Count in Airtable Registrations table |
| **Time Saved (Christopher)** | 20 hrs/month | Manual tracking vs. automated |
| **Email Deploy Frequency** | 10+ deploys/month | GitHub Actions workflow runs |
| **Workflow Count** | <100 active | HubSpot audit script |

### Logging Standards

**Log Levels**:
- **ERROR**: System failure, requires immediate attention (e.g., API authentication failed)
- **WARN**: Degraded performance, manual intervention may be needed (e.g., enrichment API slow)
- **INFO**: Normal operations (e.g., "Contact synced to HubSpot")
- **DEBUG**: Detailed trace for troubleshooting (e.g., "Airtable API response: {...}")

**Log Format** (JSON for structured logging):
```json
{
  "timestamp": "2025-12-15T18:05:32Z",
  "level": "INFO",
  "service": "events-automation",
  "workflow": "luma-to-hubspot-sync",
  "event_id": "EV-2025-042",
  "registration_id": "REG-2025-005432",
  "message": "Contact synced to HubSpot",
  "hubspot_contact_id": 98765432,
  "duration_ms": 342
}
```

**Error Log Example**:
```json
{
  "timestamp": "2025-12-15T18:06:15Z",
  "level": "ERROR",
  "service": "events-automation",
  "workflow": "luma-to-hubspot-sync",
  "event_id": "EV-2025-042",
  "registration_id": "REG-2025-005433",
  "message": "HubSpot API error",
  "error": {
    "status": 429,
    "message": "Rate limit exceeded",
    "retry_after": 30
  },
  "stack_trace": "..."
}
```

### Alerting Rules

**Critical Alerts** (Slack #mops-alerts, immediate):
- Webhook authentication failure (invalid signature from Luma)
- HubSpot API 401 Unauthorized (token expired/revoked)
- Airtable API error rate >10% (system degradation)
- Build pipeline failing for >2 consecutive runs

**Warning Alerts** (Slack #mops-alerts, within 1 hour):
- Sync latency >10 minutes (p95)
- Enrichment API error rate >20%
- Email image validation failures (broken images)
- HubSpot rate limit hit (429 errors)

**Info Notifications** (Slack #marketing-ops, daily digest):
- Total registrations synced today
- Email templates deployed today
- Workflow governance actions (sunset notifications sent, approvals pending)

---

## Security & Compliance

### API Key Management

**Storage**:
- **GitHub Secrets**: For CI/CD pipeline (HubSpot API key, Slack webhook)
- **n8n Credentials**: Encrypted credential store for Luma, Airtable, HubSpot
- **Never commit**: `.env` files, API keys in code

**Rotation Policy**:
- HubSpot Private App tokens: Rotate every 90 days
- Luma webhook secrets: Rotate every 180 days
- n8n credentials: Audit quarterly, rotate if team member leaves

**Access Control**:
- Principle of least privilege: Only grant scopes required (e.g., HubSpot contacts, lists - not settings)
- Audit trail: Log all API key usage (which workflow, when, what action)

### Data Privacy (GDPR/CCPA Compliance)

**Consent Tracking**:
- Luma registrations include implicit consent for event communications
- HubSpot subscription status: Set to "Opted In" for event registrations
- Unsubscribe link: Required in all emails (HubSpot native token)

**Data Retention**:
- **Airtable Registrations**: Retain indefinitely (business records)
- **HubSpot Contacts**: Follow HubSpot retention policies (user can request deletion)
- **Logs**: Retain 90 days (n8n execution logs, GitHub Actions logs)

**Right to Erasure**:
- Process: Contact requests deletion → Christopher manually removes from Airtable + HubSpot
- Timeline: Within 30 days of request
- (Future enhancement: Automated deletion workflow)

---

## Disaster Recovery

### Backup Strategy

**Airtable**:
- **Native Backups**: Airtable auto-snapshots every 24 hours (30-day retention)
- **Manual Export**: Christopher exports to CSV weekly (stored in Google Drive)
- **Restoration**: Import CSV back to new Airtable base if needed

**GitHub Repository**:
- **Git History**: Full version control, can restore any commit
- **Branch Protection**: Require PR reviews, prevent force pushes to main
- **Backup**: GitHub's redundant storage (99.9% uptime SLA)

**HubSpot**:
- **Native Backups**: HubSpot does not offer point-in-time restoration
- **Export**: Weekly export of contacts, workflows (via API or manual CSV)
- **Restoration**: Re-import contacts, rebuild workflows from documentation

**n8n Workflows**:
- **Export**: All workflows exported as JSON, stored in GitHub repo
- **Restoration**: Import JSON back to n8n instance

### Incident Response Plan

**Severity Levels**:

| Level | Definition | Response Time | Example |
|-------|------------|---------------|---------|
| **P0 - Critical** | System down, no workaround | <1 hour | All Luma registrations not syncing |
| **P1 - High** | Major feature broken, workaround exists | <4 hours | CSV upload failing, manual Airtable entry works |
| **P2 - Medium** | Minor feature broken, low impact | <1 business day | Email preview site slow to load |
| **P3 - Low** | Cosmetic issue, no functional impact | <1 week | Typo in documentation |

**Escalation Path**:
1. **Christopher** (first responder): Investigates, attempts fix
2. **Skydog Ops** (if within 30-day support): Zoom call, pair programming
3. **HubSpot Support** (for API issues): Submit ticket, share error logs
4. **Leadership** (if business-critical): Notify Tommy/Jeremy, request eng resources

**Runbook Template**:
```markdown
# Incident: Luma Registrations Not Syncing

## Detection
- Luma webhook logs show 500 errors
- Airtable Registrations table has no new records in 2 hours

## Diagnosis Steps
1. Check n8n workflow execution history (last successful run?)
2. Verify Luma API status page (is service down?)
3. Check Airtable API rate limits (quota exceeded?)
4. Review n8n error logs (authentication? validation?)

## Resolution
- If Luma API down: Wait for restoration, backfill registrations via API
- If n8n workflow error: Fix workflow, re-process failed registrations
- If Airtable rate limit: Pause non-critical workflows, resume after quota reset

## Prevention
- Add Airtable rate limit monitoring (alert at 80% quota)
- Implement exponential backoff in n8n workflows
```

---

## Performance Optimization

### Latency Targets

| Operation | Target | Current (Estimated) | Optimization |
|-----------|--------|---------------------|--------------|
| **Luma webhook → Airtable** | <2 sec | ~5 sec | n8n hosted on fast server, minimize workflow steps |
| **Airtable → HubSpot sync** | <3 sec | ~10 sec | Batch API calls (100 contacts per request) |
| **CSV upload (100 rows)** | <5 min | ~15 min | Parallel processing (10 rows at a time) |
| **Email template build** | <30 sec | ~60 sec | Cache npm dependencies, use esbuild instead of Babel |
| **GitHub Actions deploy** | <5 min | ~8 min | Optimize Docker image size, cache layers |

### Caching Strategy

**n8n Workflow Cache**:
- Cache Airtable event lookups (TTL: 5 minutes)
- Avoid re-fetching same event metadata multiple times in rapid succession

**HubSpot API Response Cache**:
- Cache contact lookups by email (TTL: 10 minutes)
- Invalidate cache on contact update

**Build Artifacts**:
- Cache `node_modules` in GitHub Actions (speeds up CI pipeline by 2-3 min)

---

## Future Architectural Enhancements

### Phase 2 (Q1 2026)

1. **Storyblok Integration**
   - Add serverless middleware (Vercel Edge Functions)
   - Storyblok webhook → Render email HTML → Deploy to HubSpot
   - See [04-Email Template Architecture](04-email-template-architecture.md)

2. **Customer.io Migration**
   - Move event emails from HubSpot → Customer.io (bypass contact bloat)
   - Sync Airtable Registrations → Customer.io segments
   - Trigger broadcasts via API

3. **Real-Time Progress State Cache**
   - Redis cache for application progress (fix 45-min gap)
   - Requires Engineering team to emit events on progress steps
   - See [01-Root Cause Analysis](01-root-cause-analysis.md)

### Phase 3 (Q2 2026)

4. **Self-Service Events UI**
   - Next.js web app for Sales/Partnerships to upload CSVs
   - Real-time dashboard (registration counts, sync status)
   - See [11-Events Management UI](11-events-management-ui.md)

5. **Machine Learning Enhancements**
   - Predict registration → attendance conversion (train model on historical data)
   - Auto-prioritize high-intent registrations (score based on job title, company size)

---

**Last Updated**: 2025-11-12
**Document Owner**: Christopher Cooper + Skydog Ops (technical lead)
**Next Review**: End of Week 2 (after MVP deployed)

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
