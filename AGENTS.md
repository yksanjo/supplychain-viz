# SupplyChainViz - Agent Context

## Project Identity

**Agent ID:** 3.2  
**Agent Name:** Supply Chain Monitor  
**Project Name:** SupplyChainViz  
**Status:** Concept Phase  
**Last Updated:** 2026-02-13

---

## Quick Reference

### What It Is
Mid-market supply chain visibility platform with AI-powered predictive disruption alerts.

### The Problem
- Enterprise solutions (SAP, Oracle, Kinaxis) cost $100K+ and take 6-12 months
- Mid-market manufacturers ($20M-$200M) are completely underserved
- They manage 50-500 suppliers with spreadsheets and manual tracking
- Reactive problem solving (find out about delays when customers complain)

### The Solution
- Real-time digital twin of supply chain
- Predictive disruption alerts (7-14 days advance notice)
- $499-1,999/month pricing (vs $100K+ enterprise tools)
- 2-4 week implementation (vs 6-12 months)

### Target Customer
- Mid-market manufacturers: $20M-$200M revenue
- Pain: Production delays, stockouts, spreadsheet hell
- Tech stack: NetSuite, Acumatica, SAP Business One

### Revenue Model
| Tier | Price | Target |
|------|-------|--------|
| Starter | $499/mo | 50 suppliers, basic dashboards |
| Growth | $999/mo | 200 suppliers, predictive alerts |
| Scale | $1,999/mo | Unlimited, AI predictions |

### Key Metrics
- Goal: 100 customers by Month 24
- Target ARR: $1.5M
- ACV: $15K
- CAC: <$1,500
- LTV:CAC: >30:1

---

## Important Files

| File | Purpose |
|------|---------|
| `README.md` | Project overview, value proposition |
| `docs/TECHNICAL_ARCHITECTURE.md` | System design, tech stack, data models |
| `docs/COMPETITIVE_ANALYSIS.md` | Competitor research, positioning |
| `docs/GTM_STRATEGY.md` | Go-to-market plan, pricing, marketing |
| `docs/MVP_PLAN.md` | 12-week implementation roadmap |

---

## Key Decisions

### Tech Stack
- **Frontend:** React + TypeScript + TailwindCSS
- **Backend:** Node.js + Express
- **Database:** PostgreSQL + ClickHouse (analytics)
- **AI/ML:** Python microservices
- **Hosting:** AWS (production), Railway (MVP)

### MVP Scope (8-12 weeks)
1. NetSuite integration only (focus)
2. Inventory dashboard
3. Purchase order tracking
4. Basic alerts (low stock, late POs)
5. Mock AI predictions (for demo)

### Out of Scope for MVP
- Multi-tier supplier visibility
- Real AI predictions
- EDI integrations
- Mobile app
- Multiple ERP connectors

---

## Next Actions

### Immediate (This Week)
- [ ] Validate with 3 mid-market manufacturers
- [ ] Set up NetSuite developer account
- [ ] Create simple landing page for waitlist

### Short-term (Month 1)
- [ ] Interview 20+ target customers
- [ ] Build technical proof of concept
- [ ] Close 3 design partner commitments

### Medium-term (Months 2-3)
- [ ] Build MVP
- [ ] Onboard pilot customers
- [ ] Iterate based on feedback

---

## Competitive Context

**Primary Competitors:**
- SAP IBP ($100K-500K) - too complex/expensive
- Kinaxis ($75K-300K) - enterprise only
- Anvyl ($1K-5K/mo) - limited predictive

**Our Edge:**
- AI-first (predictive vs. reactive)
- Mid-market focus (vs. enterprise)
- Fast implementation (2 weeks vs. 6 months)

---

## Resources Needed

### Team
- 2 full-stack engineers (MVP)
- 1 designer (part-time)
- 1 product manager (founder)

### Budget
- Development: $40K (3 months contractor costs)
- Infrastructure: $400/month
- Marketing: $150K (Year 1)
- Sales: $200K (Year 1)

---

## Risk Factors

1. **Integration complexity** → Mitigation: Start with NetSuite only
2. **Long sales cycles** → Mitigation: Pilot program, ROI guarantee
3. **Competitor response** → Mitigation: Speed to market, AI differentiation
4. **Market timing** → Mitigation: Post-COVID supply chain awareness is high

---

## Success Definition

**6 Months:**
- 10 paying customers
- $10K MRR
- Product-market fit signals

**12 Months:**
- 35 customers
- $35K MRR
- <$1,000 CAC

**24 Months:**
- 100 customers
- $1.5M ARR
- Series A ready

---

## Contact

**Owner:** Agent 3.2 (Supply Chain Monitor)  
**Docs:** `/supplychain-viz/`  
**Related:** See `SAAS_ENGINEER_PROJECT_IDEAS.md` for similar opportunities
