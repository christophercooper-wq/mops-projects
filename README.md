# RHO MARKETING OPERATIONS: INFRASTRUCTURE OVERHAUL PROJECT

**Project Status**: 🟡 In Progress | **Phase**: Foundation & Vendor Kickoff
**Timeline**: 8 Weeks (December 2025 - January 2026)
**Owner**: Christopher Cooper (Marketing Operations)
**Stakeholders**: Anthony Hwang (RevOps Manager), Skydog Ops (Vendor), Engineering, Leadership

---

## 🎯 PROJECT OVERVIEW

This project addresses systemic infrastructure failures that caused the Lifecycle Marketing Manager hire to fail despite qualified candidates receiving unanimous positive scores. The root cause is **not tactical execution gaps**, but **strategic data architecture and leadership alignment failures**.

### **The Problem Statement**

Rho's marketing operations infrastructure suffers from:
1. **No canonical data schemas** between Signup Services → Data Warehouse → Salesforce → HubSpot
2. **40-45 minute application status lag** making real-time lead acceleration impossible
3. **300+ HubSpot workflows built on faulty assumptions** about data availability
4. **Leadership misalignment** on demand generation vs lifecycle marketing strategy
5. **No ownership model** for data infrastructure, tools, and experimentation

**Result**: $100K+ annual inefficiency cost, 300% slower campaign deployment, failed hiring process

---

## 📚 DOCUMENTATION INDEX

### **Core Analysis Documents**

| Document | Description | Status |
|----------|-------------|--------|
| [01-root-cause-analysis.md](./01-root-cause-analysis.md) | Data flow failures, 45-minute gap, taxonomy issues | ✅ Complete |
| [02-organizational-assessment.md](./02-organizational-assessment.md) | Org structure, role misalignment, leadership gaps | ✅ Complete |
| [03-events-automation.md](./03-events-automation.md) | Luma integration, Airtable registry, automation MVP | 🟡 In Progress |
| [04-email-template-architecture.md](./04-email-template-architecture.md) | Storyblok migration, component library, scalability | 📋 Planned |
| [05-hubspot-audit-requirements.md](./05-hubspot-audit-requirements.md) | Workflow bloat analysis, property cleanup, tooling | 🟡 In Progress |
| [06-vendor-sow-skydog.md](./06-vendor-sow-skydog.md) | 3-phase engagement scope, deliverables, $32K budget | ✅ Complete |
| [07-mops-action-plan.md](./07-mops-action-plan.md) | 8-week milestones, priority matrix, success criteria | ✅ Complete |
| [08-technical-architecture.md](./08-technical-architecture.md) | Event emission specs, ETL contracts, data schemas | 📋 Planned |

### **Project Management**

| Document | Description | Status |
|----------|-------------|--------|
| [09-project-roadmap.md](./09-project-roadmap.md) | Detailed 8-week timeline, dependencies, owners | ✅ Complete |
| [CHANGELOG.md](./CHANGELOG.md) | All project changes, decisions, and milestones | 🟢 Active |

### **Additional Tooling**

| Document | Description | Status |
|----------|-------------|--------|
| [10-component-library-cicd.md](./10-component-library-cicd.md) | Email template doc site, HubSpot deployment automation | 📋 Planned |
| [11-events-management-ui.md](./11-events-management-ui.md) | CSV normalization, source-agnostic field mapping | 📋 Planned |
| [12-storyblok-integration.md](./12-storyblok-integration.md) | CMS layer for email, serverless send logic (Q1 2026) | 📋 Planned |

---

## 🗓️ 8-WEEK PROJECT TIMELINE

### **Month 1: Foundation & Vendor Kickoff** (Weeks 1-4)

**Week 1-2: Critical Path**
- 🔴 **P0**: Leadership alignment meeting (Tommy/Jeremy/Anthony)
- 🔴 **P0**: Skydog Ops kickoff with 3-phase SOW
- 🟡 **P1**: Events automation MVP (Airtable registry)
- 🟡 **P1**: Email template velocity documentation

**Week 3-4: Quick Wins & Audit**
- 🟡 **P1**: Skydog Phase 1 delivery (HubSpot dependency audit)
- 🟡 **P1**: Engineering collaboration (event emission PRD)
- 🟢 **P2**: Component library doc site setup
- 🟢 **P2**: Events management UI design

### **Month 2: Remediation & Infrastructure** (Weeks 5-8)

**Week 5-6: Skydog Phase 2 Execution**
- 🔴 **P0**: Workflow consolidation (300+ → <100)
- 🔴 **P0**: Property rationalization (archive Tier 5 legacy)
- 🔴 **P0**: Salesforce sync fixes (<5% error rate)
- 🟡 **P1**: Attribution schema implementation (PIC v3.0)

