# 10 - Component Library CI/CD Implementation

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations) + Skydog Ops
**Phase**: Week 4-6 (Automation Phase)
**Related**: [04-Email Template Architecture](04-email-template-architecture.md), [08-Technical Architecture](08-technical-architecture.md)

---

## Overview

This document provides implementation details for the **CI/CD pipeline** that automates email template deployment from GitHub → HubSpot Design Manager.

**Goal**: Commit code → GitHub Actions builds → Tests run → Auto-deploy to HubSpot → Slack notification (all in <5 minutes)

**Stack**:
- GitHub Actions (CI/CD orchestration)
- Node.js + React (component rendering)
- HubSpot Design Manager API (deployment target)
- Jest (testing framework)
- Slack Webhooks (notifications)

---

## GitHub Actions Workflow

### File: `.github/workflows/deploy-email-templates.yml`

```yaml
name: Deploy Email Templates to HubSpot

on:
  push:
    branches:
      - main
    paths:
      - 'components/**'
      - 'templates/**'
  workflow_dispatch:  # Manual trigger

env:
  NODE_VERSION: '18'

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build email templates
        run: npm run build
        env:
          NODE_ENV: production

      - name: Run tests
        run: npm test -- --coverage

      - name: Deploy to HubSpot
        if: success()
        run: npm run deploy
        env:
          HUBSPOT_API_KEY: ${{ secrets.HUBSPOT_API_KEY }}
          HUBSPOT_FOLDER: 'Email Templates / Automated'

      - name: Notify Slack on success
        if: success()
        uses: slackapi/slack-github-action@v1.25.0
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
                    "text": "*Email Templates Deployed* 🚀\n\nCommit: <https://github.com/${{ github.repository }}/commit/${{ github.sha }}|${{ github.event.head_commit.message }}>\nAuthor: ${{ github.actor }}"
                  }
                }
              ]
            }

      - name: Notify Slack on failure
        if: failure()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
          payload: |
            {
              "text": "❌ Email template deployment failed",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Deployment Failed* ⚠️\n\nCommit: <https://github.com/${{ github.repository }}/commit/${{ github.sha }}|${{ github.event.head_commit.message }}>\nSee <https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}|workflow logs>"
                  }
                }
              ]
            }
```

---

## Build Script

### File: `scripts/build.js`

```javascript
const fs = require('fs');
const path = require('path');
const React = require('react');
const { renderToStaticMarkup } = require('react-dom/server');

// Import all template files
const templates = [
  { name: 'event-invite', component: require('../templates/event-invite.jsx').default },
  { name: 'product-launch', component: require('../templates/product-launch.jsx').default },
  { name: 'newsletter', component: require('../templates/newsletter.jsx').default }
];

const OUTPUT_DIR = path.join(__dirname, '../dist/templates');

// Ensure output directory exists
if (!fs.existsSync(OUTPUT_DIR)) {
  fs.mkdirSync(OUTPUT_DIR, { recursive: true });
}

// Email client reset CSS
const emailResetCSS = `
<style>
  body { margin: 0 !important; padding: 0 !important; }
  img { border: 0; display: block; }
  table { border-collapse: collapse; }
  a { text-decoration: none; }
  @media only screen and (max-width: 620px) {
    .fluid { width: 100% !important; max-width: 100% !important; }
    .stack { display: block !important; width: 100% !important; }
  }
</style>
`;

// Build each template
templates.forEach(({ name, component: Component }) => {
  try {
    console.log(`Building ${name}...`);

    // Render React component to static HTML
    const bodyHTML = renderToStaticMarkup(<Component />);

    // Wrap in full HTML document with email resets
    const fullHTML = `
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>${name}</title>
  ${emailResetCSS}
</head>
<body style="margin: 0; padding: 0; background-color: #f4f4f4;">
  ${bodyHTML}
</body>
</html>
    `.trim();

    // Write to dist folder
    const outputPath = path.join(OUTPUT_DIR, `${name}.html`);
    fs.writeFileSync(outputPath, fullHTML);

    console.log(`✅ Built ${name}.html`);
  } catch (error) {
    console.error(`❌ Failed to build ${name}:`, error.message);
    process.exit(1);
  }
});

console.log('\n🎉 All templates built successfully!\n');
```

**package.json script**:
```json
{
  "scripts": {
    "build": "node scripts/build.js"
  }
}
```

---

## Deployment Script

### File: `scripts/deploy.js`

