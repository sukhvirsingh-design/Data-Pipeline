# Enterprise Financial Data Pipeline Architecture

This solution uses a **hub-and-spoke integration architecture** supported by an Enterprise Service Bus, API-led integration, asynchronous processing, and a centralized financial data platform.

In this architecture, the POS, payment gateways, and Yellow Dog operate as specialized transactional systems, or spokes. They send operational and financial data through an integration layer into a governed cloud data warehouse or lakehouse.

The warehouse acts as the centralized financial and analytical core, while Salesforce receives curated summary data for executive reporting, forecasting, relationship management, and exception resolution.

## Overview

This document describes the production data pipeline connecting:

* Point-of-sale transactions
* Payment gateway activity and settlements
* Yellow Dog inventory and COGS data
* A centralized financial data warehouse
* Salesforce reporting and business workflows

The objective is to provide leadership with a single, reliable view of financial performance by combining transaction-level revenue, payment settlement, inventory consumption, and cost data.

This removes the need to manually reconcile sales, settlements, processing fees, inventory usage, and COGS across multiple systems.

## Strategic Pillars of the Architecture

### Functional Specialization

Each system remains authoritative for its own business domain.

The POS system is trusted for items sold, quantities, discounts, taxes, tips, transaction timestamps, and location-level sales activity.

The payment gateway is trusted for payment authorization, refunds, chargebacks, processing fees, settlements, payouts, and payment references.

Yellow Dog is trusted for inventory management, receiving, depletion, recipes, waste, shrinkage, physical counts, and Cost of Goods Sold.

The cloud data warehouse is trusted for enterprise reconciliation, financial calculations, historical records, data lineage, and consolidated analytical reporting.

Salesforce is trusted for executive dashboards, forecasting, customer and account relationships, operational workflows, and financial exception management.

### Separation of Concerns

High-volume operational transactions remain inside the systems designed to process them.

Salesforce does not need to process individual credit card authorizations, every inventory movement, or every item-level depletion event.

Detailed records are stored and processed inside the data platform. Salesforce receives only the summarized and actionable data required for reporting, forecasting, and business workflows.

This keeps Salesforce clean, responsive, scalable, and focused on corporate-level financial visibility.

### Asynchronous Consolidation

Operational and financial data do not need to move at the same speed.

POS terminals and payment gateways process transactions in near real time. Yellow Dog records inventory activity and calculates inventory costs throughout the operational cycle.

The integration platform captures these records incrementally and performs a broader reconciliation process after the business day closes.

Nightly or scheduled processing combines the datasets, calculates approved financial metrics, identifies discrepancies, and publishes curated results to Salesforce.

### Auditability and Replay

Raw source records are preserved before transformation.

This creates a complete audit trail and allows the organization to:

* Reprocess failed batches
* Investigate financial discrepancies
* Recalculate historical periods
* Trace Salesforce totals back to source transactions
* Recover from transformation or mapping errors
* Support internal and external audits

### Governed Financial Calculations

Revenue, COGS, processing fees, settlement amounts, and margin calculations are defined centrally in the data platform.

This prevents different departments from creating conflicting versions of the same financial metric.

## System Architecture Roles

### POS System — Sales Activity Source

The POS system, such as Shift4, captures operational sales at the venue, concession, terminal, and item levels.

It records:

* Items sold
* Quantities
* Transaction timestamps
* Discounts
* Taxes
* Tips
* Voids
* Refunds
* Payment methods
* Terminal and location identifiers

The POS system is the source of truth for what was sold and when the sale occurred.

### Payment Gateway — Payment and Settlement Source

Payment processors such as Stripe, FreedomPay, or Shift4 manage card authorization, payment processing, refunds, disputes, fees, and settlement activity.

They provide:

* Payment transactions
* Authorization references
* Refunds
* Chargebacks
* Processing fees
* Adjustments
* Settlement batches
* Payout timing
* Bank references

The payment gateway is the source of truth for payment activity and settlement movement.

POS revenue and gateway settlement are intentionally treated as separate concepts because the amount sold is not always equal to the amount deposited.

### Yellow Dog — Inventory and COGS Source

Yellow Dog operates as the inventory and cost-management engine.

It tracks:

* Inventory on hand
* Receiving
* Transfers
* Depletion
* Recipes
* Ingredient usage
* Waste
* Shrinkage
* Physical counts
* Theoretical COGS
* Actual COGS
* Inventory variance

Yellow Dog is the source of truth for inventory activity and inventory-related cost calculations.

### Integration Layer — Extraction and Orchestration

