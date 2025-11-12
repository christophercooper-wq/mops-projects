# 04 - Email Template Architecture & Storyblok Integration

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Related Documents**: [10-Component Library CI/CD](10-component-library-cicd.md), [01-Root Cause](01-root-cause-analysis.md)

---

## Executive Summary

Rho's current email template system relies on manual HubSpot drag-and-drop editing, creating:
- **Inconsistent Branding**: Each email built from scratch, no component reusability
- **Slow Production**: 2-3 hours to build a single event email
- **No Version Control**: Changes made directly in HubSpot, no audit trail
- **Developer Bottleneck**: Christopher is single point of failure for all email builds

**Proposed Solution**: Storyblok-Powered Component Library with CI/CD to HubSpot

### Three-Phase Approach

**Phase 1 (Week 1-3)**: Component Library Foundation
- Build React JSX component library (Header, CTA, Footer, etc.)
- Create preview site (`index.html`) for visual QA
- Extract email-safe HTML for manual HubSpot deployment

**Phase 2 (Week 4-6)**: CI/CD Automation
- GitHub Actions pipeline: commit → build → deploy to HubSpot Design Manager API
- Automated testing (email client rendering, link validation)
- Version tagging and rollback capability

**Phase 3 (Q1 2026)**: Storyblok CMS Integration
- Headless CMS for content editing (non-technical users can update copy/images)
- Serverless middleware: Storyblok webhook → render email HTML → HubSpot API
- Bypass HubSpot contact creation bloat (send via Customer.io or Sendgrid)

**Timeline**:
- Phase 1 complete by Week 3 (ready for Demand Gen hire)
- Phase 2 complete by Week 6 (automated deployments)
- Phase 3 scoped for Q1 2026 (post-Skydog engagement)

---

## Current State: Pain Points

### Problem 1: Manual Email Building Process

**Current Workflow** (2-3 hours per email):
```
Growth Team: "We need event invite email"
    ↓ [Request via Slack]
Christopher: Draft in HubSpot drag-and-drop editor
    ↓ [30 min: Build layout]
Christopher: Add copy, images, CTA
    ↓ [30 min: Content insertion]
Christopher: Test send to self
    ↓ [15 min: QA in Gmail, Outlook]
Christopher: Fix rendering issues
    ↓ [30 min: Outlook button fix, spacing adjustments]
Christopher: Send test to stakeholder
    ↓ [30 min: Stakeholder review, change requests]
Christopher: Make edits, re-test
    ↓ [15 min: Final QA]
Growth Team: Schedule/send
```