```javascript
const fs = require('fs');
const path = require('path');
const axios = require('axios');

const HUBSPOT_API_KEY = process.env.HUBSPOT_API_KEY;
const HUBSPOT_FOLDER = process.env.HUBSPOT_FOLDER || 'Email Templates / Automated';
const TEMPLATES_DIR = path.join(__dirname, '../dist/templates');

if (!HUBSPOT_API_KEY) {
  console.error('❌ HUBSPOT_API_KEY environment variable not set');
  process.exit(1);
}

async function deployTemplate(fileName, htmlContent) {
  const templateName = fileName.replace('.html', '');
  const version = process.env.GITHUB_SHA ? process.env.GITHUB_SHA.substring(0, 7) : 'dev';
  const fullName = `${templateName} - v${version}`;

  try {
    console.log(`Deploying ${fullName}...`);

    const response = await axios.post(
      'https://api.hubapi.com/content/api/v2/templates',
      {
        name: fullName,
        source: htmlContent,
        folder: HUBSPOT_FOLDER,
        template_type: 'EMAIL',
        author_name: 'Christopher Cooper',
        author_email: 'christopher.cooper@rho.co'
      },
      {
        headers: {
          'Authorization': `Bearer ${HUBSPOT_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );

    console.log(`✅ Deployed ${fullName} (ID: ${response.data.id})`);
    return response.data;

  } catch (error) {
    if (error.response) {
      console.error(`❌ HubSpot API error for ${fullName}:`, error.response.data);
    } else {
      console.error(`❌ Failed to deploy ${fullName}:`, error.message);
    }
    process.exit(1);
  }
}

async function main() {
  const files = fs.readdirSync(TEMPLATES_DIR).filter(f => f.endsWith('.html'));

  console.log(`\nFound ${files.length} templates to deploy\n`);

  for (const file of files) {
    const htmlContent = fs.readFileSync(path.join(TEMPLATES_DIR, file), 'utf8');
    await deployTemplate(file, htmlContent);

    // Rate limiting: Wait 1 second between requests
    await new Promise(resolve => setTimeout(resolve, 1000));
  }

  console.log('\n🎉 All templates deployed successfully!\n');
}

main();
```

**package.json script**:
```json
{
  "scripts": {
    "deploy": "node scripts/deploy.js"
  }
}
```

---

## Automated Tests

### File: `tests/link-validator.test.js`

```javascript
const fs = require('fs');
const path = require('path');
const { JSDOM } = require('jsdom');

const TEMPLATES_DIR = path.join(__dirname, '../dist/templates');

describe('Link Validator', () => {
  const templates = fs.readdirSync(TEMPLATES_DIR).filter(f => f.endsWith('.html'));

  templates.forEach(templateFile => {
    test(`${templateFile}: All links should have valid href attributes`, () => {
      const html = fs.readFileSync(path.join(TEMPLATES_DIR, templateFile), 'utf8');
      const dom = new JSDOM(html);
      const links = dom.window.document.querySelectorAll('a[href]');

      links.forEach(link => {
        const href = link.getAttribute('href');

        // Check href is not empty
        expect(href).toBeTruthy();

        // Check href starts with http/https or is mailto/tel
        expect(
          href.startsWith('http://') ||
          href.startsWith('https://') ||
          href.startsWith('mailto:') ||
          href.startsWith('tel:') ||
          href.startsWith('#')  // Anchor links OK
        ).toBe(true);
      });
    });
  });
});
```

### File: `tests/image-validator.test.js`

```javascript
const fs = require('fs');
const path = require('path');
const { JSDOM } = require('jsdom');
const axios = require('axios');

const TEMPLATES_DIR = path.join(__dirname, '../dist/templates');

describe('Image Validator', () => {
  const templates = fs.readdirSync(TEMPLATES_DIR).filter(f => f.endsWith('.html'));

  templates.forEach(templateFile => {
    test(`${templateFile}: All images should have src and alt attributes`, () => {
      const html = fs.readFileSync(path.join(TEMPLATES_DIR, templateFile), 'utf8');
      const dom = new JSDOM(html);
      const images = dom.window.document.querySelectorAll('img');

      images.forEach(img => {
        const src = img.getAttribute('src');
        const alt = img.getAttribute('alt');

        // Check src exists
        expect(src).toBeTruthy();

        // Check alt exists (accessibility)
        expect(alt).toBeDefined();

        // Check src is valid URL (not broken)
        if (src.startsWith('http')) {
          expect(src).toMatch(/^https?:\/\/.+/);
        }
      });
    });

    test(`${templateFile}: All image URLs should return 200 status`, async () => {
      const html = fs.readFileSync(path.join(TEMPLATES_DIR, templateFile), 'utf8');
      const dom = new JSDOM(html);
      const images = dom.window.document.querySelectorAll('img');

      const imageURLs = Array.from(images)
        .map(img => img.getAttribute('src'))
        .filter(src => src && src.startsWith('http'));

      // Check each image URL
      for (const url of imageURLs) {
        try {
          const response = await axios.head(url, { timeout: 5000 });
          expect(response.status).toBe(200);
        } catch (error) {
          throw new Error(`Broken image: ${url} (${error.message})`);
        }
      }
    }, 30000);  // 30 second timeout for network requests
  });
});
```

### File: `tests/accessibility.test.js`

```javascript
const fs = require('fs');
const path = require('path');
const { JSDOM } = require('jsdom');

const TEMPLATES_DIR = path.join(__dirname, '../dist/templates');

