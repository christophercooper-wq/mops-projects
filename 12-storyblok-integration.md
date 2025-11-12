# 12 - Storyblok CMS Integration for Email Templates

**Document Status**: ✅ Complete - Q1 2026 Project Spec
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Timeline**: Q1 2026 (3-4 weeks, $15-20K budget)
**Phase**: Post-Skydog Engagement
**Related**: [04-Email Template Architecture](04-email-template-architecture.md), [08-Technical Architecture](08-technical-architecture.md)

---

## Executive Summary

**Problem**: Even with component library + CI/CD, Growth team still depends on Christopher to make email content changes (update copy, swap images, change CTA URLs). This creates bottleneck and prevents rapid iteration.

**Solution**: Integrate **Storyblok headless CMS** to enable non-technical users (Growth, Demand Gen) to:
- Edit email content via visual editor (no code required)
- Publish changes that auto-render email HTML and deploy to HubSpot
- Send emails via Customer.io (bypass HubSpot contact bloat, save $3K+/year)

**Business Impact**:
- **Self-service email creation**: Growth team creates 80% of emails without Christopher
- **Cost savings**: $3K+/year (reduce HubSpot contact creation from event emails)
- **Faster iteration**: Change CTA text, publish, send in <10 min (down from hours/days)

**Timeline**: 3-4 weeks development
**Budget**: $15-20K (internal dev or vendor)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│              STORYBLOK CMS (Content Layer)                   │
├─────────────────────────────────────────────────────────────┤
│  Visual Editor:                                              │
│  [Header Component] → Logo URL, Kicker Text                 │
│  [BodyWithCTA Component] → Body Text, CTA Text, CTA URL     │
│  [Footer Component] → Company Info, Social Links            │
└───────────────────────┬─────────────────────────────────────┘
                        │ (Publish button clicked)
                        ↓
