# Salesforce Agentforce Order & Inventory Backfilling Integration

A comprehensive Salesforce solution using **Agentforce** to automate the backfilling of historical online orders and inventory reconciliation from external e-commerce platforms (Shopify, Commerce Cloud, etc.) into Salesforce.

## Overview

This repository contains a production-ready SFDX project that:

- **Backfills historical orders** from online stores into Salesforce Order/OrderItem objects with idempotent, duplicate-prevention logic
- **Reconciles inventory** by adjusting product stock quantities to reflect synced orders
- **Integrates with Agentforce** so users can trigger backfills conversationally ("Backfill my Shopify orders from last month")
- **Audits all changes** with comprehensive job tracking and error logging
- **Supports multiple data sources** (Shopify, Commerce Cloud, custom APIs) via pluggable connector pattern

## Key Features

✅ **Idempotent Backfilling** — Safe to re-run; no duplicate orders  
✅ **Agentforce Integration** — Conversational orchestration of backfill operations  
✅ **Pluggable Connectors** — Support for Shopify, Commerce Cloud, mock data, and custom sources  
✅ **Job Tracking** — Detailed audit trail and error logging on `Backfill_Job__c`  
✅ **Inventory Discrepancy Detection** — Flags and logs stock reconciliation issues  
✅ **85%+ Test Coverage** — Comprehensive Apex tests including edge cases  
✅ **Production-Ready** — Follows Salesforce best practices (bulkification, error handling, security)

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Agentforce Agent                          │
│          (Conversational interface for backfills)             │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           │ Invokes
                           ▼
          ┌────────────────────────────────┐
          │  OrderBackfillAction           │
          │  (Invocable Apex method)       │
          └────────┬───────────────────────┘
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
┌──────────────────┐  ┌──────────────────────────┐
│ Order Backfill   │  │ Inventory Adjustment     │
│ Service          │  │ Service                  │
└────────┬─────────┘  └────────┬─────────────────┘
         │                     │
         │                     ▼
         │           ┌─────────────────────┐
         │           │ Inventory Discount  │
         │           │ Flagging            │
         │           └─────────────────────┘
         │
         ▼
┌──────────────────────────┐
│ External Order Connector │
│ (Pluggable interface)    │
└────────┬─────────────────┘
         │
    ┌────┴────┬─────────┬──────────┐
    ▼         ▼         ▼          ▼
 Shopify  Commerce  Mock        Custom
         Cloud    Connector       API
```

## Quick Start

### Prerequisites
- Salesforce DX CLI (`sf` or `sfdx`)
- A Salesforce org (production, sandbox, or scratch org)
- Node.js/npm (for SFDX)

### Deploy to a Scratch Org

```bash
# Clone the repository
git clone https://github.com/amashfak2020-ai/aforceAI.git
cd aforceAI

# Create and open a scratch org
sf org create scratch -f config/project-scratch-def.json --set-default

# Deploy the project
sf project deploy start

# Run tests
sf apex run test --test-level RunAllTestsInNamespace

# Open the org
sf org open
```

### Configure External API Connection

1. In your Salesforce org, navigate to **Setup** → **Named Credentials**
2. Create a Named Credential `ExternalOrderAPI` with your external store API details
3. Update `force-app/main/default/namedCredentials/ExternalOrderAPI.namedCredential-meta.xml` with your endpoint

### Set Up Agentforce Integration

See **[AGENTFORCE_SETUP.md](./docs/AGENTFORCE_SETUP.md)** for detailed instructions on:
- Registering the `OrderBackfillAction` as an Agentforce action
- Configuring agent topics and prompts
- Testing end-to-end conversational workflows

## Project Structure

```
aforceAI/
├── force-app/main/default/
│   ├── classes/
│   │   ├── OrderBackfillAction.cls
│   │   ├── OrderBackfillService.cls
│   │   ├── InventoryAdjustmentService.cls
│   │   ├── ExternalOrderConnector.cls
│   │   ├── ShopifyOrderConnector.cls
│   │   ├── MockOrderConnector.cls
│   │   ├── BackfillJobManager.cls
│   │   └── *_Test.cls (test classes)
│   ├── objects/
│   │   ├── Backfill_Job__c.object-meta.xml
│   │   └── Order.object-meta.xml
│   ├── namedCredentials/
│   │   └── ExternalOrderAPI.namedCredential-meta.xml
│   └── ...
├── docs/
│   ├── README.md (this file)
│   ├── SETUP.md (deployment guide)
│   └── AGENTFORCE_SETUP.md (Agentforce configuration)
├── sample-data/
│   ├── sample_orders.json
│   └── sample_inventory.json
├── .github/workflows/
│   └── validate-sfdx.yml
├── sfdx-project.json
├── .forceignore
└── .gitignore
```

## Key Concepts

### Idempotency
Orders are upsererted using the `External_Order_Id__c` field as the external ID. Running the backfill multiple times for the same date range will not create duplicates.

### Backfill Job Tracking
Every backfill run creates a `Backfill_Job__c` record that tracks:
- Start/end dates and data source
- Counts: created, updated, skipped, failed
- Full error log for any failures
- Execution time

### Inventory Reconciliation
After orders are synced, the `InventoryAdjustmentService` calculates inventory deltas and flags discrepancies if actual stock doesn't match expected levels post-backfill.

### Pluggable Connectors
To add support for a new data source:
1. Extend `ExternalOrderConnector`
2. Implement `fetchOrders(startDate, endDate)`
3. Register in Custom Metadata `Integration_Config__mdt`

See **[SETUP.md](./docs/SETUP.md)** for detailed examples.

## Testing

Run Apex tests with:

```bash
sf apex run test --test-level RunAllTestsInNamespace
```

Expected coverage: **≥85%** on core service classes.

Test classes included:
- `OrderBackfillAction_Test` — Invocable method logic
- `OrderBackfillService_Test` — Idempotency, upsert, error handling
- `InventoryAdjustmentService_Test` — Inventory delta, discrepancy flagging

## Documentation

- **[SETUP.md](./docs/SETUP.md)** — Deployment, configuration, and troubleshooting
- **[AGENTFORCE_SETUP.md](./docs/AGENTFORCE_SETUP.md)** — Agentforce agent integration guide
- **[ARCHITECTURE.md](./docs/ARCHITECTURE.md)** — Detailed data flow and design patterns

## License

[Your License]

## Support

For issues, questions, or contributions, please open a GitHub issue or pull request.

---

**Built with ❤️ for Salesforce order management and Agentforce automation.**
