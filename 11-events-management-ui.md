# 11 - Events Management UI (Self-Service Platform)

**Document Status**: ✅ Complete - Q1 2026 Project Spec
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Timeline**: Q1 2026 (6-8 weeks, $25-30K budget)
**Phase**: Post-Skydog Engagement
**Related**: [03-Events Automation](03-events-automation.md), [08-Technical Architecture](08-technical-architecture.md)

---

## Executive Summary

**Problem**: Currently, only Christopher can upload event CSVs and manage event data. With projected volume of 35-55 events/month in Q1 2026, this doesn't scale.

**Solution**: Self-service web application that allows Sales, Partnerships, and Growth teams to:
- Upload event CSVs with guided field mapping
- View real-time event dashboards (registration counts, sync status)
- Monitor upcoming events and download registration lists

**Business Impact**:
- **80% reduction** in Christopher's time on event operations (16 hrs/month → 3 hrs/month)
- **Sales enablement**: AEs can see high-intent registrations in real-time
- **Scalability**: Support 50+ events/month without additional headcount

**Timeline**: 6-8 weeks development + 2 weeks UAT
**Budget**: $25-30K (internal dev team or external vendor)

---

## User Personas & Requirements

### Persona 1: Christopher Cooper (Marketing Ops Admin)

**Needs**:
- Full system access (create events, upload CSVs, configure field mappings, view all data)
- Audit logs (who uploaded what, when)
- Error monitoring (see failed imports, retry manually)
- Export capabilities (download all registrations as CSV)

**Use Cases**:
1. Create new event in system → Auto-generates UTM parameters
2. Configure field mapping preset for new partner (e.g., "Ramp Events CSV format")
3. Review import errors → Fix data quality issues → Re-import
4. Monitor sync status across all events (Airtable → HubSpot → Salesforce)

### Persona 2: Partnerships Team (Emily, Partnerships Manager)

**Needs**:
- Upload CSVs for co-hosted events (Stripe, Mercury, Ramp)
- Select pre-configured field mapping preset (no manual mapping)
- See confirmation that import succeeded
- Download list of registrants for pre-event outreach

**Use Cases**:
1. Receive CSV from partner → Log into Events UI
2. Click "Upload Event CSV" → Select event → Choose "Stripe Partner Events" preset → Upload file
3. Preview 5 sample rows → Confirm → Import starts
4. See success message: "234 registrations imported, 12 duplicates skipped, 2 errors"
5. Download registration list as CSV for follow-up

### Persona 3: Sales Team (AE, SDR)