describe('Accessibility Tests', () => {
  const templates = fs.readdirSync(TEMPLATES_DIR).filter(f => f.endsWith('.html'));

  templates.forEach(templateFile => {
    test(`${templateFile}: All images must have alt text`, () => {
      const html = fs.readFileSync(path.join(TEMPLATES_DIR, templateFile), 'utf8');
      const dom = new JSDOM(html);
      const images = dom.window.document.querySelectorAll('img');

      images.forEach(img => {
        const alt = img.getAttribute('alt');
        expect(alt).not.toBeNull();
        expect(alt.length).toBeGreaterThan(0);
      });
    });

    test(`${templateFile}: No empty headings`, () => {
      const html = fs.readFileSync(path.join(TEMPLATES_DIR, templateFile), 'utf8');
      const dom = new JSDOM(html);
      const headings = dom.window.document.querySelectorAll('h1, h2, h3, h4, h5, h6');

      headings.forEach(heading => {
        const text = heading.textContent.trim();
        expect(text.length).toBeGreaterThan(0);
      });
    });

    test(`${templateFile}: All tables have role="presentation"`, () => {
      const html = fs.readFileSync(path.join(TEMPLATES_DIR, templateFile), 'utf8');
      const dom = new JSDOM(html);
      const tables = dom.window.document.querySelectorAll('table');

      tables.forEach(table => {
        const role = table.getAttribute('role');
        expect(role).toBe('presentation');
      });
    });
  });
});
```

**package.json**:
```json
{
  "scripts": {
    "test": "jest --verbose"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "jsdom": "^22.0.0",
    "axios": "^1.6.0"
  }
}
```

---

## GitHub Secrets Configuration

**Required Secrets** (set in GitHub repo settings):

| Secret Name | Value | Used By |
|-------------|-------|---------|
| `HUBSPOT_API_KEY` | HubSpot Private App access token | `deploy.js` script |
| `SLACK_WEBHOOK_URL` | Slack incoming webhook URL for #marketing-ops | GitHub Actions workflow |

**How to Set**:
1. Go to GitHub repo → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `HUBSPOT_API_KEY`, Value: `pat-na1-...` (from HubSpot)
4. Repeat for `SLACK_WEBHOOK_URL`

---

## Rollback Procedure

**Scenario**: New email template deployed, but has rendering issue in Outlook.

**Steps**:

1. **Identify Last Good Commit**
   ```bash
   git log --oneline
   # Find commit before the breaking change
   # Example: abc1234 "Update CTA button styles"
   ```

2. **Checkout Previous Commit**
   ```bash
   git checkout abc1234
   ```

3. **Rebuild and Redeploy**
   ```bash
   npm run build
   npm run deploy
   ```

4. **Verify in HubSpot**
   - Check Design Manager for new template version
   - Send test email, verify rendering

5. **Fix Issue in New Branch**
   ```bash
   git checkout main
   git checkout -b fix/outlook-rendering
   # Make fixes
   git commit -m "fix: Outlook button rendering"
   git push origin fix/outlook-rendering
   # Create PR, review, merge
   ```

---

## Performance Optimization

**Current Pipeline Duration**: ~8 minutes (target: <5 minutes)

**Optimizations**:

1. **Cache npm dependencies** (saves 2-3 min)
   ```yaml
   - name: Setup Node.js
     uses: actions/setup-node@v4
     with:
       cache: 'npm'  # ✅ Already added
   ```

2. **Parallel testing** (saves 1 min)
   ```json
   {
     "scripts": {
       "test": "jest --maxWorkers=4"
     }
   }
   ```

3. **Skip image URL validation in CI** (saves 30 sec, run manually)
   ```javascript
   // In tests/image-validator.test.js
   test.skip(`All image URLs should return 200 status`, async () => {
     // Only run this test manually, not in CI
   });
   ```

4. **Use esbuild instead of Babel** (faster JSX transformation)
   ```bash
   npm install --save-dev esbuild
   # Update build.js to use esbuild.transform()
   ```

**Expected Duration After Optimization**: ~4 minutes

---

## Monitoring & Alerts

**GitHub Actions Notifications**:
- Success: Slack message to #marketing-ops
- Failure: Slack message to #marketing-ops (ping @christopher.cooper)

**Weekly Metrics** (send to Slack every Monday):
```javascript
// scripts/weekly-report.js
// Fetch GitHub Actions API for past 7 days
// Calculate:
// - Total deploys
// - Success rate
// - Average duration
// POST to Slack webhook
```

**Alerts**:
- Pipeline failing >2 consecutive runs → Page Christopher (PagerDuty)
- Deploy duration >10 min → Investigate performance regression

---

## Future Enhancements

### Phase 2 (Q1 2026)

1. **Email Client Screenshots** (Litmus API integration)
   - After deploy, trigger Litmus test
   - Generate screenshots for Gmail, Outlook, Apple Mail
   - Attach to GitHub Actions workflow artifacts

2. **A/B Testing Support**
   - Build 2 versions of same template (e.g., green CTA vs. blue CTA)
   - Deploy both to HubSpot
   - Tag with `_variant_a` and `_variant_b` suffixes

3. **Dark Mode Testing**
   - Add CSS media query: `@media (prefers-color-scheme: dark)`
   - Test rendering in Gmail iOS dark mode

---

**Last Updated**: 2025-11-12
**Document Owner**: Skydog Ops (build) + Christopher Cooper (maintenance)
**Next Review**: Week 6 (after CI/CD deployed)
