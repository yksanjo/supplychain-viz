# SupplyChainViz Technical Architecture

## System Overview

SupplyChainViz is a cloud-native SaaS platform built on a microservices architecture, designed for horizontal scalability and real-time data processing.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│  React Dashboard    Mobile App    Embedded Widgets    API Clients           │
│     (Web)           (React        (Supplier          (ERP Systems)         │
│                      Native)       Portal)                                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              API GATEWAY                                     │
│                    (Kong / AWS API Gateway)                                  │
│         Auth (JWT) • Rate Limiting • Request Routing • SSL                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SERVICE MESH                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Data      │ │  Analytics  │ │   Alert     │ │   AI/ML Prediction  │   │
│  │ Ingestion   │ │   Engine    │ │   Service   │ │     Service         │   │
│  │  Service    │ │             │ │             │ │                     │   │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └──────────┬──────────┘   │
│         │               │               │                   │              │
│  ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐ ┌──────────▼──────────┐   │
│  │  Digital    │ │   Report    │ │   User      │ │   Integration       │   │
│  │   Twin      │ │   Engine    │ │ Management  │ │     Service         │   │
│  │  Service    │ │             │ │             │ │                     │   │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  PostgreSQL  │  │  ClickHouse  │  │    Redis     │  │    MinIO     │    │
│  │ (Primary DB) │  │ (Analytics)  │  │    (Cache)   │  │  (File Store)│    │
│  │              │  │              │  │              │  │              │    │
│  │ • Companies  │  │ • Time-series│  │ • Sessions   │  │ • Documents  │    │
│  │ • Users      │  │ • Events     │  │ • Real-time  │  │ • EDI Files  │    │
│  │ • Suppliers  │  │ • Aggregates │  │   data       │  │ • Reports    │    │
│  │ • Products   │  │              │  │              │  │              │    │
│  │ • Orders     │  │              │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        INTEGRATION LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                     ETL Pipeline (Apache Airflow)                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│         │                  │                  │                    │        │
│  ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐      ┌──────▼──────┐ │
│  │  ERP        │    │   EDI       │    │  External   │      │    IoT      │ │
│  │ Connectors  │    │  Gateway    │    │    APIs     │      │  Sensors    │ │
│  │             │    │             │    │             │      │             │ │
│  │ • NetSuite  │    │ • X12       │    │ • Weather   │      │ • RFID      │ │
│  │ • Acumatica │    │ • EDIFACT   │    │ • Freight   │      │ • GPS       │ │
│  │ • SAP B1    │    │ • CSV/XML   │    │ • Financial │      │ • Temp      │ │
│  │ • MS Dynam. │    │             │    │ • News/Risk │      │             │ │
│  └─────────────┘    └─────────────┘    └─────────────┘      └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Core Services

### 1. Data Ingestion Service

**Purpose:** Normalize and ingest supply chain data from multiple sources

**Key Components:**
```typescript
interface DataIngestionService {
  // Connector management
  registerConnector(config: ConnectorConfig): Promise<Connector>;
  syncData(connectorId: string): Promise<SyncResult>;
  
  // Data transformation
  transform(rawData: unknown, mapping: DataMapping): NormalizedRecord;
  validate(record: NormalizedRecord): ValidationResult;
  
  // Real-time streaming
  subscribeToChanges(connectorId: string, callback: ChangeHandler): Subscription;
}
```

**Data Model (Normalized):**
```typescript
interface SupplyChainEntity {
  id: string;
  companyId: string;
  entityType: 'supplier' | 'product' | 'order' | 'shipment' | 'inventory';
  sourceSystem: string;
  externalId: string;
  data: Record<string, unknown>;
  metadata: {
    ingestedAt: Date;
    lastUpdated: Date;
    version: number;
    checksum: string;
  };
}
```

### 2. Digital Twin Service

**Purpose:** Maintain real-time digital representation of physical supply chain

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                    DIGITAL TWIN ENGINE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐      ┌─────────────┐      ┌───────────┐  │
│   │   Entity    │──────▶│   State     │──────▶│  Graph    │  │
│   │   Builder   │      │  Manager    │      │  Model    │  │
│   └─────────────┘      └─────────────┘      └─────┬─────┘  │
│                                                    │        │
│   ┌─────────────┐      ┌─────────────┐            │        │
│   │  Scenario   │◀─────│  Simulator  │◀───────────┘        │
│   │   Engine    │      │             │                     │
│   └─────────────┘      └─────────────┘                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Graph Model:**
```cypher
// Neo4j-style graph representation
(Supplier)-[:SUPPLIES {leadTime: 5, minOrder: 1000}]->(Component)
(Component)-[:PART_OF {quantity: 4}]->(Product)
(Product)-[:STORED_AT]->(Warehouse)
(Warehouse)-[:SHIPS_TO {transitTime: 2}]->(Customer)
```

### 3. AI Prediction Service

**Purpose:** Predict disruptions and optimize supply chain decisions