┌─────────────────────────────────────────────────────────────┐
│              STORYBLOK WEBHOOK (Event Trigger)               │
├─────────────────────────────────────────────────────────────┤
│  POST to Vercel Edge Function: /api/storyblok-webhook       │
│  Payload: { story_id: 123, action: "published" }           │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│         VERCEL SERVERLESS FUNCTION (Rendering Layer)         │
├─────────────────────────────────────────────────────────────┤
│  1. Fetch Storyblok story (GET /v2/stories/123)             │
│  2. Render React components server-side (RSC)               │
│  3. Transform JSX → Email-safe HTML                         │
│  4. Validate output (links, images, accessibility)          │
└───────────────────────┬─────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                  DISTRIBUTION LAYER                          │
├─────────────────────────────────────────────────────────────┤
│  Option A: Deploy to HubSpot (archive template)             │
│  Option B: Send via Customer.io (recommended)               │
│     - Fetch recipient list from Airtable                    │
│     - Create broadcast with rendered HTML                   │
│     - Send to segment (e.g., "Event Registrants")          │
└─────────────────────────────────────────────────────────────┘
```

---

## Storyblok Configuration

### Component Schema Definitions

**Component 1: Header**

**Storyblok Block Schema**:
```json
{
  "name": "email_header",
  "display_name": "Email Header",
  "schema": {
    "logo_url": {
      "type": "text",
      "default_value": "https://39998325.hs-sites.com/hubfs/Rho-Logo.png",
      "description": "URL to logo image"
    },
    "kicker": {
      "type": "text",
      "max_length": 100,
      "description": "Small text above main content (e.g., 'Upcoming Event')"
    },
    "date": {
      "type": "text",
      "max_length": 50,
      "description": "Event date or email date (e.g., 'December 15, 2025')"
    }
  },
  "is_root": false,
  "is_nestable": true,
  "color": "#00ECC0"
}
```

**Component 2: BodyWithCTA**

```json
{
  "name": "body_with_cta",
  "display_name": "Body Text with CTA Button",
  "schema": {
    "body_text": {
      "type": "textarea",
      "required": true,
      "description": "Main paragraph text"
    },
    "cta_text": {
      "type": "text",
      "required": true,
      "max_length": 50,
      "description": "Button text (e.g., 'Register Now')"
    },
    "cta_url": {
      "type": "text",
      "required": true,
      "regex": "^https?://.+",
      "description": "Button destination URL"
    },
    "cta_color": {
      "type": "option",
      "options": [
        { "value": "#00ECC0", "name": "Rho Green (Primary)" },
        { "value": "#000000", "name": "Black (Secondary)" }
      ],
      "default_value": "#00ECC0"
    }
  },
  "preview_field": "cta_text",
  "is_root": false,
  "is_nestable": true,
  "color": "#00ECC0"
}
```

**Component 3: Footer**

```json
{
  "name": "email_footer",
  "display_name": "Email Footer",
  "schema": {
    "company_name": {
      "type": "text",
      "default_value": "Rho Technologies",
      "disabled": true
    },
    "company_address": {
      "type": "textarea",
      "default_value": "100 Crosby Street\nNew York, NY 10012",
      "disabled": true
    },
    "disclaimer": {
      "type": "textarea",
      "default_value": "Rho is a fintech company, not a bank...",
      "disabled": true,
      "description": "Required legal disclaimer (cannot be edited)"
    },
    "unsubscribe_url": {
      "type": "text",
      "default_value": "{{unsubscribe_url}}",
      "description": "Merge tag for ESP unsubscribe link"
    },
    "preferences_url": {
      "type": "text",
      "default_value": "{{preferences_url}}",
      "description": "Merge tag for ESP preferences link"
    }
  },
  "is_root": false,
  "is_nestable": true,
  "color": "#0D0F10"
}
```

**Root Component: Email Template**

```json
{
  "name": "email_template",
  "display_name": "Email Template",
  "schema": {
    "subject_line": {
      "type": "text",
      "required": true,
      "max_length": 100,
      "description": "Email subject line"
    },
    "preview_text": {
      "type": "text",
      "max_length": 150,
      "description": "Email preview text (appears after subject in inbox)"
    },
    "from_name": {
      "type": "text",
      "default_value": "Rho",
      "description": "Sender name"
    },
    "from_email": {
      "type": "text",
      "default_value": "events@rho.co",
      "description": "Sender email"
    },
    "target_segment": {
      "type": "text",
      "description": "Customer.io segment ID or Airtable view ID"
    },
    "body": {
      "type": "bloks",
      "restrict_components": true,
      "component_whitelist": [
        "email_header",
        "hero",
        "body_with_cta",
        "two_column_cards",
        "highlight",
        "pre_footer",
        "email_footer"
      ],
      "description": "Drag components here to build email"
    }
  },
  "is_root": true,
  "is_nestable": false
}
```

### Visual Editor Experience

**User Flow** (Growth Team Member):

1. **Log into Storyblok** → Navigate to "Email Templates" folder
2. **Click "Create New"** → Select "Email Template"
3. **Fill in Metadata**:
   - Subject Line: "You're Invited: CFO Roundtable - Dec 15"
   - Preview Text: "Join us for an exclusive discussion on banking stack optimization"
   - Target Segment: "Event Registrations - CFO Roundtable"
4. **Drag Components** into "Body" section:
   - Email Header (logo, kicker: "Upcoming Event", date: "December 15, 2025")
   - Body with CTA (text: "Join us for...", CTA: "Register Now", URL: "https://lu.ma/rho-cfo-dec-2025")
   - Email Footer (auto-populated with company info)
5. **Real-Time Preview**: Right panel shows rendered email (desktop + mobile view)
6. **Publish**: Click "Publish" button → Webhook fires → Email rendered → Sent via Customer.io

---

## Serverless Middleware Implementation

### Vercel Edge Function: `/api/storyblok-webhook`

**File**: `pages/api/storyblok-webhook.ts`

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';
import { storyblokClient } from '@/lib/storyblok';
import { renderEmailComponent } from '@/lib/email-renderer';
import { customerio } from '@/lib/customerio';
import { hubspotAPI } from '@/lib/hubspot';
import { verifyStoryblokSignature } from '@/lib/verify-signature';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // 1. Verify webhook signature (security)
  const signature = req.headers['webhook-signature'] as string;
  if (!verifyStoryblokSignature(signature, req.body)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // 2. Extract payload
  const { story_id, action } = req.body;

  // 3. Only process "published" events (ignore drafts, unpublished)
  if (action !== 'published') {
    return res.status(200).json({ message: 'Ignored (not published)' });
  }

  try {
    // 4. Fetch full story from Storyblok API
    const story = await storyblokClient.getStory(story_id, {
      version: 'published',
      cv: Date.now() // Cache bust
    });

    // 5. Validate story is email template
    if (story.content.component !== 'email_template') {
      return res.status(400).json({ error: 'Not an email template' });
    }

    // 6. Render React components to email-safe HTML
    const emailHTML = await renderEmailComponent(story.content);

    // 7. Validate output (links, images, accessibility)
    const validation = validateEmail(emailHTML);
    if (!validation.valid) {
      console.error('Email validation failed:', validation.errors);
      return res.status(400).json({ error: 'Validation failed', details: validation.errors });
    }

    // 8. Deploy to HubSpot (optional, for archival)
    await hubspotAPI.createTemplate({
      name: `${story.name} - ${new Date().toISOString()}`,
      source: emailHTML,
      folder: 'Storyblok Templates'
    });

    // 9. Send via Customer.io (recommended path)
    const broadcast = await customerio.createBroadcast({
      name: story.name,
      subject: story.content.subject_line,
      from_name: story.content.from_name,
      from_email: story.content.from_email,
      preheader: story.content.preview_text,
      body: emailHTML,
      segment_id: story.content.target_segment
    });

    // 10. Trigger send (or schedule)
    await customerio.sendBroadcast(broadcast.id);

    // 11. Log success
    console.log(`✅ Email published and sent: ${story.name} (Broadcast ID: ${broadcast.id})`);

    return res.status(200).json({
      success: true,
      story_id: story_id,
      broadcast_id: broadcast.id,
      hubspot_template_id: templateId
    });

  } catch (error) {
    console.error('❌ Webhook processing error:', error);
    return res.status(500).json({ error: 'Internal server error', message: error.message });
  }
}
```