The integration layer may use MuleSoft, Boomi, Workato, custom APIs, cloud-native integration services, or scheduled ETL processes.

It is responsible for:

* Connecting securely to each source system
* Pulling incremental data
* Receiving webhook events where supported
* Preserving raw source payloads
* Validating incoming records
* Standardizing dates, currencies, items, events, and locations
* Applying enterprise identifiers
* Detecting duplicate records
* Managing retries and failed messages
* Loading validated data into staging tables
* Coordinating nightly reconciliation workflows
* Publishing curated results to downstream systems

The integration layer should not contain all financial business logic. It should primarily handle connectivity, orchestration, validation, and controlled data movement.

### Raw Data Storage — Immutable Source Archive

Raw records from the POS, gateway, and Yellow Dog are stored in encrypted cloud object storage before transformation.

Possible platforms include:

* Amazon S3
* Azure Data Lake Storage
* Google Cloud Storage

The raw data layer retains:

* Original API responses
* Source files
* Extraction timestamps
* Batch identifiers
* Source-system identifiers
* Checksums
* Schema versions
* Integration run IDs

This provides replay capability, audit history, and protection against downstream processing errors.

### Cloud Data Warehouse or Lakehouse — Financial and Analytical Core

The cloud data warehouse or lakehouse is the centralized financial data platform.

Possible platforms include:

* Snowflake
* BigQuery
* Amazon Redshift
* Databricks
* Salesforce Data Cloud, where appropriate

The platform stores detailed and summarized records for:

* POS sales
* Payment activity
* Refunds
* Processing fees
* Disputes
* Settlements
* Inventory movements
* Inventory counts
* COGS
* Waste
* Margin
* Reconciliation results
* Data-quality exceptions

It performs the enterprise financial calculations and reconciliation processes.

This is the authoritative source for consolidated financial analytics, rather than Salesforce alone.

### Salesforce CRM — Executive Reporting and Workflow Hub

Salesforce receives curated, reconciled summary records from the data platform.

It does not store every POS transaction, inventory movement, or payment event.

Salesforce is used for:

* Revenue dashboards
* Cost and margin reporting
* Location and event performance
* Financial forecasting
* Customer lifetime value reporting
* Exception management
* Operational follow-up
* Approval workflows
* Leadership visibility

Salesforce remains the primary business-facing hub, while the detailed financial history remains inside the warehouse.

## Core Components

### 1. POS System

Point-of-sale terminals capture sales at the concession, venue, location, and terminal levels.

Typical data includes:

* Transaction ID
* Business date
* Event
* Location
* Terminal
* Item
* Quantity
* Sale amount
* Discount
* Tax
* Tip
* Payment method
* Void or refund status

### 2. Payment Gateway

The payment gateway processes and settles transactions.

It produces:

* Authorized payment records
* Captured payment records
* Refunds
* Chargebacks
* Processing fees
* Adjustments
* Settlement batches
* Net payouts
* Bank references

The gateway data is reconciled separately against POS sales and bank settlement records.

### 3. Yellow Dog Software

Yellow Dog manages inventory, receiving, recipes, stock movement, usage, physical counts, waste, shrinkage, and COGS.

It provides the operational context required to calculate:

* Theoretical inventory usage
* Actual inventory usage
* Theoretical COGS
* Actual COGS
* Waste cost
* Shrinkage
* Gross margin
* Inventory variance

### 4. Integration and Messaging Layer

The integration layer connects all source and target systems.

Its responsibilities include:

* API and file-based extraction
* Webhook ingestion
* Scheduling
* Message queuing
* Schema validation
* Deduplication
* Error handling
* Retry processing
* Dead-letter queues
* Mapping
* Data orchestration
* Pipeline monitoring

Separate integration flows should be created for POS, payment gateway, Yellow Dog, warehouse loading, reconciliation, and Salesforce publishing.

### 5. Raw Data Layer

The raw data layer stores unchanged source records.

This layer supports:

* Auditing
* Troubleshooting
* Replay
* Historical restatement
* Source-to-target traceability
* Vendor dispute investigation

### 6. Transformation and Reconciliation Layer

The transformation layer converts source-specific data into a shared enterprise schema.

It standardizes:

* Locations
* Events
* Business dates
* Items
* Inventory products
* Transactions
* Payment references
* Settlement IDs
* Currencies
* Time zones

It then reconciles:

* POS sales against payment gateway activity
* Payment activity against gateway settlements
* POS item sales against Yellow Dog depletion
* Theoretical inventory against actual inventory
* Revenue against COGS
* Gateway settlement against bank deposits, where available