**Needs**:
- View upcoming events (what's happening this week/month)
- See high-intent registrations (CFO/CEO titles, large companies)
- Export registration lists for manual outreach
- Understand which events drive most pipeline

**Use Cases**:
1. Log into Events UI → Dashboard shows "Upcoming Events" (3 this week)
2. Click event → See registration list sorted by "High Intent Score"
3. Filter by job title: "CFO" → See 12 CFOs registered
4. Download as CSV → Import into personal outreach cadence

---

## Feature Specifications

### Feature 1: Event Creation Form

**Purpose**: Allow admins (Christopher) to create new events in system.

**Fields**:
- Event Name (required, max 200 chars)
- Event Type (dropdown: Luma Event, Partner Event, Internal Roundtable, Webinar)
- Event Tier (dropdown: Tier 1, Tier 2, Tier 3)
- Event Date (date/time picker)
- Event Location (text input or "Virtual")
- Host (dropdown: Growth, Sales, Partnerships, Customer Success)
- Luma URL (optional URL input, validated)
- Requires Nurture (checkbox, default: true)

**Auto-Generated Fields** (read-only, calculated on save):
- UTM Campaign: `event_cfo-roundtable-banking-stack_2025-12-15`
- UTM Source: (selected by user, default: "luma")
- UTM Medium: (selected by user, default: "event")

**Actions**:
- Save → Creates record in Airtable Events table
- Save & Create HubSpot Campaign → Creates HubSpot campaign object via API
- Cancel → Discard changes

**Validation**:
- Event Name: Required, unique per date
- Event Date: Must be future date
- Luma URL: Must be valid URL format

**Success Message**: "Event created successfully! Event ID: EV-2025-042"

### Feature 2: CSV Upload Wizard

**Purpose**: Self-service CSV upload with guided field mapping.

**Step 1: Select Event**
- Dropdown list of events (filter: status = "Registration Open" OR "Planned")
- Display: "CFO Roundtable - Dec 15, 2025 (EV-2025-042)"
- Option: "Create New Event" (redirects to Event Creation Form)

**Step 2: Upload CSV**
- Drag-and-drop zone or "Choose File" button
- Max file size: 10 MB
- Supported formats: CSV, TSV
- File preview: Show first 5 rows in table

**Step 3: Field Mapping**
- Option A: "Use Preset" (dropdown of saved mappings: Stripe, Mercury, Zoom, etc.)
  - Loads mapping rules automatically
  - Highlights mapped columns in green
- Option B: "Manual Mapping" (for new sources)
  - Left column: CSV column names (detected from file)
  - Right column: Target field dropdowns (email, first_name, last_name, company, etc.)
  - Transformation options (lowercase, date format, phone format)
- Option C: "Save as New Preset" (after manual mapping)
  - Name preset (e.g., "Ramp Partner Events")
  - Store for future reuse

**Step 4: Preview**
- Show 5 sample rows with transformations applied
- Highlight errors (invalid email format, missing required fields)
- Summary: "234 valid rows, 2 errors"
- Option: "Download Errors CSV" (see which rows failed validation)

**Step 5: Confirm Import**
- Button: "Import 234 Registrations"
- Progress bar: "Importing... 45% complete"
- Real-time updates: "Inserting to Airtable... Syncing to HubSpot..."

**Step 6: Success Summary**
- "Import Complete! ✅"
- Stats:
  - 234 registrations imported
  - 12 duplicates skipped (already registered for this event)
  - 2 errors (invalid data, see error log)
- Actions:
  - "Download Registration List" (all registrations for this event)
  - "View Event Dashboard" (navigate to event detail page)
  - "Upload Another CSV" (return to Step 1)

**Error Handling**:
- CSV parsing error (invalid format) → Show error message, allow re-upload
- Validation failures (invalid emails) → Show error rows, allow download, continue with valid rows
- Airtable API error → Retry 3x, then show error message, allow manual retry
- HubSpot sync error → Log error, continue import (sync will retry automatically)

### Feature 3: Event Dashboard

**Purpose**: Real-time view of all events with key metrics.

**Layout**:

**Header**:
- Title: "Events Dashboard"
- Filters:
  - Date Range: "This Month" | "Next 30 Days" | "All Upcoming" | Custom
  - Event Type: All | Luma Event | Partner Event | Roundtable | Webinar
  - Status: All | Planned | Registration Open | Completed | Cancelled
- Actions:
  - Button: "Create New Event"
  - Button: "Upload CSV"

**Event List (Table View)**:

| Event Name | Type | Date | Registrations | Attendance | Status | Actions |
|------------|------|------|---------------|------------|--------|---------|
| CFO Roundtable: Banking Stack | Luma Event | Dec 15, 2025 | 42 | TBD | Registration Open | View \| Edit \| Export |
| Rho x Stripe: Fintech Panel | Partner Event | Dec 20, 2025 | 68 | TBD | Registration Open | View \| Edit \| Export |
| Webinar: Cash Management 101 | Webinar | Dec 18, 2025 | 156 | TBD | Completed | View \| Export |

**Sorting**:
- Click column header to sort (date, registrations, etc.)

**Pagination**:
- 20 events per page, load more on scroll

**Quick Stats (Top of Page)**:
- Total Upcoming Events: 8
- Total Registrations (This Month): 542
- Avg Registrations per Event: 67.8
- High-Intent Registrations: 23 (CFO/CEO titles, 500+ employees)

### Feature 4: Event Detail Page

**Purpose**: Deep dive into single event with registration list and sync status.

**Header**:
- Event Name: "CFO Roundtable: Banking Stack Optimization"
- Event ID: EV-2025-042
- Status Badge: 🟢 Registration Open
- Date: December 15, 2025, 6:00 PM ET
- Location: Rho Office - 100 Crosby St, NY
- Host: Growth Team

**Key Metrics (Cards)**:
- Registrations: 42
- Attendance: TBD (update post-event)
- Sync Status: 98% (41 synced to HubSpot, 1 error)
- High-Intent: 7 (CFO/CEO, 500+ employees)

**Registration List (Table)**:

| Name | Email | Company | Job Title | Registered | Status | Sync Status |
|------|-------|---------|-----------|------------|--------|-------------|
| Jane Doe | jane.doe@acme.com | Acme Corp | CFO | Dec 1, 2:32 PM | Registered | ✅ Synced |
| John Smith | john.smith@startup.io | Startup Inc | CEO | Dec 1, 3:45 PM | Registered | ✅ Synced |
| Bob Wilson | bob@example.com | Example Co | VP Finance | Dec 2, 10:12 AM | Registered | ❌ Error |

**Filters**:
- Job Title: (dropdown: All | CFO | CEO | VP Finance | etc.)
- Company Size: (dropdown: All | 1-50 | 51-200 | 201-500 | 500+)
- Sync Status: (dropdown: All | Synced | Pending | Error)

**Actions**:
- Button: "Export as CSV" (download all registrations)
- Button: "Retry Failed Syncs" (manually trigger HubSpot sync for error rows)
- Button: "Upload Additional CSV" (add more registrants to this event)

**Sync Error Details**:
- Click "❌ Error" → Modal shows error message
- Example: "HubSpot API error: Invalid email format"
- Action: "Edit Contact" → Fix email → "Retry Sync"

**Charts**:
- Registrations Over Time (line chart, daily registration count)
- Top Companies (bar chart, which companies have most registrants)
- Job Title Breakdown (pie chart, CFO vs. VP vs. Director, etc.)

### Feature 5: Field Mapping Manager (Admin Only)

**Purpose**: Configure CSV mapping presets for different partner sources.

**Mapping List (Table)**:

| Preset Name | Created By | Created Date | Last Used | Actions |
|-------------|------------|--------------|-----------|---------|
| Stripe Partner Events | Christopher | Nov 1, 2025 | Dec 5, 2025 | Edit \| Delete \| Test |
| Mercury Co-hosted Events | Christopher | Nov 15, 2025 | Never | Edit \| Delete \| Test |
| Zoom Webinars | Christopher | Oct 20, 2025 | Dec 10, 2025 | Edit \| Delete \| Test |

**Create/Edit Mapping Form**:

| Source Column (CSV) | Target Field (Airtable) | Transformation | Required |
|---------------------|-------------------------|----------------|----------|
| attendee_email_address | email | Lowercase | ✅ |
| attendee_first_name | first_name | None | ✅ |
| attendee_last_name | last_name | None | ✅ |
| company_name | company | None | ❌ |
| title | job_title | None | ❌ |
| registration_timestamp | registration_date | Date Format: MM/DD/YYYY → ISO 8601 | ✅ |

**Transformation Options**:
- None
- Lowercase
- Uppercase
- Titlecase
- Date Format: (dropdown of common formats)
- Phone Format: E.164
- Custom (JavaScript function, advanced)

**Test Mapping**:
- Upload sample CSV (5-10 rows)
- Show preview of transformed data
- Verify output looks correct before saving preset

---

## Technical Architecture

### Frontend Stack

**Framework**: Next.js 14 (App Router)
**Styling**: Tailwind CSS
**UI Components**: shadcn/ui (React components)
**State Management**: React Query (server state) + Zustand (client state)
**Authentication**: Clerk or Auth0 (SSO with Google Workspace)
**File Upload**: React Dropzone
**Charts**: Recharts or Chart.js
**CSV Parsing**: Papa Parse

**Why Next.js?**
- Fast, modern React framework
- Server-side rendering (SSR) for better performance
- API routes (serverless functions) for backend logic
- Vercel deployment (matches Rho's existing stack)

### Backend Stack

**API**: Next.js API Routes (serverless functions)
**Database**: Airtable (existing source of truth, no new DB needed)
**Background Jobs**: Inngest or Trigger.dev (handle long-running CSV imports)
**File Storage**: AWS S3 or Vercel Blob (temporary CSV storage before processing)
**Monitoring**: Sentry (error tracking), Vercel Analytics (performance)

**Why Serverless?**
- Low ops overhead (no servers to manage)
- Auto-scaling (handles 50+ events/month easily)
- Cost-efficient (pay per request, not fixed hosting)

### API Endpoints

**Events**:
- `GET /api/events` - List all events (with filters)
- `GET /api/events/:id` - Get single event details
- `POST /api/events` - Create new event
- `PATCH /api/events/:id` - Update event
- `DELETE /api/events/:id` - Delete event (soft delete, set status = "Cancelled")

**Registrations**:
- `GET /api/events/:id/registrations` - List registrations for event
- `POST /api/events/:id/registrations` - Create single registration (manual entry)
- `POST /api/events/:id/registrations/import` - Bulk CSV import (triggers background job)
- `GET /api/events/:id/registrations/export` - Export as CSV

**Field Mappings**:
- `GET /api/field-mappings` - List all presets
- `GET /api/field-mappings/:id` - Get single preset
- `POST /api/field-mappings` - Create preset
- `PATCH /api/field-mappings/:id` - Update preset
- `DELETE /api/field-mappings/:id` - Delete preset

**CSV Import (Background Job)**:
- `POST /api/csv-import` - Trigger import job
- `GET /api/csv-import/:jobId` - Check job status
- Returns: `{ status: "pending" | "processing" | "completed" | "failed", progress: 45, totalRows: 234, processedRows: 105, errors: [...] }`

### Data Flow

```
User uploads CSV
    ↓
Next.js API Route: /api/events/:id/registrations/import
    ↓
Upload CSV to S3 (temporary storage)
    ↓
Trigger background job (Inngest)
    ↓
Job: Parse CSV with Papa Parse
    ↓
Job: Apply field mapping transformations
    ↓
Job: Validate each row (email format, required fields)
    ↓
Job: Insert valid rows to Airtable (batch of 10 at a time)
    ↓
Job: Update progress (emit status updates to client via SSE)
    ↓
Client: Display progress bar in real-time
    ↓
Job Complete: Return summary (234 imported, 12 duplicates, 2 errors)
    ↓
Client: Show success message
```

---

## User Interface Mockups

### Dashboard (Desktop View)

```
┌──────────────────────────────────────────────────────────────┐
│  Rho Events Management                       [Christopher ▼] │
├──────────────────────────────────────────────────────────────┤
│  📊 Events Dashboard                                          │
│                                                                │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐│
│  │ Upcoming   │ │ Total Reg  │ │ Avg Reg/   │ │ High-Intent││
│  │ Events     │ │ This Month │ │ Event      │ │ Leads      ││
│  │    8       │ │    542     │ │    67.8    │ │    23      ││
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘│
│                                                                │
│  [Create Event] [Upload CSV]        Filters: [This Month ▼]  │
│                                              [All Types ▼]    │
│  ┌──────────────────────────────────────────────────────────┐│
│  │ Event Name              │ Type │ Date │ Reg │ Status     ││
│  ├──────────────────────────────────────────────────────────┤│
│  │ CFO Roundtable: Banking │ Luma │ 12/15│ 42  │ 🟢 Reg Open││
│  │ Rho x Stripe Panel      │ Part │ 12/20│ 68  │ 🟢 Reg Open││
│  │ Cash Management Webinar │ Web  │ 12/18│ 156 │ ✅ Complete││
│  └──────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────┘
```

### CSV Upload Wizard (Mobile Responsive)

```
┌────────────────────────────┐
│  📤 Upload Event CSV        │
├────────────────────────────┤
│  Step 2 of 6: Upload File  │
│                             │
│  ┌────────────────────────┐│
│  │  Drop CSV file here    ││
│  │  or click to browse    ││
│  │                         ││
│  │  Max 10 MB             ││
│  └────────────────────────┘│
│                             │
│  [Cancel]       [Next →]   │
└────────────────────────────┘
```

---

## Security & Access Control

### Authentication

**SSO Integration**: Google Workspace
- Users log in with @rho.co email
- Auth0/Clerk handles OAuth flow
- Session stored in encrypted HTTP-only cookie

### Role-Based Access Control (RBAC)

| Role | Permissions |
|------|-------------|
| **Admin** (Christopher) | Full access: Create/edit/delete events, configure field mappings, view all data, export all |
| **Power User** (Growth, Partnerships) | Create events, upload CSVs, view all events, export own events |
| **Viewer** (Sales, SDRs) | View events, view registrations, export registrations (read-only) |

**Implementation**:
```javascript
// Middleware in Next.js API routes
async function requireRole(req, allowedRoles) {
  const user = await auth.getUser(req);
  if (!allowedRoles.includes(user.role)) {
    throw new ForbiddenError();
  }
}

// Usage
app.post('/api/events', requireRole(['admin', 'power_user']), createEvent);
app.get('/api/events', requireRole(['admin', 'power_user', 'viewer']), listEvents);
```

### Data Security

**PII Protection**:
- All API requests over HTTPS (TLS 1.3)
- Airtable API keys stored in environment variables (never committed to Git)
- CSV files deleted from S3 after import completes (7-day retention for debugging)

**Audit Logs**:
- Log all actions: Who, What, When
- Store in Airtable "Audit Log" table
- Example: "Christopher Cooper uploaded CSV for Event EV-2025-042 (234 rows) at 2025-12-15 14:32"

---

## Development Roadmap

### Week 1-2: Foundation

**Tasks**:
- Set up Next.js project, Tailwind CSS, shadcn/ui
- Implement Auth0/Clerk authentication (SSO with Google)
- Build Event Creation Form (frontend + API)
- Build Event Dashboard (list view, filters)

**Deliverable**: Basic UI with authentication, can create/view events

### Week 3-4: CSV Upload

**Tasks**:
- Build CSV Upload Wizard (6-step flow)
- Implement Papa Parse integration
- Build Field Mapping UI (select preset, manual mapping)
- Implement background job (Inngest) for CSV processing
- Real-time progress updates (Server-Sent Events or polling)

**Deliverable**: Can upload CSV, apply mapping, import to Airtable

### Week 5-6: Event Detail & Registrations

**Tasks**:
- Build Event Detail page (registration list, charts)
- Implement filters (job title, company size, sync status)
- Build export functionality (download CSV)
- Add sync error handling (retry failed syncs)

**Deliverable**: Full event management (view, export, retry errors)

### Week 7: Field Mapping Manager

**Tasks**:
- Build Field Mapping Manager (admin only)
- CRUD operations for presets
- Test mapping functionality (upload sample CSV, preview)

**Deliverable**: Admins can create/edit mapping presets

### Week 8: UAT & Polish

**Tasks**:
- User acceptance testing with Christopher, Emily (Partnerships), Sales team
- Bug fixes
- Performance optimization (lazy loading, caching)
- Mobile responsiveness testing

**Deliverable**: Production-ready application

---

## Success Metrics

### Week 8 Targets (Launch)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **User Adoption** | 10+ users registered | Auth0/Clerk user count |
| **CSV Uploads** | 5+ uploads by non-Christopher users | Audit logs |
| **Time Saved** | 80% reduction (16 hrs → 3 hrs/month) | Christopher time tracking |
| **System Uptime** | 99.5% | Vercel monitoring |

### Q2 2026 Targets (Mature Product)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Events Managed** | 50+ events/month | Event count in Airtable |
| **CSV Uploads** | 30+ uploads/month (80% by non-admins) | Audit logs (group by user role) |
| **User Satisfaction** | 4.5/5 rating | Quarterly survey |
| **Error Rate** | <5% of CSV imports have errors | Job status logs |

---

## Cost Estimate

### Development (6-8 weeks)

| Resource | Hours | Rate | Cost |
|----------|-------|------|------|
| **Frontend Developer** | 120 hrs | $150/hr | $18,000 |
| **Backend Developer** | 80 hrs | $150/hr | $12,000 |
| **UI/UX Designer** | 20 hrs | $100/hr | $2,000 |
| **QA Tester** | 20 hrs | $75/hr | $1,500 |
| **Project Manager** | 20 hrs | $100/hr | $2,000 |
| **Total** | | | **$35,500** |

**Negotiated Budget**: $25-30K (if using freelancers or offshore team)

### Ongoing Costs (Monthly)

| Service | Cost | Notes |
|---------|------|-------|
| **Vercel Hosting** | $20/month | Pro plan (supports team, unlimited bandwidth) |
| **Auth0/Clerk** | $25/month | Up to 25 users |
| **Inngest** | $0-50/month | Free tier: 10K events/month, likely sufficient |
| **AWS S3** | $5/month | Temporary CSV storage |
| **Sentry** | $26/month | Error tracking (up to 5K events) |
| **Total** | **~$76/month** | **~$900/year** |

**ROI**: $30K one-time + $900/year → Saves 16 hrs/month ($1,200/month labor cost) → Breakeven in 2 months

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Dev team unavailable** | High (project delayed) | Start recruiting in Q4 2025, have backup vendor |
| **Airtable API rate limits** | Medium (slow imports) | Batch API calls, exponential backoff, upgrade to Enterprise plan if needed |
| **User adoption low** | Medium (tool unused, wasted investment) | Comprehensive training, mandate CSV uploads via UI (deprecate manual Airtable entry) |
| **Security breach** | High (PII exposed) | Penetration testing before launch, regular security audits |

---

## Out of Scope (Future Enhancements)

**Phase 2 (Q2 2026)**:
1. **Mobile App** (iOS/Android native)
2. **Slack Integration** (slash commands: `/rho-event create`)
3. **Automated Event Reminders** (email registrants 1 day before event)
4. **Event Check-In App** (iPad app for on-site check-in, auto-update attendance)
5. **Predictive Analytics** (ML model to predict registration → attendance conversion)

---

**Last Updated**: 2025-11-12
**Document Owner**: Christopher Cooper
**Next Review**: Q1 2026 (before project kickoff)

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
