# 07 - Marketing Operations Action Plan (90-Day Milestones)

**Document Status**: ✅ Complete
**Last Updated**: 2025-11-12
**Owner**: Christopher Cooper (Marketing Operations)
**Timeline**: 8 weeks (December 2025 - January 2026) + 30-day support period
**Related Documents**: [09-Project Roadmap](09-project-roadmap.md) (detailed weekly tasks)

---

## Executive Summary

This document provides a high-level milestone view of the Marketing Operations Infrastructure Overhaul project, extracted from the detailed 8-week roadmap. Use this for executive reporting, leadership updates, and tracking critical path progress.

**Project Goal**: Infrastructure readiness for Q1 2026 Demand Generation hire
**Budget**: $32,000 (Skydog Ops engagement)
**Success Criteria**: 70% reduction in Christopher's operational burden (30 hrs/week → 10 hrs/week)

---

## Week 1-2 Milestones (Foundation Phase)

### Critical Path Items

**Week 1**:
- [ ] **Leadership Alignment Meeting** (BLOCKING)
  - **Owner**: Christopher schedules with Tommy McNulty
  - **Attendees**: Tommy (CRO), Jeremy (Head of Growth), Christopher
  - **Agenda**: Confirm project scope, RACI decisions, Demand Gen hire timeline
  - **Success Criteria**: Tommy approves workflow deletion authority, confirms attribution methodology ownership
  - **Deadline**: December 6, 2025

- [ ] **Skydog Ops Kickoff**
  - **Owner**: Christopher + Skydog Ops Lead
  - **Deliverable**: Project plan confirmed, API credentials shared, week 1 tasks assigned
  - **Success Criteria**: Skydog Ops has access to Luma API, HubSpot API, Airtable
  - **Deadline**: December 3, 2025

**Week 2**:
- [ ] **Events Automation MVP Live** (P0 - CRITICAL)
  - **Deliverable**: Luma webhook → Airtable → HubSpot sync operational
  - **Success Criteria**: 3 test events synced, <5 min latency, <2% error rate
  - **Impact**: Christopher saves 6 hours/month on manual event imports
  - **Deadline**: December 13, 2025

- [ ] **Email Component Library Built**
  - **Deliverable**: 7 React JSX components + preview site functional
  - **Success Criteria**: Christopher can build event email in <30 min (down from 2-3 hours)
  - **Impact**: Reduces email production time by 80%
  - **Deadline**: December 20, 2025

### Key Metrics (End of Week 2)

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| **Luma Events Automated** | 3 events | | |
| **Email Templates Created** | 3 templates | | |
| **Time Saved** | 8 hours/month | | |
| **Leadership Sign-Off** | Approved | | |

---

## Week 3-4 Milestones (Automation Phase)

### Critical Path Items

**Week 3**:
- [ ] **CI/CD Pipeline Deployed**
  - **Deliverable**: GitHub Actions auto-deploys email templates to HubSpot on commit
  - **Success Criteria**: 1 test deployment successful (<5 min commit → HubSpot)
  - **Impact**: Zero manual HubSpot template uploads
  - **Deadline**: December 27, 2025

- [ ] **HubSpot Workflow Audit Complete**
  - **Deliverable**: Python script generates full workflow inventory (CSV + Airtable)
  - **Success Criteria**: 300+ workflows cataloged, categorized, prioritized
  - **Impact**: Visibility into technical debt, deletion plan ready
  - **Deadline**: January 3, 2026

**Week 4**:
- [ ] **Workflow Cleanup - Phase 1**
  - **Deliverable**: 60-80 workflows deactivated (orphaned, duplicates, abandoned)
  - **Success Criteria**: Active workflow count <150 (50% reduction)
  - **Impact**: Reduced HubSpot performance issues, fewer conflicts
  - **Deadline**: January 10, 2026