Exceptions are separated from successful records and routed for review.

### 7. Curated Financial Data Mart

The curated data mart contains approved financial datasets.

Typical datasets include:

* Daily revenue by location
* Revenue by event
* Revenue by item category
* Processing fees
* Refunds and disputes
* Settlement status
* Theoretical COGS
* Actual COGS
* Waste cost
* Gross margin
* Inventory variance
* Reconciliation status
* Financial exceptions

Detailed records remain in the data platform, while summarized records are sent to Salesforce.

### 8. Salesforce CRM

Salesforce receives summary data at a controlled reporting grain, such as:

* Location by business date
* Event by business date
* Location by event
* Gateway settlement
* Financial exception
* Integration run

Salesforce custom objects may include:

* Financial Performance
* Settlement Summary
* Inventory Variance
* Financial Exception
* Integration Run

### 9. Salesforce Revenue Management and Analytics

Salesforce reporting capabilities sit on top of the curated CRM records.

They provide:

* Revenue dashboards
* Cost dashboards
* Margin dashboards
* Event-performance reporting
* Location comparisons
* Executive forecasting
* Settlement status
* Exception queues
* Operational alerts
* Approval workflows

Detailed drill-down reporting may remain in Tableau, Power BI, Looker, or another business-intelligence platform connected directly to the warehouse.

## Recommended Data Flow

```text
POS terminals
    |
    | Sales transactions and item details
    v
POS platform
    |
    | API, webhook or secure file
    v
Integration and messaging layer
    |
    | Preserve original records
    v
Raw cloud storage
    |
    | Validate, deduplicate and standardize
    v
Warehouse staging layer
    |
    | Transform and map enterprise identifiers
    v
Curated financial data platform
    |
    | Reconcile sales, costs, payments and settlements
    |
    +-----------------------------+
    |                             |
    v                             v
Salesforce CRM               BI and Finance Reporting
    |                             |
    v                             v
Executive dashboards         Detailed analysis
Forecasting                  Audit and reconciliation
Exception workflows          Transaction drill-down
```

Yellow Dog and the payment gateway follow parallel paths:

```text
Payment gateway
    -> Integration layer
    -> Raw storage
    -> Warehouse staging
    -> Payment and settlement reconciliation

Yellow Dog
    -> Integration layer
    -> Raw storage
    -> Warehouse staging
    -> Inventory and COGS calculations
```

The reconciled outputs are combined inside the warehouse before curated summary records are published to Salesforce.

## Financial Data Flow

```text
POS sales
    -> Gross sales
    -> Discounts
    -> Refunds
    -> Net revenue

Payment gateway
    -> Payment activity
    -> Processing fees
    -> Chargebacks
    -> Adjustments
    -> Settlements

Yellow Dog
    -> Inventory depletion
    -> Waste and shrinkage
    -> Theoretical COGS
    -> Actual COGS

Financial data platform
    -> Revenue reconciliation
    -> Settlement reconciliation
    -> COGS reconciliation
    -> Margin calculation
    -> Variance identification

Salesforce
    -> Executive reporting
    -> Forecasting
    -> Exception workflows
    -> Operational follow-up
```

## Important Financial Distinction

The architecture must keep the following concepts separate:

```text
Gross sales
- Discounts
- Refunds
= Net revenue

Net revenue
- COGS
= Gross margin

Payment activity
- Refunds
- Chargebacks
- Processing fees
+/- Adjustments
= Net gateway activity

Net gateway activity
+/- Settlement timing differences
= Settlement or payout
```

POS revenue should not automatically be treated as equal to gateway settlement or bank deposit.

## Processing Schedule

A recommended operating schedule is:

```text
Continuous or every 15 minutes:
Capture available POS and gateway changes

After business-day close:
Extract finalized POS transactions

Nightly:
Extract Yellow Dog inventory and COGS data

Nightly:
Validate, transform and reconcile data

Early morning:
Publish preliminary financial summaries to Salesforce

When settlement becomes available:
Reconcile gateway activity against settlement batches

After reconciliation:
Update Salesforce settlement and exception statuses

At financial close:
Mark the reporting period as closed or restated
```

Salesforce records may display statuses such as:

* Operationally Complete
* Settlement Pending
* Settlement Reconciled
* Exception Identified
* Financially Closed
* Restated

## Implementation Approach

The recommended production implementation combines several integration methods rather than relying on only one.

### API-Led Integration

MuleSoft or another enterprise integration platform manages reusable connections to:

* POS systems
* Payment gateways
* Yellow Dog
* Cloud storage
* Data warehouse
* Salesforce