**Week 7-8: Strategic Infrastructure & Hiring Prep**
- 🟡 **P1**: Skydog Phase 3 advisory (event bus architecture)
- 🟡 **P1**: Demand Gen role preparation (infrastructure checklist)
- 🟢 **P2**: Component library CI/CD deployment
- 🟢 **P2**: Events UI MVP launch

---

## 🎯 SUCCESS METRICS

### **By End of Week 4 (Month 1)**
- ✅ Leadership alignment meeting held, RACI chart documented
- ✅ Skydog Phase 1 complete: Audit delivered, gaps identified
- ✅ Events automation MVP live (zero manual uploads)
- ✅ Email deployment <1 day (interim goal)
- ✅ Engineering has PRD for event emission

### **By End of Week 8 (Month 2)**
- ✅ Workflow count <100 (from 300+)
- ✅ SFDC sync error rate <5% (from ~15%)
- ✅ Property count reduced 30%+
- ✅ Lifecycle stages defined and implemented
- ✅ Demand Gen role reopened with correct JD
- ✅ Component library CI/CD live
- ✅ Events UI handling 100% of uploads

---

## 🏗️ PROJECT PHASES OVERVIEW

### **Phase 1: Discovery & Audit** (Weeks 1-3) - $15K
**Owner**: Skydog Ops
**Deliverables**:
- HubSpot Health Assessment Report
- Data Architecture Documentation
- Priority Recommendations Matrix

### **Phase 2: Remediation & Quick Wins** (Weeks 4-6) - $12K
**Owner**: Skydog Ops + Christopher Cooper (MOps)
**Deliverables**:
- Workflow Consolidation
- Property Rationalization
- Salesforce Sync Fixes
- Events Automation MVP

### **Phase 3: Strategic Infrastructure** (Weeks 7-8) - $5K Advisory
**Owner**: Skydog Ops (Advisory) + Engineering (Implementation)
**Deliverables**:
- Real-Time Event Architecture Design
- Progress State Service Design
- Marketing Automation Migration Plan

---

## 🔑 KEY DECISIONS REQUIRED

### **Leadership (Tommy McNulty + Jeremy Liang + Anthony Hwang)**

| Decision | Status | Blocker Impact |
|----------|--------|----------------|
| **Role Definition**: Demand Gen vs Lifecycle? | 🔴 Pending | Blocks hiring |
| **Ownership Model**: RACI for data, tools, MOF/BOF | 🔴 Pending | Blocks all strategic work |
| **Budget Approval**: $32K Skydog 3-phase SOW | 🟢 Approved | Execution enabled |
| **Timeline Commitment**: Reopen role Q1 2026? | 🟡 TBD | Sets expectations |

### **Engineering & Data Teams**

| Decision | Status | Blocker Impact |
|----------|--------|----------------|
| **Event Emission**: Add real-time events from Signup Services | 🔴 Pending | Blocks lead acceleration |
| **ETL Contracts**: Document data schema flows | 🔴 Pending | Blocks attribution fixes |
| **API Access**: Provide endpoints for MOps tooling | 🟡 TBD | Delays automation |

---

## 🚧 CURRENT BLOCKERS

| Blocker | Impact | Owner | Resolution Timeline |
|---------|--------|-------|---------------------|
| **Leadership alignment meeting not scheduled** | 🔴 Critical - blocks all strategic decisions | Christopher Cooper | Week 1 |
| **Lifecycle stage definitions on hold** | 🔴 Critical - blocks funnel reporting | Leadership | Week 2 |
| **No real-time event emission from Signup Services** | 🔴 Critical - 45-min gap persists | Engineering | Q1 2026 (advisory only) |
| **SFDC-HubSpot sync errors (~15% failure rate)** | 🟡 High - causes data loss | Skydog Ops | Week 5-6 |
| **300+ HubSpot workflows creating maintenance burden** | 🟡 High - slows all changes | Skydog Ops | Week 4-6 |

---

## 📊 PROJECT HEALTH DASHBOARD

**Overall Status**: 🟡 At Risk (Leadership decisions pending)

| Category | Status | Health |
|----------|--------|--------|
| **Documentation** | ✅ Complete | 🟢 Green |
| **Vendor Engagement** | 🟡 Kickoff Pending | 🟡 Yellow |
| **Leadership Alignment** | 🔴 Meeting Not Scheduled | 🔴 Red |
| **Technical Execution** | 🟢 Quick Wins in Progress | 🟢 Green |
| **Budget** | ✅ $32K Approved | 🟢 Green |
| **Timeline** | 🟡 On Track (if decisions made) | 🟡 Yellow |

---

## 🤝 STAKEHOLDER RESPONSIBILITIES