- [ ] **Property Audit & "Primary Inbound Channel" Fix**
  - **Deliverable**: Consolidate 3 conflicting attribution properties → 1 canonical
  - **Success Criteria**: 15 workflows updated, 8 reports fixed, old properties archived
  - **Impact**: Attribution data trustworthy for first time
  - **Deadline**: January 10, 2026

### Key Metrics (End of Week 4)

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| **Active Workflows** | <150 | | |
| **Workflows Documented** | 50 critical | | |
| **Email Deploy Time** | <5 min | | |
| **Attribution Fix** | 1 property | | |

---

## Week 5-6 Milestones (Optimization Phase)

### Critical Path Items

**Week 5**:
- [ ] **CSV Normalization Engine Live**
  - **Deliverable**: Airtable CSV upload + field mapping for 3 partner sources (Stripe, Mercury, Zoom)
  - **Success Criteria**: Christopher uploads 100-row CSV in <10 min (down from 2 hours)
  - **Impact**: Scales event management to 35-55 events/month (Q1 2026 volume)
  - **Deadline**: January 17, 2026

- [ ] **Governance Framework Implemented**
  - **Deliverable**: Workflow approval form + sunset policy automation live
  - **Success Criteria**: 5+ workflow requests submitted via new process
  - **Impact**: Prevents future technical debt accumulation
  - **Deadline**: January 17, 2026

**Week 6**:
- [ ] **Workflow Cleanup - Phase 2**
  - **Deliverable**: Additional 50 workflows deactivated (target: <100 total active)
  - **Success Criteria**: Event workflows consolidated into 1 template workflow
  - **Impact**: 67% workflow reduction (300+ → <100)
  - **Deadline**: January 24, 2026

- [ ] **Documentation Complete**
  - **Deliverable**: All systems documented (Markdown in GitHub + PDFs in Drive)
  - **Success Criteria**: Christopher can troubleshoot issues independently using docs
  - **Impact**: Reduces reliance on Skydog Ops post-engagement
  - **Deadline**: January 24, 2026

### Key Metrics (End of Week 6)

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| **Active Workflows** | <100 | | |
| **CSV Import Time** | <10 min | | |
| **Governance Adoption** | 5+ requests | | |
| **Documentation** | 100% complete | | |

---

## Week 7-8 Milestones (Handoff Phase)

### Critical Path Items

**Week 7**:
- [ ] **Training Complete**
  - **Deliverable**: 5 video tutorials + 2-hour live training with Christopher + Demand Gen hire
  - **Success Criteria**: Team can perform all key tasks without Skydog Ops help
  - **Impact**: Knowledge transfer successful, engagement can close
  - **Deadline**: January 31, 2026

**Week 8**:
- [ ] **Final Review & Acceptance**
  - **Deliverable**: All Phase 3 acceptance criteria met, Christopher signs off
  - **Success Criteria**: No blocking issues, 30-day support plan active
  - **Impact**: Project complete, final payment released
  - **Deadline**: February 7, 2026

- [ ] **30-Day Support Begins**
  - **Deliverable**: Office hours scheduled (2 hrs/week), Slack channel active
  - **Success Criteria**: Skydog Ops responds to questions within 24 hours
  - **Impact**: Safety net for Christopher during transition
  - **Duration**: February 7 - March 7, 2026

### Key Metrics (End of Week 8)

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| **Active Workflows** | <100 | | |
| **Time Saved** | 20 hrs/month | | |
| **Training** | 100% complete | | |
| **Project Status** | Closed | | |

---

## Success Metrics Dashboard (Cumulative)

### Efficiency Gains

| Metric | Before | After (Week 8) | Improvement |
|--------|--------|----------------|-------------|
| **Event Import Time** | 2-3 hours | <10 minutes | 95% reduction |
| **Email Build Time** | 2-3 hours | <30 minutes | 85% reduction |
| **Email Deploy Time** | 30 minutes | <5 minutes | 85% reduction |
| **Workflow Troubleshooting** | 10 hrs/month | <3 hrs/month | 70% reduction |
| **Christopher's Weekly Hours** | 30 hrs (100% tactical) | 10 hrs (70% strategic) | 67% time freed |