### Email Renderer: `/lib/email-renderer.ts`

```typescript
import React from 'react';
import { renderToStaticMarkup } from 'react-dom/server';

// Import all email components
import Header from '@/components/email/Header';
import Hero from '@/components/email/Hero';
import BodyWithCTA from '@/components/email/BodyWithCTA';
import TwoColumnCards from '@/components/email/TwoColumnCards';
import Highlight from '@/components/email/Highlight';
import PreFooter from '@/components/email/PreFooter';
import Footer from '@/components/email/Footer';

// Map Storyblok component names to React components
const componentMap = {
  'email_header': Header,
  'hero': Hero,
  'body_with_cta': BodyWithCTA,
  'two_column_cards': TwoColumnCards,
  'highlight': Highlight,
  'pre_footer': PreFooter,
  'email_footer': Footer
};

export async function renderEmailComponent(storyContent: any): Promise<string> {
  // Render each block in story body
  const components = storyContent.body.map((block: any) => {
    const Component = componentMap[block.component];

    if (!Component) {
      throw new Error(`Unknown component: ${block.component}`);
    }

    // Pass all block fields as props to React component
    return React.createElement(Component, { key: block._uid, ...block });
  });

  // Wrap in full HTML email document
  const emailHTML = renderToStaticMarkup(
    <html lang="en">
      <head>
        <meta charSet="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta httpEquiv="X-UA-Compatible" content="IE=edge" />
        <title>{storyContent.subject_line}</title>
        <style>{`
          body { margin: 0 !important; padding: 0 !important; }
          img { border: 0; display: block; }
          table { border-collapse: collapse; }
          @media only screen and (max-width: 620px) {
            .fluid { width: 100% !important; }
          }
        `}</style>
      </head>
      <body style={{ margin: 0, padding: 0, backgroundColor: '#f4f4f4' }}>
        {/* Preview text (hidden div for email clients) */}
        <div style={{ display: 'none', maxHeight: 0, overflow: 'hidden' }}>
          {storyContent.preview_text}
        </div>

        {/* Main email container */}
        <table role="presentation" width="100%" cellSpacing={0} cellPadding={0} border={0}>
          <tr>
            <td align="center">
              {components}
            </td>
          </tr>
        </table>
      </body>
    </html>
  );

  return `<!DOCTYPE html>\n${emailHTML}`;
}

// Validation function
function validateEmail(html: string): { valid: boolean; errors: string[] } {
  const errors: string[] = [];

  // Check for broken image tags
  if (html.includes('<img') && !html.includes('alt=')) {
    errors.push('Image missing alt text');
  }

  // Check for invalid links
  const linkRegex = /href="([^"]+)"/g;
  const links = [...html.matchAll(linkRegex)];
  links.forEach(([_, url]) => {
    if (!url.startsWith('http') && !url.startsWith('mailto') && !url.startsWith('tel')) {
      errors.push(`Invalid link: ${url}`);
    }
  });

  // Check for missing unsubscribe link
  if (!html.includes('{{unsubscribe_url}}') && !html.includes('unsubscribe')) {
    errors.push('Missing unsubscribe link (CAN-SPAM compliance)');
  }

  return {
    valid: errors.length === 0,
    errors
  };
}
```