**ML Pipeline:**
```python
# Simplified architecture
data_ingestion → feature_engineering → model_training → prediction_api

# Models:
1. Delay Prediction (LSTM/Transformer)
   - Input: Historical shipments, weather, port data
   - Output: Probability of delay, estimated delay days
   
2. Demand Forecasting (Prophet + XGBoost)
   - Input: Sales history, seasonality, trends
   - Output: 30/60/90 day demand predictions
   
3. Supplier Risk Scoring (Ensemble)
   - Input: Financial data, news, performance history
   - Output: Risk score (0-100), risk factors
   
4. Inventory Optimization (Reinforcement Learning)
   - Input: Demand forecast, carrying costs, stockout costs
   - Output: Optimal reorder points and quantities
```

**Prediction API:**
```typescript
interface PredictionService {
  predictDelay(shipmentId: string): Promise<DelayPrediction>;
  forecastDemand(productId: string, horizon: number): Promise<DemandForecast>;
  scoreSupplierRisk(supplierId: string): Promise<RiskScore>;
  optimizeInventory(warehouseId: string): Promise<InventoryRecommendation[]>;
  simulateScenario(scenario: ScenarioConfig): Promise<SimulationResult>;
}
```

### 4. Alert Service

**Purpose:** Multi-channel notification system for supply chain events

**Alert Types:**
1. **Threshold Alerts** - Inventory below safety stock
2. **Predictive Alerts** - Predicted delay 7 days out
3. **Anomaly Alerts** - Unusual patterns detected
4. **Risk Alerts** - Supplier risk score changed
5. **System Alerts** - Integration failures, data issues

**Routing Rules:**
```yaml
alert_rules:
  - name: critical_inventory
    condition: inventory.days_of_supply < 7
    severity: critical
    channels: [email, sms, in_app, webhook]
    recipients: [supply_manager, buyer]
    
  - name: delay_prediction
    condition: delay.confidence > 0.8 AND delay.impact_score > 7
    severity: warning
    channels: [email, in_app]
    recipients: [logistics_team]
```

---

## Database Schema

### PostgreSQL (Primary)

```sql
-- Companies
CREATE TABLE companies (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    tier VARCHAR(50) CHECK (tier IN ('starter', 'growth', 'scale', 'enterprise')),
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Suppliers (multi-tenant)
CREATE TABLE suppliers (
    id UUID PRIMARY KEY,
    company_id UUID REFERENCES companies(id),
    name VARCHAR(255) NOT NULL,
    tier INTEGER CHECK (tier IN (1, 2, 3)), -- Tier 1, 2, 3
    risk_score INTEGER CHECK (risk_score BETWEEN 0 AND 100),
    contact_info JSONB,
    capabilities JSONB, -- ISO certs, capacity, etc.
    location GEOGRAPHY(POINT),
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Products / SKUs
CREATE TABLE products (
    id UUID PRIMARY KEY,
    company_id UUID REFERENCES companies(id),
    sku VARCHAR(100) NOT NULL,
    name VARCHAR(255) NOT NULL,
    category VARCHAR(100),
    unit_cost DECIMAL(12,2),
    bom JSONB, -- Bill of materials
    created_at TIMESTAMP DEFAULT NOW()
);

-- Inventory
CREATE TABLE inventory (
    id UUID PRIMARY KEY,
    product_id UUID REFERENCES products(id),
    warehouse_id UUID REFERENCES warehouses(id),
    quantity_on_hand INTEGER DEFAULT 0,
    quantity_reserved INTEGER DEFAULT 0,
    quantity_available INTEGER GENERATED ALWAYS AS (quantity_on_hand - quantity_reserved) STORED,
    safety_stock INTEGER DEFAULT 0,
    reorder_point INTEGER DEFAULT 0,
    last_counted_at TIMESTAMP,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Purchase Orders
CREATE TABLE purchase_orders (
    id UUID PRIMARY KEY,
    company_id UUID REFERENCES companies(id),
    supplier_id UUID REFERENCES suppliers(id),
    po_number VARCHAR(100) NOT NULL,
    status VARCHAR(50) CHECK (status IN ('draft', 'sent', 'acknowledged', 'in_production', 'shipped', 'delivered', 'cancelled')),
    order_date DATE NOT NULL,
    expected_delivery DATE,
    total_amount DECIMAL(12,2),
    terms JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Shipments
CREATE TABLE shipments (
    id UUID PRIMARY KEY,
    po_id UUID REFERENCES purchase_orders(id),
    tracking_number VARCHAR(255),
    carrier VARCHAR(100),
    origin JSONB,
    destination JSONB,
    status VARCHAR(50) CHECK (status IN ('pending', 'picked_up', 'in_transit', 'out_for_delivery', 'delivered', 'exception')),
    estimated_delivery TIMESTAMP,
    actual_delivery TIMESTAMP,
    events JSONB[], -- Tracking events
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Predictions (for audit trail)
CREATE TABLE predictions (
    id UUID PRIMARY KEY,
    company_id UUID REFERENCES companies(id),
    prediction_type VARCHAR(50),
    entity_type VARCHAR(50),
    entity_id UUID,
    prediction JSONB NOT NULL,
    confidence DECIMAL(3,2),
    actual_outcome JSONB, -- For model training feedback
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_suppliers_company ON suppliers(company_id);
CREATE INDEX idx_inventory_product ON inventory(product_id);
CREATE INDEX idx_po_company_date ON purchase_orders(company_id, order_date);
CREATE INDEX idx_predictions_company ON predictions(company_id, prediction_type, created_at);
```