### System Health

| Metric | Before | After (Week 8) | Improvement |
|--------|--------|----------------|-------------|
| **Active HubSpot Workflows** | 300+ | <100 | 67% reduction |
| **Documented Workflows** | ~5% | 100% | 20x increase |
| **Attribution Properties** | 3 conflicting | 1 canonical | Consolidated |
| **Event Sync Latency** | 24-72 hours | <5 minutes | 99% faster |
| **Workflow Error Rate** | 15% | <5% | 67% reduction |

### Financial Impact

| Category | Annual Savings | Source |
|----------|----------------|--------|
| **Time Savings** | $30,000 | Christopher's time freed (20 hrs/month × $75/hr × 12 months) |
| **HubSpot Contact Reduction** | $3,000 | Reduced contact bloat from event emails |
| **Faster Lead Routing** | $20,000+ | Sales can engage leads 45 min faster → higher conversion |
| **Total ROI** | **$53,000/year** | vs. $32K one-time investment = **166% ROI** |

---

## Blockers & Escalation Path

### Red Flags (Immediate Escalation Required)

| Blocker | Owner | Escalation Path | Deadline |
|---------|-------|----------------|----------|
| **Leadership alignment meeting not scheduled** | Christopher | Escalate to Tommy via email (Dec 3) | Dec 6 |
| **API credentials not provided** | Christopher | Skydog Ops blocks, notify Tommy | Dec 3 |
| **Skydog Ops team member unavailable** | Skydog Ops | Assign backup team member | Within 24 hrs |
| **Critical workflow deleted by mistake** | Christopher + Skydog | Reactivate immediately, document issue | Within 4 hrs |
| **HubSpot API changes break integration** | Skydog Ops | Allocate 4 hrs buffer for migration | Within 48 hrs |

### Weekly Blocker Review

**Every Monday Sync Call**:
1. Review previous week's blockers (resolved?)
2. Identify new blockers for current week
3. Assign owner + escalation path
4. Confirm resolution deadline

---

## Communication Cadence

### Weekly Updates (Every Monday)

**Format**: 30-min Zoom call
**Attendees**: Christopher + Skydog Ops Lead
**Agenda**:
- Metrics review (compare targets vs. actuals)
- Demo new deliverables
- Blocker review
- Confirm next week priorities

### Leadership Updates (Bi-Weekly)

**Format**: 5-min Slack update or 15-min Zoom (if requested)
**Attendees**: Christopher → Tommy (CRO) + Jeremy (Head of Growth)
**Content**:
- High-level progress (on track / at risk / behind)
- Key wins (e.g., "60 workflows deactivated, no incidents")
- Risks/blockers requiring leadership decisions
- Upcoming milestones

**Schedule**:
- Week 2 (Dec 13): Phase 1 progress
- Week 4 (Jan 10): Phase 2 progress
- Week 6 (Jan 24): Phase 3 progress
- Week 8 (Feb 7): Final wrap-up

### Phase Demos (End of Each Phase)

**Phase 1 Demo** (Week 3, Dec 20):
- Live demo: Luma registration → Airtable → HubSpot
- Live demo: Build email using component library
- Q&A: Tommy, Jeremy, Christopher

**Phase 2 Demo** (Week 6, Jan 24):
- Live demo: Commit code → GitHub Actions → HubSpot deploy
- Show workflow audit results (300+ → <100)
- Q&A: Leadership + Demand Gen hire (if onboarded)

**Phase 3 Demo** (Week 8, Feb 7):
- Show CSV upload workflow
- Show governance framework (approval form, sunset policy)
- Review all documentation
- Final acceptance sign-off

---

## Risk Register (Top 5)