### **Christopher Cooper (Marketing Operations)**
- ✅ Project documentation and roadmap management
- ✅ Quick wins execution (events automation, email templates)
- ✅ Skydog Ops liaison and QA
- ✅ Engineering collaboration (event emission PRD)
- ✅ Changelog maintenance

### **Anthony Hwang (RevOps/MOps Manager)**
- 🔴 Schedule leadership alignment meeting
- 🔴 Approve Skydog 3-phase SOW scope
- 🟡 Facilitate engineering collaboration
- 🟡 Review/approve vendor deliverables

### **Skydog Ops (Vendor)**
- 🔴 Deliver Phase 1 audit (Weeks 1-3)
- 🟡 Execute Phase 2 remediation (Weeks 4-6)
- 🟢 Provide Phase 3 advisory (Weeks 7-8)

### **Leadership (Tommy McNulty + Jeremy Liang)**
- 🔴 Attend alignment meeting (Week 1)
- 🔴 Decide: Demand Gen vs Lifecycle role
- 🔴 Approve RACI chart (ownership model)
- 🟡 Review hiring timeline for Q1 2026

### **Engineering & Data Teams**
- 🔴 Review event emission PRD
- 🟡 Commit to Q1 2026 implementation timeline
- 🟢 Provide API access for MOps tooling

---

## 🛠️ TECHNICAL ARCHITECTURE OVERVIEW

### **Current State Problems**
```
Signup Services (Application Start)
    ↓ [NO REAL-TIME EVENTS] ❌
    ↓ [40-45 min batch sync]
Data Warehouse (Snowflake)
    ↓ [Scheduled sync]
Salesforce
    ↓ [HubSpot integration - 15% error rate] ❌
HubSpot
    ↓ [300+ workflows - bloated] ❌
Marketing Automation (too late)
```

### **Target State Architecture**
```
Signup Services
    ↓ [REAL-TIME EVENTS via webhook/Kafka] ✅
    ├→ Event Bus (Segment/RudderStack)
    │   ├→ Analytics (Amplitude/PostHog) [instant visibility]
    │   ├→ Data Warehouse [async enrichment]
    │   ├→ CRM (Salesforce) [qualified leads only]
    │   └→ Marketing Automation (Customer.io) [<5min triggers]
    └→ Progress State Cache (Redis/Firebase)
        └→ Sales Dashboard [real-time "who's stuck"]
```

---

## 📝 QUICK START GUIDE

### **For Team Members**
1. Read [02-organizational-assessment.md](./02-organizational-assessment.md) for context
2. Review [09-project-roadmap.md](./09-project-roadmap.md) for timeline
3. Check [CHANGELOG.md](./CHANGELOG.md) for latest updates
4. Identify your action items in roadmap

### **For Leadership**
1. Review [Key Decisions Required](#-key-decisions-required) section above
2. Schedule alignment meeting (Week 1 priority)
3. Review [06-vendor-sow-skydog.md](./06-vendor-sow-skydog.md) for budget details

### **For Vendors (Skydog Ops)**
1. Review [06-vendor-sow-skydog.md](./06-vendor-sow-skydog.md) for full SOW
2. Review [05-hubspot-audit-requirements.md](./05-hubspot-audit-requirements.md) for audit scope
3. Access HubSpot/Salesforce environments (credentials via Anthony)

---

## 🔗 EXTERNAL RESOURCES

- **GitHub Repository**: [christophercooper-wq/mops-projects](https://github.com/christophercooper-wq/mops-projects)
- **Google Drive Folder**: `/My Drive/MOps Infrastructure Project/` (auto-synced)
- **HubSpot Instance**: [Portal ID 39998325](https://app.hubspot.com/contacts/39998325/)
- **Existing Component Library**: `/Users/christopher.cooper/Documents/MOps/rho-hubspot-deployment/`

---

## 📞 CONTACT & ESCALATION

**Project Owner**: Christopher Cooper (christopher.cooper@rho.co)
**Manager**: Anthony Hwang (RevOps/MOps Manager)
**Vendor**: Skydog Ops (engagement pending)

**Escalation Path**:
1. Week 1-4 blockers → Anthony Hwang
2. Leadership decisions → Tommy McNulty + Jeremy Liang
3. Engineering dependencies → [TBD - Eng lead contact]

---

## 🔄 DOCUMENT MAINTENANCE

**Last Updated**: 2025-11-12
**Next Review**: Weekly (every Monday)
**Changelog**: See [CHANGELOG.md](./CHANGELOG.md)
**Version**: 1.0.0

---

**Status Legend**:
- ✅ Complete
- 🟢 On Track
- 🟡 At Risk / In Progress
- 🔴 Blocked / Critical
- 📋 Planned / Not Started
