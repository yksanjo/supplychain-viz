# SupplyChainViz MVP Implementation Plan

## MVP Scope

**Goal:** Build a minimal viable product that delivers core value in 8-12 weeks.

**MVP Definition:**
> A supply chain visibility dashboard for NetSuite users that shows real-time inventory, purchase orders, and basic alerts.

**Out of Scope for MVP:**
- AI predictions (will be mocked)
- Multi-tier supplier visibility
- Digital twin simulation
- EDI integrations
- Mobile app

---

## MVP Features

### Tier 1: Must-Have (Weeks 1-6)

| Feature | Description | Value |
|---------|-------------|-------|
| **NetSuite Integration** | OAuth connection, sync inventory/POs | Core functionality |
| **Inventory Dashboard** | Real-time inventory levels by SKU/warehouse | Visibility |
| **PO Tracking** | Purchase order status and timeline | Basic tracking |
| **Alerts** | Email notifications for low stock, late POs | Proactive management |
| **User Auth** | Login, roles (admin/viewer) | Security |

### Tier 2: Should-Have (Weeks 7-9)

| Feature | Description | Value |
|---------|-------------|-------|
| **Basic Reporting** | Inventory valuation, PO spend | Insights |
| **Supplier Directory** | List of suppliers with contact info | Organization |
| **CSV Export** | Download data for analysis | Flexibility |
| **Dashboard Widgets** | Customizable home dashboard | UX |

### Tier 3: Nice-to-Have (Weeks 10-12)

| Feature | Description | Value |
|---------|-------------|-------|
| **Mock AI Predictions** | Simulated delay predictions | Demo value |
| **Mobile Responsive** | Works on phone/tablet | Convenience |
| **Onboarding Wizard** | Self-service setup | Time to value |

---

## Technical Implementation

### Architecture (MVP Simplified)

```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND                                │
│                   React + TypeScript                         │
│              TailwindCSS + Recharts                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      BACKEND                                 │
│                    Node.js / Express                         │
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Auth      │  │  NetSuite   │  │   Alerts    │         │
│  │   Service   │  │  Connector  │  │   Engine    │         │
│  └─────────────┘  └──────┬──────┘  └─────────────┘         │
│                          │                                  │
│  ┌───────────────────────▼──────────────────────────┐       │
│  │              PostgreSQL Database                  │       │
│  │  • Users  • Companies  • Inventory  • POs        │       │
│  └───────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology | Why |
|-------|------------|-----|
| Frontend | React + Vite | Fast dev, great ecosystem |
| Styling | TailwindCSS | Rapid UI development |
| Charts | Recharts | React-native charting |
| Backend | Node.js + Express | JavaScript full stack |
| Database | PostgreSQL | Reliable, scalable |
| ORM | Prisma | Type-safe database |
| Auth | Auth0 / Clerk | Secure, fast to implement |
| Email | SendGrid | Transactional emails |
| Hosting | Railway / Render | Simple deployment |
| Monitoring | LogRocket | Session replay |

### Database Schema (MVP)

```sql
-- Simplified MVP Schema

CREATE TABLE companies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    netsuite_account_id VARCHAR(100),
    netsuite_access_token TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID REFERENCES companies(id),
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(50) DEFAULT 'viewer', -- admin, viewer
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE suppliers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID REFERENCES companies(id),
    netsuite_id VARCHAR(100),
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    phone VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID REFERENCES companies(id),
    netsuite_id VARCHAR(100),
    sku VARCHAR(100) NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    unit_cost DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id UUID REFERENCES products(id),
    warehouse_name VARCHAR(255),
    quantity_on_hand INTEGER DEFAULT 0,
    quantity_available INTEGER DEFAULT 0,
    safety_stock INTEGER DEFAULT 0,
    last_updated TIMESTAMP DEFAULT NOW(),
    UNIQUE(product_id, warehouse_name)
);

CREATE TABLE purchase_orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID REFERENCES companies(id),
    supplier_id UUID REFERENCES suppliers(id),
    netsuite_id VARCHAR(100),
    po_number VARCHAR(100) NOT NULL,
    status VARCHAR(50), -- pending, partial, received
    order_date DATE,
    expected_date DATE,
    total DECIMAL(12,2),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE po_lines (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    po_id UUID REFERENCES purchase_orders(id),
    product_id UUID REFERENCES products(id),
    quantity INTEGER,
    quantity_received INTEGER DEFAULT 0,
    unit_price DECIMAL(10,2)
);

CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID REFERENCES companies(id),
    alert_type VARCHAR(50), -- low_stock, late_po
    severity VARCHAR(20), -- info, warning, critical
    title VARCHAR(255),
    description TEXT,
    entity_type VARCHAR(50),
    entity_id UUID,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_inventory_company ON products(company_id);
CREATE INDEX idx_po_company ON purchase_orders(company_id);
CREATE INDEX idx_alerts_company ON alerts(company_id, is_read, created_at);
```

---

## Week-by-Week Plan

### Week 1: Setup & Foundation

**Day 1-2: Project Setup**
```bash
# Initialize project
mkdir supplychain-viz-mvp
cd supplychain-viz-mvp
npm init -y

# Setup structure
mkdir -p frontend backend shared
cd frontend && npm create vite@latest . -- --template react-ts
cd ../backend && npm init -y