**Pain Points**:
- Christopher is bottleneck for every single email
- No reusable components (rebuild CTA buttons, headers, footers every time)
- HubSpot drag-and-drop editor fragile (one wrong click breaks layout)
- Testing is manual and time-consuming (send to 5+ email clients)
- No version history (can't roll back to previous email design)

**Volume Impact**:
- Current: 8-12 emails/month (manageable, barely)
- Q1 2026 Projection: 30-40 emails/month (unsustainable without automation)

### Problem 2: Inconsistent Branding

**Example**: CFO Roundtable email (Dec 2025) vs. Product Launch email (Nov 2025)

| Element | CFO Email | Product Email | Issue |
|---------|-----------|---------------|-------|
| **CTA Button Color** | `#00ECC0` (correct) | `#00D4A8` (wrong shade) | No brand enforcement |
| **Font Family** | Helvetica | Arial | Inconsistent typography |
| **Header Logo Size** | 120px width | 150px width | No standard |
| **Footer Disclaimer** | Full legal text | Abbreviated text | Compliance risk |
| **Button Padding** | 12px vertical | 16px vertical | Visual inconsistency |

**Root Cause**: No single source of truth for email components.

### Problem 3: HubSpot Template Limitations

**Technical Constraints**:
1. **Contact Creation Required**: Every email send creates/updates HubSpot contact → inflates contact count → $$$ (Rho pays per contact)
2. **Limited Personalization**: HubSpot tokens only work with CRM data (can't pull from external APIs)
3. **No Real-Time Data**: Can't fetch live event availability, stock prices, or user account data
4. **Slow Editor**: Drag-and-drop builder laggy, especially with image-heavy emails
5. **No Code Reusability**: Custom modules exist, but still require manual setup per email

**Example Impact**:
- Event invite emails sent to 500 registrants → HubSpot creates 500 contacts (many non-qualified)
- Cost: $0.50/contact/month → $250/month → $3K/year just for event emails
- Solution: Send via Customer.io (synced from Airtable), only create HubSpot contact if they engage

---

## Proposed Architecture

### Phase 1: Component Library Foundation (Week 1-3)

**Goal**: Build reusable, email-safe React JSX components with visual preview site.

#### Component Inventory

| Component | Purpose | Props | HubSpot Usage |
|-----------|---------|-------|---------------|
| **Header** | Logo + date/kicker text | `logoUrl`, `kicker`, `date` | Every email |
| **Hero** | Main heading + subheading | `heading`, `subheading`, `backgroundImage` | Announcements, events |
| **BodyWithCTA** | Paragraph + CTA button | `bodyText`, `ctaText`, `ctaUrl` | Every email |
| **TwoColumnCards** | Side-by-side content blocks | `leftCard: {title, text, image}`, `rightCard: {...}` | Case studies, features |
| **Highlight** | Callout box with border | `highlightText`, `borderColor` | Key stats, quotes |
| **PreFooter** | Testimonials or founder quote | `quote`, `author`, `authorTitle`, `authorImage` | Social proof emails |
| **Footer** | Company info, social links, legal | `unsubscribeUrl`, `preferencesUrl` | Every email |

**File Structure**:
```
rho-hubspot-deployment/
├── index.html                          # Component library preview site
├── components/
│   ├── Header.jsx
│   ├── Hero.jsx
│   ├── BodyWithCTA.jsx
│   ├── TwoColumnCards.jsx
│   ├── Highlight.jsx
│   ├── PreFooter.jsx
│   ├── Footer.jsx
│   └── FullEmailTemplate.jsx          # Combines all components
├── templates/
│   ├── event-invite.html               # Rendered output (HubSpot-ready)
│   ├── product-launch.html
│   └── newsletter.html
├── tests/
│   ├── email-client-tests.js           # Litmus API integration
│   └── link-validator.js               # Check all CTAs work
└── README.md                            # Usage docs
```

#### Key Technical Requirements

**Email-Safe HTML**:
- Table-based layouts (no flexbox, CSS Grid)
- Inline styles only (no `<style>` tags in body)
- Outlook VML fallbacks for buttons
- Max-width 600px for mobile responsiveness
- Images: `display: block`, `border: 0`, proper alt text

**Example Component** (BodyWithCTA.jsx):
```jsx
export default function BodyWithCTA({ bodyText, ctaText, ctaUrl }) {
  return (
    <table role="presentation" cellSpacing="0" cellPadding="0" border="0" width="100%">
      <tr>
        <td style={{
          padding: '20px 20px 30px 20px',
          fontFamily: 'Helvetica, Arial, sans-serif',
          fontSize: '16px',
          lineHeight: '24px',
          color: '#000000'
        }}>
          {bodyText}
        </td>
      </tr>
      <tr>
        <td style={{ padding: '0 20px 20px 20px' }}>
          {/* Outlook VML Button */}
          {`<!--[if mso]>
          <v:roundrect xmlns:v="urn:schemas-microsoft-com:vml"
                       xmlns:w="urn:schemas-microsoft-com:office:word"
                       href="${ctaUrl}"
                       style="height:44px;v-text-anchor:middle;width:200px;"
                       arcsize="9%"
                       fillcolor="#00ECC0"
                       strokecolor="#00ECC0">
            <w:anchorlock/>
            <center style="color:#000000;font-family:Helvetica,Arial,sans-serif;font-size:18px;font-weight:700;">
              ${ctaText}
            </center>
          </v:roundrect>
          <![endif]-->`}

          {/* Standard HTML Button */}
          {`<!--[if !mso]><!-->` }
          <a href={ctaUrl} style={{
            display: 'inline-block',
            background: '#00ECC0',
            color: '#000000',
            fontFamily: 'Helvetica, Arial, sans-serif',
            fontSize: '18px',
            fontWeight: '700',
            textDecoration: 'none',
            borderRadius: '4px',
            padding: '12px 30px',
            border: '1px solid #00ECC0'
          }}>
            {ctaText}
          </a>
          {`<!--<![endif]-->`}
        </td>
      </tr>
    </table>
  );
}
```

**Rendered HTML Output** (for HubSpot):
```html
<table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%">
  <tr>
    <td style="padding:20px 20px 30px 20px;font-family:Helvetica,Arial,sans-serif;font-size:16px;line-height:24px;color:#000000">
      Join us for an exclusive CFO roundtable discussion...
    </td>
  </tr>
  <tr>
    <td style="padding:0 20px 20px 20px">
      <!--[if mso]>
      <v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" xmlns:w="urn:schemas-microsoft-com:office:word"
        href="https://lu.ma/rho-cfo-dec-2025" style="height:44px;v-text-anchor:middle;width:200px;" arcsize="9%"
        fillcolor="#00ECC0" strokecolor="#00ECC0">
        <w:anchorlock/>
        <center style="color:#000000;font-family:Helvetica,Arial,sans-serif;font-size:18px;font-weight:700;">
          Register Now
        </center>
      </v:roundrect>
      <![endif]-->
      <!--[if !mso]><!-->
      <a href="https://lu.ma/rho-cfo-dec-2025" style="display:inline-block;background:#00ECC0;color:#000000;font-family:Helvetica,Arial,sans-serif;font-size:18px;font-weight:700;text-decoration:none;border-radius:4px;padding:12px 30px;border:1px solid #00ECC0">
        Register Now
      </a>
      <!--<![endif]-->
    </td>
  </tr>
</table>
```

#### Preview Site (index.html)

**Purpose**: Visual component library for QA, stakeholder review, and code extraction.

**Layout**:
```
┌─────────────────────────────────────────────────────────────┐
│  Rho Email Component Library                                 │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────┐  ┌──────────────────────────────┐    │
│  │                  │  │                              │    │
│  │  FULL EMAIL      │  │  INDIVIDUAL COMPONENTS       │    │
│  │  PREVIEW         │  │                              │    │
│  │  (Sticky)        │  │  1. Header                   │    │
│  │                  │  │     [Preview]                │    │
│  │  [Complete       │  │     [Copy HTML] [Copy JSX]   │    │
│  │   email with     │  │                              │    │
│  │   all components │  │  2. Hero                     │    │
│  │   rendered]      │  │     [Preview]                │    │
│  │                  │  │     [Copy HTML] [Copy JSX]   │    │
│  │                  │  │                              │    │
│  │                  │  │  3. BodyWithCTA              │    │
│  │                  │  │     [Preview]                │    │
│  │                  │  │     [Copy HTML] [Copy JSX]   │    │
│  │                  │  │                              │    │
│  │                  │  │  ... (scrollable)            │    │
│  │                  │  │                              │    │
│  └──────────────────┘  └──────────────────────────────┘    │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

**Key Features**:
- **Live Rendering**: JSX components rendered via Babel standalone
- **Code Extraction**: "Copy HTML" button copies rendered output (for HubSpot paste)
- **Props Editing**: Form inputs to adjust props in real-time (e.g., change CTA text, see update)
- **Responsive Preview**: Toggle mobile/desktop view
- **Dark Mode Test**: Preview email in dark mode (Gmail iOS)

**Tech Stack**:
- React via CDN (unpkg)
- Babel standalone for JSX transformation
- Tailwind CSS for UI (not for email components)
- Prism.js for syntax highlighting

**Usage**:
1. Open `index.html` in browser
2. Scroll to component you need (e.g., "BodyWithCTA")
3. Adjust props via form inputs (change text, URL)
4. Click "Copy HTML" button
5. Paste into HubSpot email editor (or Storyblok later)

### Phase 2: CI/CD Automation (Week 4-6)

**Goal**: Automate component deployment to HubSpot Design Manager API.

#### GitHub Actions Pipeline

**Trigger**: Push to `main` branch or manual workflow dispatch

**Steps**:
1. **Checkout Code**: Fetch repo
2. **Install Dependencies**: `npm install` (React, Babel, testing libs)
3. **Build Components**: Transform JSX → email-safe HTML
   - Use `@babel/standalone` or `jsx-email` library
   - Output to `dist/templates/` directory
4. **Run Tests**:
   - **Link Validator**: Check all `href` attributes are valid URLs
   - **Image Validator**: Verify all `src` URLs return 200 status
   - **Email Client Tests**: Send rendered HTML to Litmus API, get screenshots
   - **Accessibility**: Check alt text on images, sufficient color contrast
5. **Deploy to HubSpot**:
   - Use HubSpot Design Manager API (`/cms/v3/hubdb/tables`)
   - Upload each template as custom module or design manager file
   - Tag with version number (e.g., `v1.2.3`)
6. **Notify Team**:
   - POST to Slack webhook: "#marketing-ops: New email templates deployed (v1.2.3)"
   - Include link to preview site

**GitHub Actions YAML** (`.github/workflows/deploy-email-templates.yml`):
```yaml
name: Deploy Email Templates to HubSpot

on:
  push:
    branches: [ main ]
    paths:
      - 'rho-hubspot-deployment/components/**'
      - 'rho-hubspot-deployment/templates/**'
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: |
          cd rho-hubspot-deployment
          npm install

      - name: Build email templates
        run: |
          cd rho-hubspot-deployment
          npm run build  # Transforms JSX → HTML

      - name: Run tests
        run: |
          cd rho-hubspot-deployment
          npm test  # Link validation, image checks

      - name: Deploy to HubSpot
        env:
          HUBSPOT_API_KEY: ${{ secrets.HUBSPOT_API_KEY }}
        run: |
          cd rho-hubspot-deployment
          npm run deploy  # Calls HubSpot API

      - name: Notify Slack
        if: success()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
          payload: |
            {
              "text": "✅ Email templates deployed to HubSpot",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*New Email Templates Deployed* 🚀\n\nVersion: `${{ github.sha }}`\n<https://github.com/${{ github.repository }}/commit/${{ github.sha }}|View Commit>"
                  }
                }
              ]
            }