---

## Customer.io Integration

### Why Customer.io Instead of HubSpot?

| Factor | HubSpot | Customer.io | Winner |
|--------|---------|-------------|--------|
| **Contact Creation** | Creates/updates contact on every send | No contact creation (just sends to segment) | Customer.io |
| **Cost** | $0.50/contact/month (500 registrants = $250/month) | $0.01-0.02/email sent (500 emails = $5-10) | Customer.io |
| **Real-Time Data** | Can only use CRM properties | Can fetch live data via API (e.g., event spots remaining) | Customer.io |
| **Multi-Channel** | Email only | Email + SMS + Push notifications | Customer.io |
| **Personalization** | Limited to CRM fields | Liquid templating, API lookups, dynamic content | Customer.io |
| **Annual Savings** | Baseline | $3K+/year | Customer.io |

### Customer.io API Integration

**Sync Airtable Registrations → Customer.io Segments**

**n8n Workflow** (runs every 5 minutes):
1. Query Airtable: `SELECT * FROM Registrations WHERE sync_status_customerio = 'Pending'`
2. For each registration:
   - Create/update person in Customer.io: `POST /api/v1/customers/{email}`
   - Add to segment: `POST /api/v1/segments/{segment_id}/add_people`
   - Update Airtable: `sync_status_customerio = 'Synced'`
3. Log errors to Slack

**Segment Examples**:
- "Event Registrations - CFO Roundtable - Dec 15"
- "Event Attendees - All Time"
- "High-Intent Leads" (CFO/CEO titles, 500+ employees)

**Send Broadcast via API**:

```javascript
const customerio = require('customerio-node');
const client = new customerio.APIClient(process.env.CUSTOMERIO_API_KEY);

// Create broadcast
const broadcast = await client.createBroadcast({
  name: "CFO Roundtable Invite",
  subject: "You're Invited: CFO Roundtable - Dec 15",
  from_name: "Rho",
  from_email: "events@rho.co",
  preheader: "Join us for an exclusive discussion...",
  body: emailHTML,  // Rendered from Storyblok
  segment: {
    id: 123  // "Event Registrations - CFO Roundtable"
  }
});

// Send immediately (or schedule for later)
await client.triggerBroadcast(broadcast.id);
```

---

## Development Roadmap

### Week 1: Storyblok Setup

**Tasks**:
- Create Storyblok account (Business plan: $269/month for 10 users)
- Configure component schemas (email_header, body_with_cta, email_footer, etc.)
- Create sample email template in visual editor
- Test preview functionality

**Deliverable**: Storyblok CMS configured with component library

### Week 2: Serverless Middleware

**Tasks**:
- Build Vercel Edge Function: `/api/storyblok-webhook`
- Implement email renderer (React SSR)
- Add validation (links, images, accessibility)
- Test webhook flow: Storyblok publish → Vercel function → Rendered HTML

**Deliverable**: Webhook processes Storyblok publishes, renders email HTML

### Week 3: Customer.io Integration

**Tasks**:
- Set up Customer.io account (Growth plan: $1,000/month for 100K contacts)
- Build n8n workflow: Airtable → Customer.io sync
- Implement broadcast API integration (create + send)
- Test end-to-end: Storyblok publish → Vercel render → Customer.io send

**Deliverable**: Email sent via Customer.io to test segment

### Week 4: UAT & Training