Separate system, process, and publishing interfaces should be used to avoid tightly coupling source systems directly to Salesforce.

### Event-Assisted Data Capture

Webhooks should be used where vendors support them.

Webhook events provide faster data capture, while scheduled incremental extraction provides recovery for delayed or missed events.

### Scheduled Batch Reconciliation

Nightly batch processing remains appropriate for:

* Finalized business-day sales
* Inventory depletion
* COGS
* Waste
* Settlement reconciliation
* Margin calculations
* Salesforce summary publishing

### Data Lakehouse or Warehouse Processing

Detailed financial calculations, historical storage, and reconciliation should occur inside the data platform.

Salesforce should receive only approved, summarized, and actionable records.

### Salesforce Bulk Upserts

Salesforce records should be loaded using deterministic external identifiers.

This allows the pipeline to rerun safely without creating duplicate records.

Example external identifiers include:

```text
Financial summary:
Location + Event + Business Date + Currency

Settlement:
Gateway + Settlement ID

Exception:
Pipeline Run + Exception Type + Source Record
```

## Error Handling and Recovery

The production pipeline should include:

* Automatic retry for temporary API failures
* Exponential backoff
* Dead-letter queues
* Record quarantine
* Duplicate detection
* Replay by business date
* Replay by location
* Replay by source record
* Integration run tracking
* Source-to-target lineage
* Alerts for failed nightly processing

Invalid records should not be silently discarded or replaced with zero values.

They should be recorded as exceptions with:

* Source system
* Source record
* Failure reason
* Severity
* Business impact
* Assigned owner
* Resolution status
* Replay status

## Security Requirements

The integration should use:

* OAuth or secure service credentials
* Least-privilege service accounts
* Encrypted communication
* Encrypted storage
* Centralized secret management
* API allowlists
* Private networking where supported
* Credential rotation
* Role-based data access
* Audit logging

The pipeline should not store:

* Full card numbers
* CVV values
* PIN data
* Magnetic-stripe data
* Unnecessary payment credentials

Only tokenized or masked payment references should be retained.

## Monitoring Requirements

The platform should monitor:

* Source-system availability
* Data freshness
* Records extracted
* Records processed
* Records rejected
* Duplicate records
* Queue backlog
* Unmapped items
* Unmapped locations
* Reconciliation differences
* Settlement delays
* Salesforce load failures
* Nightly pipeline completion
* Data-quality test results

Critical alerts should be generated when:

* No sales are received for an active location
* A source extract is missing
* A nightly pipeline fails
* A settlement is significantly delayed
* A reconciliation exceeds its threshold
* Salesforce publishing fails
* A breaking schema change is detected

## Open Questions and Next Steps

1. Confirm the authentication method between the integration layer and every source system, including OAuth, API keys, certificates, or secure file transfer.

2. Confirm whether each vendor supports webhooks, incremental API extraction, scheduled exports, or only full reports.

3. Define enterprise identifiers for locations, venues, terminals, events, items, transactions, and settlements.

4. Define the financial ownership of each field and calculation.

5. Confirm the approved revenue, COGS, margin, tax, tip, refund, and processing-fee definitions.

6. Define the reconciliation rules for matching POS activity to payment gateway records.

7. Define the reconciliation rules for matching gateway activity to settlement batches.

8. Define how POS sales periods align with Yellow Dog inventory depletion periods.

9. Decide whether detailed reporting will use Snowflake, BigQuery, Redshift, Databricks, Salesforce Data Cloud, or another platform.

10. Decide which summarized records should be loaded into Salesforce.

11. Define the synchronization schedule for normal days and high-volume event days.

12. Define the variance thresholds that should generate warnings, critical alerts, or Salesforce exception records.

13. Define the accounting-period close and historical-restatement process.

14. Confirm data-retention, security, audit, and disaster-recovery requirements.

## Final Architecture Summary

This architecture is best described as:

**An API-led, event-assisted, batch-reconciled financial data platform using a hub-and-spoke integration model.**

The POS, payment gateways, and Yellow Dog remain authoritative for their individual operational domains.

The integration layer securely extracts, validates, standardizes, and transports the data.

The cloud data warehouse or lakehouse preserves detailed records, performs reconciliation, calculates approved financial metrics, and provides the auditable analytical core.

Salesforce acts as the executive reporting, forecasting, relationship-management, and exception-workflow hub.

This approach provides leadership with a reliable financial view without forcing Salesforce to operate as a payment processor, inventory engine, detailed transaction warehouse, or accounting ledger.