```

**HubSpot Deployment Script** (`deploy.js`):
```javascript
const axios = require('axios');
const fs = require('fs');
const path = require('path');

const HUBSPOT_API_KEY = process.env.HUBSPOT_API_KEY;
const TEMPLATES_DIR = path.join(__dirname, 'dist/templates');

async function deployTemplate(fileName, htmlContent) {
  try {
    const response = await axios.post(
      'https://api.hubapi.com/content/api/v2/templates',
      {
        name: fileName.replace('.html', ''),
        source: htmlContent,
        folder: 'Email Templates / Automated',
        template_type: 'email'
      },
      {
        headers: {
          'Authorization': `Bearer ${HUBSPOT_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    console.log(`✅ Deployed: ${fileName} (ID: ${response.data.id})`);
  } catch (error) {
    console.error(`❌ Failed to deploy ${fileName}:`, error.response?.data || error.message);
    process.exit(1);
  }
}

async function main() {
  const files = fs.readdirSync(TEMPLATES_DIR);

  for (const file of files) {
    if (file.endsWith('.html')) {
      const htmlContent = fs.readFileSync(path.join(TEMPLATES_DIR, file), 'utf8');
      await deployTemplate(file, htmlContent);
    }
  }
}

main();
```

#### Benefits of CI/CD

| Benefit | Before (Manual) | After (Automated) |
|---------|----------------|-------------------|
| **Deployment Time** | 30 min (manual copy/paste, test send) | 5 min (commit → deploy) |
| **Testing** | Manual (5+ email clients) | Automated (Litmus screenshots) |
| **Version Control** | None (HubSpot editor only) | Full Git history + rollback |
| **Team Collaboration** | Christopher only | Any dev can submit PR |
| **Error Rate** | High (manual copy/paste errors) | Low (automated validation) |

### Phase 3: Storyblok CMS Integration (Q1 2026)

**Goal**: Empower non-technical users (Growth, Demand Gen) to create/edit emails without code.

#### Why Storyblok?

**Key Advantages**:
1. **Headless CMS**: Content stored separately from rendering logic
2. **Visual Editor**: Drag-and-drop interface (like HubSpot), but outputs structured JSON (not HTML)
3. **Component-Based**: Maps perfectly to our React component library
4. **Real-Time Preview**: See changes instantly before publishing
5. **Webhooks**: Notify external systems (HubSpot, Customer.io) when content published
6. **API-First**: Easy to integrate with serverless functions

**How It Works**:
```
Content Editor (Growth Team)
    ↓ [Opens Storyblok Visual Editor]
Storyblok CMS
    ↓ [Drags "BodyWithCTA" component onto canvas]
    ↓ [Fills in: bodyText, ctaText, ctaUrl]
    ↓ [Clicks "Publish"]
Storyblok Webhook
    ↓ [POST to Vercel serverless function]
Vercel Function
    ↓ [Fetches Storyblok JSON]
    ↓ [Renders React components → HTML]
    ↓ [Validates output]
    ↓ [Calls HubSpot API to create/update template]
HubSpot Template Available
    ↓ [Growth team schedules send]
Email Sent (via Customer.io or HubSpot)
```

#### Storyblok Schema Setup

**Component: BodyWithCTA**

**Storyblok Configuration**:
```json
{
  "name": "BodyWithCTA",
  "display_name": "Body Text with CTA Button",
  "schema": {
    "bodyText": {
      "type": "textarea",
      "description": "Main paragraph text",
      "required": true
    },
    "ctaText": {
      "type": "text",
      "description": "Button text (e.g., 'Register Now')",
      "required": true
    },
    "ctaUrl": {
      "type": "text",
      "description": "Button destination URL",
      "required": true,
      "regex": "^https?://.+"
    }
  },
  "preview_field": "ctaText",
  "color": "#00ECC0"
}
```

**Visual Editor Experience**:
1. Growth team opens Storyblok email editor
2. Sees list of available components (Header, Hero, BodyWithCTA, Footer)
3. Drags "BodyWithCTA" onto canvas
4. Right panel shows form:
   - **Body Text**: [Textarea input]
   - **CTA Text**: [Text input] (e.g., "Register Now")
   - **CTA URL**: [Text input] (e.g., "https://lu.ma/rho-cfo")
5. Real-time preview shows rendered email component
6. Clicks "Publish" → Webhook fires → Email template created in HubSpot

#### Serverless Middleware Architecture

**Tech Stack**:
- **Hosting**: Vercel (serverless functions, Edge Runtime)
- **Framework**: Next.js API routes
- **Rendering**: React Server Components (transform JSX → HTML)
- **Validation**: Zod (schema validation for Storyblok JSON)
- **Email Service**: Customer.io (preferred) or SendGrid (bypass HubSpot contact bloat)

**Vercel Function** (`/api/storyblok-webhook.js`):
```javascript
import { storyblokClient } from '@/lib/storyblok';
import { renderEmailComponent } from '@/lib/email-renderer';
import { hubspotAPI } from '@/lib/hubspot';
import { customerio } from '@/lib/customerio';

export default async function handler(req, res) {
  // 1. Verify Storyblok webhook signature
  const signature = req.headers['webhook-signature'];
  if (!verifySignature(signature, req.body)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // 2. Extract story ID from webhook payload
  const { story_id, action } = req.body;

  if (action !== 'published') {
    return res.status(200).json({ message: 'Ignored (not published)' });
  }

  // 3. Fetch full story content from Storyblok API
  const story = await storyblokClient.getStory(story_id);

  // 4. Validate story structure
  if (story.content.component !== 'email_template') {
    return res.status(400).json({ error: 'Not an email template' });
  }

  // 5. Render React components to email-safe HTML
  const emailHTML = await renderEmailComponent(story.content);

  // 6. Deploy to HubSpot (optional, for archival)
  await hubspotAPI.createTemplate({
    name: story.name,
    source: emailHTML,
    folder: 'Storyblok Templates'
  });

  // 7. Send via Customer.io (bypasses HubSpot contact creation)
  // Assumes recipient list already in Airtable/Customer.io from Events Automation
  await customerio.sendBroadcast({
    segment_id: story.content.target_segment,  // e.g., "Event Registrants - CFO Roundtable"
    subject: story.content.subject_line,
    from_name: "Rho",
    from_email: "events@rho.co",
    html_body: emailHTML
  });

  return res.status(200).json({
    message: 'Email template published and sent',
    template_url: `https://app.hubspot.com/templates/${templateId}`
  });
}
```

**Email Renderer** (`/lib/email-renderer.js`):
```javascript
import React from 'react';
import { renderToStaticMarkup } from 'react-dom/server';
import Header from '@/components/Header';
import Hero from '@/components/Hero';
import BodyWithCTA from '@/components/BodyWithCTA';
import Footer from '@/components/Footer';

const componentMap = {
  'header': Header,
  'hero': Hero,
  'body_with_cta': BodyWithCTA,
  'footer': Footer
};

export function renderEmailComponent(storyContent) {
  const components = storyContent.body.map(block => {
    const Component = componentMap[block.component];
    if (!Component) {
      throw new Error(`Unknown component: ${block.component}`);
    }
    return <Component key={block._uid} {...block} />;
  });

  const emailHTML = renderToStaticMarkup(
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>{storyContent.subject_line}</title>
      </head>
      <body style={{ margin: 0, padding: 0, backgroundColor: '#f4f4f4' }}>
        <table role="presentation" width="100%" cellSpacing="0" cellPadding="0" border="0">
          <tr>
            <td align="center">
              {components}
            </td>
          </tr>
        </table>
      </body>
    </html>
  );

  return emailHTML;
}
```

#### Benefits of Storyblok Integration

| Benefit | Impact |
|---------|--------|
| **Self-Service Email Creation** | Growth/Demand Gen team can create emails without Christopher (80% time savings) |
| **Bypass HubSpot Contact Bloat** | Send via Customer.io → no unnecessary HubSpot contacts → save $3K+/year |
| **Real-Time Personalization** | Fetch live data (event spots remaining, user account balance) via API |
| **Content Versioning** | Storyblok stores all versions, easy to revert if mistake |
| **Multi-Channel Distribution** | Same content → email, SMS, push notifications (Customer.io supports all) |
| **Component Governance** | Only approved components available in Storyblok (enforces brand consistency) |

---

## Implementation Roadmap

### Week 1-2: Component Library Build

**Tasks**:
1. **Set up file structure** (2 hours)
   - Create `rho-hubspot-deployment/` directory
   - Initialize npm project, install dependencies (React, Babel)

2. **Build core components** (12 hours)
   - Header.jsx (logo, date, kicker)
   - Hero.jsx (heading, subheading, background image)
   - BodyWithCTA.jsx (text + button)
   - Footer.jsx (company info, social links, legal)
   - Test in browser (manual render)

3. **Create preview site** (8 hours)
   - Build `index.html` with 50/50 split layout
   - Add React/Babel CDN imports
   - Implement "Copy HTML" buttons (clipboard API)
   - Add props editing form for each component

4. **Extract 3 full email templates** (6 hours)
   - Event invite (Header + Hero + BodyWithCTA + Footer)
   - Product launch (Header + BodyWithCTA + TwoColumnCards + Footer)
   - Newsletter (Header + Highlight + BodyWithCTA + PreFooter + Footer)
   - Save rendered HTML to `templates/` directory

**Deliverables**:
- ✅ 7 React JSX components
- ✅ Preview site (`index.html`) functional
- ✅ 3 full email templates (HubSpot-ready HTML)

**Owner**: Skydog Ops (with Christopher review/QA)

### Week 3: Documentation & Handoff

**Tasks**:
1. **Write usage documentation** (4 hours)
   - How to use preview site
   - How to extract HTML for HubSpot
   - Component props reference
   - Brand guidelines (colors, fonts, spacing)

2. **Train Demand Gen hire** (2 hours)
   - Walkthrough of component library
   - Demo: Build email from scratch using components
   - Q&A session

3. **Create HubSpot email templates** (4 hours)
   - Manually paste 3 template HTMLs into HubSpot
   - Save as reusable templates in Design Manager
   - Test sends to multiple email clients (Gmail, Outlook, Apple Mail)

**Deliverables**:
- ✅ README.md with usage instructions
- ✅ Brand guidelines doc
- ✅ 3 email templates live in HubSpot
- ✅ Demand Gen hire trained

**Owner**: Christopher (documentation), Skydog Ops (HubSpot setup)

### Week 4-5: CI/CD Pipeline Build

**Tasks**:
1. **Set up GitHub repo** (2 hours)
   - Move `rho-hubspot-deployment/` to dedicated repo (or keep in monorepo)
   - Configure branch protection (require PR reviews before merge to main)

2. **Build transformation script** (6 hours)
   - `npm run build` script: JSX → email-safe HTML
   - Use `jsx-email` library or custom Babel transform
   - Output to `dist/templates/` directory

3. **Write automated tests** (8 hours)
   - Link validator: Check all `href` attributes
   - Image validator: Verify `src` URLs return 200
   - Accessibility: Check alt text, color contrast (WCAG AA)
   - Litmus integration: Send HTML, retrieve screenshots (optional, $$)

4. **Create deployment script** (6 hours)
   - `deploy.js`: Call HubSpot Design Manager API
   - Upload each template with version tag
   - Handle API errors (rate limits, authentication)

5. **Configure GitHub Actions** (4 hours)
   - Create `.github/workflows/deploy-email-templates.yml`
   - Add HubSpot API key as GitHub secret
   - Add Slack webhook URL for notifications
   - Test workflow (commit to main, verify deploy)

**Deliverables**:
- ✅ Automated build pipeline (JSX → HTML)
- ✅ Automated tests running on every commit
- ✅ GitHub Actions deploys to HubSpot on main branch push
- ✅ Slack notifications on deploy

**Owner**: Skydog Ops (build), Christopher (HubSpot API setup, testing)

### Week 6: Testing & Iteration

**Tasks**:
1. **End-to-end testing** (6 hours)
   - Make change to component (e.g., update CTA button color)
   - Commit to main branch
   - Verify GitHub Actions runs successfully
   - Check HubSpot Design Manager has new template version
   - Send test email, verify rendering in Gmail/Outlook

2. **Error scenario testing** (4 hours)
   - Simulate HubSpot API failure → Verify retry logic
   - Simulate invalid HTML → Verify build fails before deploy
   - Simulate broken link → Verify test suite catches it

3. **Performance optimization** (4 hours)
   - Reduce build time (parallel component rendering)
   - Optimize image sizes (compress PNGs, use WebP fallbacks)

**Deliverables**:
- ✅ CI/CD pipeline battle-tested
- ✅ Rollback procedure documented (revert Git commit, redeploy)
- ✅ Known issues documented with workarounds

**Owner**: Skydog Ops (testing), Christopher (QA signoff)

### Q1 2026: Storyblok Integration (Post-Skydog)

**Tasks** (Owner: Internal Dev Team or new vendor):
1. **Storyblok account setup** (2 hours)
   - Create Storyblok space
   - Configure component schemas (map to React components)
   - Set up webhook to Vercel function

2. **Vercel serverless function build** (16 hours)
   - `/api/storyblok-webhook.js` endpoint
   - Fetch story from Storyblok API
   - Render React components server-side
   - Call HubSpot API to create template
   - Call Customer.io API to send email

3. **Customer.io integration** (8 hours)
   - Sync Airtable event registrations → Customer.io segments
   - Create broadcast templates in Customer.io
   - Test send flow: Storyblok publish → Customer.io send

4. **Training & documentation** (4 hours)
   - Train Growth team on Storyblok visual editor
   - Document workflows (create email, publish, monitor delivery)

**Deliverables**:
- ✅ Storyblok CMS live with component library
- ✅ Serverless middleware deployed to Vercel
- ✅ Customer.io integration functional (bypasses HubSpot contact creation)
- ✅ Growth team can create/send emails without Christopher

**Timeline**: 3-4 weeks (Q1 2026)
**Budget**: $15-20K (dev time) or in-house if bandwidth available

---

## Success Metrics

### Week 3 Targets (Component Library Complete)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Components Built** | 7 core components | Count of `.jsx` files |
| **Templates Created** | 3 full templates (event, product, newsletter) | Count of `.html` files in `templates/` |
| **Preview Site Functional** | 100% uptime | Manual QA (open `index.html`, verify all components render) |
| **Time to Build Email** | <30 min (down from 2-3 hours) | Christopher tracks time for next 3 emails |

### Week 6 Targets (CI/CD Deployed)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Deployment Time** | <5 min (commit → live in HubSpot) | GitHub Actions workflow duration |
| **Test Coverage** | 100% of components have link/image validation | Test suite output |
| **Deployment Success Rate** | >95% (no manual fixes needed) | GitHub Actions pass rate over 20 deployments |
| **Team Adoption** | Demand Gen hire creates 3 emails using components (without Christopher) | Survey/interview |

### Q1 2026 Targets (Storyblok Integrated)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Self-Service Email Creation** | 80% of emails created by Growth team (not Christopher) | Storyblok activity log |
| **HubSpot Contact Savings** | $3K+/year saved (send via Customer.io instead) | Finance: Compare HubSpot contact count before/after |
| **Time Saved** | 30 hours/month (Christopher no longer builds emails) | Time tracking |
| **Email Quality** | <5% of emails require rework post-publish | QA audit log |

---

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **HubSpot API changes break deployment** | High (deploy pipeline fails) | Low | Pin to specific API version, monitor HubSpot changelog, build fallback (manual upload script) |
| **Storyblok costs exceed budget** | Medium (may need to pause Phase 3) | Low | Storyblok has generous free tier (up to 3 users), only pay if scale beyond Growth team |
| **Growth team resists Storyblok learning curve** | Medium (self-service adoption fails) | Medium | Comprehensive training, create video tutorials, Christopher available for Q&A first 2 weeks |
| **Email rendering breaks in Outlook** | High (emails unreadable for 20% of recipients) | Medium | Automated Litmus testing in CI/CD, manual QA in Outlook before launch |
| **Customer.io integration delays** | Low (can still use HubSpot for Phase 3) | Medium | Build Phase 3 in stages: Storyblok → HubSpot first, then swap HubSpot for Customer.io |

---

## Open Questions & Decisions

### Week 1 (Blocking)

1. **Component Scope**: Which components to build first?
   - Option A: 7 core components (Header, Hero, BodyWithCTA, TwoColumnCards, Highlight, PreFooter, Footer)
   - Option B: Start with 4 (Header, BodyWithCTA, Footer, + 1 custom for next event)
   - **Decision Maker**: Christopher (based on upcoming email calendar)

2. **Preview Site Hosting**: Where to host `index.html`?
   - Option A: GitHub Pages (free, public)
   - Option B: Vercel (free, can add authentication later)
   - Option C: Local only (Christopher runs `open index.html`)
   - **Decision Maker**: Christopher

### Week 4 (Important, Not Blocking)

3. **Litmus Integration**: Worth the cost for automated email client testing?
   - Litmus: $99/month for API access
   - Alternative: Manual testing (Christopher sends to 5 email clients)
   - **Decision Maker**: Christopher + Anthony (budget approval)

4. **Git Workflow**: How should team submit component changes?
   - Option A: Anyone can commit to main (fast, risky)
   - Option B: Require PR reviews (slower, safer)
   - **Decision Maker**: Christopher

### Q1 2026 (Phase 3)

5. **Email Service Provider**: Customer.io or SendGrid?
   - Customer.io: Better for lifecycle emails, has CDP features ($$$)
   - SendGrid: Transactional focus, cheaper ($$)
   - **Decision Maker**: Tommy (CRO) + Christopher (based on Events Automation needs)

---

## Related Documentation

- **[CLAUDE.md](../CLAUDE.md)**: Email-safe HTML requirements, brand guidelines
- **[10-Component Library CI/CD](10-component-library-cicd.md)**: Technical specs for GitHub Actions pipeline
- **[03-Events Automation](03-events-automation.md)**: Customer.io integration for event emails
- **Storyblok Article** (User-provided): [Storyblok + Email Integration Guide](https://www.storyblok.com/tp/integrate-emails-with-storyblok)

---

**Next Steps**:
1. Week 1: Christopher prioritizes component list based on upcoming email needs
2. Week 1: Skydog Ops begins component build (Header, BodyWithCTA, Footer first)
3. Week 2: Preview site MVP for Christopher QA
4. Week 3: Extract first full email template, test in HubSpot

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