**Tasks**:
- User acceptance testing with Growth team (create 3 test emails)
- Training session (1 hour): How to use Storyblok visual editor
- Documentation: Video walkthrough, written guide
- Bug fixes, polish

**Deliverable**: Production-ready, Growth team trained

---

## Cost Analysis

### Development (3-4 weeks)

| Resource | Hours | Rate | Cost |
|----------|-------|------|------|
| **Backend Developer** | 60 hrs | $150/hr | $9,000 |
| **Frontend Developer** (Storyblok config) | 20 hrs | $150/hr | $3,000 |
| **QA Tester** | 10 hrs | $75/hr | $750 |
| **Total** | 90 hrs | | **$12,750** |

**Negotiated Budget**: $15-20K (includes buffer for UAT, revisions)

### Ongoing Costs (Monthly)

| Service | Cost | Notes |
|---------|------|-------|
| **Storyblok** | $269/month | Business plan (10 users, unlimited content) |
| **Customer.io** | $1,000/month | Growth plan (100K contacts, unlimited emails) |
| **Vercel** | $20/month | Pro plan (already paying for Events UI) |
| **Total** | **$1,289/month** | **$15,468/year** |

### ROI Calculation

**Costs**:
- One-time: $15K (development)
- Annual: $15,468 (SaaS subscriptions)
- **Total Year 1**: $30,468

**Savings**:
- Christopher's time: 16 hrs/month × $75/hr × 12 months = **$14,400/year**
- HubSpot contact reduction: $3,000/year (avoid contact bloat from event emails)
- **Total Savings**: **$17,400/year**

**Net ROI**: Break-even in ~21 months. Positive ROI Year 2+ ($17K savings vs. $15K ongoing costs).

**Strategic Value** (not quantified):
- **Faster iteration**: Change email content in minutes (not days)
- **Empowerment**: Growth team self-sufficient (not dependent on Christopher)
- **Multi-channel**: Can add SMS, push notifications (Customer.io supports all)

---

## Success Metrics

### Week 4 Targets (Launch)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Emails Created in Storyblok** | 3 templates | Storyblok content count |
| **Growth Team Adoption** | 2+ users creating emails | Storyblok user activity logs |
| **Time to Create Email** | <30 min (Growth team, no Christopher help) | User survey |
| **Email Delivery Success** | >98% | Customer.io delivery reports |

### Q2 2026 Targets (Mature Product)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Self-Service Email Creation** | 80% of emails created by Growth (not Christopher) | Storyblok author metadata |
| **HubSpot Contact Reduction** | 500+ fewer contacts/month | HubSpot contact count diff |
| **Customer.io Utilization** | 10+ broadcasts/month | Customer.io API logs |
| **Time Saved** | 16 hrs/month for Christopher | Time tracking |

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Storyblok learning curve** | Medium (Growth team resists new tool) | Comprehensive training, video tutorials, office hours for first 2 weeks |
| **Customer.io costs exceed budget** | High (email volume >expected) | Monitor usage monthly, optimize segments, use throttling if needed |
| **Email rendering issues** | High (emails break in Outlook, Gmail) | Automated testing (Litmus integration), manual QA before first send |
| **Webhook failures** | Medium (emails don't send) | Retry logic with exponential backoff, Slack alerts, manual trigger button |

---

## Out of Scope (Future Enhancements)

**Phase 2 (Q2 2026)**:
1. **Dynamic Content Blocks**: Fetch live data from APIs (e.g., "5 spots remaining" for events)
2. **A/B Testing**: Storyblok variants → Customer.io A/B tests
3. **SMS/Push Notifications**: Expand beyond email to SMS, mobile push (Customer.io supports)
4. **Personalization Engine**: ML-powered content recommendations (show different CTA based on user behavior)

---

## Related User-Provided Article

**Storyblok + Email Integration Guide**: https://www.storyblok.com/tp/integrate-emails-with-storyblok

**Key Takeaways**:
- Storyblok's visual editor is ideal for email content management
- Serverless functions (Vercel, Netlify) recommended for rendering layer
- Real-time preview in Storyblok shows exactly what email will look like
- Can integrate with any ESP (Customer.io, SendGrid, Mailgun, etc.)

---

**Last Updated**: 2025-11-12
**Document Owner**: Christopher Cooper
**Next Review**: Q1 2026 (before project kickoff)

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