# Install dependencies
# Frontend: React, Tailwind, React Query, Axios, Recharts
# Backend: Express, Prisma, PostgreSQL driver
```

**Day 3-4: Database & Models**
- Set up PostgreSQL (local or Supabase)
- Define Prisma schema
- Create initial migrations
- Seed with test data

**Day 5: Auth Foundation**
- Set up Auth0/Clerk account
- Implement login/logout
- Protect routes

### Week 2: NetSuite Integration

**Day 1-2: NetSuite Research**
- Sign up for NetSuite Developer Account
- Study REST API documentation
- Understand OAuth flow

**Day 3-4: Connector Development**
```typescript
// Core NetSuite connector
class NetSuiteConnector {
  async authenticate(credentials: Credentials): Promise<Token>;
  async syncInventory(): Promise<Inventory[]>;
  async syncPurchaseOrders(): Promise<PurchaseOrder[]>;
  async syncSuppliers(): Promise<Supplier[]>;
  async syncProducts(): Promise<Product[]>;
}
```

**Day 5: Data Sync**
- Build sync job (runs every 15 min)
- Handle incremental updates
- Error handling and retries

### Week 3: Core Backend APIs

**Endpoints to Build:**
```
GET    /api/me                    # Current user
GET    /api/dashboard             # Dashboard summary
GET    /api/inventory             # List inventory
GET    /api/inventory/:id         # Inventory detail
GET    /api/purchase-orders       # List POs
GET    /api/purchase-orders/:id   # PO detail
GET    /api/suppliers             # List suppliers
GET    /api/alerts                # List alerts
POST   /api/alerts/:id/read       # Mark alert read
POST   /api/netsuite/connect      # Connect NetSuite
POST   /api/netsuite/sync         # Trigger sync
```

### Week 4: Frontend Dashboard

**Pages to Build:**
1. **Login Page**
2. **Dashboard (Home)**
   - KPI cards (total inventory, pending POs, alerts)
   - Recent activity feed
   - Low stock warning

3. **Inventory Page**
   - Table with sorting/filtering
   - Search by SKU/name
   - Stock level indicators

4. **Purchase Orders Page**
   - PO list with status
   - Timeline view
   - Filter by supplier/status

5. **Suppliers Page**
   - Supplier directory
   - Contact info

### Week 5: Alerts & Polish

**Alert System:**
- Background job checks for:
  - Inventory below safety stock
  - POs past expected date
  - Unusual inventory changes
- Email notifications via SendGrid
- In-app notification center

**UI Polish:**
- Loading states
- Error handling
- Empty states
- Mobile responsiveness pass

### Week 6: Testing & Fixes

**Testing:**
- Manual end-to-end testing
- Fix critical bugs
- Performance optimization
- Security review

**Documentation:**
- API documentation
- User guide
- Admin setup guide

### Week 7-8: Pilot Customer Onboarding

**Activities:**
- Onboard 3 design partners
- Gather feedback
- Fix issues
- Iterate on UX

### Week 9-12: Enhancement & Launch Prep

**Features:**
- Add reporting (basic)
- CSV export
- Onboarding wizard
- Mock AI predictions (for demo)

**Launch Prep:**
- Landing page
- Pricing page
- Demo video
- Signup flow

---

## Development Guidelines

### Code Structure

```
supplychain-viz/
├── frontend/
│   ├── src/
│   │   ├── components/     # Reusable UI components
│   │   ├── pages/          # Route-level pages
│   │   ├── hooks/          # Custom React hooks
│   │   ├── api/            # API client
│   │   ├── types/          # TypeScript types
│   │   └── utils/          # Helper functions
│   └── public/
├── backend/
│   ├── src/
│   │   ├── routes/         # API routes
│   │   ├── services/       # Business logic
│   │   ├── connectors/     # ERP integrations
│   │   ├── models/         # Database models
│   │   ├── jobs/           # Background jobs
│   │   └── utils/          # Helper functions
│   └── prisma/
│       └── schema.prisma   # Database schema
└── shared/
    └── types/              # Shared TypeScript types
```

### Key Decisions

1. **Monorepo vs. Separate:** Start with separate repos for simplicity
2. **Testing:** Unit tests for critical logic, E2E tests for core flows
3. **Deployment:** Railway for easy deployment
4. **Scaling:** Design for 10 companies in MVP, but architecture supports 100+

---

## Success Criteria for MVP

### Technical
- [ ] 99% uptime
- [ ] <2 second page load times
- [ ] Handles 100 concurrent users
- [ ] Syncs data within 5 minutes of NetSuite changes

### Business
- [ ] 3 pilot customers actively using
- [ ] >50% daily active usage
- [ ] Positive NPS (>30)
- [ ] Clear path to first 10 paying customers

### Product
- [ ] Complete data sync from NetSuite
- [ ] Working alerts (email + in-app)
- [ ] All core dashboard features functional
- [ ] <4 hour setup time for new customer

---

## Post-MVP Roadmap

### Month 4-6: Product-Market Fit
- Add Acumatica connector
- AI predictions (real, not mocked)
- Multi-tier supplier visibility
- Advanced reporting

### Month 7-12: Scale
- Additional ERP connectors (SAP B1, MS Dynamics)
- Mobile app
- Supplier portal
- EDI integration

---

## Budget & Resources

### Team (MVP)
- 1 Full-stack Engineer (lead)
- 1 Frontend Engineer
- 1 Part-time Designer

### Timeline
- **8-12 weeks** for MVP
- 2-3 weeks per major feature

### Costs
| Category | Cost |
|----------|------|
| Infrastructure | $200/month |
| Auth0/Clerk | $100/month |
| SendGrid | $50/month |
| Monitoring | $50/month |
| **Total Monthly** | **$400** |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| NetSuite API complexity | Start with developer account, test thoroughly |
| Slow development | Cut scope aggressively, focus on core loop |
| No pilot customers | Start customer conversations Week 1 |
| Data sync issues | Build robust retry logic, manual fallback |

---

**Version:** 1.0  
**Last Updated:** 2026-02-13