| Risk | Probability | Impact | Mitigation | Status |
|------|-------------|--------|------------|--------|
| **Leadership alignment delayed** | Medium | High | Christopher escalates to Tommy Dec 3 | Open |
| **Workflow deletion breaks automation** | Low | High | 30-day "soft delete" with monitoring | Open |
| **Demand Gen hire not onboarded by Week 3** | Medium | Low | Record training, provide async | Open |
| **HubSpot API rate limits** | Medium | Medium | Exponential backoff, batch calls | Open |
| **Skydog Ops resource unavailable** | Low | Medium | Backup team member assigned | Open |

---

## Post-Engagement Roadmap (Q1 2026+)

### Immediate Next Steps (Weeks 9-12, With Skydog Support)

1. **Monitor & Stabilize** (Weeks 9-10)
   - Christopher monitors all systems for issues
   - Uses Skydog Ops office hours for troubleshooting (2 hrs/week)
   - Documents any edge cases discovered

2. **Demand Gen Onboarding** (Weeks 10-11)
   - Train Demand Gen hire on Events Automation (1 hour)
   - Train on Email Component Library (1 hour)
   - Train on HubSpot governance (workflow approval process)

3. **Iterate Based on Feedback** (Week 12)
   - Collect feedback from Growth team (what's working, what needs adjustment)
   - Minor enhancements (within 30-day support scope)
   - Plan Q1 2026 enhancements (out of scope, separate project)

### Future Enhancements (Q1-Q2 2026, Separate Projects)

**Q1 2026 Priorities**:
1. **Storyblok CMS Integration** (3-4 weeks, $15-20K)
   - Enable Growth team to create emails without Christopher
   - Bypass HubSpot contact creation (save $3K+/year)
   - See [04-Email Template Architecture](04-email-template-architecture.md)

2. **Self-Service Events Management UI** (6-8 weeks, $25-30K)
   - Web app for Sales/Partnerships to upload CSVs
   - Real-time event dashboard (registration counts, sync status)
   - See [11-Events Management UI](11-events-management-ui.md)

3. **Engineering Event Emission** (Engineering team project)
   - Fix 40-45 minute gap (emit events on app progress, not just submission)
   - Requires Engineering sprint allocation (not MOps budget)
   - See [01-Root Cause Analysis](01-root-cause-analysis.md) for technical specs

**Q2 2026 Priorities**:
4. **Customer.io Migration** (4-5 weeks, $20K)
   - Move event emails from HubSpot → Customer.io
   - Integrate with Airtable Events Automation
   - Reduce HubSpot contact count (cost savings)

5. **Attribution Model Redesign** (Leadership project)
   - Workshop with Tommy to define attribution methodology
   - Then technical implementation (2-3 weeks, $10K)
   - Consolidate all attribution tracking → single source of truth

---

## Appendix: Weekly Task Summary (Quick Reference)

| Week | Phase | Key Deliverables | Hours (Skydog) | Budget |
|------|-------|------------------|----------------|--------|
| **Week 1** | Foundation | Kickoff, Leadership Meeting, Airtable Setup | 15 | $1,500 |
| **Week 2** | Foundation | Events Automation MVP, Component Library | 40 | $5,000 |
| **Week 3** | Foundation | Email Templates Deployed, Training | 30 | $5,500 |
| **Week 4** | Automation | CI/CD Pipeline, Workflow Audit | 35 | $6,000 |
| **Week 5** | Automation | Workflow Cleanup, Property Audit | 30 | $4,000 |
| **Week 6** | Automation | Documentation, Governance Framework | 25 | $3,000 |
| **Week 7** | Handoff | CSV Normalization, Training Videos | 25 | $3,500 |
| **Week 8** | Handoff | Final Review, Support Plan Active | 25 | $3,500 |
| **Total** | | | **225 hrs** | **$32,000** |

---

**Last Updated**: 2025-11-12
**Document Owner**: Christopher Cooper
**Next Review**: End of Week 2 (December 13, 2025)

**Questions?** Contact Christopher Cooper (christopher.cooper@rho.co)