### ClickHouse (Analytics)

```sql
-- Time-series events
CREATE TABLE supply_chain_events (
    event_id UUID,
    company_id UUID,
    event_type Enum('inventory_change', 'shipment_update', 'order_status', 'prediction'),
    entity_type String,
    entity_id UUID,
    timestamp DateTime64(3),
    data JSON,
    metadata JSON
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (company_id, event_type, timestamp);

-- Aggregated metrics
CREATE TABLE daily_metrics (
    company_id UUID,
    date Date,
    metric_name String,
    metric_value Float64,
    dimensions JSON
) ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (company_id, date, metric_name);
```

---

## Integration Architecture

### ERP Connectors

```typescript
interface ERPConnector {
  name: string;
  auth: AuthMethod;
  
  // Data extraction
  extractSuppliers(): Promise<Supplier[]>;
  extractProducts(): Promise<Product[]>;
  extractInventory(): Promise<Inventory[]>;
  extractPurchaseOrders(since: Date): Promise<PurchaseOrder[]>;
  
  // Real-time sync
  subscribeToChanges(): Observable<ChangeEvent>;
  
  // Write-backs (future)
  createPurchaseOrder(po: PurchaseOrder): Promise<void>;
  updateInventory(sku: string, qty: number): Promise<void>;
}

// Example: NetSuite Connector
class NetSuiteConnector implements ERPConnector {
  private client: NetSuiteClient;
  
  async extractSuppliers(): Promise<Supplier[]> {
    const vendors = await this.client.search({
      type: 'vendor',
      columns: ['entityid', 'companyname', 'email', 'address']
    });
    
    return vendors.map(v => this.normalizeVendor(v));
  }
  
  subscribeToChanges(): Observable<ChangeEvent> {
    // Webhook or polling based on NetSuite capabilities
    return this.client.webhookStream();
  }
}
```

### EDI Gateway

```
┌─────────────────────────────────────────────────────────────┐
│                     EDI GATEWAY                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │   Receive    │───▶│   Parse      │───▶│   Transform  │   │
│  │   (AS2/SFTP) │    │   (X12/EDIFACT)│   │   (to JSON)  │   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│                                                        │     │
│  ┌──────────────┐    ┌──────────────┐    ┌────────────▼──┐  │
│  │   Send       │◀───│   Generate   │◀───│   Validate    │  │
│  │   (AS2/SFTP) │    │   (X12/EDIFACT)│   │               │  │
│  └──────────────┘    └──────────────┘    └───────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Security & Compliance

### Data Security
- **Encryption at Rest:** AES-256 for all databases
- **Encryption in Transit:** TLS 1.3 for all communications
- **PII Handling:** Tokenization for sensitive supplier/customer data
- **Backup:** Encrypted daily backups with 30-day retention

### Compliance
- **SOC 2 Type II:** In progress
- **GDPR:** Data residency controls, right to deletion
- **ISO 27001:** Target for Year 2

### Access Control
```typescript
interface RBACConfig {
  roles: {
    admin: ['read:*', 'write:*', 'delete:*'];
    manager: ['read:*', 'write:suppliers', 'write:orders'];
    analyst: ['read:*'];
    supplier: ['read:own_orders', 'write:own_confirmations'];
  };
  
  // Row-level security
  policies: [
    { resource: 'suppliers', condition: 'company_id = current_user.company_id' },
    { resource: 'orders', condition: 'company_id = current_user.company_id' }
  ];
}
```

---

## Scalability Plan

### Current: MVP Phase
- Single region (US-East)
- Shared database instances
- 100 companies, 10K suppliers

### Phase 2: Growth
- Multi-region (US-East, US-West, EU)
- Database read replicas
- 1,000 companies, 100K suppliers

### Phase 3: Scale
- Sharded database by company_id
- Dedicated ML inference clusters
- 10,000+ companies, 1M+ suppliers

---

## Monitoring & Observability

### Metrics (Prometheus/Grafana)
- API response times (p50, p95, p99)
- Integration sync success rates
- Prediction accuracy scores
- Data freshness (time since last update)

### Logging (ELK Stack)
- Structured JSON logging
- Correlation IDs across services
- Error tracking and alerting

### Alerting (PagerDuty)
- Critical: Integration failures > 1 hour
- Warning: Prediction accuracy < 80%
- Info: Data sync delays > 15 minutes

---

**Version:** 1.0  
**Last Updated:** 2026-02-13
